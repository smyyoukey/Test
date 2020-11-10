
### (2020.10.27 - 11.2)
关于悬浮窗权限的申请，它的检查权限流程并不处于一般权限的请求流程之中，需要在跳转设置界面之后额外创建handler进行监听，并检查是否获取权限。并且通话结果页权限申请需要在结果页展示之后弹出，那么相当于要对很多界面都进行权限处理。目前是将悬浮窗权限逻辑流程写在baseactivity中，之后打算抽出一个专门处理权限的父类activity。

### (2020.10.20 - 10.26)
1.功能引导界面使用AlertDialog实现，需要覆盖整个屏幕，但是当展示出来的时候总是会留出下方空白，无法覆盖。之后尝试了直接给window设置背景，目标区域盖住了，但之前留出下方空白的区域无法接收点击事件（说明布局还是没有覆盖整个屏幕，只有底层window的背景改变了）。最后是用一个子View充当背景，主布局才覆盖全屏幕。

创建项目模板时Fragment切换用的是官方默认的navigation框架，但是发现每次切换fragment都不保存fragment状态，看navigation中fragment切换的逻辑是调用replace方法，在fragment里打点也发现它并不是只重新初始化fragment的view而是重新初始化一个fragment对象。查资料后发现官方说明的状态存储需要自行保存在viewmodel里，navigation里并不缓存fragment，于是打算还是先使用viewPager实现主界面框架。


### (2020.9.21 - 10.12)
1.superCleaner需要将文件管理的界面集成到主界面的tab中，目前是用一个fragment作为载体嵌套2个功能界面fragment，之后发现权限回调和一些操作不生效，发现是onActivityResult回调的问题，解决了之后目前没有发现其他问题，但界面的数据刷新时机依然需要斟酌一下，每次切换到该页面时都刷新数据对于内容量较大的界面来说消耗性能较多，低配置机型可能会卡顿。

2.许多类似于文件管理界面这样的功能界面逻辑需要提供给应用主界面使用，那么其中许多类型强转的地方都需要注意，不然就会由于强转类型不对crash。目前对于一定要这样使用的地方替换成了抽象接口实现，进行解耦合。

### (2020.8.31-9.6)
遇到的问题及思路：
1.小米机型要从后台弹出应用界面需要start in background权限，android 10需要overlay权限，理论上来说基于android10的小米系统这两权限都需要申请。但用android10的小米机型发现，这两个权限在设置界面的权限描述变成了一个权限，而android10以下的小米机型start in background权限的权限描述就只是为了start in background。目前不确定这只是小米系统的一个过渡方式还是之后固定如此，所以权限引导流程和之前保持一致。

2.目前界面外弹出清理界面是用一个透明activity再去启动清理界面，原因是为了让清理activity兼容各个启动场景，但好像会在某些机型上导致清理界面显示较慢，之后可以看看是否可以优化。

### (2020.8.24-8.31)
1.关于获取系统传感器的响应很简单，调用系统API getSystemService（）获取传感器的manager即可。由于可开启传感器监听的场景较多，一开始的设想是在各个场景中执行注销监听操作再设置新的listener，结果发现之前注册的listener注销不了，调查之后发现传感器服务中并不是只维持一个listener实例，而是使用一个map，以listener为key，封装的回调处理类为value，保存所有注册的listener实例。为了使应用内保持一个listener实例，所以在application类内初始化并随时使用。

2.关于preferenceScreen添加自定义布局，直接添加一个perference项是不行的，因为该类inflater的布局是固定的，在layout属性中设置的布局无效。最后只能重写该类，并重写oncreateview（）方法，来实现布局自定义。

3.由于小米手机对于后台唤起应用有一个专门的权限，所以在摇晃清理功能开启前在小米手机中都得请求这个权限，但由于开启该功能的场景较多，每个场景都处理不光是交互体验不好，而且UI flow复杂。所以关于权限交互需要再讨论一下。



### (2020.8.17-8.23)
1.上两周相似照片功能技术总结：https://github.com/smyyoukey/Test/blob/master/note/%E7%9B%B8%E4%BC%BC%E7%85%A7%E7%89%87%E7%AE%97%E6%B3%95%E6%80%BB%E7%BB%93.md

2.相似照片功能中由于数据加载时间较长，所以基于交互考虑，数据的移除，界面的更新基本上是直接操作list数据，而不是重新走一遍数据加载。

### (2020.7.27-8.2)
1.隐藏应用入口功能在android 10 手机上失效，发现是：
Android 10 限制了在启动器中隐藏应用图标的功能。除非满足以下任一条件，否则应用必须具有图标：
它是系统应用，即使是更新后的应用。
它是托管式配置文件管理应用（工作资料所有者）。
它未请求任何权限。
它不包含任何组件（例如，Activity、内容提供程序、广播接收器和服务）。
具有图标且拥有已启用的可启动 Activity 的应用不受影响。除了上面列出的例外情况，所有应用均会显示一个图标。如果应用没有图标，则会显示默认的系统图标。点按没有可启动 Activity 的应用图标时会打开应用信息屏幕。

2.部分机型即使使用了私密消息封锁短信功能后，短信通知依然会弹出，但通知栏上明显是cancel了并在应用内显示了。目前初步判断该通知机制与普通的系统通知不同。

### (2020.7.20-7.26)
1.提示新文件时需要与上一次的本地数据库数据做对比，于是打算将数据记录为json文件存储在本地，使用了getCacheDir（）与createTempFile（）这两个函数来创建文件（使用这两个函数可以将文件写入Android/data/包名/cache 目录下，而且不需要文件读写权限），尝试创建文件成功，虽然在应用的cache文件夹下找不到该文件，但读取文件的数据是成功的，但后来简化逻辑后发现只要对比文件的数据库id即可，所以还是使用MMKV存储。至于这种创建文件的方式还是可以测试并完善一下，虽然Android/data/在应用删除/清空数据时会被删除，但在该目录下存储文件貌似是不需要存储权限的，在很多情况下都可以使用。（之前使用的少个人认为是当时的手机内部存储空间较少，文件都存在sd卡下比较好，但现在都属于一个存储介质之中。）

2.之前看竞品搜索功能的逻辑，有一种这样的case，如果2次的搜索输入字符串相同，直接复用第一次的搜索结果。这种优化case的效果不错，之后有时间也可以研究一下该逻辑的实现并应用到搜索或数据量大但需要多次更新的业务场景中。

3.之前有的通知无法收入的原因是判断了通知的title或content为空就停止执行方法，测试发现的通知就是没有title的。之后将该限制删除，并且两者都为空时文本显示为“未知”。

### (2020.7.13-7.19)
1.之前遇到的搜索逻辑UI更新卡顿的问题主要有2个方面，一是UI更新频繁，二是使用的数据结构的问题。这里不细说UI更新的优化（主要是在数据处理完后找时机统一更新），主要研究了一下为什么hashset性能会优于arraylist。首先主要是这两个集合类的contains（）函数实现，Hash  类集合，在存储时 会存以hash 值为数组下标的数组，这样contains 时 只用判断  数组对应位置是否有值，没有直接返回，有的话再用equals 判断内容是否相等。所以 contains 的效率是o(1) 的（需要注意的是HashMap 对value 的查找是o(n) 的）。所以要大量使用包含有contains（）的操作时（比如说一万多条数据部分全选，还有数据集合的remove（）操作等），最好使用Hash类集合。其次是对象的equals函数，重写之后equals中最好使用唯一键（最好是基础类型）对比，也能提高效率。

 2.之前使用contentObserver来实现监听手机文件更新，发现部分机型对于MideaStore.Files的监听状态与众不同。比如说小米机型无法监听到文件的添加和更新，只能监听到文件的删除;moto机型可以监听到文件的添加，更新，删除但不会返回有效Uri，导致无法分辨是何文件发生了什么操作。所以在这些机型上无法弹出提醒文件新增的提醒。

3.发送intent打开第三方应用预览文件时会出现无效path的问题，找不到根路径，目前猜测是文件处于其他外部存储介质，但被系统数据库扫入，之后外部存储移除但数据库未更新导致。目前该问题需要继续跟踪。

### (2020.7.6-7.12)
1.一开始计划文件搜索功能的实现时认为和之前音频搜索或联系人搜索一样,直接检索相关字段返回搜索结果即可。当开发时才发现MideaStore中name字段并不是所有文件都有，只有audio、video有该字段，而display_name字段并不是所有的文件都有值，许多文件的该字段为空。而title字段是每个文件都有值的，但是为不包含后缀的名字，并且有的只是路径而已。那么只有搜索data字段中存储的绝对路径了，但这样又会扫出多余的文件（比如说路径中包含搜索内容）。这样就需要在数据库查询语句中写入字符串切割的逻辑才能精准查找了。于是反编译小米的文件管理应用，发现它的确是这么做的。切割绝对路径中的文件名出来，并通过title字段验证，于是直接将搜索逻辑拿来用上。

2.随之而来的第二个问题是查询的数据，操作的数据量都非常之大。首先是每在搜索栏输入一个字符都要重新查询一次数据并展示，这就会导致一个问题，上一次查询流程还没走完又开始了下次，下下次。其次是刷新数据以及进行文件操作时由于数据过多直接未响应。比如说有一万多个文件进行全选，必须将所有item都设为选择状态，这个过程就足以导致ANR。并且选择打开特定文件夹时，需要逐步遍历其父文件夹，这些操作都是非常耗时且需要为同步操作。

3.数据库中查询的数据无法区分是文件还是文件夹。查找了许多方式，既没有专门记录的方式也没有相关api，只能用File类提供的isDireactory（）函数进行判断。几万条数据一一调用该函数还是挺消耗资源的。目前是在查询数据后判断是否为文件夹，并记录在数据结构中。

### (2020.6.29-7.5)
1.显示最近操作过的文件记录，由于文件种类较多，最好是直接使用mediastore.files直接从系统数据库获取所有文件的记录，而不是使用mediastore.image/audio/video一个个查再merge cursor。但是当直接读取mediastore.files获取手机上的所有文件时，会将含有.nomedia文件下的媒体文件也读出来，但使用mediastore.image/audio/video不会。经过一系列测试研究，发现有一项表内属性为“media_type”，比如说处于.nomedia文件所在路径中的图片，media_type为type_none，而相册图片为type_image，借此区别是否为nomedia文件。

2.关于下拉刷新的实现，之前打算直接根据recyclerview添加头部view实现，但这样实现基本上所有的view都得加进recyclerview，所以最后是使用第三方控件在主布局外套一层，下拉刷新中的header view放在主布局外层，下拉时展示，这样既不会导致布局错乱，也不会因为recyclerview添加多个布局导致item position计算特别复杂。

3. 由于最近操作文件列表的实现是由一个recycleview展示不同的item布局，导致不同item之间的间距样式很难分别设置，不同布局设置不同的item间距仍需研究实现方式。

### (2020.6.22-6.28)
recycleview中的setStableId（）为true后，一定要重写adapter的getitemid（）函数，并且返回的id一定要唯一。之前使用过item的属性来当作唯一id，比如说数据库id，修改日期，文件名的一部分，但总的来说这些属性都有可能不是唯一的。所以当不需要通过id来更新adapter时，直接将realPosition设置给stableid就行了，使用时机基本相同且肯定不重复。

### (2020.6.15-6.21)
1.关于之前的applocker与私密模块应用内上锁逻辑，需要对于5.0以下的系统特殊处理，因为当使用startActivityforResult（）启动acitivity时，5.0以下的系统会在acitivity第一次启动的时候就回调一次onActivityResult，而5.0及以上不会，这刚好影响到了应用内上锁逻辑。尝试了使用标志位，或判断onActivityResult返回的resultcode值来纠正上锁行为，都无法顾及所有场景。最后解决方式是自定义resultCode值，并在需要处理的界面中setResult（），以此返回自定义resultCode值，来处理对应的上锁行为。

2.文件加密由于不同的文件需要以不同的加密方式应对（纯文本文件需要全文件异或混淆，而pdf或是zip文件只需要混淆文件头文件结构就已经被损坏），目前的区分方式是以文件后缀拓展名进行格式区分，比如说".pdf"就被识别为pdf文件;该属性被记录在文件名中，所以在业务流程中的加密解密皆可以依赖该属性来进行区分。特殊情况：比如说用户自己将txt文件的后缀名改为pdf，导致加密不完善的情况不予考虑。之后测得有几种相同后缀名不同mimetype的格式文件，比如说pdf，有的pdf文件混淆文件头就可以使其无法打开，有的PDF却必须全文件混淆处理。所以关于文件加密方面还需要多加测试。

### (2020.6.8-6.14)
1.音频文件的加密方式使用的是全文件异或混淆，单单像视频文件那样混淆文件头并不能使音频文件不能播放，依然可以正常播放未混淆的部分（测试文件的格式为mp3）。

2.文件加密由于不同的文件需要以不同的加密方式应对（纯文本文件需要全文件异或混淆，而pdf或是zip文件只需要混淆文件头文件结构就已经被损坏），目前的区分方式是以文件后缀拓展名进行格式区分，比如说".pdf"就被识别为pdf文件;该属性被记录在文件名中，所以在业务流程中的加密解密皆可以依赖该属性来进行区分。特殊情况：比如说用户自己将txt文件的后缀名改为pdf，导致加密不完善的情况不予考虑。

3.目前文件的加密方式有多种，但业务流程上的逻辑不同，导致相同的加密方法却无法复用。之后得再次考虑将加密方法从业务逻辑中抽离的方式。

### (2020.6.1-6.7)
1.关于文件的mimetype属性不确定的问题，目前使用mimeType判断文件分类的方式仍是无可取代的。有2个方面：1是文件的加密方式不同：图片使用AES加密;视频或其他对文件完整性要求较高的文件可以使用混淆文件头的方式加密;而音频，纯文本等文件必须混淆整个文件来达到加密的目的。2是android系统中文件的打开方式，扫描方式都是依赖于mimeType属性。如果使用文件的后缀名来判断文件类型，1是当后缀名不匹配（比如说mimeType是音频文件，后缀变成了视频文件的后缀）的时候，可能文件的加密就无效了，在手机文件管理器中可以正常播放;并且在应用内界面发送intent使用第三方应用打开文件时，不会显示正确的打开方式，在文件管理器中却可以正常播放。所以目前使用mimetype判断文件类型更为准确。

2.多线程并发写入文件还是会抛出异常的。之前没抛出异常是因为文件分块与写入操作并不是并发而是在同一个子线程，只是在并发加密文件而已。当一个线程在写入该文件时，其他线程应该阻塞，然后写完释放再轮到下一个，整个过程依赖于文件锁机制。

3.AES加密对于有些格式的文件，在加密后会无法复原，目前不知道原因所在，但看竞品大多数格式文件加密都是使用AES加密。这个问题有时间的时候再研究一下。

### (2020.5.25-5.31)
1.测试正式版本时遇到的restore复原失败的bug，原因是当应用清除数据（被卸载或清空数据）后，reload本地文件会给它设置默认的还原路径：系统DCIM下的相册。这是一个系统指定的照片路径，基本上所有的android手机都一样。但是在三星的几个比较新的手机上发现：如果没有拍照就不会产生相册文件夹，导致还原路径为无效url。所以之后在还原操作中都添加了保护逻辑：判断原路径是否存在，不存在就新建原路径文件夹。

2.关于视频的mimetype属性杂乱无章，在测​试中发现读不出来的视频，是由于mimitype与相同格式文件的mimitype不同导致无法指定获取。比如说avi格式视频，有的avi格式视频mimetype可能是video/avi，有的可能是video/msvideo，video/x-msvideo,甚至一些没见过在网络上也查不到的mimitype。所以mimitype这个属性应该只是大致统一的。之后将加密视频获取系统媒体库中的视频的mimeType限制取消，直接获取所有视频。

3.由于私密模块界面中添加了上锁逻辑，导致fragment中的onActivityResult失效了（因为直接从acitivity调用startActivityForResult（））。所以目前的解决方法是在acitvity收到了onActivityResult后再更新fragment。由于打开权限界面依然是用fragment起的，所以权限界面的result fragment中是可以收到的。


### (2020.5.11-5.17)
1.之前加密大文件的时候会直接OOM（因为是相当于直接读写再复写文件，直接占用内存资源），于是打算将文件分割再写入，然后合并。结果做完发现拷贝时间过长，又尝试了其他几种拷贝方式，结果如下：
目标文件大小652MB，      耗时时间单位：毫秒
1.RandomAccessFile 文件分割（block 32MB）: 54840
2.RandomAccessFile 文件分割（block 128MB）: 73465
3.BufferedOutputStream + FileInputStream:     30830
4.BufferedOutputStream + BufferedInputStream: 26023
5.FileChannel：31002
由上结果可知，采用内存缓冲区读写拷贝文件的方式耗时较短，但仍需要26秒，想要达到瞬间完成的程度，那么就不可能是文件读写操作，肯定是直接修改文件路径。所以加密流程为更改文件路径后再加密文件（由于是更改文件路径，所以也不用考虑剩余存储空间问题了）。

2.使用popupwindow完成了一个列表中item的菜单界面，并根据屏幕位置计算弹出位置（如果点击的item靠下展示区域不够就向上展示）。但在完成的过程中发现写定宽高的布局在获取布局宽高的时候数值居然有变化，目前的解决方式是写定子布局的宽高属性从而固定布局宽高，但layout为什么会发生裁剪效果还不得而知。

3.时间戳转换时要看它是否为13位，java的时间戳为13位，媒体数据库中的时间戳可能只有10位。这个时候要将其换算成13位才是正确的时间。（乘以1000就可以，应该只是时间单位不同）

### (2020.5.6-5.10)
1.为了验证魔数的可靠性测试了一下手机录制视频的头是否能有对应的魔数：该视频文件的格式为mp4，可与魔数中对应的MP4不同，但相差只有一个byte的数据。于是决定还是主要依靠在文件名中存储的文件格式来确定文件的后缀名，魔数只作为一个备选方案。

2.关于存储的问题还需要考虑完善。比如在手机存储将满的时候，目前的处理是当加密的视频总大小大于剩余空间时提示空间不足，但还有恢复文件的时机，播放加解密操作文件的时机应该怎样去预防超过存储，还需要验证一下。

3.还一个问题是如果多处同时操作一个文件该怎么办，目前是决定尽量避免这个情况，但是判断文件是否被占用的方法是否可靠，多处同时操作一个文件的可能场景有哪些还需要测试。

### (2020.4.27-5.1)
1.目前的加密方式主要是视频文件转化为2进制文件流的前128位取反，如果视频需要播放的话，就得先解密再发送intent跳转到系统播放器播放。所以有2个问题：1.能播放什么格式的视频不是应用决定的而是用户选择的播放器决定的，我们主要测试哪些格式可以成功加密/解密；2.当用户播放视频时如果直接中途退出应用，如何再次加密视频？目前定了三个时机：1.播放完视频返回应用；2.用户锁屏时去检查视频是否加密，再把没加密的加密上；3.定时检查视频是否加密。

2.如果用户清除了应用数据如何将加密文件复原呢？可以恢复到默认文件夹，但加密状态和后缀拓展名的获取需要检验。目前是将这2个信息写在文件名中，在加密/解密前再2次检测文件的加密状态（获取文件流的前数位对比判断）。后缀名也是如此。

3.目前有几个功能还需要测试，比如加密方式是否对所有格式都有效果，文件魔数是否能获取到正确的后缀拓展名等。

### (2020.4.20-4.26)
1.新增的推荐位由firebase控制，但是之前的逻辑只是单次获取firebase配置，没有加上获取配置后成功的异步回调。于是将其补上，并完善了付费广告去广告前后的UI更新逻辑。

2.统一所有界面的UI风格的需求中需要在每个头部toolbar下方加上阴影，一般可以在xml布局文件中添加属性实现，也可以直接在代码里动态添加阴影。但是要注意，代码中设置阴影后该设置阴影的布局会被置为最上层，如果toolbar view上的返回或其他按钮是处于该布局上层的，该布局设置阴影后就会置为最上层盖住其他按钮。目前解决方式是将toolbar布局与按钮外再包一层布局并设置阴影。

3.关于私密视频功能，如何获取视频文件是否是加密状态与文件加密后如何获取视频文件格式是目前需要研究的问题。

### (2020.4.13-4.19)
1.关于自适应图标，如果资源中加入了自适应图标的xml文件的话，应用内就不能再用ic_launcher这个图标资源了。因为应用内引用后，会和应用外一样显示为rom的launcher的默认style，style是圆的图标就显示为圆的，是方的就显示为方的，会和之前的UI有出入。所以应用内使用icon需要引用单独的图片资源，而不是使用minmap中的ic_launcher，遂替换所有应用内使用ic_launcher的位置的资源。

2.关于修改本地化翻译的需求，比如说这次需要修改应用名，但该应用名已经翻译成了各国语言版本，那么如果没有给翻译后的新应用名，那么其他语言中的应用名对应本地化翻译应该逐一查找并删除。如果下次要本地化的时候，再一一查找位置去修改。

3.关于上周订阅界面布局初始化的问题，这周时间较少没有将其重写成自定义view的形式，之后如果这方面逻辑不变的话就将其重写。目标是尽量以几行代码完成订阅界面的初始化与回调的注册等逻辑。

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
