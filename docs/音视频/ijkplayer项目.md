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

从上面可以看出ijkplayer是暂时不支持音频硬件解码的。

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
- 







