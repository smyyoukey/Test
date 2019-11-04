# 临时备忘录

### (2019.11.4)

###### 一.谈谈Glide

作者：蓝师傅_Android
链接：https://juejin.im/post/5dbeda27e51d452a161e00c8
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

###### 二.阿里巴巴规约插件

昨日（10/14）日，阿里巴巴在杭州云栖大会上，正式发布了由阿里巴巴 P3C 项目组，经过 247 天的持续研发，正式发布众所期待的 《阿里巴巴 Java 开发规约》的扫描插件！
P3C 是世界知名的反潜机，专门对付水下潜水艇，寓意是扫描出所有潜在的代码隐患。这个项目组是阿里巴巴开发爱好者自发组织的虚拟项目组，把《阿里巴巴 Java 开发规约》强制条目转化自动插件，并实现部分的自动编码。
该插件已经在 Github 上开源，有兴趣的可以直接去看看。

     github.com/alibaba/p3c
      或者在Github直接搜索p3c

4.2 Inspections 支持
Inspections 相信大家应该都不陌生，它会自动在我们编码的阶段，进行快速灵活的静态代码分析，自动检测编译器和运行时错误，并提示开发人员再编译之前就进行有效的改正和改进。
这里举个简单的例子。
<div align="center">
<img src="./recent temp note/规约插件0.png"  alt="tools 实例" />
</div>

可以看到，它会个我指出我这里编写不规范的地方，如果想要查看更多细节，点击 more 按钮即可。

<div align="center">
<img src="./recent temp note/规约插件1.png"  alt="tools 实例" />
</div>

当然，所有的规范，都可以再 Inspections 中查看到。

<div align="center">
<img src="./recent temp note/规约插件2.png"  alt="tools 实例" />
</div>

在 Inspections 中，以 All-Check 区分，以下是它支持的所有检查，有兴趣可以一个个点击查看细节，右侧为检查出问题之后的提示信息，如果不想要的检测条件，还可以将它反选掉。


作者：承香墨影
链接：https://juejin.im/post/59e2e0bd6fb9a0450d101de9
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。


### (2019.11.1)
###### Android Tools属性
日常开发过程中，我们都会遇到这样一种场景：我们写出的 UI 效果在对接数据之前需要提前进行预览，进而调整  UI 细节和排版问题。我们一般的做法是什么样的？如果存在像 TextView 或者 ImageView 这种基础控件，你是不是还在通过诸如 android:text="xxx" 和 android:src="@drawable/xxx" 的方式来测试和预览UI效果？当然你肯定也会遇到这些“脏数据”给你带来的困扰：测试的时候某些地方出现了本不该出现的数据，事后可能一拍脑门才发现，原来是布局中控件预览数据没有清除导致的。
如果是 RecyclerView，在后台接口尚能测试的情况下，你是否又要自己生成“假数据”并手写 Adapter 呢？这时候你不禁会问：有没有一种方法，既能够做到布局时预览数据方便排版，又能够在对接真实数据运行后动态替换和移除这些无关数据呢？

只需要在 XML 布局文件的根元素中添加即可：
```xml
<RootTag xmlns:android="http://schemas.android.com/apk/res/android"
  xmlns:tools="http://schemas.android.com/tools" >
```

Design-time View Attributes
这就是我们先前效果图中的重要功臣了，即：布局设计时的控件属性。这类属性主要作用于 View 控件，如上文所说的 tools:context 就是“成员”之一，下面我们来介绍其他重要成员。
在此之前，我们需要先揭开 tools 命名空间的另一层神秘面纱：tools: 可以替换任何以 android: 为前缀的属性，并为其设置样例数据（sample data）。当然，正如我们前面所说，tools 属性只能在布局编辑期间有效，App真正运行后就毫无意义了，所以，我们就可以像下面这样来在运行前预览布局效果：

<div align="center">
<img src="./recent temp note/tools实例.png"  alt="tools 实例" />
</div>

上图对应的布局为：
```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout
        xmlns:app="http://schemas.android.com/apk/res-auto"
        xmlns:tools="http://schemas.android.com/tools"
        xmlns:android="http://schemas.android.com/apk/res/android"
        android:background="@android:color/white"
        android:clickable="true"
        android:focusable="true"
        android:foreground="?attr/selectableItemBackground"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginStart="2dp"
        android:layout_marginEnd="2dp"
        tools:targetApi="m"
        tools:ignore="UnusedAttribute">

    <ImageView
            android:id="@+id/card_item_avatar"
            android:layout_width="38dp"
            android:layout_height="38dp"
            app:layout_constraintStart_toStartOf="parent"
            android:layout_marginStart="16dp"
            android:layout_marginTop="16dp"
            app:layout_constraintTop_toTopOf="parent"
            app:layout_constraintBottom_toBottomOf="parent"
            android:layout_marginBottom="16dp"
            app:layout_constraintVertical_bias="0.0"
            tools:ignore="ContentDescription"
            tools:srcCompat="@drawable/user_other"/>
    <TextView
            android:id="@+id/card_item_username"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="16dp"
            app:layout_constraintTop_toTopOf="parent"
            app:layout_constraintStart_toStartOf="@+id/card_item_title"
            app:layout_constraintEnd_toEndOf="@+id/card_item_title"
            app:layout_constraintHorizontal_bias="0.0"
            android:textSize="12sp"
            android:textColor="@color/username_text_color"
            android:layout_marginEnd="16dp"
            android:paddingEnd="16dp"
            tools:text="水月沐风" />
    <TextView
            android:id="@+id/card_item_title"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textSize="16sp"
            android:textColor="@color/title_text_color"
            app:layout_constraintStart_toEndOf="@+id/card_item_avatar"
            android:layout_marginStart="12dp"
            app:layout_constraintTop_toBottomOf="@+id/card_item_username"
            android:layout_marginTop="8dp"
            android:maxLines="1"
            tools:text="今天上海的夜色真美！"/>
    <TextView
            android:id="@+id/card_item_content"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            app:layout_constraintStart_toStartOf="parent"
            android:layout_marginTop="24dp"
            app:layout_constraintTop_toBottomOf="@+id/card_item_avatar"
            app:layout_constraintBottom_toBottomOf="parent"
            android:layout_marginBottom="16dp"
            app:layout_constraintVertical_bias="1.0"
            android:maxLines="3"
            android:ellipsize="end"
            android:textColor="@color/content_text_color"
            android:textStyle="normal"
            app:layout_constraintEnd_toEndOf="@+id/card_item_bottom_border"
            android:layout_marginEnd="16dp"
            android:layout_marginStart="16dp"
            app:layout_constraintHorizontal_bias="0.0"
            tools:text="人生若只如初见，何事秋风悲画扇..."/>
    <ImageView
            android:id="@+id/card_item_poster"
            android:layout_width="0dp"
            android:layout_height="200dp"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@+id/card_item_content"
            app:layout_constraintEnd_toEndOf="parent"
            android:layout_marginEnd="16dp"
            android:scaleType="centerCrop"
            android:layout_marginTop="8dp"
            android:layout_marginStart="16dp"
            app:layout_constraintBottom_toBottomOf="parent"
            android:layout_marginBottom="16dp"
            app:layout_constraintVertical_bias="0.0"
            tools:ignore="ContentDescription"
            android:visibility="visible"
            tools:src="@drawable/shanghai_night"/>
    <View
            android:id="@+id/card_item_bottom_border"
            android:layout_width="0dp"
            android:layout_height="2dp"
            app:layout_constraintTop_toBottomOf="@+id/card_item_poster"
            android:background="#ffededfe"
            app:layout_constraintEnd_toEndOf="parent"
            android:layout_marginTop="16dp"
            app:layout_constraintStart_toStartOf="parent"/>
    <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:id="@+id/card_item_date"
            android:layout_marginTop="16dp"
            app:layout_constraintTop_toTopOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            android:layout_marginEnd="16dp"
            android:textColor="@color/date_text_color"
            android:textSize="12sp"
            tools:text="2019-08-10"/>
</android.support.constraint.ConstraintLayout>
```

通过上面代码我们可以发现：通过 对 TextView 使用 tools:text 属性代替 android:text 就可以实现文本具体效果的预览，然而这项设置并不会对我们 App 实际运行效果产生影响。同理，通过将 tools:src 作用于 ImageView 也可以达到预览图片的效果。此外。我们还可以对其他以 android: 为前缀的属性进行预览而不影响实际运行的效果，例如：上面布局代码中的底部分割线 <View>，我们想将其在 App 实际运行的时候隐藏掉，但我们还是需要知道它的预览效果和所占高度。

完整内容链接如下：

作者：水月沐风
链接：https://juejin.im/post/5d500b1a6fb9a06b1417d5c9
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

### (8.12-8.18)

1.有关fake icon功能：真正地去切换应用的项目icon挺麻烦的，fake icon功能的实现我个人认为是一种取巧行为，并能达到目的。主要实现有人写了我就直接贴[链接](https://github.com/panwj/TestDemo/wiki/Android%E5%8A%A8%E6%80%81%E6%9B%B4%E6%96%B0%E5%BA%94%E7%94%A8%E5%9B%BE%E6%A0%87
)了这里对一下这种情况做一点情景上的补充：“更改状态时我们传入的flag可以是PackageManager.DONT_KILL_APP或者是0，0表示会杀死包含该组件的app, 图标也会立即更改，但是该方法之后的方法都不会调用了，因为组件被kill。”

我在activity的onDestory（）回调中去执行changeIcon（），并在changeIcon（）的方法中去用sharePrefrence保存当前使用的是哪一个icon。但是在onDestory（）回调中去执行changeIcon（）虽然可以切换icon，sharePrefrence却不能执行保存。于是正常的切换icon并使用back键一栈一栈退出应用时，会导致一次“当前icon”的状态信息错误（因为没有保存）。


而用home键退出则可以正常保存，不过执行顺序有所不同：changeIcon（）（change（）————>save（））然后再主动执行activity的finish（）方法;

而点Back键是在onDestory（）中执行changeIcon（）（change（）————>save（）），不知道是否是这个原因导致结果有所区别，还需研究。


2


### (8.5-8.11)
1.接着上周：监听app安装卸载时有一种情况，那就是覆盖安装的时候会发remove和replace两种intent。这个时候有2种方式判断：1是判断接受了remove的intent后接收并判断Intent.EXTRA_REPLACING，2是Intent.ACTION_PACKAGE_FULLY_REMOVED这个广播是应用被卸载，数据被清除时才会发，所以，这是真正的被卸载的时候会发的。

上周发现对监听其他app卸载与更新的情况与预期不符（并没有分辨出卸载还是更新），所以看了一下这个问题，发现是由于对应用卸载残留的数据是个耗时操作，于是业务逻辑是通过receiver发送intent启动intentService扫描卸载残留数据的。于是receiver中的intent与intentService中的intent不是一个，导致receiver中的intent中的Intent.EXTRA_REPLACING并没有传入service，所以目前是在receiver中获取Intent.EXTRA_REPLACING的值，通过intent传到service。

2.之前发现appbarlayout与recycleview嵌套在某些小屏幕手机上会出现item显示不全的问题，这是一个挺棘手的问题，发现原因是view层级的显示中recycleview绘制区域大于父级view的有效操作区域。于是依靠设置recycleview的内部margin来解决这个问题。

### (7.29-8.4)
1.列表刷新列表中每个itemUI闪烁的问题，一开始以为是RecycleView导致的，尝试了很多种RecycleView相关的解决方法（关闭动画，设置固定id等）。结果不起作用，然后发现UI更新时只有使用glide加载的图片会闪烁，遂从glide方面入手解决。最后换了一种图片的加载方式。

2.判断scrollview中某子控件是否可见,普通得使用visible方法是有问题的。应该去获取该View当前绘制出的Rect区域来判断。

### (7.22-7.28)

1.之前试着将一个List完全写入到一个文件当中，使用的是ioStream方法，但是生成文件的时候总是报警告（报warning而不是error），并且生成文件失败。之后发现是路径问题，最后发现文件的.mDir（）与.mDirs()方法是有所不同的。一般要生成文件的上层文件夹还是要判断一下情况使用对应的方法。

2.String可以插入html语句进行动态赋值，直接使用比整理好string语句再赋值更为方便。用例如下：
string.xml文件中：
```xml
<string name="dlg_app_uninstalling_content">
     <Data><![CDATA[<font color="#ff4f3d"><b>%1$s</b></font>]]></Data> residual files left after
     <Data><![CDATA[<font color="#4d69ff"><b>%2$s</b></font>]]></Data>
    uninstalled.Suggest to clean up.</string>
```
代码中：
```java
mContentTv.setText(Html.fromHtml(String.format(getResources().getString(R.string.dlg_app_uninstalling_content), "42MB", "Messenger")));
```

### (7.1-7.7)
1.这次的版本改动在UI交互方面较多，由于解决之前的一些棘手的问题花了很多的时间导致在发test之前才合并代码调整UI所以导致在发布之前有很多UI更新与状态变换时的问题。这样的问题应该注意。

2.之前绘制波形图时产生ANR的问题在之后查明了是canvas绘制过多导致的。该custom view的更新时至多会有6个paint对象同时进行绘制更新，并且在放大和缩小的时候更加的消耗性能：首先在放大时总的波形图的帧数会发生变化，所以需要重新加载一次;然后再开始绘制背景，波形等一系列控件，当操作过多时产生ANR。目前是在onDraw（）方法中添加了限制条件，只有当特定情况下去绘制有必要重新绘制的view。

3.之前出现了一个关于recycleView的java.lang.IndexOutOfBoundsException: Inconsistency detected. Invalid item position.问题，这个问题很明显是notify（）的时候更新数据与使用的时候不一致。但很明显代码中notify的时机都没有问题，不然之前早就报出这个crash了。仔细查了一下调用栈，发现是分发更新消息的时候分发错了（recycleView的更新有很多种，changed，remove，add等），但android怎么可能有这样的不稳定问题，于是老老实实打断点调试，发现是recycleView的一个拓展工具diffUtill类的使用方面有问题，由于数据源不同当然不能走同一个diffUtill类的区分方法，遂改之。

4.反编译了一个竞品Message pro主要看了一下聊天室的实现，发现他们是引入了com.google.firebase.database这个包实现的，也就是说聊天功能的实现应该是借由google端的数据库搭载聊天消息的，但具体用的啥完成的即时通讯没有看出来。

### (6.24-6.30)
1.在复用展示列表的fragment的过程中，初始化item时报出了NullPointerException - Attempt to invoke virtual method RecyclerView$ViewHolder.shouldIgnore()' on a null object reference的错误。查找引起该问题的原因，发现主要是在复用itemView的时候改变了item的布局，但是代码中并没有做这种处理;所以得找其他原因。最后发现得关闭对recycleView动画的设置，终于发现了主要问题，切换tab页时在adapter中设置的stableId发生了改变，所以导致了这个问题。

2.在activity中的fragment中新建的二级fragment，有许多的状态识别回调方法与直接在activity中创建的fragment并不相同了。这个时候类似于isShow（），isVisible（），setVisibleHint（）等方法对于该fragment的判断都甚不准确。所以
这个时候应该基于这些方法自己处理一下判断当前显示的是哪个fragment的判断方法。

3.（上周）研究了一下ringtone maker中绘制音频波形图的代码，发现其是以每一帧的sampleRate为高，用canvas将线画出，然后连线成面将波形图画出。当波形图的尺寸超过屏幕宽度时（比如放大时），拖动时是实时更新波形图的UI。所以只改变初始化UI时的绘制方式是不行的。此时的目的是绘制出不连续的，有间隔的波形图，于是我降低绘制该图形用的sampleRate数组的保真程度，但出现的问题这样的话UI更新时给人的体验不好（刷新率低下），所以尝试用其他方式来完成目的。

之后换了一种绘制方式，不降低sampleRate的保真度，而是每隔几帧绘制一次来达到目的。那么之后需要解决的问题就是在这种情况下MakerView的绘制方式，与进度条的绘制方式了。


### (6.17-6.23)
1.搜索栏移动到toolbar中，直接获取toolbar中的menuItem并不能拿到搜索栏的实例对象，需要通过menuItemCompat对象获取。

2.研究了一下ringtone maker中绘制音频波形图的代码，发现其是以每一帧的sampleRate为高，用canvas将线画出，然后连线成面将波形图画出。当波形图的尺寸超过屏幕宽度时（比如放大时），拖动时是实时更新波形图的UI。所以
只改变初始化UI时的绘制方式是不行的。此时的目的是绘制出不连续的，有间隔的波形图，于是我降低绘制该图形用的sampleRate数组的保真程度，但出现的问题这样的话UI更新时给人的体验不好（刷新率低下），所以尝试用其他方式来完成目的。

3。已编辑音频的列表映射的本地数据库将和显示录音的列表分开，主要是因为2者的交互与增删改查的行为都有一定的差异。
### (6.10-6.16)

1.申请权限的过程中如果用户选择拒绝授予并不再提示，此时直接跳转setting界面，授予之后返回界面继续执行逻辑;该业务主要依赖activity的onActivityResult（）方法实现。目前这部分的逻辑还需要整理一下，尽量保持和应用内的权限回调的处理逻辑一致。

2.目前player和相关的UI更新逻辑需要拿到context与一些生命周期较长的引用，所以这部分暂时不做修改，先将不需要context之类的引用的方法对象移出。

3.有关之前getVisibility（）方法的问题，发现当app在后台，所有的View.getVisibility()都返回View.INVISIBLE， 即使getWindow().getDecorView().getVisibility()也是View.INVISIBLE。注意需要强调的是整个app在后台。
但是如果Activity由于其他方式导致不可见，例如被其他Activity覆盖，此时所有的View仍然是View.VISIBLE。

### (6.3-6.9)

1.之前发现打开google play页面的操作可能发生在应用外从而给intent加上了NEW_TASK的flag，但是又发现了一个问题，NEW_TASK这个flag在5.0之前的某些机型中可能会被识别为其他的非法flag导致依然发生crash。这个问题仍需要
分析并解决。

2.将每次挂电话就刷新界面的逻辑优化为每次拨打服务号之后挂断电话才会刷新，减少不必要的刷新次数。

3.整理需求“google帐号与后台录音账户唯一绑定”时可能发生的一些问题与使用方面的case，并与后台，产品，研发整个团队一起讨论，商量好可行的实现方式。

4.分析了一下目前产生的几个ANR的log，有几个发生的场景是列表界面刚刚初始化UI的时候。得在之后的开发过程中跟踪并分析一下究竟是因为此时View的初始化操作过多，还是刷新频繁导致的。

5.一般给view setVisibility（）之前也不需要判断该view是否可见，因为setVisibility方法中自带判断逻辑。

### (5.27-6.2)

1.在做修改录音名称的时候遇到dialog弹出查询后调用dismiss（），再次show（）dialog的时候popListView并没有展示在dialog中（此时应该是没有将dialog中的editText作为锚点）。主动为popListView添加锚点发现也不起作用，这时候发现应该是得判断是不是处于同一个dialog实例中，于是在初始化dialog的逻辑中加上了该判断。

2.承接之前减少fragment中代码量的问题，此次将其中所有初始化dialog的逻辑放在一个新建的类中，进行统一调用。唯一需要注意的是，该类中的context一定要和mainActivity中的一致，而不能将其作为一个单例，这样会导致context泄漏。比如说将context传入单例中，按back建退出应用再重新打开应用，这时候打印出该单例的内存地址和退出应用之前的内存地址一样，也就是说在这个行为下activity被销毁了但是这个单例没有销毁，从而导致单例中的context与当前acticity的不同，产生内存泄露问题。

### (5.13-5.19)

1.研究了很久"获取联系人电话功能",最终讨论后得出结论在没有权限的情况下无法实现.获取联系人电话功能的主要场景是:获取到用户拨打出的电话号码或是接听的电话号码,与后台的录音数据进行一一匹配.但是可以用来匹配的属性只有时间.android手机上监听不到通话接通的回调,所以和后台录音的start_time(也就是时间戳)的误差特别大。(比如说用户在拨打电话后1分钟对方才接听,那么应用内的时间戳就是拨打开始的时间,后台录音的start_time就是1分钟后对方接听的时间,然后还有几秒的误差。)所以录音名称目前依然显示时间.

2.RecycleView列表的局部更新在下载成功后调用总是不起作用,所以目前调用notifysetDataChanged()进行全更新,目前认为问题应该出在adapter中的条件判断不适用,之后得解决这个问题.

3录音的文件播放依然是沿用之前其他应用的播放逻辑,之前出现解码问题在看了后台接口的文档后发现是和文件名称有关,于是修改了下载完成后对文件名的处理.目前使用WAV格式音频.

4.数据的更新也是进行差量更新,目前同步的触发场景只有2处,一是用户主动刷新,二是通话结束刷新。列表的更新也已经拆分了请求后台数据和请求本地数据库。
