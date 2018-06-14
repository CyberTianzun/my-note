---
title: 交叉编译 OpenCV for Android
date: 2018-6-14
tags: "opencv"
categories: "opencv"
---

最近因为搞了搞一些需要用到 OpenCV 的东西，并且有些功能还需要写到 Android 上，折腾的过程中还是遇到了不少的问题，这篇文章主要是记录一下我踩过的坑吧，并且会讲完整一点，也作为我查阅了各种各样的资料之后写的一个总结篇。因为 NDK 支持 Cmake了，这是好东西啊，所以这篇文章就只讲 Cmake 相关的配置吧。正文开始之前，先介绍一下问题概要：

* 我有一部分现成的 C++ 代码在 macOS/Linux 已经调试完成了，毕竟在 maxOS上开发和调试都方便，现在要尽量少的修改能移植到 Android 端上面。如何最优地迁移代码？

* OpenCV 有提供 Android SDK，我也不再详细介绍使用方法了，问题在于这个 SDK 中使用已经编译好的 .so 最大能到 19M，很明显一个正经的App产品发布，肯定不会携带这么大的库的。Apk包大小可是会影响升级/推广转化率的重要指标之一。如何压缩 OpenCV 库？


## 准备工作


1. Android SDK

    确保你的 Android Studio 的版本已经升级到 2.2 及以上，确保 Tools -> Android -> SDK Manager -> SDK Tools 里面安装好 CMake, LLDB, NDK，注意确保 NDK 的版本不要太老太旧。

    ![](android-sdk-preferences.png)


2. 去 [OpenCV 官网(传送门)](https://www.opencv.org/releases.html) 下载 OpenCV 源代码

    这里选择了源代码而不是 Android Pack，我简单说一下，Android Pack 是一个已经编译好的 Android SDK，提供了**已经编译好的各个 CPU 架构下的 .so 动态链接库**，以及一些常用类的 **Java 封装**，比如 `Mat` 类的 Java 封装。这样的包当然用起来是非常简单的，特别是提供给写 Demo 或者学生做研究，问题在于这个 Android Pack 里面的 .so 动态链接库真的是巨大无比，最大能到 19M，难免以后不会更多，在**生产环境中使用这样的库明显是不太现实的**，所以我们选择下载 OpenCV 的源代码，这样可以自己选择性的编译自己需要的模块。下图中画了两个红框，可以视自己的情况而定下载自己需要版本的 OpenCV Source。

    ![](download-opencv-source.png)


## 交叉编译 OpenCV

解压你下载的 OpenCV Source，我的解压到了 `/java-frameworks/opencv-3.4.1` 这里，后面使用命令的时候注意替换成自己的文件夹地址。在 OpenCV 的源代码中，已经准备好了为 Android 交叉编译的 CMakeLists.txt 了，所以我们用好 CMake 就好了。参考一下下面的bash命令，我先编译一个 `armeabi-v7a` 架构的 OpenCV 库吧：

```bash
# 初始化一些变量，方便按自己的需要更改
export ANDROID_ABI="armeabi-v7a"
export OPENCV_ROOT="/java-frameworks/opencv-3.4.1"
export ANDROID_NDK="/java-frameworks/android-sdk-macosx/ndk-bundle"
export ANDROID_API="15"

# 准备编译源代码
cd $OPENCV_ROOT
mkdir build && cd build

# 生成交叉编译的脚本
cmake -DCMAKE_TOOLCHAIN_FILE=$ANDROID_NDK/build/cmake/android.toolchain.cmake -DANDROID_NDK=${ANDROID_NDK} -DANDROID_NATIVE_API_LEVEL=$ANDROID_API -DCMAKE_BUILD_TYPE=Release -DANDROID_ABI="${ANDROID_ABI}" $OPENCV_ROOT
make -j8
make install
```


如果编译了 armeabi-v7a，还想编译别的架构的库，可以使用 `make clean` 命令之后，再把上面的命令 `ANDROID_ABI`参数修改一下，再执行一遍，最后所有架构都会自己分好类分在该在的地方的。在一堆耗时任务全部执行完成之后，在 `build/install` 目录下，得到了编译的成果：

![](cross-compiled-opencv-complete.png)


所有的模块都被编译了，而且，真的是太多了，实际上我需要尽量减少依赖和我不需要的东西，下面重新调整一下命令吧，我的期望是我只需要 core 模块就足够了，其他的我一概不需要。至于其他同学可以参考自己的需求调整了。按照下面的方法调整 cmake 指令：

```bash
export ANDROID_ABI="armeabi-v7a"
export OPENCV_ROOT="/java-frameworks/opencv-3.4.1"
export ANDROID_NDK="/java-frameworks/android-sdk-macosx/ndk-bundle"
export ANDROID_API="15"

cd $OPENCV_ROOT
# 删除之前的编译成果
rm -rf build
mkdir build && cd build

# 生成编译脚本
cmake -DCMAKE_TOOLCHAIN_FILE=$ANDROID_NDK/build/cmake/android.toolchain.cmake \
      -DANDROID_NDK=${ANDROID_NDK}            -DANDROID_ABI="${ANDROID_ABI}" \
      -DANDROID_NATIVE_API_LEVEL=$ANDROID_API -DCMAKE_BUILD_TYPE=Release \
      -DWITH_CUDA=off                         -DBUILD_DOCS=off \
      -DBUILD_SHARED_LIBS=off                 -DBUILD_FAT_JAVA_LIB=off \
      -DBUILD_TESTS=off                       -DBUILD_TIFF=off \
      -DBUILD_JASPER=off                      -DBUILD_JPEG=off \
      -DBUILD_OPENEXR=off                     -DBUILD_PNG=off \
      -DBUILD_TIFF=off                        -DBUILD_ZLIB=off \
      -DBUILD_opencv_calib3d=off              -DBUILD_opencv_core=on \
      -DBUILD_opencv_cudaarithm=off           -DBUILD_opencv_cudabgsegm=off \
      -DBUILD_opencv_cudacodec=off            -DBUILD_opencv_cudafeatures2d=off \
      -DBUILD_opencv_cudafilters=off          -DBUILD_opencv_cudaimgproc=off \
      -DBUILD_opencv_cudalegacy=off           -DBUILD_opencv_cudaobjdetect=off \
      -DBUILD_opencv_cudaoptflow=off          -DBUILD_opencv_cudastereo=off \
      -DBUILD_opencv_cudawarping=off          -DBUILD_opencv_cudev=off \
      -DBUILD_opencv_dnn=off                  -DBUILD_opencv_features2d=off \
      -DBUILD_opencv_flann=off                -DBUILD_opencv_highgui=off \
      -DBUILD_opencv_imgcodecs=off            -DBUILD_opencv_imgproc=off \
      -DBUILD_opencv_java=off                 -DBUILD_opencv_js=off \
      -DBUILD_opencv_ml=off                   -DBUILD_opencv_objdetect=off \
      -DBUILD_opencv_photo=off                -DBUILD_opencv_python=off \
      -DBUILD_opencv_shape=off                -DBUILD_opencv_stitching=off \
      -DBUILD_opencv_superres=off             -DBUILD_opencv_ts=off \
      -DBUILD_opencv_video=off                -DBUILD_opencv_videoio=off \
      -DBUILD_opencv_videostab=off            -DBUILD_opencv_viz=off \
      -DBUILD_opencv_world=off                -DWITH_1394=off \
      -DWITH_EIGEN=off                        -DWITH_FFMPEG=off \
      -DWITH_GIGEAPI=off                      -DWITH_GSTREAMER=off \
      -DWITH_GTK=off                          -DWITH_PVAPI=off \
      -DWITH_V4L=off                          -DWITH_LIBV4L=off \
      -DWITH_CUDA=off                         -DWITH_CUFFT=off \
      -DWITH_OPENCL=off                       -DBUILD_EXAMPLES=off \
      -DBUILD_opencv_apps=off                 -DBUILD_DOCS=off \
      -DBUILD_PERF_TESTS=off                  -DBUILD_TESTS=off \
      ${OPENCV_ROOT}
make -j8
make install
```


又是一段漫长的耗时任务，然后得到了如下的成果：

![](cross-compiled-opencv-onlycore.png)


编译结果就只有一个 `libopencv_core.a`，现在看起来是想当干净的了。实际上在上面的编译过程中遇到的坑不少，首先是因为 Android NDK 更新的缘故，与网络上非常多教程中提供的 android cmake toolchain 配置都无法对应上，OpenCV 代码库内的 cmake toolchain 也是无法使用的。如果 NDK 有提供 cmake toolchain 的话，一定要尽量使用 NDK 提供的。编译成功之后，下面要简单讲一讲怎么使用了。


## 在 JNI 代码中使用 


首先，创建新的 Android 项目，记得勾选 “Include C++ Support”。

![](create-android-project.png)


然后 Android Studio 已经生成好一个模板了，在 `native-lib.cpp` 文件中，能看到下面这段代码：

```cpp
#include <jni.h>
#include <string>

extern "C"
JNIEXPORT jstring JNICALL
Java_com_mi_mier_is_myapplication_MainActivity_stringFromJNI(
        JNIEnv* env,
        jobject /* this */) {
    std::string hello = "Hello from C++";
    return env->NewStringUTF(hello.c_str());
}
```

增加几句测试一下吧：


```cpp
#include <jni.h>
#include <string>
#include <opencv2/opencv.hpp>

extern "C"
JNIEXPORT jstring JNICALL
Java_com_mi_mier_is_myapplication_MainActivity_stringFromJNI(
        JNIEnv* env,
        jobject /* this */) {

    auto height = 500, width = 500;
    cv::Mat mat = cv::Mat::zeros(height, width, cv::CV_8UC3);

    std::string hello = "Hello from C++";
    return env->NewStringUTF(hello.c_str());
}
```


然后在 `CMakeLists.txt` 要添加一下刚才编译的 OpenCV 配置才行。


```cmake
set(OpenCV_ROOT_DIR /java-frameworks/opencv-3.4.1/build/install/sdk/native)
find_package(OpenCV REQUIRED)
include_directories(${OpenCV_ROOT_DIR}/jni/include)
```


以及，在 `target_link_libraries` 的语句里面，链接一下 OpenCV 的静态库 `opencv_core`，然后还是会有一些压缩相关的方法找不到，原因是在交叉编译 OpenCV for Android 的时候，是没有引入 zlib 库的，所以在链接库里面还要增加一个叫做 `z` 的库。大概就是下面这样：


```cmake
target_link_libraries( # Specifies the target library.
                       native-lib

                       opencv_core

                       libIlmImf
                       liblibjasper
                       liblibjpeg
                       liblibpng
                       liblibtiff
                       libzlib

                       # Links the target library to the log library
                       # included in the NDK.
                       android
                       z
                       ${log-lib})
```


大功告成，就可以编译执行了。

