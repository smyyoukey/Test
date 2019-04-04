# Memory Profiler的使用实例

#### 以下内容以flash项目为例:  

>首先我在项目中集成了Matrix与LeakCanary,分析后台OOM信息并进行测试:

>如下图所示,这是Matrix抛出的memory leak与大量I/O操作的隐患;根据调用栈可以判断出这些操作是在哪里发生的.(由图中可以看出Matrix单从检查memory leak方面来说没有LeakCanary的信息详细)  

<div align="center">
<img src="./Memory Profiler的使用实例pic/Screenshot_2019-02-19-12-12-34-112_best.flashlight.png"  alt="Memory Profiler 界面" />
</div>

>接下来是LeakCanary查找出来的内存泄漏问题

<div align="center">
<img src="./Memory Profiler的使用实例pic/Screenshot_2019-03-15-10-40-06-124_best.flashlight.png"  alt="Memory Profiler 界面" />
</div>

<div align="center">
<img src="./Memory Profiler的使用实例pic/Screenshot_2019-03-15-10-40-24-372_best.flashlight.png"  alt="Memory Profiler 界面" />
</div>

<div align="center">
<img src="./Memory Profiler的使用实例pic/Screenshot_2019-03-15-10-40-59-491_best.flashlight.png"  alt="Memory Profiler 界面" />
</div>

>如以上几张图所示,谁持有谁已经一目了然了.接下来,我们再进一步用Profiler去查看实时的内存堆栈,就拿以上2个log作为例子:

首先打开一次所有的界面(比如说设置页面,皮肤选择页面等),最后回到主界面,这个时候点击"捕获堆转储"的按钮,显示此时时的内存分布,这时候可以看见:
<div align="center">
<img src="./Memory Profiler的使用实例pic/1cut.png"  alt="Memory Profiler 界面" />
 </div>   

>然后可以查找一下关于使用该资源的Activity,如下图所示,activity也在.这个时候手动触发GC:

<div align="center">
<img src="./Memory Profiler的使用实例pic/1cut1.png"  alt="Memory Profiler 界面" />
 </div>

> **发现这个实例依然在**.结果很明显,这属于内存泄漏.然后再看右下框内的References,第一条就是持有该实例的对象.从图中可以看出,该对象应该是Reward广告的一个实现类.这时候多次打开关闭SettingCsActivity试试看:

<div align="center">
<img src="./Memory Profiler的使用实例pic/1cut2.png"  alt="Memory Profiler 界面" />
 </div>

>果然实例数在不停地累计着,说明这个activity leak 了.这个时候再手动触发GC,发现SettingCsActivity的实例数又减少了,减少到了5个,从依赖里可以看出广告里应该是使用了弱引用来使用Context对象的,在内存紧张的时候会释放一下这些资源来缓解内存占用.**但是至少有一个SettingCsActivity的实例被广告对象持有后是一直存在的,它所占用的资源也不会得到释放**.目前单从这一方面说问题是不全面的.因为该资源一定会释放,只是有风险发生OOM,并且在一般情况下导致crash的概率基本为0.那么继续查看其他地方的内存问题.

>接下来看看cleaner功能中:点击Memory Boost,将所有清理执行几遍,然后退出到主界面,这时候看内存:

<div align="center">
<img src="./Memory Profiler的使用实例pic/2cut.png"  alt="Memory Profiler 界面" />
</div>

  >SettingCsActivity依然存在.再使用强制GC:

  <div align="center">
  <img src="./Memory Profiler的使用实例pic/2cut1.png"  alt="Memory Profiler 界面" />
  </div>

  >如上图所示,RubbishCleanActivity的context也依然被持有着,不过也是以弱引用的形式.

     所以我认为:目前这些对象的生命周期状态可分为以下几种:

     理想的健康生命周期:使用时加载资源,不使用时立即销毁资源释放内存;

     健康的生命周期:使用时加载资源,不使用时标记需要释放的资源,周期释放内存;(如频繁操作bitmap,或一直持有hardware.camera资源)

     
