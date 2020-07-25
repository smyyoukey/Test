



### (2020.4.6-4.12)
1.在scrollview中的子布局设置居中无效，解决方式是在scrollview外层载套一个布局，并且将子布局设置为match_parent（前提是scrollview设置了fillViewport为true），才能成功设置子布局居中。

2.之前想用animation完成动画的轮播效果，即在动画结束的回调中去启动另一个动画，但是发现启动失败，于是改成了animatorSet实现。为什么失败还需要研究一下。

3.目前项目中的启动页（是一个activity）和订阅界面（是一个dialog）用的同一个布局，所以使用了一个静态函数完成布局控件的初始化。但有些静态实例需要在activity的ondestroy（）的时候将其释放，于是又返回了一个回调实例用于监听，但这样的回调显得很繁琐，之后得优化这一部分。

### (2020.3.30-4.5)
1.实时存储功能在7.0以上的手机上如果要获取每个应用占用手机存储空间的大小，需要获取用户使用痕迹组权限。这个权限不能直接申请，只有跳到系统的设置界面引导用户打开才行。目前实时存储条处于应用的主界面，不适合主动申请该权限，当前该应用的applock功能和大文件清理功能都会申请这个权限，所以目前只有当用户打开这2个功能并给予权限时，实时存储条才会显示分类。

2.pointer out of index这个exception目前是当可伸缩大小的view与viewpager嵌套导致onTouchEvent冲突才会出现。这个问题据查询资料显示无可避免，解决方法是给viewpager的onTouchEvent加try catch。

3.关于视频的格式方面测试发现，视频的层级结构取决于格式，类似于H.263，H.264 ，H.265等，而mp4，mkv等所谓格式不过相当于视频结构外层的壳。操作视频时，视频文件的层级结构决定了视频的操作方式。

### (2020.3.23-3.29)
1.实时存储需要展示不同存储资源的占用空间，比如说图片占有多少空间，视频占有多少空间，操作系统占用多少空间等。首先研究了一下如何获取操作系统占用空间数据，发现这些数据android并没有提供任何api可以去获取。之后又去查找是否有其他的方法获取这类数据，最后总结下来均不可行。因为比如说一个手机它的内部存储为16G，但可用空间仅有13G多，操作系统占用空间就在这个数据差中，但这个数据差中还有另一个变量，那就是存储硬件厂商存储容量的统一规格。每个厂商的规格不一，导致这个差值不可估算，从而也无法算出操作系统占用多少空间。

2.Activity要更新内部intent状态得在onNewintent（）里setIntent（）（目前都是从外部startActivity进入的情况，其他情况如何尚不确定）

3.目前好像只有acitvity的startActivity函数可以传递或者更新intent，如果acitvityA start了activityB，在new Task的intentFlag情况下，acitivityB finish时如何给activityA传递intent，还需要研究一下

### (2020.3.16-3.22)
1.上周发现了线上出现了一个新的错误，java.lang.IndexOutOfBoundsException: Invalid index 0, size is 0，发生在viewpager设置的indicator中，开始看这个错误认为是indicator设置的时候viewpager的adapter还没有初始化，于是先初始化viewpager再调用indicator，结果发现这个错误依然存在。之后上传mapping把可能走进的调用栈全查了一遍，发现的确是数据源没有初始化，但这个indicator中如果传入的viewpager与界面上的viewpager不是同一个的话，indicator里的数据源是不对应的（indicator和viewpager的数据源不对应）。由于indicator的主要作用是显示，所以目前的解决方式是屏蔽点击事件。

2.最初对加密视频播放方案的设想是能不能将本地文件转换成流媒体，然后模仿播放网络视频那样，解密一部分资源就相当于缓存一部分进度，最后播放完整个视频。但在查找过很多资料后好像没有人这么做过，目前打算研究一下这个方案的可行性

### (2020.3.9-3.15)
1.照片展示模糊，是由于之前进行压缩时设置的采样率，长，宽以及质量参数均较低，之后将其参数进行调整，又发现照片清晰了，但截图等图片依然模糊。调查后发现是获取相片详细信息的时候使用了Exif-ExifInterface获取，这个类只能获取jpeg等几个格式的图片信息，如果传入png格式的图片直接就返回空。所以没有照片信息的时候使用默认规格进行压缩，导致图片不清晰。目前改成了使用bitmapFactory获取图片信息。

2.Ad sdk更新后初始化方式改变，传入的广告id最好新建一个，初始化时传入广告id的广告会在初始化时请求，但不展示导致广告数据不正常。

3.关于surfaceView，测试发现并不是在所有的手机上都会进行“镂空”，有的手机会露出下方背景，有的不会，目前猜测是使用windowmanager提高了activity的安全级别产生的影响，具体原因待察。

### (2020.3.2-3.8)
1.在有的手机上照片展示界面会不断闪烁的问题：之前认为是在adapter里启用异步任务，在item不可见时仍然执行，导致item不断刷新。之后设置了tag并进行判断，同时在item被回收的时候取消异步任务操作，但发现该问题依然有。最终发现是在有的手机上，系统媒体数据库会经常刷新，导致cursorLoader观察到系统媒体数据库变化就调用onloadfinished（）回调，使照片界面不断刷新造成闪烁问题。

2.还有一个刷新时闪烁的问题，于是给adapter设置了固定ID，取消glide与recycleview的动画，遂解决。

3.最近工作中解决bug经常有一个问题，那就是对开发来说无法详细了解bug出现的详细情况表现与流程（并不了解是啥bug），导致寻找bug的过程需要花费很多问题且效率很低。

### (2020.2.24-3.1)

1.加密功能遇到的一些问题：1，如果报错：java.security.InvalidKeyException: Illegal key size
     请更换   JDK路径\jre\lib\security 文件底下的local_policy.jar和US_export_policy.jar
2、如果报错：java.lang.SecurityException: Cannot set up certs for trusted CAs
     请更换  jce.jar 包 或者使用更高版本的JDK
    3.如果报错：javax.crypto.BadPaddingException: pad block corrupted，可能是设置的key长度不对，或是被改变了
     4.如果报错：java.nio.channels.OverlappingFileLockException：可能是不同线程同时操作了同一个文件

2.之前在子线程完成io操作发现速度较慢，于是打算使用asynctask搭配线程池实现并行任务操作，结果发现会出现几个问题：1.创建的并行任务过多导致的exception；2。多个任务访问同一个文件的情况不好处理。最后还是换了一个方式，引入Rxjava实现并行操作，目前测试情况良好。

# 临时备忘录
### (2019.12.18)

#### 关于SurfaceView使用过程中遇到的问题:

#### 总结几个要点:
1.SurfaceView拥有独立的Surface（绘图表面）

2.SurfaceView是用Zorder排序的，他默认在宿主Window的后面，SurfaceView通过在Window上面“挖洞”（设置透明区域）进行显示.

#### SurfaceView与View的区别

1.View的绘图效率不高，主要用于动画变化较少的程序;

2.SurfaceView 绘图效率较高，用于界面更新频繁的程序;

3.SurfaceView拥有独立的Surface（绘图表面），即它不与其宿主窗口共享同一个Surface。

4.一般来说，每一个窗口在SurfaceFlinger服务中都对应有一个Layer，用来描述它的绘图表面。对于那些具有SurfaceView的窗口来说，每一个SurfaceView在SurfaceFlinger服务中还对应有一个独立的Layer或者LayerBuffer，用来单独描述它的绘图表面，以区别于它的宿主窗口的绘图表面。
因此SurfaceView的UI就可以在一个独立的线程中进行绘制，可以不会占用主线程资源。

5.SurfaceView使用双缓冲机制，播放视频时画面更流畅.

#### 使用场景

所以SurfaceView一方面可以实现复杂而高效的UI，另一方面又不会导致用户输入得不到及时响应。常用于画面内容更新频繁的场景，比如游戏、视频播放和相机预览。

#### 遇到的问题：
  1.当SurfaceView可见时背景变成透明;

  2.尝试给SurfaceView设置背景，遂添加以下代码，但发现在各个回调或是生命周期中holder.lockCanvas()方法获得的canvas都为空;

```java
if(mSnapSurfaceView != null)mSnapSurfaceView.getHolder().addCallback(new SurfaceHolder.Callback() {
       @Override
       public void surfaceCreated(SurfaceHolder holder) {
           if (mNeedPaint) {
               mNeedPaint = false;
               Canvas canvas = holder.lockCanvas();
               if (canvas != null) {
                   canvas.drawColor(Color.BLACK);
                   holder.unlockCanvasAndPost(canvas);
               }
           }
       }

       @Override
       public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) { }
       @Override
       public void surfaceDestroyed(SurfaceHolder holder) { }
   });
```   
#### 后续解决：
1.背景透明问题要点2就说明了，SurfaceView通过在Window上面“挖洞”（设置透明区域）进行显示.也就是说，如果按view的覆盖层级来说明，处于SurfaceView下层的都会被“挖开”，盖在SurfaceView上层的才会正常显示。

2.关于这个问题，有很多需要注意的地方：

1.将surfaceview的初始工作放在onCreate中，将surfaceHolder.lockCanvas()放到onCreate外，因为在onCreate中surfaceHolder画布未创建完，所以返回会为空。

2.在Android开发官网上有描述，只能有一个用户使用TextureView，在提供camera preview的同时是不可以进行Canvas的绘制的，lockCanvas()的值就是null。 详情可参考：https://blog.csdn.net/tinyzhao/article/details/52681600

我所碰到的是第二种情况，所以我最终的解决方式是在SurfaceView的上层盖一层ImageView。

#### 拓展：什么是双缓冲机制

在运用时可以理解为：SurfaceView在更新视图时用到了两张 Canvas，一张 frontCanvas 和一张 backCanvas ，每次实际显示的是 frontCanvas ，backCanvas 存储的是上一次更改前的视图。当你在播放这一帧的时候，它已经提前帮你加载好后面一帧了，所以播放起视频很流畅。

当使用lockCanvas（）获取画布时，得到的实际上是backCanvas 而不是正在显示的 frontCanvas ，之后你在获取到的 backCanvas 上绘制新视图，再 unlockCanvasAndPost（canvas）此视图，那么上传的这张 canvas 将替换原来的 frontCanvas 作为新的frontCanvas ，原来的 frontCanvas 将切换到后台作为 backCanvas 。例如，如果你已经先后两次绘制了视图A和B，那么你再调用 lockCanvas（）获取视图，获得的将是A而不是正在显示的B，之后你将重绘的 A 视图上传，那么 A 将取代 B 作为新的 frontCanvas 显示在SurfaceView 上，原来的B则转换为backCanvas。

相当与多个线程，交替解析和渲染每一帧视频数据。


### (2019.12.9-12.15)

1.界面中元素都用ScrollView包裹后，无法使ScrollView充满剩下的父布局。解决这个问题的办法就是，在ScrollView中添加一个Android:fillViewport="true"属性就可以了。顾名思义，这个属性允许 ScrollView中的组件去充满它。然后将包裹在其中的子布局修改为“match_parent”，即可让ScrollView充满整个父布局（当然ScrollView中子布局足够多时也可以撑满父布局，但一般ScrollView为动态数据填充，所以一般不这样）。

2.之前测试的时候发现在小米手机上不给start from background 权限时，如果该activity启动模式被设置为singleInstance，即使在应用内（应用处于前台）该activity也弹不出来。估计是由于处于的栈不同所导致的，但后来思考了一下同一个应用进程下应用是否处于前台难道小米系统是依靠该activty所处栈的状态来判断的？该问题之后有时间需要再研究一下。首先打算试试android 10上不给相关权限，该activity是否会成功弹出。


### (2019.12.11)
队列是一种特殊的线性表，它只允许在表的前端（front）进行删除操作，而在表的后端（rear）进行插入操作。进行插入操作的端称为队尾，进行删除操作的端称为队头。队列中没有元素时，称为空队列。

在队列这种数据结构中，最先插入的元素将是最先被删除的元素；反之最后插入的元素将是最后被删除的元素，因此队列又称为“先进先出”（FIFO—first in first out）的线性表。

在java5中新增加了java.util.Queue接口，用以支持队列的常见操作。该接口扩展了java.util.Collection接口。

Queue继承了Collection接口

Queue使用时要尽量避免Collection的add()和remove()方法，add()和remove()方法在失败的时候会抛出异常。

要使用offer()来加入元素，使用poll()来获取并移出元素。

它们的优点是通过返回值可以判断成功与否， 如果要使用前端而不移出该元素，使用element()或者peek()方法。

值得注意的是LinkedList类实现了Queue接口，因此我们可以把LinkedList当成Queue来用。
　　add        增加一个元索                     如果队列已满，则抛出一个IIIegaISlabEepeplian异常
　　remove   移除并返回队列头部的元素    如果队列为空，则抛出一个NoSuchElementException异常
　　element  返回队列头部的元素             如果队列为空，则抛出一个NoSuchElementException异常
　　offer       添加一个元素并返回true       如果队列已满，则返回false
　　poll         移除并返问队列头部的元素    如果队列为空，则返回null
　　peek       返回队列头部的元素             如果队列为空，则返回null
　　put         添加一个元素                      如果队列满，则阻塞
　　take        移除并返回队列头部的元素     如果队列为空，则阻塞
————————————————
版权声明：本文为CSDN博主「五山口老法师」的原创文章，遵循 CC 4.0 BY-SA 版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/Fly_as_tadpole/article/details/86536539

### (2019.11.21)

#### 01.ViewStub解析
ViewStub 是一个看不见的，没有大小，不占布局位置的 View，可以用来懒加载布局。  
当 ViewStub 变得可见或 inflate() 的时候，布局就会被加载（替换 ViewStub）。因此，ViewStub 一直存在于视图层次结构中直到调用了 setVisibility(int) 或 inflate()。  
在 ViewStub 加载完成后就会被移除，它所占用的空间就会被新的布局替换。

Inflate使用特点：

ViewStub只能被Inflate一次，inflate之后ViewStub对象就会被置为空。即某个被ViewStub指定的布局被Inflate后，就不能够再通过ViewStub来控制它了。  
ViewStub只能用来Inflate一个布局文件，而不是某个具体的View，当然也可以把View写在某个布局文件中。

ViewStub不支持merge ：

不能引入包含merge标签的布局到ViewStub中。否则会报错：  
android.view.InflateException: Binary XML file line #1:  can be used only with a valid ViewGroup root and attachToRoot=true

作者：杨充
链接：https://juejin.im/post/5dd6176c6fb9a05a9d6bf2ba
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
作者：杨充
链接：https://juejin.im/post/5dd6176c6fb9a05a9d6bf2ba
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。




### (2019.11.18)
#### Activity中的几种启动模式(复习)
activity的几种启动模式是android中常考的知识点，一般会考察有哪几种启动模式，以及每种启动模式在什么场景下使用：  
standard:这个是android默认的Activity启动模式，每启动一个Activity都会被实例化一个Activity，并且新创建的Activity在堆栈中会在栈顶。  
singleTop:如果当前要启动的Activity就是在栈顶的位置，那么此时就会复用该Activity，并且不会重走onCreate方法，会直接它的onNewIntent方法，如果不在栈顶，就跟standard一样的。如果当前activity已经在前台显示着，突然来了一条推送消息，此时不想让接收推送的消息的activity再次创建，那么此时正好可以用该启动模式，如果之前activity栈中是A-->B-->C如果点击了推动的消息还是A-->B--C，不过此时C是不会再次创建的，而是调用C的onNewIntent。而如果现在activity中栈是A-->C-->B，再次打开推送的消息，此时跟正常的启动C就没啥区别了，当前栈中就是A-->C-->B-->C了。  
singleTask:该种情况下就比singleTop厉害了，不管在不在栈顶，在Activity的堆栈中永远保持一个。这种启动模式相对于singleTop而言是更加直接，比如之前activity栈中有A-->B-->C---D，再次打开了B的时候，在B上面的activity都会从activity栈中被移除。下面的acitivity还是不用管，所以此时栈中是A-->B，一般项目中主页面用到该启动模式。
singleInstance:该种情况就用得比较少了，主要是指在该activity永远只在一个单独的栈中。一旦该模式的activity的实例已经存在于某个栈中，任何应用在激活该activity时都会重用该栈中的实例，解决了多个task共享一个activity。其余的基本和上面的singleTask保持一致。  
上面的各种启动模式主要是通过配置清单文件，常见还有在代码中设置flag也能实现上面的功能：   
FLAG_ACTIVITY_CLEAR_TOP：这种启动的话，只能单纯地清空栈上面的acivity，而自己会重新被创建一次，如果当前栈中有A-->B-->C这几种情况，重新打开B之后，此时栈会变成了A-->B，但是此时B会被重新创建，不会走B的onNewIntent方法。这就是单独使用FLAG_ACTIVITY_CLEAR_TOP的用处，能清空栈上面的activity，但是自己会重新创建。    
如果在上面的基础上再加上FLAG_ACTIVITY_SINGLE_TOP此时就不重新创建B了，也就直接走B的onNewIntent。它两者结合着使用就相当于上面的singleTask模式。  
如果只是单独的使用FLAG_ACTIVITY_SINGLE_TOP跟上面的singleTop就没啥区别了。  
FLAG_ACTIVITY_CLEAR_TOP+FLAG_ACTIVITY_SINGLE_TOP=singleTask,此时要打开的activity不会被重建，只是走onNewIntent方法。  
FLAG_ACTIVITY_SINGLE_TOP=singleTop
FLAG_ACTIVITY_NEW_TASK


在相同taskAffinity情况下：启动activity是没有任何作用的。


在不同taskAffinity情况下：
如果启动不同栈中的activity已经存在了某一个栈中的activity，那么此时是启动不了该activity的，因为栈中已经存在了该activity；如果栈中不存在该要启动的activity，那么会启动该acvitity，并且将该activity放入该栈中。


FLAG_ACTIVITY_NEW_TASK和FLAG_ACTIVITY_CLEAR_TOP一起使用，并且要启动的activity的taskAffinity和当前activity的taskAffinity不一样才会和singleTask一样的效果，因为要启动的activity和原先的activity不在同一个taskAffinity中，所以能启动该activity，这个地方有点绕，写个简单的公式:

FLAG_ACTIVITY_NEW_TASK如果启动同一个不同taskAffinity的activity才会有效果。
FLAG_ACTIVITY_NEW_TASK和FLAG_ACTIVITY_CLEAR_TOP如果一起使用要开启的activity和现在的activity处于同一个taskAffinity，那么效果还是跟没加FLAG_ACTIVITY_NEW_TASK是一样的效果。
FLAG_ACTIVITY_NEW_TASK和FLAG_ACTIVITY_CLEAR_TOP启动和现在的activity不是同一个taskAffinity才会和singleTask一样的效果。

FLAG_ACTIVITY_CLEAR_TASK

在相同taskAffinity情况下：和FLAG_ACTIVITY_NEW_TASK一起使用，启动activity是没有任何作用的。
在不同taskAffinity情况下：和FLAG_ACTIVITY_NEW_TASK一起使用，如果要启动的activity不存在栈中，那么启动该acitivity，并且将该activity放入该栈中，如果该activity已经存在于该栈中，那么会把当前栈中的activity先移除掉，然后再将该activity放入新的栈中。

FLAG_ACTIVITY_NEW_TASK+FLAG_ACTIVITY_SINGLE_TOP用在当app正在运行点击push消息进到某个activity中的时候，如果当前处于该activity，此时会触发activity的onNewIntent。
FLAG_ACTIVITY_NEW_TASK+FLAG_ACTIVITY_CLEAR_TOP用在app没在运行中，启动主页的activity，然后在相应的activity中做相应的activity跳转。

作者：xiangcman
链接：https://juejin.im/post/5d9fe662f265da5b783f0651
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。


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
