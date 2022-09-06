# ijkplayer项目

# 环境配置

> NDK全称：Native Development Kit。
>
> 1、NDK是一系列工具的集合。NDK提供了一系列的工具，帮助开发者快速开发C（或C++）的动态库，并能自动将so和java应用一起打包成apk。这些工具对开发者的帮助是巨大的。NDK集成了[交叉编译器](https://baike.baidu.com/item/交叉编译器)，并提供了相应的mk文件隔离平台、CPU、API等差异，开发人员只需要简单修改mk文件（指出“哪些文件需要编译”、“编译特性要求”等），就可以创建出so。NDK可以自动地将so和Java应用一起打包，极大地减轻了开发人员的打包工作。
>
> 2、NDK提供了一份稳定、功能有限的API头文件声明。Google明确声明该API是稳定的，在后续所有版本中都稳定支持当前发布的API。从该版本的NDK中看出，这些API支持的功能非常有限，包含有：C标准库（libc）、标准数学库（libm）、压缩库（libz）、Log库（liblog）。
>
> SDK：（software development kit）软件。
>
> Gradle是一个基于[JVM](https://so.csdn.net/so/search?q=JVM&spm=1001.2101.3001.7020)的构建工具，是一款通用灵活的构建工具，支持maven， Ivy仓库，支持传递性依赖管理，而不需要远程仓库或者是pom.xml和ivy.xml配置文件，基于Groovy，build脚本使用Groovy编写。

- [【错误记录】编译 Android 版本的 ijkplayer 报错 ( You must define ANDROID_NDK before starting. | 下载指定版本 NDK )_韩曙亮的博客-CSDN博客](https://hanshuliang.blog.csdn.net/article/details/123598841?spm=1001.2014.3001.5502)

- [AndroidDevTools - Android开发工具 Android SDK下载 Android Studio下载 Gradle下载 SDK Tools下载](https://www.androiddevtools.cn/)

- git大文件下载

   ```php
   brew install git-lfs
   git lfs install
   git lfs pull
   ```

> JNI是Java Native Interface的缩写，通过使用 [Java](https://baike.baidu.com/item/Java/85979)本地接口书写程序，可以确保代码在不同的平台上方便移植。从Java1.1开始，JNI标准成为java平台的一部分，它允许Java代码和其他语言写的[代码](https://baike.baidu.com/item/代码/86048)进行交互。JNI一开始是为了本地已[编译](https://baike.baidu.com/item/编译/1258343)语言，尤其是C和C++而设计的，但是它并不妨碍你使用其他编程语言，只要调用约定受支持就可以了。使用java与本地已编译的代码[交互](https://baike.baidu.com/item/交互/6964417)，通常会丧失平台[可移植性](https://baike.baidu.com/item/可移植性/6931884)。但是，有些情况下这样做是可以接受的，甚至是必须的。例如，使用一些旧的库，与硬件、操作系统进行交互，或者为了提高程序的性能。JNI标准至少要保证[本地代码](https://baike.baidu.com/item/本地代码)能工作在任何Java [虚拟机](https://baike.baidu.com/item/虚拟机)环境。

# 文件结构

ijkplayer在底层重写了ffplay.c文件，主要是去除ffplay中使用sdl音视频库播放音视频的部分；并且增加了对移动端硬件解码部分，视频渲染部分，以及音频播放部分的实现，这些部分在android和ios下有不同的实现，具体如下：

| Platform | 硬件解码     | 视频渲染                 | 音频播放              |
| :------- | :----------- | ------------------------ | --------------------- |
| IOS      | VideoToolBox | OpenGL ES                | AudioQueue            |
| Android  | MediaCodec   | OpenGL ES、ANativeWindow | OpenSL ES、AudioTrack |

从上面可以看出ijkplayer是暂时不支持音频硬件解码的，只支持软解。

主要目录结构：

| 目录      | 解释                                                         |
| --------- | ------------------------------------------------------------ |
| android   | android平台上的上层接口封装以及平台相关方法                  |
| config    | 存放编译ijkplayer所需的依赖源文件, 如ffmpeg、openssl等       |
| ijkmedia  | 核心代码                                                     |
| ijkj4a    | android平台下使用，用来实现c代码调用java层代码。这个文件夹是通过bilibili的另一个开源项目jni4android自动生成的。 |
| ijkplayer | 播放器数据下载及解码相关                                     |
| ijksdl    | 音视频数据渲染相关                                           |
| ios       | iOS平台上的上层接口封装以及平台相关方法                      |
| tool      | 初始化项目工程脚本                                           |

# 源码解析

[![v4oWoq.png](https://s1.ax1x.com/2022/08/31/v4oWoq.png)](https://imgse.com/i/v4oWoq)

[![v4ooSU.png](https://s1.ax1x.com/2022/08/31/v4ooSU.png)](https://imgse.com/i/v4ooSU)

## 初始化

结构体：

- SDL_Vout表示一个显示上下文，或者理解为一块画布，ANativeWindow，控制如何显示overlay。
- SDL_VoutOverlay表示显示层，或者理解为一块图像数据，表达如何显示。

初始化：

1. 创建IJKMediaPlayer对象， 通过`ffp_create`方法创建了FFPlayer对象，并设置消息处理函数；
2. 创建图像渲染对象SDL_Vout；
3. 创建平台相关的IJKFF_Pipeline对象，包括视频解码以及音频输出部分；

简单来说，就是创建播放器对象，完成音视频解码、渲染的准备工作。

当外部调用prepareToPlay启动播放后，ijkplayer内部最终会调用到ffplay.c中的方法`int ffp_prepare_async_l(FFPlayer *ffp, const char *file_name)`，该方法是启动播放器的入口函数，在此会设置player选项，打开audio output，最重要的是调用`stream_open`方法。

```php
static VideoState *stream_open(FFPlayer *ffp, const char *filename, AVInputFormat *iformat)
{  
    ......           
    /* start video display */
    if (frame_queue_init(&is->pictq, &is->videoq, ffp->pictq_size, 1) < 0)
        goto fail;
    if (frame_queue_init(&is->sampq, &is->audioq, SAMPLE_QUEUE_SIZE, 1) < 0)
        goto fail;
 
    if (packet_queue_init(&is->videoq) < 0 ||
        packet_queue_init(&is->audioq) < 0 )
        goto fail;
 
    ......
    
    is->video_refresh_tid = SDL_CreateThreadEx(&is->_video_refresh_tid, video_refresh_thread, ffp, "ff_vout");
    
    ......
    
    is->read_tid = SDL_CreateThreadEx(&is->_read_tid, read_thread, ffp, "ff_read");
    
    ......
}
```

从代码中可以看出，stream_open主要做了以下几件事情：

1. 创建存放video/audio解码前数据的videoq/audioq；
2. 创建存放video/audio解码后数据的pictq/sampq；
3. 创建读数据线程read_thread；
4. 创建视频渲染线程video_refresh_thread；

说明：subtitle是与video、audio平行的一个stream，ffplay中也支持对它的处理，即创建存放解码前后数据的两个queue，并且当文件中存在subtitle时，还会启动subtitle的解码线程。

## 数据读取

数据读取的整个过程都是由ffmpeg内部完成的，接收到网络过来的数据后，ffmpeg根据其封装格式，完成了解复用的动作，得到音视频分离开的解码前的数据，步骤如下：

1. 创建上下文结构体，这个结构体是最上层的结构体，表示输入上下文

```php
ic = avformat_alloc_context();
```

2. 设置中断函数，如果出错或者退出，就可以立刻退出

```php
ic->interrupt_callback.callback = decode_interrupt_cb;
ic->interrupt_callback.opaque = is；
```

3. 打开文件，主要是探测协议类型，如果是网络文件则创建网络链接等

```php
err = avformat_open_input(&ic, is->filename, is->iformat, &ffp->format_opts);
```

4. 探测媒体类型，可得到当前文件的封装格式，音视频编码参数等信息

```php
err = avformat_find_stream_info(ic, opts);
```

5. 打开视频、音频解码器。在此会打开相应解码器，并创建相应的解码线程。

```php
stream_component_open(ffp, st_index[AVMEDIA_TYPE_AUDIO]);
```

6. 读取媒体数据，得到的是音视频分离的解码前数据

```php
ret = av_read_frame(ic, pkt);
```

7. 将音视频数据分别送入相应的queue中

```php
if (pkt->stream_index == is->audio_stream && pkt_in_play_range) {
    packet_queue_put(&is->audioq, pkt);
} else if (pkt->stream_index == is->video_stream && pkt_in_play_range && !(is->video_st && (is->video_st->disposition & AV_DISPOSITION_ATTACHED_PIC))) {
    packet_queue_put(&is->videoq, pkt);
    ......
} else {
    av_packet_unref(pkt);
}
```


重复6、7两步，即可不断获取待播放的数据。

## 音视频解码

ijkplayer在视频解码上支持软解和硬解两种方式，可在起播前配置优先使用的解码方式，播放过程中不可切换。iOS平台上硬解使用VideoToolbox，Android平台上使用MediaCodec。ijkplayer中的音频解码只支持软解，暂不支持硬解。

> - 硬解，用自带播放器播放，android中的VideoView；
> - 软解，使用音视频解码库，比如FFmpeg；

### 视频解码方式选择

在打开解码器的方法中：

```php
static int stream_component_open(FFPlayer *ffp, int stream_index)
{
    ......
    codec = avcodec_find_decoder(avctx->codec_id);
    ......
    if ((ret = avcodec_open2(avctx, codec, &opts)) < 0) {
        goto fail;
    }
    ......  
    case AVMEDIA_TYPE_VIDEO:
        ......
        decoder_init(&is->viddec, avctx, &is->videoq, is->continue_read_thread);
        ffp->node_vdec = ffpipeline_open_video_decoder(ffp->pipeline, ffp);
        if (!ffp->node_vdec)
            goto fail;
        if ((ret = decoder_start(&is->viddec, video_thread, ffp, "ff_video_dec")) < 0)
            goto out;       
    ......
}
```

首先会打开ffmpeg的解码器，然后通过`ffpipeline_open_video_decoder`创建IJKFF_Pipenode。在创建IJKMediaPlayer对象时，通过`ffpipeline_create_from_android`创建了pipeline。该函数实现如下：

```php
IJKFF_Pipenode* ffpipeline_open_video_decoder(IJKFF_Pipeline *pipeline, FFPlayer *ffp)
{
    return pipeline->func_open_video_decoder(pipeline, ffp);
}
```

`func_open_video_decoder`函数指针最后指向的是ffpipeline_android.c中的`func_open_video_decoder`，其定义如下：

```php
static IJKFF_Pipenode *func_open_video_decoder(IJKFF_Pipeline *pipeline, FFPlayer *ffp)
{
    IJKFF_Pipeline_Opaque *opaque = pipeline->opaque;
    IJKFF_Pipenode        *node = NULL;
    if (ffp->mediacodec_all_videos || ffp->mediacodec_avc || ffp->mediacodec_hevc || ffp->mediacodec_mpeg2)
        node = ffpipenode_create_video_decoder_from_android_mediacodec(ffp, pipeline, opaque->weak_vout);
    if (!node) {
        node = ffpipenode_create_video_decoder_from_ffplay(ffp);
    }
    return node;
}
```


首先通过`ffp->mediacodec_all_videos || ffp->mediacodec_avc || ffp->mediacodec_hevc || ffp->mediacodec_mpeg2`判断是否支持硬件解码，如果支持会优先去尝试打开硬件解码器，如果打开失败会自动切换使用ffmpeg软解码。

关于ffp->mediacodec_all_videos 、ffp->mediacodec_avc 、ffp->mediacodec_hevc 、ffp->mediacodec_mpeg2它们的值需要在起播前通过如下方法配置：

```php
ijkmp_set_option_int(_mediaPlayer, IJKMP_OPT_CATEGORY_PLAYER,   "xxxxx", 1);
```

### 视频解码

video的解码线程为video_thread，audio的解码线程为audio_thread。

- 视频解码线程

```php
static int video_thread(void *arg)
{
    FFPlayer *ffp = (FFPlayer *)arg;
    int       ret = 0;
 
    if (ffp->node_vdec) {
        ret = ffpipenode_run_sync(ffp->node_vdec);
    }
    return ret;
}
```

`ffpipenode_run_sync` 中调用的是IJKFF_Pipenode对象中的 `func_run_sync`：

```php
int ffpipenode_run_sync(IJKFF_Pipenode *node)
{
    return node->func_run_sync(node);
}
```

`func_run_sync` 取决于播放前配置的软硬解，假设为**硬解**，`func_run_sync`函数指针最后指向的是ffpipenode_android_mediacodec_vdec.c中的`func_run_sync`，其定义如下：

```php
static int func_run_sync(IJKFF_Pipenode *node)
{
    .......
    opaque->enqueue_thread = SDL_CreateThreadEx(&opaque->_enqueue_thread, enqueue_thread_func, node, "amediacodec_input_thread");
    if (!opaque->enqueue_thread) {
        ALOGE("%s: SDL_CreateThreadEx failed\n", __func__);
        ret = -1;
        goto fail;
    }
    while (!q->abort_request) {
        int64_t timeUs = opaque->acodec_first_dequeue_output_request ? 0 : AMC_OUTPUT_TIMEOUT_US;
        got_frame = 0;
        ret = drain_output_buffer(env, node, timeUs, &dequeue_count, frame, &got_frame);
        .......
        if (got_frame) {
            duration = (frame_rate.num && frame_rate.den ? av_q2d((AVRational){frame_rate.den, frame_rate.num}) : 0);
            pts = (frame->pts == AV_NOPTS_VALUE) ? NAN : frame->pts * av_q2d(tb);
            ret = ffp_queue_picture(ffp, frame, pts, duration, av_frame_get_pkt_pos(frame), is->viddec.pkt_serial);
            ......
        }
    }
}
```

1. 首先该函数启动一个输入线程，线程的执行函数为enqueue_thread_func，函定义如下:

```php
static int enqueue_thread_func(void *arg)
{
    ......
    while (!q->abort_request) {
        ret = feed_input_buffer(env, node, AMC_INPUT_TIMEOUT_US, &dequeue_count);
        if (ret != 0) {
            goto fail;
        }
    }
    ......
}
```

该函数在循环中通过`feed_iput_buffer`调用`ffp_packet_queue_get_or_buffering`一直不停的取数据，并将取得的数据交给硬件解码器。

2. 创建完输入线程后，直接进入while循环，循环中调用`drain_output_buffer`去获取硬件解码后的数据，该函数最后一个参数用来标记是否接收到完整的一帧数据。

3. 当`got_frame`为true时，将接收的帧通过`ffp_queue_picture`送入pictq队列里。

若为**软解**，`func_run_sync`函数指针最后指向的是ffpipenode_ffplay_vdec.c中的func_run_sync，其定义如下：

```php
static int func_run_sync(IJKFF_Pipenode *node)
{
    IJKFF_Pipenode_Opaque *opaque = node->opaque;
    return ffp_video_thread(opaque->ffp);
}

static int ffplay_video_thread(void *arg) {
    AVFrame *frame = av_frame_alloc();
    for(;;){
        ret = get_video_frame(ffp, frame); // avcodec_receive_frame软解码获取一帧
        queue_picture(ffp, frame, pts, duration, frame->pkt_pos, is->viddec.pkt_serial);
    }
}
```

### 音频解码

ijkplayer的音频解码线程的入口函数是ff_ffplayer.c中的`audio_thread()`：

```php
static int audio_thread(void *arg)
{
.....
    do {
        ffp_audio_statistic_l(ffp);
        if ((got_frame = decoder_decode_frame(ffp, &is->auddec, frame, NULL)) < 0)
            goto the_end;
            ......
            while ((ret = av_buffersink_get_frame_flags(is->out_audio_filter, frame, 0)) >= 0) {
              	.....
                if (!(af = frame_queue_peek_writable(&is->sampq)))
                    goto the_end;
								.....
                av_frame_move_ref(af->frame, frame);
                frame_queue_push(&is->sampq);
                .....
        		}
    		} while (ret >= 0 || ret == AVERROR(EAGAIN) || ret == AVERROR_EOF);
 the_end:
		......
    av_frame_free(&frame);
    return ret;
}
```

1. 一开始就进入循环，然后调用`decoder_decode_frame()`进行解码，调用传进去的codec的`codec->decode()`方法解码，解码后的帧存放到frame中；

2. 然后调用`frame_queue_peek_writable()`判断是否能把刚刚解码的frame写入`is->sampq`中。会判断sampq队列是否满了，如果已满，会调用`pthread_cond_wait()`方法阻塞队列；如果未满，就会返回frame应该放置的位置的地址。`is->sampq`是音频解码帧列表，播放线程从这里面读取数据，然后播放出来；
3. 最后 `av_frame_move_ref(af->frame, frame);`把frame放入到sampq相应位置。由于前面`af = frame_queue_peek_writable(&is->sampq)`，af为指向这一帧frame存放位置的指针，所以直接把值赋值给它的结构体里面的frame就行了。

4. `frame_queue_push(&is->sampq);`里面是一个唤醒线程的操作，如果音频播放线程因为sampq队列为空而阻塞，这里可以唤醒它。

## 音视频渲染和同步

### 音频输出

ijkplayer中Android平台使用OpenSL ES或 输出音频，iOS平台使用AudioQueue输出音频。

audio output节点，在ffp_prepare_async_l方法中被创建：

```php
ffp->aout = ffpipeline_open_audio_output(ffp->pipeline, ffp);
```

`ffpipeline_open_audio_output`方法实际上调用的是IJKFF_Pipeline对象的函数指针`func_open_audio_utput`，该函数指针在初始化中的`ijkmp_android_create`方法中被赋值，最后指向的是ffpipeline_android.c中的函数`func_open_audio_output`：

```php
static SDL_Aout *func_open_audio_output(IJKFF_Pipeline *pipeline, FFPlayer *ffp)
{
    SDL_Aout *aout = NULL;
    if (ffp->opensles) {
        aout = SDL_AoutAndroid_CreateForOpenSLES();
    } else {
        aout = SDL_AoutAndroid_CreateForAudioTrack();
    }
    if (aout)
        SDL_AoutSetStereoVolume(aout, pipeline->opaque->left_volume, pipeline->opaque->right_volume);
    return aout;
}
```

该函数会根据ffp->opensles来决定是否使用openSLES来进行音频播放，后面的分析是基于openSLES方式处理音频的。

`SDL_AoutAndroid_CreateForOpenSLES`定义如下，主要完成的是创建SDL_Aout对象：

```php
SDL_Aout *SDL_AoutAndroid_CreateForOpenSLES()
{
    SDLTRACE("%s\n", __func__);
    SDL_Aout *aout = SDL_Aout_CreateInternal(sizeof(SDL_Aout_Opaque));
    if (!aout)
        return NULL;
    SDL_Aout_Opaque *opaque = aout->opaque;
    opaque->wakeup_cond = SDL_CreateCond();
    opaque->wakeup_mutex = SDL_CreateMutex();

    int ret = 0;
 
    SLObjectItf slObject = NULL;
    ret = slCreateEngine(&slObject, 0, NULL, 0, NULL, NULL);
    CHECK_OPENSL_ERROR(ret, "%s: slCreateEngine() failed", __func__);
    opaque->slObject = slObject;
 
    ret = (*slObject)->Realize(slObject, SL_BOOLEAN_FALSE);
    CHECK_OPENSL_ERROR(ret, "%s: slObject->Realize() failed", __func__);
 
    SLEngineItf slEngine = NULL;
    ret = (*slObject)->GetInterface(slObject, SL_IID_ENGINE, &slEngine);
    CHECK_OPENSL_ERROR(ret, "%s: slObject->GetInterface() failed", __func__);
    opaque->slEngine = slEngine;
 
    SLObjectItf slOutputMixObject = NULL;
    const SLInterfaceID ids1[] = {SL_IID_VOLUME};
    const SLboolean req1[] = {SL_BOOLEAN_FALSE};
    ret = (*slEngine)->CreateOutputMix(slEngine, &slOutputMixObject, 1, ids1, req1);
    CHECK_OPENSL_ERROR(ret, "%s: slEngine->CreateOutputMix() failed", __func__);
    opaque->slOutputMixObject = slOutputMixObject;
 
    ret = (*slOutputMixObject)->Realize(slOutputMixObject, SL_BOOLEAN_FALSE);
    CHECK_OPENSL_ERROR(ret, "%s: slOutputMixObject->Realize() failed", __func__);
 
    aout->free_l       = aout_free_l;
    aout->opaque_class = &g_opensles_class;
    aout->open_audio   = aout_open_audio;
    aout->pause_audio  = aout_pause_audio;
    aout->flush_audio  = aout_flush_audio;
    aout->close_audio  = aout_close_audio;
    aout->set_volume   = aout_set_volume;
    aout->func_get_latency_seconds = aout_get_latency_seconds;
 
    return aout;
fail:
    aout_free_l(aout);
    return NULL;
}
```

回到ffplay.c中，如果发现待播放的文件中含有音频，那么在调用 `stream_component_open` 打开解码器时，该方法里面也调用 `audio_open` 打开了audio output设备。

```php
static int audio_open(FFPlayer *opaque, int64_t wanted_channel_layout, int wanted_nb_channels, int wanted_sample_rate, struct AudioParams *audio_hw_params)
{
    FFPlayer *ffp = opaque;
    VideoState *is = ffp->is;
    SDL_AudioSpec wanted_spec, spec;
    ......
    wanted_nb_channels = av_get_channel_layout_nb_channels(wanted_channel_layout);
    wanted_spec.channels = wanted_nb_channels;
    wanted_spec.freq = wanted_sample_rate;
    wanted_spec.format = AUDIO_S16SYS;
    wanted_spec.silence = 0;
    wanted_spec.samples = FFMAX(SDL_AUDIO_MIN_BUFFER_SIZE, 2 << av_log2(wanted_spec.freq / SDL_AoutGetAudioPerSecondCallBacks(ffp->aout)));
    wanted_spec.callback = sdl_audio_callback;
    wanted_spec.userdata = opaque;
    while (SDL_AoutOpenAudio(ffp->aout, &wanted_spec, &spec) < 0) {
        .....
    }
    ......
    return spec.size;
}
```

在 audio_open中配置了音频输出的相关参数 SDL_AudioSpec ，并通过

```php
int SDL_AoutOpenAudio(SDL_Aout *aout, const SDL_AudioSpec *desired, SDL_AudioSpec *obtained)
{
    if (aout && desired && aout->open_audio)
        return aout->open_audio(aout, desired, obtained);
    return -1;
}
```

设置给了Audio Output，android平台上即为OpenSLES。OpenSLES模块在工作过程中，通过不断的callback来获取pcm数据进行播放。

### 视频渲染

若Android平台上采用OpenGL渲染解码后的YUV图像，渲染线程为video_refresh_thread，最后渲染图像的方法为video_image_display2，定义如下：

```php
static void video_image_display2(FFPlayer *ffp)
{
    VideoState *is = ffp->is;
    Frame *vp;
    Frame *sp = NULL;
 
    vp = frame_queue_peek_last(&is->pictq);
    ......
    
    SDL_VoutDisplayYUVOverlay(ffp->vout, vp->bmp);
    ......
}
```

从代码实现上可以看出，该线程的主要工作为：

1. 调用`frame_queue_peek_last`从pictq中读取当前需要显示视频帧；
2. 调用`SDL_VoutDisplayYUVOverlay`进行绘制；

`display_overlay`函数指针在函数SDL_VoutAndroid_CreateForANativeWindow()中被赋值为`vout_display_overlay`，该方法就是调用OpengGL绘制图像。

### 音视频同步

ijkplayer在默认情况下也是使用音频作为参考时钟源，处理同步的过程主要在视频渲染`video_refresh_thread`的线程中：

```php
static int video_refresh_thread(void *arg)
{
    FFPlayer *ffp = arg;
    VideoState *is = ffp->is;
    double remaining_time = 0.0;
    while (!is->abort_request) {
        if (remaining_time > 0.0)
            av_usleep((int)(int64_t)(remaining_time * 1000000.0));
        remaining_time = REFRESH_RATE;
        if (is->show_mode != SHOW_MODE_NONE && (!is->paused || is->force_refresh))
            video_refresh(ffp, &remaining_time);
    }
 
    return 0;
}
```

从上述实现可以看出，该方法中主要循环做两件事情：

1. 休眠等待，`remaining_time`的计算在`video_refresh`中；
2. 调用`video_refresh`方法，刷新视频帧；

可见同步的重点是在`video_refresh`中，下面着重分析该方法：

```php
lastvp = frame_queue_peek_last(&is->pictq);
vp = frame_queue_peek(&is->pictq);
......
  /* compute nominal last_duration */
last_duration = vp_duration(is, lastvp, vp);
delay = compute_target_delay(ffp, last_duration, is);
```

lastvp是上一帧，vp是当前帧，last_duration则是根据当前帧和上一帧的pts，计算出来上一帧的显示时间，经过 `compute_target_delay` 方法，计算出显示当前帧需要等待的时间。

```php
static double compute_target_delay(FFPlayer *ffp, double delay, VideoState *is)
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
        /* -- by bbcallen: replace is->max_frame_duration with AV_NOSYNC_THRESHOLD */
        if (!isnan(diff) && fabs(diff) < AV_NOSYNC_THRESHOLD) {
            if (diff <= -sync_threshold)
                delay = FFMAX(0, delay + diff);
            else if (diff >= sync_threshold && delay > AV_SYNC_FRAMEDUP_THRESHOLD)
                delay = delay + diff;
            else if (diff >= sync_threshold)
                delay = 2 * delay;
        }
    }
 
    .....
 
    return delay;
}
```

- 如果当前视频帧落后于主时钟源，则需要减小下一帧画面的等待时间；
- 如果视频帧超前，并且该帧的显示时间大于显示更新门槛，则显示下一帧的时间为超前的时间差加上上一帧的显示时间;
- 如果视频帧超前，并且上一帧的显示时间小于显示更新门槛，则采取加倍延时的策略。

回到video_refresh中：

```php
time= av_gettime_relative()/1000000.0;
if (isnan(is->frame_timer) || time < is->frame_timer)
  is->frame_timer = time;
if (time < is->frame_timer + delay) {
  *remaining_time = FFMIN(is->frame_timer + delay - time, *remaining_time);
  goto display;
}
```


frame_timer实际上就是上一帧的播放时间，而frame_timer + delay实际上就是当前这一帧的播放时间，如果系统时间还没有到当前这一帧的播放时间，直接跳转至display，而此时is->force_refresh变量为0，不显示当前帧，进入`video_refresh_thread`中下一次循环，并睡眠等待。

```php
is->frame_timer += delay;
  if (delay > 0 && time - is->frame_timer > AV_SYNC_THRESHOLD_MAX)
      is->frame_timer = time;
 
  SDL_LockMutex(is->pictq.mutex);
  if (!isnan(vp->pts))
         update_video_pts(is, vp->pts, vp->pos, vp->serial);
  SDL_UnlockMutex(is->pictq.mutex);
 
  if (frame_queue_nb_remaining(&is->pictq) > 1) {
       Frame *nextvp = frame_queue_peek_next(&is->pictq);
       duration = vp_duration(is, vp, nextvp);
       if(!is->step && (ffp->framedrop > 0 || (ffp->framedrop && get_master_sync_type(is) != AV_SYNC_VIDEO_MASTER)) && time > is->frame_timer + duration) {
           frame_queue_next(&is->pictq);
           goto retry;
       }
  }
```

如果当前这一帧的播放时间已经过了，并且其和当前系统时间的差值超过了AV_SYNC_THRESHOLD_MAX，则将当前这一帧的播放时间改为系统时间，并在后续判断是否需要丢帧，其目的是为后面帧的播放时间重新调整frame_timer，如果缓冲区中有更多的数据，并且当前的时间已经大于当前帧的持续显示时间，则丢弃当前帧，尝试显示下一帧。

```php
{
   frame_queue_next(&is->pictq);
   is->force_refresh = 1;
 
   SDL_LockMutex(ffp->is->play_mutex);
   
    ......
    
display:
    /* display picture */
    if (!ffp->display_disable && is->force_refresh && is->show_mode == SHOW_MODE_VIDEO && is->pictq.rindex_shown)
        video_display2(ffp);
```

否则进入正常显示当前帧的流程，调用 `video_display2` 开始渲染。

## 事件处理

在播放过程中，某些行为的完成或者变化，如prepare完成，开始渲染等，需要以事件形式通知到外部，以便上层作出具体的业务处理。ijkplayer支持的事件比较多，具体定义在ijkplayer/ijkmedia/ijkplayer/ff_ffmsg.h中

```php
#define FFP_MSG_FLUSH                       0
#define FFP_MSG_ERROR                       100     /* arg1 = error */
#define FFP_MSG_PREPARED                    200
#define FFP_MSG_COMPLETED                   300
#define FFP_MSG_VIDEO_SIZE_CHANGED          400     /* arg1 = width, arg2 = height */
#define FFP_MSG_SAR_CHANGED                 401     /* arg1 = sar.num, arg2 = sar.den */
#define FFP_MSG_VIDEO_RENDERING_START       402
#define FFP_MSG_AUDIO_RENDERING_START       403
#define FFP_MSG_VIDEO_ROTATION_CHANGED      404     /* arg1 = degree */
#define FFP_MSG_BUFFERING_START             500
#define FFP_MSG_BUFFERING_END               501
#define FFP_MSG_BUFFERING_UPDATE            502     /* arg1 = buffering head position in time, arg2 = minimum percent in time or bytes */
#define FFP_MSG_BUFFERING_BYTES_UPDATE      503     /* arg1 = cached data in bytes,            arg2 = high water mark */
#define FFP_MSG_BUFFERING_TIME_UPDATE       504     /* arg1 = cached duration in milliseconds, arg2 = high water mark */
#define FFP_MSG_SEEK_COMPLETE               600     /* arg1 = seek position,                   arg2 = error */
#define FFP_MSG_PLAYBACK_STATE_CHANGED      700
#define FFP_MSG_TIMED_TEXT                  800
#define FFP_MSG_VIDEO_DECODER_OPEN          10001
```

### 消息上报初始化

在IJKMediaPlayer的初始化方法中:

```php
static void
IjkMediaPlayer_native_setup(JNIEnv *env, jobject thiz, jobject weak_this)
{
    MPTRACE("%s\n", __func__);
    IjkMediaPlayer *mp = ijkmp_android_create(message_loop);
    ......
}
```

可以看到在创建播放器时， `message_loop` 函数地址作为参数传入了 `ijkmp_android_create` ，继续跟踪代码，可以发现，该函数地址最终被赋值给了IjkMediaPlayer中的 `msg_loop` 函数指针：

```php
IjkMediaPlayer *ijkmp_create(int (*msg_loop)(void*))
{
    ......
    mp->msg_loop = msg_loop;
    ......
}
```

开始播放时，会启动一个消息线程：

```php
static int ijkmp_prepare_async_l(IjkMediaPlayer *mp)
{
    ......
    mp->msg_thread = SDL_CreateThreadEx(&mp->_msg_thread, ijkmp_msg_loop, mp, "ff_msg_loop");
    ......
}
```

`ijkmp_msg_loop` 方法中调用的即是 `mp->msg_loop` 。

### 消息上报处理

播放器底层上报事件时，实际上就是将待发送的消息放入消息队列，另外有一个线程会不断从队列中取出消息，上报给外部，其代码流程大致如下图所示：

[![vIMfoD.png](https://s1.ax1x.com/2022/09/02/vIMfoD.png)](https://imgse.com/i/vIMfoD)

# 框架分析

ijkplayer（android）融合了mediacodec，实现了硬解支持。由于MediaCodec的使用与一般的解码API有所不同，其在应用中的使用层级更偏向于播放器层面。所以ijkplayer并没有将MediaCodec作为“解码器”扩展到ffmpeg中，而是另外定义了一个封装层，同时也统一了软解的接口，**这个封装层即ffpipeline**。

ijkplayer中的音频是走的软解，后续提到的解码无特别说明都是指视频解码。

## 基础概念

解码封装层中有两个重要的结构体，`ffpipeline`和`ffpipenode`。ffpipeline表示解码器和音频输出的提供者，ffpipenode表示解码器。

**ffpipeline**定义如下：

```php
struct IJKFF_Pipeline {
    SDL_Class             *opaque_class;
    IJKFF_Pipeline_Opaque *opaque;
    void            (*func_destroy)             (IJKFF_Pipeline *pipeline);
    IJKFF_Pipenode *(*func_open_video_decoder)  (IJKFF_Pipeline *pipeline, FFPlayer *ffp);
    SDL_Aout       *(*func_open_audio_output)   (IJKFF_Pipeline *pipeline, FFPlayer *ffp);
    IJKFF_Pipenode *(*func_init_video_decoder)  (IJKFF_Pipeline *pipeline, FFPlayer *ffp);
    int           (*func_config_video_decoder)  (IJKFF_Pipeline *pipeline, FFPlayer *ffp);
};
```

`IJKFF_Pipeline`主要定义了3组函数：

- 获取音频输出：func_open_audio_output；
- 同步方式获取视频解码器：func_open_video_decoder；
- 异步方式获取视频解码器：func_init_video_decoder、func_config_video_decoder；

这里以IJKFF_Pipenode表示一个视频解码器，以SDL_Aout表示音频输出。

ijkplayer中的音频是软解码的，所以不需要像视频一样去作封装，而是直接基于ffplay修改的。

异步创建视频解码器的用意不是很明白，实际使用中也没需求需要用到，后面分析，先只关注同步创建解码器的部分，即func_open_video_decoder的使用。

**ffpipenode**定义如下：

```php
struct IJKFF_Pipenode {
    SDL_mutex *mutex;
    void *opaque;
    void (*func_destroy) (IJKFF_Pipenode *node);
    int  (*func_run_sync)(IJKFF_Pipenode *node);
    int  (*func_flush)   (IJKFF_Pipenode *node); // optional
};
```

IJKFF_Pipenode中的主要函数是：

- func_run_sync：表示解码主循环，从这个函数返回代表解码结束了；
- func_flush：清空解码器，一般在seek时用到；

## 目录结构

解码封装层的源码文件及目录结构如下：

```php
// +表示目录，目录下文件递进4个空格，-表示文件
+ijkmedia/ijkplayer
    -ff_ffpipenode.c/h                          //pipeline定义与封装
    -ff_ffpipeline.c/h                          //pipenode定义与封装
    +pipeline
        -ffpipeline_ffplay.c/h                  //ffplay的pipeline定义(未使用)
        -ffpipenode_ffplay_vdec.c/h             //ffplay的pipenode定义
    +android/pipeline
        -ffpipeline_android.c/h                 //android的ffpipeline定义
        -ffpipenode_android_mediacodec_vdec.c/h //android的(mediacodec)ffpipenode定义
```

上面提到的pipeline和pipenode的实现由两种（android上）：

- ffplay软解封装：在ijkmedia/ijkplayer/pipeline目录下，其中ffpipeline_ffplay并未用到；
- mediacodec硬解封装：在ijkmedia/ijkplayer/android/pipeline目录下；

## 流程分析

解码器在播放器使用过程中的主要功能可以分为：创建、解码、帧入队、seek处理、销毁。解码器的调用主要在ff_ffplay。

### 创建

首先需要创建pipeline，pipeline的创建流程和SDL_Vout一样：

```php
new IjkMediaPlayer() 
    -> initPlayer() 
        -> native_setup() 
            -> IjkMediaPlayer_native_setup() 
                -> ijkmp_android_create() 
                    -> SDL_VoutAndroid_CreateForAndroidSurface() //SDL_Vout在这里创建
                    -> ffpipeline_create_from_android() //android pipeline在这里创建
```

接着在ff_ffplay中创建解码器(pipenode)：

```php
static int stream_component_open(FFPlayer *ffp, int stream_index)
{
    //……
    switch (avctx->codec_type) {
        case AVMEDIA_TYPE_VIDEO:
            if (ffp->async_init_decoder) {
                //这里是异步创建解码器的代码
            }
            else{
                decoder_init(&is->viddec, avctx, &is->videoq, is->continue_read_thread);
              	// 默认情况，调用android pipeline的func_open_video_decoder，
              	// 其内部会根据所配置的选项和实际支持的编码类型，自动回退到软解，即创建ffplay pipenode
                ffp->node_vdec = ffpipeline_open_video_decoder(ffp->pipeline, ffp);
                if (!ffp->node_vdec)
                    goto fail;
            }
            if ((ret = decoder_start(&is->viddec, video_thread, ffp, "ff_video_dec")) < 0)
            goto out;
            break;
            //……
    }
}
```

解码器的创建时在`stream_component_open`里调用`ffpipeline_open_video_decoder`完成的，创建好后的decoder保存在`ffp->node_vdec`中。接着调用`decoder_start`启动了`video_thread`。

`video_thread`的实现很简单：

```php
static int video_thread(void *arg)
{
    FFPlayer *ffp = (FFPlayer *)arg;
    int       ret = 0;

    if (ffp->node_vdec) {
        ret = ffpipenode_run_sync(ffp->node_vdec);
    }
    return ret;
}
```

直接调用循环的主体`ffpipenode_run_sync`，即pipenode的`func_run_sync`函数。

### 解码

解码工作是在`video_thread`中完成的，也就是由具体pipenode的实现决定。在ijkplayer中，这分为两种情况，一种是硬解，一种是软解。软解的实现基本是ffplay的改造，硬解的实现是调用的MediaCodec。





















### 帧入队

在解码的过程中，需要将已经解码好的帧放入帧队列FrameQueue中。该工作由ff_ffplay中的ffp_queue_picture/queue_picture完成。

```php
int ffp_queue_picture(FFPlayer *ffp, AVFrame *src_frame, double pts, double duration, int64_t pos, int serial)
{
    return queue_picture(ffp, src_frame, pts, duration, pos, serial);
}

static int queue_picture(FFPlayer *ffp, AVFrame *src_frame, double pts, double duration, int64_t pos, int serial)
{
    //……
}
```

`ffp_queue_picture`只是简单调用的`queue_picture`，这是为了改变它的可见性（去掉static），方便在其他文件调用该函数。

`queue_picture`的主要功能和结构与ffplay的类似，可以参考[ffplay video显示线程分析](https://zhuanlan.zhihu.com/p/44122324)。所不同的是ijk的显示流程和解码流程与ffplay不同，所以在queue_picture的时候调用`SDL_VoutOverlay`的`func_fill_frame`把帧画面“绘制”到最终的显示图层上。

### seek处理

seek时需要调用解码器的flush：

```php
static int read_thread(void *arg)
{
    //……
    ret = avformat_seek_file(is->ic, -1, seek_min, seek_target, seek_max, is->seek_flags);
    if (ret < 0) {
        //……
    }
    else {
        //……
        if (is->video_stream >= 0) {
            if (ffp->node_vdec) {
              	// 这里调用解码器的flush，即pipenode的func_flush函数
                ffpipenode_flush(ffp->node_vdec);
            }
            packet_queue_flush(&is->videoq);
            packet_queue_put(&is->videoq, &flush_pkt);
        }
        //……
    }
    //……
}
```

### 销毁

按预期pipenode的销毁应该出现在其创建函数`stream_component_open`对应的`stream_component_close`中，然而并没有。根据`IJKFF_Pipenode`的定义，其销毁应调用`func_destroy`，或者其封装函数`ffpipenode_free`/`ffpipenode_free_p`。

正常流程只在整个播放器销毁时有调用到：

```php
void ffp_destroy(FFPlayer *ffp)
{
    //……
    ffpipenode_free_p(&ffp->node_vdec);
    ffpipeline_free_p(&ffp->pipeline);
    //……
}
```

这就意味着，按代码分析的情况看，除第一次外，每次选择视频轨道都会造成一次内存泄漏！不过毕竟还未验证，待验证后再给结论。

# 音视频解码

## 硬解

在android中的ijkplayer是通过封装MediaCodec实现的硬解，以下将从解码器的创建、解码、帧入队三个方面介绍。

### 创建

硬解pipenode的创建是在`stream_component_open`中调用`ffpipeline_open_video_decoder`创建的。`ffpipeline_open_video_decoder`是pipeline的封装，在Android上调用的是ffpipeline_andriod.c中的`func_open_video_decoder`：

```php
static IJKFF_Pipenode *func_open_video_decoder(IJKFF_Pipeline *pipeline, FFPlayer *ffp)
{
    IJKFF_Pipeline_Opaque *opaque = pipeline->opaque;
    IJKFF_Pipenode        *node = NULL;
    if (ffp->mediacodec_all_videos || ffp->mediacodec_avc || ffp->mediacodec_hevc || ffp->mediacodec_mpeg2)
        node = ffpipenode_create_video_decoder_from_android_mediacodec(ffp, pipeline, opaque->weak_vout);
    if (!node) {
        node = ffpipenode_create_video_decoder_from_ffplay(ffp);
    }
    return node;
}
```

这里启用了硬解会调用`ffpipenode_create_video_decoder_from_android_mediacodec`：

```php
IJKFF_Pipenode *ffpipenode_create_video_decoder_from_android_mediacodec(FFPlayer *ffp, IJKFF_Pipeline *pipeline, SDL_Vout *vout)
{
    //……
    //1. 初始化node
    IJKFF_Pipenode *node = ffpipenode_alloc(sizeof(IJKFF_Pipenode_Opaque));
    node->func_destroy  = func_destroy;
    if (ffp->mediacodec_sync) {
        node->func_run_sync = func_run_sync_loop;
    } else {
        node->func_run_sync = func_run_sync;
    }
    node->func_flush    = func_flush;

    //2. 硬解选项检查
    switch (opaque->codecpar->codec_id) {
    case AV_CODEC_ID_H264:
        if (!ffp->mediacodec_avc && !ffp->mediacodec_all_videos) {
            ALOGE("%s: MediaCodec: AVC/H264 is disabled. codec_id:%d \n", __func__, opaque->codecpar->codec_id);
            goto fail;
        }
        opaque->mcc.profile = opaque->codecpar->profile;
        opaque->mcc.level   = opaque->codecpar->level;
    //……
    }

    //3. 创建MediaFormat
    ret = recreate_format_l(env, node);

    //4. 选择codec(选择最佳codec name)
    if (!ffpipeline_select_mediacodec_l(pipeline, &opaque->mcc) || !opaque->mcc.codec_name[0]) {
        ALOGE("amc: no suitable codec\n");
        goto fail;
    }

    //5. 配置codec(创建MediaCodec)
    ret = reconfigure_codec_l(env, node, jsurface);

    //一些特殊的解码器需要在MediaCodec解码后，增加一级帧队列，队列按pts排序，然后再送出到FrameQueue，源码分析中不考虑该特殊情况。
    if (opaque->n_buf_out) {
        int i;

        opaque->amc_buf_out = calloc(opaque->n_buf_out, sizeof(*opaque->amc_buf_out));
        assert(opaque->amc_buf_out != NULL);
        for (i = 0; i < opaque->n_buf_out; i++)
            opaque->amc_buf_out[i].pts = AV_NOPTS_VALUE;
    }
    //……
}
```

在pipenode的创建中，经历以下步骤：

1. 初始化node；
2. 硬解选项检查；
3. 创建MediaFormat。在函数`recreate_format_l`中实现，主要设置mime type、width、height和csd-0；
4. 选择codec(选择最佳codec name)。主要是调用`ffpipeline_select_mediacodec_l`函数进行选择，`ffpipeline_select_mediacodec_l`会回调到`IjkMediaPlayer`的`onSelectCodec`。在`onSelectCodec`中会根据自己的一套规则取选择合适的codec name（与`MediaCodecList.findDecoderForFormat`的工作类似，不过更灵活）；
5. 配置codec(创建MediaCodec)。在函数`reconfigure_codec_l`中实现。

接下来看下`reconfigure_codec_l`：

```php
static int reconfigure_codec_l(JNIEnv *env, IJKFF_Pipenode *node, jobject new_surface)
{
    //……
    //acodec = new MediaCodec
    if (!opaque->acodec) {
        opaque->acodec = create_codec_l(env, node);
    }

    //MediaCodec.setSurface
    amc_ret = SDL_AMediaCodec_configure_surface(env, opaque->acodec, opaque->input_aformat, opaque->jsurface, NULL, 0);

    //MediaCodec.start
    amc_ret = SDL_AMediaCodec_start(opaque->acodec);
    //……
}
```

reconfigure的主要流程与java api的使用差不多。典型的`new -> setSurface -> start`。到这里，MediaCodec就创建好，准备接收数据了。

### 解码

解码调用过程：`stream_component_open -> decoder_start -> video_thread -> fun_run_sync`。

在ffpipenode_android_mediacodec_vdec中有两个fun_run_sync的实现，可以通过mediacodec_sync选项进行切换：

```php
//ffpipenode_create_video_decoder_from_android_mediacodec
    if (ffp->mediacodec_sync) {
        node->func_run_sync = func_run_sync_loop;
    } else {
        node->func_run_sync = func_run_sync;
    }
```

默认使用的是`func_run_sync`:

```php
static int func_run_sync(IJKFF_Pipenode *node)
{
    //……
    //A. 创建enqueue_thread，喂原始数据
    opaque->enqueue_thread = SDL_CreateThreadEx(&opaque->_enqueue_thread, enqueue_thread_func, node, "amediacodec_input_thread");

    //B. 循环拉取解码数据
    while (!q->abort_request) {
        got_frame = 0;
        //1. drain_output_buffer获取frame
        ret = drain_output_buffer(env, node, timeUs, &dequeue_count, frame, &got_frame);
        //……
        if (ret != 0) {
            //拉取出错，release buffer false通知MediaCodec丢弃这一帧
        }
        if (got_frame) {
            duration = (frame_rate.num && frame_rate.den ? av_q2d((AVRational){frame_rate.den, frame_rate.num}) : 0);
            pts = (frame->pts == AV_NOPTS_VALUE) ? NAN : frame->pts * av_q2d(tb);
            //2. 解码速度过慢，丢帧
            if (ffp->framedrop > 0 || (ffp->framedrop && ffp_get_master_sync_type(is) != AV_SYNC_VIDEO_MASTER)) {
                //逻辑与软解丢帧类似，可以参考ffplay解码线程分析
            }
            //3. 帧入队
            ret = ffp_queue_picture(ffp, frame, pts, duration, av_frame_get_pkt_pos(frame), is->viddec.pkt_serial);
            if (ret) {
                //入队出错，release buffer false通知MediaCodec丢弃这一帧
            }
            av_frame_unref(frame);
    }
    //……
}
```

`func_run_sync`的函数比较长，上述代码仅抽取了主干代码。主干代码分为两个部分：

- 创建enqueue_thread，喂原始数据；
- 循环拉取解码数据；

也就是ijkplayer中把MediaCodec的`queueInputBuffer`和`dequeueOutputBuffer`两个过程分离到了两个线程：`func_run_sync`负责dequeue，dequeue的主要实现在`drain_output_buffer`。

`drain_output_buffer`调用后会取到一帧填充好的AVFrame，之后调用`ffp_queue_picture`将这一帧放入到FrameQueue中。

在分析`drain_output_buffer`前先看下`enqueue_thread_func`：

```php
static int enqueue_thread_func(void *arg)
{
    while (!q->abort_request && !opaque->abort) {
        ret = feed_input_buffer(env, node, AMC_INPUT_TIMEOUT_US, &dequeue_count);
        if (ret != 0) {
            goto fail;
        }
    }
    ret = 0;
fail:
    SDL_AMediaCodecFake_abort(opaque->acodec);
    ALOGI("MediaCodec: %s: exit: %d", __func__, ret);
    return ret;
}
```

`enqueue_thread_func`的实现比较简单，即循环调用`feed_input_buffer`。

因此，在分析了上述两个线程后，我们找到的3个主要函数是：

- feed_input_buffer：主要调用MediaCodec.queueInputBuffer给MediaCodec喂原始数据；
- drain_output_buffer：主要调用MediaCodec.dequeueOutputBuffer从MediaCodec拉解码数据；
- ffp_queue_picture：将解码后的AVFrame送入FrameQueue；

**feed_input_buffer**

```php
static int feed_input_buffer(JNIEnv *env, IJKFF_Pipenode *node, int64_t timeUs, int *enqueue_count)
{
    //……
    //1. 读Packet，及不连续Packet的处理
    if (!d->packet_pending || d->queue->serial != d->pkt_serial) {
        do {
            ffp_packet_queue_get_or_buffering(ffp, d->queue, &pkt, &d->pkt_serial, &d->finished)；
            if (ffp_is_flush_packet(&pkt) || opaque->acodec_flush_request) {
                SDL_AMediaCodec_flush()；
            }
        }while (ffp_is_flush_packet(&pkt) || d->queue->serial != d->pkt_serial);
        av_packet_unref(&d->pkt);
        d->pkt_temp = d->pkt = pkt;
        d->packet_pending = 1;
    }

    //2. 喂数据
    if (d->pkt_temp.data) {
        //如果需要重新配置MediaCodec，则重新创建一个
        if (ffpipeline_is_surface_need_reconfigure_l(pipeline)) {
            ret = reconfigure_codec_l(env, node, new_surface);
        }
        input_buffer_index = SDL_AMediaCodec_dequeueInputBuffer(opaque->acodec, timeUs);
        copy_size = SDL_AMediaCodec_writeInputData(opaque->acodec, input_buffer_index, d->pkt_temp.data, d->pkt_temp.size);
        amc_ret = SDL_AMediaCodec_queueInputBuffer(opaque->acodec, input_buffer_index, 0, copy_size, time_stamp, queue_flags);
    }

    //3. pkt_temp更新
    if (copy_size < 0) {//无需分包
        d->packet_pending = 0;
    } else {
        d->pkt_temp.dts =
        d->pkt_temp.pts = AV_NOPTS_VALUE;
        if (d->pkt_temp.data) {//分包更新
            d->pkt_temp.data += copy_size;
            d->pkt_temp.size -= copy_size;
            if (d->pkt_temp.size <= 0)
                d->packet_pending = 0;
        } else {//解码结束
            d->packet_pending = 0;
            d->finished = d->pkt_serial;
        }
    }
}
```

这段代码有3个步骤：

1. 读Packet，及不连续Packet的处理。主要是通过`ffp_packet_queue_get_or_buffering`读一个Packet，如果不连续，就丢Packet和Flush MediaCodec；
2. 喂数据。MediaCodec的典型步骤：`dequeueInputBuffer -> writeInputData -> queueInputBuffer`；
3. pkt_temp更新；

导致这段代码不好理解的一个地方是**分包发送**的处理。因为在调用`SDL_AMediaCodec_writeInputData`发送Packet数据的时候，不一定能恰好完整发送，所以需要分多次发送。

分包发送代码中pkt_temp表示要发送的pkt，packet_pending表示pkt_temp中有未发送完的数据。每次发送后，根据已发送大小copy_size更新pkt_temp，直到pkt_temp.size小于0，置packet_pending为0。在读Packet前会先判断packet_pending是否为1，如果为1，则不拉取新的Packet，而是先消耗pkt_temp。这样就达到循环发送Packet的目的了。

> 上述代码是经过大量省略的，被省略的代码还有几个功能点：
>
> - reconfig的具体实现，以及如何与fun_run_sync线程同步
> - mediacodec_handle_resolution_change处理
> - H264/H265特殊处理
> - fake frame处理

**drain_output_buffer**

```php
//drain_output_buffer = lock(opaque->acodec_mutex) + drain_output_buffer_l + unlock(opaque->acodec_mutex)
static int drain_output_buffer_l(JNIEnv *env, IJKFF_Pipenode *node, int64_t timeUs, int *dequeue_count, AVFrame *frame, int *got_frame)
{
    output_buffer_index = SDL_AMediaCodecFake_dequeueOutputBuffer(opaque->acodec, &bufferInfo, timeUs);
    if (output_buffer_index == AMEDIACODEC__INFO_OUTPUT_BUFFERS_CHANGED) {
        ALOGI("AMEDIACODEC__INFO_OUTPUT_BUFFERS_CHANGED\n");
    }
    else if (output_buffer_index == AMEDIACODEC__INFO_OUTPUT_FORMAT_CHANGED) {
        ALOGI("AMEDIACODEC__INFO_OUTPUT_FORMAT_CHANGED\n");
    }
    else if (output_buffer_index == AMEDIACODEC__INFO_TRY_AGAIN_LATER) {
        AMCTRACE("AMEDIACODEC__INFO_TRY_AGAIN_LATER\n");
    }
    else if (output_buffer_index < 0) {
        goto done;
    }
    else if (output_buffer_index >= 0) {
        if (opaque->n_buf_out) {
            // 如果开启了缓冲区，则对缓冲区内的帧进行pts排序后输出。
          	// 看代码，目前只有codec是OMX.TI.DUCATI1.才启用。也就是默认MediaCodec的输出都是pts排序好的
        }else {
            ret = amc_fill_frame(node, frame, got_frame, output_buffer_index, SDL_AMediaCodec_getSerial(opaque->acodec), &bufferInfo);
        }
    }

done:
    if (opaque->decoder->queue->abort_request)
        ret = -1;
    else
        ret = 0;
fail:
    return ret;
}
```

`drain_output_buffer`相比`feed_input_buffer`来的简单，主要是调用`SDL_AMediaCodecFake_dequeueOutputBuffer`（即MediaCodec.dequeueOutputBuffer），根据返回值打印调试信息。如果是`output_buffer_index >= 0`，则调用`amc_fill_frame`把bufferinfo填充到AVFrame中。

`amc_fill_frame`主要是填充Frame的宽、高、pts等信息，把bufferinfo填入到Opaque中，并没有填充真正的图像数据。Frame显示的时候再利用这些信息通过MediaCodec的releasseOutputBuffer进行显示。

### 帧入队

在[ijkplayer 解码实现分析——软解篇](https://zhuanlan.zhihu.com/p/45262272)中我们分析了`queue_picture`(`ffp_queue_picture`封装的是`queue_picture`)的逻辑。与ffplay的queue_picture的差异在于增加了一些“绘图操作”，这些绘图操作是通过`SDL_Vout_CreateOverlay -> SDL_VoutFillFrameYUVOverlay`完成。

`SDL_Vout_CreateOverlay`会调用具体`vout->create_overlay`，`SDL_VoutFillFrameYUVOverlay`会调用`overlay->func_fill_frame`。

对于MediaCodec而言，`vout->create_overlay`会调用到ijksdl_vout_overlay_android_mediacodec.c中的`SDL_VoutAMediaCodec_CreateOverlay`，这个函数中关键的几行是：

```php
SDL_VoutOverlay_Opaque *opaque = overlay->opaque;
opaque->buffer_proxy  = NULL;
overlay->opaque_class = &g_vout_overlay_amediacodec_class;
overlay->format       = SDL_FCC__AMC;
```

即：mediacodec的overlay的format指定为SDL_FCC__AMC，并且在opauqe中有一个buffer_proxy用于保存mediacodec解码后的buffer_index和bufferinfo。

> SDL_FCC__AMC主要用于在显示函数（ijksdl_vout_android_nativewindow.c）func_display_overlay_l中判断是否应该调用`SDL_VoutOverlayAMediaCodec_releaseFrame_l`来显示。分析见[ijkplayer video显示分析](https://zhuanlan.zhihu.com/p/45237178)

对于`overlay->func_fill_frame`会调用到ijksdl_vout_overlay_android_mediacodec.c中的`func_fill_frame`，这个函数中关键的几行是：

```php
opaque->buffer_proxy = (SDL_AMediaCodecBufferProxy *)frame->opaque;
overlay->opaque_class = &g_vout_overlay_amediacodec_class;
overlay->format     = SDL_FCC__AMC;
overlay->w = (int)frame->width;
overlay->h = (int)frame->height;
```

在`drain_output_buffer`中分析过，`amc_fill_frame`会把bufferinfo填入到AVFrame的Opaque中，这里只是再复制到了overlay->opaque中，方便在显示时访问该变量。

## 软解

软解的pipenode定义在ffpipenode_ffplay_vdec.h/c中。通过函数`ffpipenode_create_video_decoder_from_ffplay`来创建一个软解码器。不过ijkplayer中ffplay pipenode并不是由ffplay pipeline创建，而是由android pipeline创建：

```php
//ffpipeline_android.c
static IJKFF_Pipenode *func_open_video_decoder(IJKFF_Pipeline *pipeline, FFPlayer *ffp)
{
    IJKFF_Pipeline_Opaque *opaque = pipeline->opaque;
    IJKFF_Pipenode        *node = NULL;

    //如果有启用任何一种硬解选项，则创建mediacodec video decoder
    if (ffp->mediacodec_all_videos || ffp->mediacodec_avc || ffp->mediacodec_hevc || ffp->mediacodec_mpeg2)
        node = ffpipenode_create_video_decoder_from_android_mediacodec(ffp, pipeline, opaque->weak_vout);

    //如果没有启用硬解，或者创建失败，则返回ffplay video decoder
    if (!node) {
        node = ffpipenode_create_video_decoder_from_ffplay(ffp);
    }
    return node;
}
```

`ffpipenode_create_video_decoder_from_ffplay`定义如下：

```php
IJKFF_Pipenode *ffpipenode_create_video_decoder_from_ffplay(FFPlayer *ffp)
{
    IJKFF_Pipenode *node = ffpipenode_alloc(sizeof(IJKFF_Pipenode_Opaque));
    if (!node)
        return node;

    IJKFF_Pipenode_Opaque *opaque = node->opaque;
    opaque->ffp         = ffp;

    node->func_destroy  = func_destroy;
    node->func_run_sync = func_run_sync;

    ffp_set_video_codec_info(ffp, AVCODEC_MODULE_NAME, avcodec_get_name(ffp->is->viddec.avctx->codec_id));
    ffp->stat.vdec_type = FFP_PROPV_DECODER_AVCODEC;
    return node;
}
```

软解的关键实现在`func_run_sync`:

```php
static int func_run_sync(IJKFF_Pipenode *node)
{
    IJKFF_Pipenode_Opaque *opaque = node->opaque;
    return ffp_video_thread(opaque->ffp);
}

int ffp_video_thread(FFPlayer *ffp)
{
    return ffplay_video_thread(ffp);
}
```

这里的`ffplay_video_thread`实现基本与ffplay的`video_thread`是一样的，其分析参考[ffplay解码线程分析](https://zhuanlan.zhihu.com/p/43948483)

### queue_picture

软解与ffplay有所不同的地方在于`queue_picture`(将解码帧放入FrameQueue中)：

```php
static int queue_picture(FFPlayer *ffp, AVFrame *src_frame, double pts, double duration, int64_t pos, int serial)
{
    //……
    if (!(vp = frame_queue_peek_writable(&is->pictq)))
        return -1;

    //创建overlay
    if (!vp->bmp || !vp->allocated ||
        vp->width  != src_frame->width ||
        vp->height != src_frame->height ||
        vp->format != src_frame->format) {

        if (vp->width != src_frame->width || vp->height != src_frame->height)
            ffp_notify_msg3(ffp, FFP_MSG_VIDEO_SIZE_CHANGED, src_frame->width, src_frame->height);

        vp->allocated = 0;
        vp->width = src_frame->width;
        vp->height = src_frame->height;
        vp->format = src_frame->format;

        alloc_picture(ffp, src_frame->format);

        if (is->videoq.abort_request)
            return -1;
    }

    //填充overlay
    if (vp->bmp) {
        SDL_VoutLockYUVOverlay(vp->bmp);
        if (SDL_VoutFillFrameYUVOverlay(vp->bmp, src_frame) < 0) {
            av_log(NULL, AV_LOG_FATAL, "Cannot initialize the conversion context\n");
            exit(1);
        }
        SDL_VoutUnlockYUVOverlay(vp->bmp);

        //和ffplay类似，保存frame信息。不同的是，不保存frame数据
        vp->pts = pts;
        vp->duration = duration;
        vp->pos = pos;
        vp->serial = serial;
        vp->sar = src_frame->sample_aspect_ratio;
        vp->bmp->sar_num = vp->sar.num;
        vp->bmp->sar_den = vp->sar.den;

        frame_queue_push(&is->pictq);
        if (!is->viddec.first_frame_decoded) {
            ALOGD("Video: first frame decoded\n");
            ffp_notify_msg1(ffp, FFP_MSG_VIDEO_DECODED_START);
            is->viddec.first_frame_decoded_time = SDL_GetTickHR();
            is->viddec.first_frame_decoded = 1;
        }
    }

    return 0;
}
```

可以看到queue_picture中做了一些“绘图”相关的操作，即把AVFrame的数据绘制到vout overlay上。（这样就可以在显示的时候不关心具体的解码类型了）

#### 创建overlay

正常情况下`vp->bmp`创建一次后可重复使用，不需要重新创建。只有格式变化后，才需要调用`alloc_picture`重新创建。

```php
static void alloc_picture(FFPlayer *ffp, int frame_format)
{
    //……
    vp = &is->pictq.queue[is->pictq.windex];	//取当前要写入的帧，即peek_writabe的帧
    free_picture(vp);													//SDL_VoutFreeYUVOverlay(vp->bmp);

    SDL_VoutSetOverlayFormat(ffp->vout, ffp->overlay_format);

    vp->bmp = SDL_Vout_CreateOverlay(vp->width, vp->height,
                                   frame_format,
                                   ffp->vout);
    //……
    SDL_LockMutex(is->pictq.mutex);
    vp->allocated = 1;
    SDL_CondSignal(is->pictq.cond);
    SDL_UnlockMutex(is->pictq.mutex);
}
```

`alloc_picture`主要是通过调用SDL_Vout的接口，根据frame_format来创建一个Overlay。在[ijkplayer video显示分析](https://zhuanlan.zhihu.com/p/45237178)中分析过对于android默认通过`SDL_VoutAndroid_CreateForAndroidSurface`创建vout。该vout实现对应的overlay创建函数是：

```php
//SDL_LockMutex(vout->mutex);
static SDL_VoutOverlay *func_create_overlay_l(int width, int height, int frame_format, SDL_Vout *vout)
{
    switch (frame_format) {
    case IJK_AV_PIX_FMT__ANDROID_MEDIACODEC:
        return SDL_VoutAMediaCodec_CreateOverlay(width, height, vout);
    default:
        return SDL_VoutFFmpeg_CreateOverlay(width, height, frame_format, vout);
    }
}
//SDL_UnlockMutex(vout->mutex);
```

如果配置的是硬解，这里的frame_format将是IJK_AV_PIX_FMT__ANDROID_MEDIACODEC，会创建mediacodec“显示”所需的overlay，否则，创建的ffmpeg overlay。

#### 填充overlay

overlay的填充是调用的`SDL_VoutFillFrameYUVOverlay`，即`overlay->func_fill_frame`。对于软解调用的是ijksdl_vout_overlay_ffmpeg中的`func_fill_frame`。

```php
static int func_fill_frame(SDL_VoutOverlay *overlay, const AVFrame *frame)
{
    //……
    //1. 根据format决定后面的填充方法
    switch (overlay->format) {
        case SDL_FCC_YV12:
            need_swap_uv = 1;
            // no break;
        case SDL_FCC_I420:
            if (frame->format == AV_PIX_FMT_YUV420P || frame->format == AV_PIX_FMT_YUVJ420P) {
                // ALOGE("direct draw frame");
                use_linked_frame = 1;
                dst_format = frame->format;
            } else {
                // ALOGE("copy draw frame");
                dst_format = AV_PIX_FMT_YUV420P;
            }
            break;
        //……
    }

    //2. 准备内部frame用于填充；如果是use_linked_frame，则引用输入参数frame；否则开辟内存，存放到managed_frame
    if (use_linked_frame) {
        // linked frame
        av_frame_ref(opaque->linked_frame, frame);

        overlay_fill(overlay, opaque->linked_frame, opaque->planes);

        if (need_swap_uv)
            FFSWAP(Uint8*, overlay->pixels[1], overlay->pixels[2]);
    } else {
        // managed frame
        AVFrame* managed_frame = opaque_obtain_managed_frame_buffer(opaque);
        if (!managed_frame) {
            ALOGE("OOM in opaque_obtain_managed_frame_buffer");
            return -1;
        }

        overlay_fill(overlay, opaque->managed_frame, opaque->planes);

        // setup frame managed
        for (int i = 0; i < overlay->planes; ++i) {
            swscale_dst_pic.data[i] = overlay->pixels[i];
            swscale_dst_pic.linesize[i] = overlay->pitches[i];
        }

        if (need_swap_uv)
            FFSWAP(Uint8*, swscale_dst_pic.data[1], swscale_dst_pic.data[2]);
    }

    //3. 执行填充动作；对于use_linked_frame引用frame后即已填充好；否则需要用sws_scale转化到目标格式(可能是为了方便opengl绘图)
    if (use_linked_frame) {
        // do nothing
    } else if (ijk_image_convert(frame->width, frame->height,
                                 dst_format, swscale_dst_pic.data, swscale_dst_pic.linesize,
                                 frame->format, (const uint8_t**) frame->data, frame->linesize)) {
        opaque->img_convert_ctx = sws_getCachedContext(opaque->img_convert_ctx,
                                                       frame->width, frame->height, frame->format, frame->width, frame->height,
                                                       dst_format, opaque->sws_flags, NULL, NULL, NULL);
        if (opaque->img_convert_ctx == NULL) {
            ALOGE("sws_getCachedContext failed");
            return -1;
        }

        sws_scale(opaque->img_convert_ctx, (const uint8_t**) frame->data, frame->linesize,
                  0, frame->height, swscale_dst_pic.data, swscale_dst_pic.linesize);

        if (!opaque->no_neon_warned) {
            opaque->no_neon_warned = 1;
            ALOGE("non-neon image convert %s -> %s", av_get_pix_fmt_name(frame->format), av_get_pix_fmt_name(dst_format));
        }
    }
}
```

在queue_picture中将frame填充到opaque后，就可以调用`SDL_VoutDisplayYUVOverlay`去显示了。显示的逻辑可以参考[ffplay video显示线程分析](http://zhuanlan.zhihu.com/p/44122324)和[ijkplayer video显示分析](https://zhuanlan.zhihu.com/p/45237178)。

# 视频显示

ffplay基于sdl显示图像，ijkplayer在显示上摒弃了sdl，而是另辟蹊径封装了一套自己的显示接口。

## 基础概念

还是从显示函数开始看起(ff_ffplay.c)：

```php
stream_open -> SDL_CreateThreadEx video_refresh_thread
    ->video_refresh
        ->video_display2
            ->video_image_display2
                ->SDL_VoutDisplayYUVOverlay
```

整个调用链和ffplay保持一致，只是显示线程从主线程改变了到了一个独立线程中。最后在显示一帧图像的时候调用的是`SDL_VoutDisplayYUVOverlay`

```php
int SDL_VoutDisplayYUVOverlay(SDL_Vout *vout, SDL_VoutOverlay *overlay)
{
    if (vout && overlay && vout->display_overlay)
        return vout->display_overlay(vout, overlay);

    return -1;
}
```

在`SDL_VoutDisplayYUVOverlay`的函数里，我们看到了两个新的概念：`SDL_Vout`，`SDL_VoutOverlay`

这两个是ijk中才有的概念，是为了封装“显示上下文”和“显示层”准备的。

**SDL_Vout**

ijk中使用SDL_Vout表示一个显示上下文，或者理解为一块画布。比较接近于SDL中的Render。SDL_VoutOverlay表示显示层，或者理解为一块图像数据。比较接近于SDL中的Texture。

SDL_Vout的定义如下：

```c
struct SDL_Vout {
    SDL_mutex *mutex;

    SDL_Class       *opaque_class;
    SDL_Vout_Opaque *opaque;
    SDL_VoutOverlay *(*create_overlay)(int width, int height, int frame_format, SDL_Vout *vout);
    void (*free_l)(SDL_Vout *vout);
    int (*display_overlay)(SDL_Vout *vout, SDL_VoutOverlay *overlay);

    Uint32 overlay_format;
};
```

有了解过[C语言中的子类](https://zhuanlan.zhihu.com/p/42919249)的应该对于这样的结构体并不陌生。它定义了一个“类/接口”，支持的方法有`create_overlay`，`free_l`，`display_overlay`。其中，最重要的是`display_overlay`方法，即如何去呈现一个overlay.

既然是一个接口，就有对应的实现类：

- dummy: 这是一个空实现。定义在ijk_sdl_vout_dummy.c。
- android surface vout: 这是基于Android的surface实现的。定义在ijk_sdl_android_surface.c，ijk_vout_android_nativewindow.c

**SDL_VoutOverlay**

SDL_VoutOverlay的定义如下：

```c
struct SDL_VoutOverlay {
    int w; /**< Read-only */
    int h; /**< Read-only */
    Uint32 format; /**< Read-only */
    int planes; /**< Read-only */
    Uint16 *pitches; /**< in bytes, Read-only */
    Uint8 **pixels; /**< Read-write */

    int is_private;

    int sar_num;
    int sar_den;

    SDL_Class               *opaque_class;
    SDL_VoutOverlay_Opaque  *opaque;

    void    (*free_l)(SDL_VoutOverlay *overlay);
    int     (*lock)(SDL_VoutOverlay *overlay);
    int     (*unlock)(SDL_VoutOverlay *overlay);
    void    (*unref)(SDL_VoutOverlay *overlay);

    int     (*func_fill_frame)(SDL_VoutOverlay *overlay, const AVFrame *frame);
};
```

同样也是一个c风格的“类/接口”，定义的方法有`free_l/lock/unlock/unref/func_fill_frame`，其中最重要的是`func_fill_frame`，也就是把AVFrame的图像“画”到overlay上。

它也有对应的几种实现：

- ffmpeg overlay: 用于软解绘图，主要是内存图像数据的格式转换。定义在ijksdl_vout_ffmpeg_overlay.c
- mediacodec overlay: 用于mediacodec硬解绘图，主要用于MediaCodec的buffer index wrap和管理。定义在ijksdl_vout_overlay_android_mediacodec.c。（mediacodec可以直接绑定Surface解码，以提升效率，这种方式的解码是拿不到图像数据的，所以这里要wrap index）



**目录结构**

上面提到的一些结构体和函数，都在目录ijkmedia/ijksdl/下：

```text
+ ijkmedia/ijksdl
    - ijk_sdl.h                             //包含其他sdl头文件
    - ijksdl_vout.h/c                       //封装层，提供SDL_VoutXXX的函数调用vout和overlay
    - ijksdl_vout_internal.h                //ijksdl目录内部使用的一些util函数
    + dummy                                 //dummy vout实现
    + ffmpeg
        - ijksdl_vout_overlay_ffmpeg.h/c    //ffmpeg overlay实现
    + android
        - ijksdl_vout_android_nativewindow.c//android vout的主要实现
        - ijksdl_vout_android_surface.h/c   //android vout与Java层Surface连接层
        - ijksdl_vout_overlay_android_mediacodec.h/c    //mediacodec overlay实现
```

当然，目录里还包括的timer/mutex/thread等的封装，另外音频相关的sdl封装也在这个目录里，将在音频输出一文中分析。

## 流程分析

Android上的SDL_Vout是通过ijksdl_vout_android_surface.c中的`SDL_VoutAndroid_CreateForAndroidSurface`函数创建的，对应的SDL_Vout的实现在ijksdl_vout_android_nativewindow.c。

`SDL_VoutAndroid_CreateForAndroidSurface`调用流程如下：

```text
new IjkMediaPlayer() 
    -> initPlayer() 
        -> native_setup() 
            -> IjkMediaPlayer_native_setup() 
                -> ijkmp_android_create() 
                    -> SDL_VoutAndroid_CreateForAndroidSurface()
```

SDL_Vout创建后就可以用来显示SDL_VoutOverlay了，overlay的创建和填充是在解码线程中完成，将在解码线程一文中分析，这里略过。直接看显示部分。

前面分析了overlay的显示是在video_display2中调用`SDL_VoutDisplayYUVOverlay`显示的。`SDL_VoutDisplayYUVOverlay`只是封装了具体SDL_Vout实现类的`display_overlay`方法。对于Android，对应的是ijksdl_vout_android_nativewindow.c中的`func_display_overlay`：

```c
static int func_display_overlay(SDL_Vout *vout, SDL_VoutOverlay *overlay)
{
    SDL_LockMutex(vout->mutex);
    int retval = func_display_overlay_l(vout, overlay);
    SDL_UnlockMutex(vout->mutex);
    return retval;
}
```

加锁调用`func_display_overlay_l`(精简了代码，直接看正常流程代码)：

```c
static int func_display_overlay_l(SDL_Vout *vout, SDL_VoutOverlay *overlay)
{
    switch(overlay->format) {
    case SDL_FCC__AMC: {
        // only ANativeWindow support
        IJK_EGL_terminate(opaque->egl);
        return SDL_VoutOverlayAMediaCodec_releaseFrame_l(overlay, NULL, true);
    }
    case SDL_FCC_RV24:
    case SDL_FCC_I420:
    case SDL_FCC_I444P10LE: {
        // only GLES support
        if (opaque->egl)
            return IJK_EGL_display(opaque->egl, native_window, overlay);
        break;
    }
    case SDL_FCC_YV12:
    case SDL_FCC_RV16:
    case SDL_FCC_RV32: {
        // both GLES & ANativeWindow support
        if (vout->overlay_format == SDL_FCC__GLES2 && opaque->egl)
            return IJK_EGL_display(opaque->egl, native_window, overlay);
        break;
    }
    }

    // fallback to ANativeWindow
    IJK_EGL_terminate(opaque->egl);
    return SDL_Android_NativeWindow_display_l(native_window, overlay); 
}
```

这里主要是根据传入overlay的format选择具体的显示方法。这里有3种显示方法：

- 如果是SDL_FCC__AMC，则是MediaCodec的特定format，用`SDL_VoutOverlayAMediaCodec_releaseFrame_l`显示
- 如果是其他EGL支持的格式，则用`IJK_EGL_display`显示
- 最后，如果都无法显示，就Fallback到直接用ndk nativewindow的api显示

其中第一种是针对MediaCodec的硬解显示方式，后两种都是软解的显示方式。

接下来分别从硬解和软解分类分析显示流程。



**硬解显示**

MediaCodec是Android硬解的统一API，方便了不同芯片厂商接入。关于MediaCodec的使用，可以参考这篇：https://zhuanlan.zhihu.com/p/45224834。

MediaCodec解码时设置一个Surface以减少显示时的数据拷贝，可以提高效率。此时解码后拿到的是一个index，并非解码后的图像数据，ijk中将其封装为`SDL_AMediaCodecBufferProxy`，定义在jksdl_vout_android_nativewindow.c中：

```c
struct SDL_AMediaCodecBufferProxy
{
    int buffer_id;
    int buffer_index;
    int acodec_serial;
    SDL_AMediaCodecBufferInfo buffer_info;
};
```

SDL_AMediaCodecBufferProxy的实例在android overlay的SDL_VoutOverlay_Opaque中定义：

```c
typedef struct SDL_VoutOverlay_Opaque {
    SDL_mutex *mutex;

    SDL_Vout                   *vout;
    SDL_AMediaCodec            *acodec;

    SDL_AMediaCodecBufferProxy *buffer_proxy;//这个是dequeueOutputBuffer的封装

    Uint16 pitches[AV_NUM_DATA_POINTERS];
    Uint8 *pixels[AV_NUM_DATA_POINTERS];
} SDL_VoutOverlay_Opaque;
```

MediaCodec要显示一帧，是通过调用`releaseOutputBuffer`通知MediaCodec该显示哪一帧的。这在ijk中是封装为`SDL_AMediaCodec_releaseOutputBuffer`。

回到刚才的思路，看下`SDL_VoutOverlayAMediaCodec_releaseFrame_l`函数：

```c
int  SDL_VoutOverlayAMediaCodec_releaseFrame_l(SDL_VoutOverlay *overlay, SDL_AMediaCodec *acodec, bool render)
{
    if (!check_object(overlay, __func__))
        return -1;

    SDL_VoutOverlay_Opaque *opaque = overlay->opaque;
    return SDL_VoutAndroid_releaseBufferProxyP_l(opaque->vout, &opaque->buffer_proxy, render);
}
```

`SDL_VoutAndroid_releaseBufferProxyP_l`调用了`SDL_VoutAndroid_releaseBufferProxy_l`：

```c
//这里省略了打印调试信息的代码
static int SDL_VoutAndroid_releaseBufferProxy_l(SDL_Vout *vout, SDL_AMediaCodecBufferProxy *proxy, bool render)
{
    SDL_Vout_Opaque *opaque = vout->opaque;

    //归还到SDL_AMediaCodecBufferProxy对象池
    ISDL_Array__push_back(&opaque->overlay_pool, proxy);

    //如果serial已经变化，说明该帧已经无效(比如是seek前的帧)，不显示，丢弃
    if (!SDL_AMediaCodec_isSameSerial(opaque->acodec, proxy->acodec_serial)) {
        return 0;
    }

    //buffer_index即dequeueOutputBuffer的返回值，小于0说明不是一个图像帧(具体参考官方API)，无需显示
    if (proxy->buffer_index < 0) {
        return 0;
    } 
    //FAKE_FRAME是一个特殊标志，表示当前帧只是占位，不应该显示(具体分析将在ijkplay硬解一文分析)
    else if (proxy->buffer_info.flags & AMEDIACODEC__BUFFER_FLAG_FAKE_FRAME) {
        proxy->buffer_index = -1;
        return 0;
    }

    //到这里，就可以显示了
    sdl_amedia_status_t amc_ret = SDL_AMediaCodec_releaseOutputBuffer(opaque->acodec, proxy->buffer_index, render);    
    if (amc_ret != SDL_AMEDIA_OK) {//显示失败，返回-1
        proxy->buffer_index = -1;
        return -1;
    }

    proxy->buffer_index = -1;
    return 0;
}
```

上面代码，主要做了3件事情：

1. 归还proxy到SDL_AMediaCodecBufferProxy对象池。因为解码的调用很频繁，如果重复分配释放proxy对象，内存压力会比较大，所以这里引入了一个对象池进行优化。
2. 做一些合法性检查。比如检查serial变化、检查index值、检查占位符等
3. 调用SDL_AMediaCodec_releaseOutputBuffer（也就是MediaCodec.releaseOutputBuffer）显示



**软解显示**

这块没怎么接触，有空分析后补充。



















# 音频输出

















# 参考资料

- [Android视频播放软解与硬解的区别_Dawish_大D的博客-CSDN博客_android硬解码与软解码](https://blog.csdn.net/u010072711/article/details/52413766)
- [什么是JNI？为什么会有Native层？如何使用？ - 简书 (jianshu.com)](https://www.jianshu.com/p/9adf0c716566)
- [开源播放器 ijkplayer (一) ：使用Ijkplayer播放直播视频 - 灰色飘零 - 博客园 (cnblogs.com)](https://www.cnblogs.com/renhui/p/6420140.html)
- [Android ijkplayer详解使用教程 - 星辰之力 - 博客园 (cnblogs.com)](https://www.cnblogs.com/zhujiabin/p/7211983.html)
- [ijkplayer-android框架详解_Suk_39799839的博客-CSDN博客_ijkplayer](https://blog.csdn.net/weixin_39799839/article/details/79186034)
- [ijkplayer中遇到的问题汇总 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/116008680)
- [ijkplayer源码分析 整体概述_baiiu的博客-CSDN博客_ijkplayer源码解析](https://blog.csdn.net/u014099894/article/details/112969853)
- [ijkplayer 源码分析 - 简书 (jianshu.com)](https://www.jianshu.com/p/32a1d821189b)
- [带着问题，再读ijkplayer源码_mob604756ffeae8的技术博客_51CTO博客](https://blog.51cto.com/u_15127656/2783837?abTest=51cto)
- [Android NDK MediaCodec在ijkplayer中的实践 - 简书 (jianshu.com)](https://www.jianshu.com/p/41d3147a5e07)
- [ijkplayer 解码框架分析 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/45251444?ivk_sa=1024320u)
- [ijkplayer 解码实现分析——硬解篇 - 知乎 (zhihu.com)](https://www.zhihu.com/column/p/45441051)
- [初识MediaCodec - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/45224834)
- [ijkplayer video显示分析 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/45237178)
- [ijkplayer audio输出分析 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/45518054)
- 







