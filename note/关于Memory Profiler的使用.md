# Memory Profiler

 Memory Profiler 是可帮助识别导致应用卡顿、冻结甚至崩溃的内存泄漏和流失。 它显示一个应用内存使用量的实时图表，可以捕获堆转储、强制执行垃圾回收以及跟踪内存分配。

Memory Profiler 概览

<div align="center">
<img src="./pic about Memory Profiler/memory_ui.png"  alt="Memory Profiler 界面" />
 </div>

#### 如上图所示，Memory Profiler 的默认视图包括以下各项：

① 强制执行垃圾回收 Event 。

② 捕获堆转储。

③ 记录内存分配情况。 此按钮仅在运行 Android 7.1 或更低版本的设备时才会显示。

④ 放大/缩小时间线。

⑤ 跳转至实时内存数据。

⑥ Event 时间线，其显示 Activity 状态、用户输入 Event 和屏幕旋转 Event。

⑦ 内存使用量时间线，包含以下内容：

显示每个内存类别使用多少内存的堆叠图表，如左侧的 y 轴以及顶部的彩色键所示。
虚线表示分配的对象数，如右侧的 y 轴所示。
用于表示每个垃圾回收 Event 的图标。

#### 同如上图，内存计数中的类别如下所示:

**Java**：从 Java 或 Kotlin 代码分配的对象内存。

**Native**：从 C 或 C++ 代码分配的对象内存。

**Graphics**：图形缓冲区队列向屏幕显示像素（包括 GL 表面、GL 纹理等等）所使用的内存。 （注意，这是与 CPU 共享的内存，不是 GPU 专用内存。）

**Stack**： 应用中的原生堆栈和 Java 堆栈使用的内存，通常与应用运行多少线程有关。

**Code**：应用用于处理代码和资源（如 dex 字节码、已优化或已编译的 dex 码、.so 库和字体）的内存。

**Other**：应用使用的系统不确定如何分类的内存。

**Allocated**：应用分配的 Java/Kotlin 对象数。 没有计入 C 或 C++ 中分配的对象。

注：目前，Memory Profiler 还会显示应用中的一些误报的原生内存使用量，而这些内存实际上是分析工具使用的。 对于大约 100000 个对象，最多会使报告的内存使用量增加 10MB。


#### 查看内存分配

Memory Profiler 可显示有关对象分配的以下信息：

分配哪些类型的对象以及它们使用多少空间。

每个分配的堆叠追踪，包括在哪个线程中。

对象在何时被取消分配（Android 8.0+）。

具体操作:点击上图中的 "② 捕获堆转储"或 "③ 记录内存分配情况"

<div align="center">
<img src="./pic about Memory Profiler/memory_record.png"  alt="Memory Profiler 界面" />
 </div>

已分配对象的列表将显示在时间线下方，按类名称进行分组，并按其堆计数排序。

注：在 Android 7.1 及更低版本上，最多可以记录 65535 个分配。 如果记录超出此限值，则记录中仅保存最新的 65535 个分配。 在 Android 8.0 及更高版本中，没有此限制。

要检查分配记录，请按以下步骤操作：

1. 浏览列表以查找堆计数异常大且可能存在泄漏的对象。 点击 Class Name 列标题可以按字母顺序排序。 然后点击一个类名称。 此时在右侧将出现 Instance View 窗格，显示该类的每个实例。

2. 在 Instance View 窗格中，点击一个实例。 此时下方将出现 Call Stack 标签，显示该实例被分配到何处以及在哪个线程中，如上图所示。

3. 在 Call Stack 标签中，双击任意行以在编辑器中跳转到该代码。

##### 默认情况下，左侧的分配列表按类名称排列。 在列表顶部，可以使用右侧的下拉列表在以下排列方式之间进行切换：

**Arrange by class**：基于类名称对所有分配进行分组。

**Arrange by package**：基于软件包名称对所有分配进行分组。

**Arrange by callstack**：将所有分配分组到其对应的调用堆栈。

<div align="center">
<img src="./pic about Memory Profiler/memory_dump.png"  alt="Memory Profiler 界面" />
 </div>

##### 捕获堆转储

堆转储显示在捕获堆转储时应用中哪些对象正在使用内存。 特别是在长时间的用户会话后，堆转储会显示您认为不应再位于内存中却仍在内存中的对象，从而帮助识别内存泄漏。 在捕获堆转储后，可以查看以下信息：

应用已分配哪些类型的对象，以及每个类型分配多少。

每个对象正在使用多少内存。

在代码中的何处仍在引用每个对象。

对象所分配到的调用堆栈。 （目前，如果在记录分配时捕获堆转储，则只有在 Android 7.1 及更低版本中，堆转储才能使用调用堆栈。）
要捕获堆转储，在 Memory Profiler 工具栏中点击 Dump Java heap 。 在转储堆期间，Java 内存量可能会暂时增加， 这很正常，因为堆转储与您的应用发生在同一进程中，并需要一些内存来收集数据。

堆转储显示在内存时间线下，显示堆中的所有类类型，如上图所示。


要检查堆，请按以下步骤操作：

1. 浏览列表以查找堆计数异常大且可能存在泄漏的对象。 点击 Class Name 列标题可以按字母顺序排序。 然后点击一个类名称。 此时在右侧将出现 Instance View 窗格，显示该类的每个实例。

2. 在 Instance View 窗格中，点击一个实例，此时下方将出现 References，显示该对象的每个引用。点击实例名称旁的箭头可以查看其所有字段，然后点击一个字段名称查看其所有引用。 如果要查看某个字段的实例详情，右键点击该字段并选择 Go to Instance，如下图所示。

3. 在 References 标签中，如果发现某个引用可能在泄漏内存，则右键点击它并选择 Go to Instance。

在类列表中，可以查看以下信息：

**Heap Count**：堆中的实例数。

**Shallow Size**：此堆中所有实例的总大小（以字节为单位）。

**Retained Size**：为此类的所有实例而保留的内存总大小（以字节为单位）。

在类列表顶部，可以使用左侧下拉列表在以下堆转储之间进行切换：

**Default heap**：系统未指定堆时。

**App heap**：应用在其中分配内存的主堆。

**Image heap**：系统启动映像，包含启动期间预加载的类。 此处的分配保证绝不会移动或消失。

**Zygote heap**：写时复制堆，其中的应用进程是从 Android 系统中派生的。

默认情况下，此列表按 Retained Size 列排序，可以点击任意列标题以更改列表的排序方式。

在 Instance View 中，每个实例都包含以下信息：

**Depth**：从任意 GC root 到所选实例的最短 hop 数。

**Shallow Size**：此实例的大小。

**Retained Size**：此实例支配的内存大小。

##### 此外,使用此工具也能查看许多详细内容:

比如说:

<div align="center">
<img src="./pic about Memory Profiler/check_finalizerReference.png"  alt="Memory Profiler 界面" />
 </div>

随机查看FinalizerReference中某一个对象,具体如上图所示:
可以清楚地看到这个Reference的object是什么,它所引用和所依赖的对象和其他的一些属性,并分析在当前情景下它的状态是否正常.

又或是:

<div align="center">
<img src="./pic about Memory Profiler/memory_bitmap_pre.png"  alt="Memory Profiler 界面" />
 </div>
查看内存中Graphics的占用(比如说bitmap类),然后随机查看其中的一个对象,通过preview甚至可以看到是哪一个图片资源.

**注意**:由于内存dump里的信息实在太过于繁杂,所以在使用的时候请务必进行信息筛选,不要花费时间去一个一个看,
也不要看见一个Object的内存使用过多就紧抓不放,内存问题就算是相对比较也有可能出现偏差,主观印象很容易把人带偏,
所以目前我是使用如下图所示工具进行筛选来查找问题的,如果有更好的意见欢迎提出.
<div align="center">
<img src="./pic about Memory Profiler/finder.png"  alt="Memory Profiler 界面" />
 </div>
这就是Memory Profiler的基本功能了.CPU与NetWork Profiler在此就不过多介绍了.
----------

具体参考于:https://blog.csdn.net/u013541140/article/details/81026089


#### 拓展阅读 :Shallow heap (浅堆） vs. Retained Heap (保留堆）
>浅堆是一个对象消耗的内存。根据情况，一个对象需要 32 位或 64 位（取决于其操作系统架构），对于整型为 4 字节，对于 Long 型为 8 字节等等。依据堆转储格式，其内存大小（比如，向 8 对齐）或许适应于更好地塑造虚拟机的真实消耗。

>X 的保留集合是当 X 被垃圾回收时，那些将要被移除的对象集合。 
X 的保留堆是在 X 的保留集合中所有对象的浅堆之和，也就是 X 存留的内存。
总体讲，一个对象的浅堆就是其在堆中的大小。同一个对象的保留大小就是当对象被垃圾回收时堆内存的总量。
一些对象的主要集合，比如某一特定类的所有对象、或是由某一特定类加载器加载的所有类的所有对象、或仅仅是一些任意的对象，它们的保留集是如果那些主要集的所有对象变得不可接近时所释放的对象集。
保留集包括这些对象和仅通过这些对象才能获取的其它对象。保留集的大小是包含在保留集中的所有对象的堆的大小。
