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

   ```c
   brew install git-lfs
   git lfs install
   git lfs pull
   ```

> JNI是Java Native Interface的缩写，通过使用 [Java](https://baike.baidu.com/item/Java/85979)本地接口书写程序，可以确保代码在不同的平台上方便移植。从Java1.1开始，JNI标准成为java平台的一部分，它允许Java代码和其他语言写的[代码](https://baike.baidu.com/item/代码/86048)进行交互。JNI一开始是为了本地已[编译](https://baike.baidu.com/item/编译/1258343)语言，尤其是C和C++而设计的，但是它并不妨碍你使用其他编程语言，只要调用约定受支持就可以了。使用java与本地已编译的代码[交互](https://baike.baidu.com/item/交互/6964417)，通常会丧失平台[可移植性](https://baike.baidu.com/item/可移植性/6931884)。但是，有些情况下这样做是可以接受的，甚至是必须的。例如，使用一些旧的库，与硬件、操作系统进行交互，或者为了提高程序的性能。JNI标准至少要保证[本地代码](https://baike.baidu.com/item/本地代码)能工作在任何Java [虚拟机](https://baike.baidu.com/item/虚拟机)环境。

# 资源

- [开源播放器 ijkplayer (一) ：使用Ijkplayer播放直播视频 - 灰色飘零 - 博客园 (cnblogs.com)](https://www.cnblogs.com/renhui/p/6420140.html)
- [Android ijkplayer详解使用教程 - 星辰之力 - 博客园 (cnblogs.com)](https://www.cnblogs.com/zhujiabin/p/7211983.html)
- [ijkplayer-android框架详解_Suk_39799839的博客-CSDN博客_ijkplayer](https://blog.csdn.net/weixin_39799839/article/details/79186034)
- [ijkplayer中遇到的问题汇总 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/116008680)
- [ijkplayer源码分析 整体概述_baiiu的博客-CSDN博客_ijkplayer源码解析](https://blog.csdn.net/u014099894/article/details/112969853)
- [ijkplayer 源码分析 - 简书 (jianshu.com)](https://www.jianshu.com/p/32a1d821189b)
- [带着问题，再读ijkplayer源码_mob604756ffeae8的技术博客_51CTO博客](https://blog.51cto.com/u_15127656/2783837?abTest=51cto)
- 

# 记录

ijkplayer核心源码主要在ijkmedia文件夹下ijkplayer、ijksdl及ijkutils。

android相关源码结构：

- ijkmediademo: 播放器实例demo
- ijkmediawidget: 播放器组件封装,类似于系统播放器、vitamio结构，如mediacontroller、videoView。
- ijkmediaplayer: cpu armv7库。播放器核心jni层及相关上层调用接口开放(包括系统播放器封装切换)。
- ijkmediaplayer--* 其他cpu兼容库

jni API接口及主要核心流程：

- ijkmedia/ijkplayer/android/ijkplayer-android.c
- ijkmedia/ijkplayer/android/ijkplayer-jni.c
- ijkmedia/ijkplayer/ff_ffplay.c
- ijkmedia/ijkplayer/ijkplayer.c









