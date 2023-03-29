# 音视频同步基础概念

由于音频和视频的输出不在同一个线程，而且，也不一定会同时解出同一个pts的音频帧和视频帧。更有甚者，编码或封装的时候可能pts还是不连续的，或有个别错误的。因此，在进行音频和视频的播放时，需要对音频和视频的播放速度、播放时刻进行控制，以实现音频和视频保持同步，即所谓的音视频同步。

在ffplay中，音频（audio）和视频（video）有各自的输出线程，其中音频的输出线程是sdl的音频输出回调线程，video的输出线程是程序的主线程。（参考：https://zhuanlan.zhihu.com/p/44139512 和 https://zhuanlan.zhihu.com/p/44122324）

音视频的同步策略，一般有如下几种：

- 视频同步到音频，即音频为主时钟
- 音频同步到视频，即视频为主时钟
- 视频、音频同步到外部时钟，即外部时钟（系统时间）为主时钟
- 视频和音频各自输出，即不作同步处理，或称之为各自为主时钟

由于人耳对于声音变化的敏感度比视觉高，因此，一般采样的策略是将视频同步到音频，即对画面进行适当的丢帧或重复以追赶或等待音频。特殊地，有时候会碰到一些特殊封装（或者有问题的封装），此时就不作同步处理，各自为主时钟，进行播放。

在ffplay中实现了上述前3种的同步策略。由`sync`参数控制：

```c
{ "sync", HAS_ARG | OPT_EXPERT, { .func_arg = opt_sync }, "set audio-video sync. type (type=audio/video/ext)", "type" },
```

在深入代码了解其实现前，需要先简单了解下一些结构体和概念。

- pts
- timebase
- ffplay中的pts
- Clock

pts是presentation timestamp的缩写，即显示时间戳，用于标记一个帧的呈现时刻。它的单位由timebase决定。timebase的类型是结构体AVRational（用于表示分数）：

```c
typedef struct AVRational{
    int num; ///< Numerator
    int den; ///< Denominator
} AVRational;
```

如`timebase={1, 1000}`表示千分之一秒，那么pts=1000，即为1秒，那么这一帧就需要在第一秒的时候呈现在ffplay中，将pts转化为秒，一般做法是：`pts * av_q2d(timebase)`。

ffplay的很多自定义结构体中也有pts字段，只不过是double类型，其实就是已经转化为秒为单位的pts值。

在做同步的时候，我们需要一个"时钟"的概念，ffplay定义的结构体是Clock：

```c
typedef struct Clock {
    double pts;           /* clock base */
    double pts_drift;     /* clock base minus time at which we updated the clock */
    double last_updated;
    double speed;
    int serial;           /* clock is based on a packet with this serial */
    int paused;
    int *queue_serial;    /* pointer to the current packet queue serial, used for obsolete clock detection */
} Clock;
```

这个时钟的工作原理是这样的：

1. 需要不断“对时”。对时的方法`set_clock_at(Clock *c, double pts, int serial, double time)`，需要用pts、serial、time（系统时间）进行对时。
2. 获取的时间是一个估算值。估算是通过对时时记录的pts_drift估算的。

可以看这个图来帮助理解：

![img](https://pic4.zhimg.com/80/v2-9a80421d4096d85ce4c5e395ca1b35ab_720w.jpg)

图中央是一个时间轴，从左往右看。首先我们调用`set_clock_at`进行一次对时，假设这时的`pts`是落后系统时间`time`的，那么计算`pts_drift = pts - time`。

接着，过了一会儿，且在下次对时前，通过`get_clock`来查询时间，因为这时的`pts`已经过时，不能直接拿pts当做这个时钟的时间。不过我们前面计算过`pts_drift`，也就是`pts`和`time`的差值，所以我们可以通过当前时刻的系统时间来估算这个时刻的pts：`pts = time + pts_drift`.

当然，由于pts_drift是一直在变动的(drift与漂移、抖动的意思)，所以get_clock是估算值，真实的pts可能落在比如图示虚线圆的位置。

> 一般time会取CLOCK_MONOTONIC，即系统开机到现在的时间，一般都有几个小时；而pts是节目的播放时刻，比如从0开始，播放了10分钟，就是600s。所以，真实情况下pts_drift可能要比图示的大。

在了解了这些基础概念后，就可以开始阅读音视频同步的代码了。

# 视频同步音频

ffplay默认采用视频同步音频的同步策略。

## 主流程

ffplay中将视频同步到音频的主要方案是，如果视频播放过快，则重复播放上一帧，以等待音频；如果视频播放过慢，则丢帧追赶音频。

这一部分的逻辑实现在视频输出函数`video_refresh`中，分析代码前，我们先来回顾下这个函数的流程图：

![img](https://pic2.zhimg.com/80/v2-6316d7bd07a4cc541da4866b632aa559_720w.jpg)

在这个流程中，“计算上一帧显示时长”这一步骤至关重要。先来看下代码：

```c
static void video_refresh(void *opaque, double *remaining_time)
{
    //……
    //lastvp上一帧，vp当前帧 ，nextvp下一帧

    last_duration = vp_duration(is, lastvp, vp);//计算上一帧的持续时长
    delay = compute_target_delay(last_duration, is);//参考audio clock计算上一帧真正的持续时长

    time= av_gettime_relative()/1000000.0;//取系统时刻
    if (time < is->frame_timer + delay) {//如果上一帧显示时长未满，重复显示上一帧
        *remaining_time = FFMIN(is->frame_timer + delay - time, *remaining_time);
        goto display;
    }

    is->frame_timer += delay;//frame_timer更新为上一帧结束时刻，也是当前帧开始时刻
    if (delay > 0 && time - is->frame_timer > AV_SYNC_THRESHOLD_MAX)
        is->frame_timer = time;//如果与系统时间的偏离太大，则修正为系统时间

    //更新video clock
    //视频同步音频时没作用
    SDL_LockMutex(is->pictq.mutex);
    if (!isnan(vp->pts))
        update_video_pts(is, vp->pts, vp->pos, vp->serial);
    SDL_UnlockMutex(is->pictq.mutex);

    //……

    //丢帧逻辑
    if (frame_queue_nb_remaining(&is->pictq) > 1) {
        Frame *nextvp = frame_queue_peek_next(&is->pictq);
        duration = vp_duration(is, vp, nextvp);//当前帧显示时长
        if(time > is->frame_timer + duration){//如果系统时间已经大于当前帧，则丢弃当前帧
            is->frame_drops_late++;
            frame_queue_next(&is->pictq);
            goto retry;//回到函数开始位置，继续重试(这里不能直接while丢帧，因为很可能audio clock重新对时了，这样delay值需要重新计算)
        }
    }
}
```

这段代码的逻辑在上述流程图中有包含。主要思路就是一开始提到的如果视频播放过快，则重复播放上一帧，以等待音频；如果视频播放过慢，则丢帧追赶音频。实现的方式是，参考audio clock，计算上一帧（在屏幕上的那个画面）还应显示多久（含帧本身时长），然后与系统时刻对比，是否该显示下一帧了。

这里与系统时刻的对比，引入了另一个概念——frame_timer。可以理解为帧显示时刻，如更新前，是上一帧的显示时刻；对于更新后（`is->frame_timer += delay`），则为当前帧显示时刻。

上一帧显示时刻加上delay（还应显示多久（含帧本身时长））即为上一帧应结束显示的时刻。具体原理看如下示意图：

![img](https://pic2.zhimg.com/80/v2-67dc4599167db6cf2592d418f12f5dc5_720w.jpg)



这里给出了3种情况的示意图：

- time1：系统时刻小于lastvp结束显示的时刻（frame_timer+dealy），即虚线圆圈位置。此时应该继续显示lastvp
- time2：系统时刻大于lastvp的结束显示时刻，但小于vp的结束显示时刻（vp的显示时间开始于虚线圆圈，结束于黑色圆圈）。此时既不重复显示lastvp，也不丢弃vp，即应显示vp
- time3：系统时刻大于vp结束显示时刻（黑色圆圈位置，也是nextvp预计的开始显示时刻）。此时应该丢弃vp。

## delay的计算

那么接下来就要看最关键的lastvp的显示时长delay是如何计算的。

这在函数compute_target_delay中实现：

```c
static double compute_target_delay(double delay, VideoState *is)
{
    double sync_threshold, diff = 0;

    /* update delay to follow master synchronisation source */
    if (get_master_sync_type(is) != AV_SYNC_VIDEO_MASTER) {
        /* if video is slave, we try to correct big delays by
           duplicating or deleting a frame */
        diff = get_clock(&is->vidclk) - get_master_clock(is);

        /* skip or repeat frame. We take into account the
           delay to compute the threshold. I still don't know
           if it is the best guess */
        sync_threshold = FFMAX(AV_SYNC_THRESHOLD_MIN, FFMIN(AV_SYNC_THRESHOLD_MAX, delay));
        if (!isnan(diff) && fabs(diff) < is->max_frame_duration) {
            if (diff <= -sync_threshold)
                delay = FFMAX(0, delay + diff);
            else if (diff >= sync_threshold && delay > AV_SYNC_FRAMEDUP_THRESHOLD)
                delay = delay + diff;
            else if (diff >= sync_threshold)
                delay = 2 * delay;
        }
    }

    av_log(NULL, AV_LOG_TRACE, "video: delay=%0.3f A-V=%f\n",
            delay, -diff);

    return delay;
}
```

上面代码中的注释全部是源码的注释，代码不长，注释占了快一半，可见这段代码重要性。

这段代码中最难理解的是sync_threshold，画个图帮助理解：

![img](https://pic3.zhimg.com/80/v2-a814b6fcc7521ec6c53cbf6b5e65092a_720w.jpg)



图中坐标轴是diff值大小，diff为0表示video clock与audio clock完全相同，完美同步。图纸下方色块，表示要返回的值，色块值的delay指传入参数，结合上一节代码，即lastvp的显示时长。

从图上可以看出来sync_threshold是建立一块区域，在这块区域内无需调整lastvp的显示时长，直接返回delay即可。也就是在这块区域内认为是准同步的。

如果小于-sync_threshold，那就是视频播放较慢，需要适当丢帧。具体是返回一个最大为0的值。根据前面frame_timer的图，至少应更新画面为vp。

如果大于sync_threshold，那么视频播放太快，需要适当重复显示lastvp。具体是返回2倍的delay，也就是2倍的lastvp显示时长，也就是让lastvp再显示一帧。

如果不仅大于sync_threshold，而且超过了AV_SYNC_FRAMEDUP_THRESHOLD，那么返回delay+diff，由具体diff决定还要显示多久（这里不是很明白代码意图，按我理解，统一处理为返回2*delay，或者delay+diff即可，没有区分的必要）

至此，基本上分析完了视频同步音频的过程，简单总结下：

- 基本策略是：如果视频播放过快，则重复播放上一帧，以等待音频；如果视频播放过慢，则丢帧追赶音频。
- 这一策略的实现方式是：引入frame_timer概念，标记帧的显示时刻和应结束显示的时刻，再与系统时刻对比，决定重复还是丢帧。
- lastvp的应结束显示的时刻，除了考虑这一帧本身的显示时长，还应考虑了video clock与audio clock的差值。
- 并不是每时每刻都在同步，而是有一个“准同步”的差值区域。

# packet/frame queue

因为FrameQueue是基于固定长度的数组实现的队列，与链表队列不同，其节点在初始化的时候已经在队列中了，push所要做的只是通过某种标志记录该节点是否是写入未读的。ffplay的做法是对windex加1，将写指针移动到下一个元素，凡是windex“之前”的节点，都是写过的。（至于是否可读，rindex知道；至于后续有多少空间可写，size知道）

# 参考资料

- [音视频技术 - 知乎 (zhihu.com)](https://www.zhihu.com/column/avtec)
- [ffplay音视频同步分析——基础概念 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/44615185)
- [ffplay音视频同步分析——视频同步音频 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/44615401)
- 