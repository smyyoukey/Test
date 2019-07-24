## 2019 Google IO分享

### 1.What's New in Android C++ Development (Google I/O'19)

&emsp;&emsp;这个主题主要涉及到一些NDK的变化。之前项目中引入过NDK的应该都经历过NDK的兼容性问题，项目编不过然后去找官方NDK下载，从 NDK r11一个个试试到r18,与SDK不兼容，或是与gradle不兼容的问题比比皆是。根据会中内容所说，担任NDK研发的团队这些年主要的任务就是重构NDK部分架构，该工作从r11开始到r14初有进展，然后r17到
r18有较大的改动，在r19之后的NDK版本这些问题应该会好很多（但前提是gradle的版本和SDK的版本得非常高）。下面我总结一下会中内容：

#### 变化

##### 1.GCC ：用C / C ++编写的代码可以使用Android Native Development Kit（NDK）编译为ARM或x86本机代码（或其64位变体）。 NDK使用Clang编译器编译C / C ++。 GCC被纳入NDK r17，同时Clang编译器被设置为默认编译器（为了更好的兼容，移植）。但GCC在2018年的r18中被移除。

##### 2.LLVM libc++（r16-r18）：在NDK r17版本中被设置为默认库

##### 3.NDK r19以上官方支持ndk—build（android）与CMake（跨平台）

#### 开发过程中：

##### 1.会catch到更多的bug（异常）比如说 invalid pthread_t passed to libc (无效的pid)

&emsp;&emsp;这里解释一下平常native层代码的问题抛出调用栈，但该调用栈不是可以直接看明白的调用栈。必须根据产生bug当时抛出的trace文件去一一对应pid，才有可读性。而且就算解读了，也不一定能根据该输出找到问题所在，更别说还有一大堆异常根本不会被catch到。这个变化的意思就是，可以在这个过程中可以catch到更多异常了。


比如说下面这个metter在老版本的ndk中不会报错，程序会继续运行，然后可能会发生其他问题（因为这个问题没有catch到）
<div align="center">
   <img src="./2019 Google IO分享/TIM截图20190724114940metter.png"  alt="Memory Profiler 界面" />
</div>

然而现在的版本中如果出现了这个问题，程序会crash。

##### 2.之前log输出调用栈信息在进入java棧会停止，因为java的调用栈运行与维护方式与C/C++中的不同。

<div align="center">
   <img src="./2019 Google IO分享/TIM截图20190724114514.png"  alt="Memory Profiler 界面" />
</div>

##### 但在Android P/Q中优化了对其的识别，之后C/C++与java调用栈会连续输出。

<div align="center">
   <img src="./2019 Google IO分享/TIM截图20190724114538.png"  alt="Memory Profiler 界面" />
</div>

##### 3.simplepref——一个查看native层对cpu资源占用的工具（有GUI界面）

<div align="center">
   <img src="./2019 Google IO分享/TIM截图20190724114712simplepref.png"  alt="Memory Profiler 界面" />
</div>

##### 4.C++中有许多undefined行为，有时会导致很多问题，Clang提供了一个查找该问题的方式，可以详细地追踪调用栈，只需要在Android.mk中设置;                                          
PS：但这个方面并没有详细的演示，估计是还有其他问题（以上很多功能都是Clang或是LLVM 9 的新功能，我认为有演示的功能都是基本上可以在开发android 项目的时候用了，没有演示的应该是还没做好兼容）

NDK方面新特性的内容在wiki百科android页面NDK词条中有很详细的收录（wiki更新的真快）我把词条贴在下面：

#### NDK

&emsp;&emsp;Code written in C/C++ can be compiled to ARM, or x86 native code (or their 64-bit variants) using the Android Native Development Kit (NDK). The NDK uses the Clang compiler to compile C/C++. GCC was included until NDK r17, but removed in r18 in 2018.

&emsp;&emsp;Native libraries can be called from Java code running under the Android Runtime using System.loadLibrary, part of the standard Android Java classes.[23][24]

&emsp;&emsp;Command-line tools can be compiled with the NDK and installed using adb.[25]

&emsp;&emsp;Android uses Bionic as its C library, and the LLVM libc++ as its C++ Standard Library. The NDK also includes a variety of other APIs:[26] zlib compression, OpenGL ES or Vulkan graphics, OpenSL ES audio, and various Android-specific APIs for things like logging, access to cameras, or accelerating neural networks.

&emsp;&emsp;The NDK includes support for CMake and its own ndk-build (based on GNU Make). Android Studio supports running either of these from Gradle. Other third-party tools allow integrating the NDK into Eclipse[27] and Visual Studio.[28]

&emsp;&emsp;For CPU profiling, the NDK also includes simpleperf[29] which is similar to the Linux perf tool, but with better support for Android and specifically for mixed Java/C++ stacks.

我觉得这一节知道以上词条内容就行。下面是翻译：

用C / C ++编写的代码可以使用Android Native Development Kit（NDK）编译为ARM或x86本机代码（或其64位变体）。 NDK使用Clang编译器编译C / C ++。 GCC被纳入NDK r17，但在2018年的r18中被移除。

可以使用System.loadLibrary从Android Runtime下运行的Java代码调用本机库，System.loadLibrary是标准Android Java类的一部分。[23] [24]

命令行工具可以使用NDK编译并使用adb进行安装。[25]

Android使用Bionic作为其C库，并使用LLVM libc ++作为其C ++标准库。 NDK还包括各种其他API：[26] zlib压缩，OpenGL ES或Vulkan图形，OpenSL ES音频以及各种特定于Android的API，用于记录，访问摄像机或加速神经网络等。

NDK包括对CMake及其自己的ndk-build（基于GNU Make）的支持。 Android Studio支持从Gradle运行其中任何一个。其他第三方工具允许将NDK集成到Eclipse [27]和Visual Studio中。[28]

对于CPU分析，NDK还包括simpleperf [29]，类似于Linux perf工具，但更好地支持Android，特别是混合Java / C ++堆栈。

### 2.Best Practices in Using the Android Emulator (Google I/O'19)

&emsp;&emsp;这个主题主要涉及到一些android模拟器的变化。会中主题是android模拟器在数次的迭代的过程中，有一个最大的目标，就是模拟器的环境，运行情况要与真机无异，并且在运行速度，渲染速度，等各项性能指标中完全超过真机，使虚拟机成为开发，测试的第一选择。


#### 1.可折叠技术（foldable）：可以创造折叠手机型的模拟器了。

#### 2.运行速度提升

<div align="center">
   <img src="./2019 Google IO分享/TIM截图20190724120504gpu.png"  alt="Memory Profiler 界面" />
</div>

之前的模拟器运行时是翻译每一条指令，从客户端的每一个操作指令进行解码使X86架构的CPU可以理解。但现在是在X86的架构上运行X86（一个比方）

#### 3.通信与传输更方便，快捷
<div align="center">
   <img src="./2019 Google IO分享/TIM截图20190724121257tcp.png"  alt="Memory Profiler 界面" />
</div>

键入adb push 命令通过共享内存上的虚拟TCP链接进行通信，传输

#### 4.模拟器启动方式与快照（snapshot）

通常启动虚拟机叫冷启动（cold boot），snapshot可以迅速还原到当时的状态。

#### 5.真机与模拟器对比测试
<div align="center">
   <img src="./2019 Google IO分享/TIM截图20190724122029same test.png"  alt="Memory Profiler 界面" />
</div>

该测试通过对比真机与模拟器运行同一套环境，确保模拟器行为和真实设备统一，在不同机型，不同sdk版本的情况下的运行情况，来体现其强大的兼容性，且模拟器通常比硬件设备快的多。

#### 6.同时运行模拟器实例

可以创建多个模拟器实例，也可以创建不同sdk版本的模拟器，可以通过一条命令同时启动。

#### 7.模拟器无头模式
模拟器的无头模式是创建一个没有GUI的模拟器，由于不需要GPU渲染也不需要各种更新操作，相当于一个虚拟终端。
<div align="center">
   <img src="./2019 Google IO分享/TIM截图20190724143859headless.png"  alt="Memory Profiler 界面" />
</div>

在logcat中可以看到该模拟器的输出：
<div align="center">
   <img src="./2019 Google IO分享/TIM截图20190724144007headless1.png"  alt="Memory Profiler 界面" />
</div>
该虚拟终端的优点是，由于没有界面，占用的资源非常少，当进行大量并行测试或压力测试时可以用到。

------------------
### 内容就是这些，本次分享到此结束。
