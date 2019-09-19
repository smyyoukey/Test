### DownloadManager的使用
#### 1.使用
申请权限
文件下载和保存需要网络跟写存储卡权限
```xml
	<uses-permission android:name="android.permission.INTERNET" />
	<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
```
8.0针对未知来源应用，在应用权限设置的“特殊访问权限”中，加入了“安装其他应用”的设置，这主要是为了防止应用内引导用户安装其他无关应用，特别是针对一些流氓应用会比较有效。添加下面权限后就可以正常跳转安装界面
```xml
    <uses-permission android:name="android.permission.REQUEST_INSTALL_PACKAGES"/>
```
在Manifast中注册provider：
```xml
    <provider
               android:name="androidx.core.content.FileProvider"
               android:authorities="com.flash.fileprovider"
               android:exported="false"
               android:grantUriPermissions="true">
               <meta-data
                   android:name="android.support.FILE_PROVIDER_PATHS"
                   android:resource="@xml/filepath" />
           </provider>
```

```java
public class DownLoadUtil {
    private static final int MSG_SHOW = 1;
    private static final int MSG_COMPLETED = 2;

    //下载器
    private DownloadManager mDownloadManager;
    private Context mContext;
    //下载的ID
    private long mDownloadId;
    private String name;
    private String pathstr;

    private View updateView;
    private UiHandler mHandler; // 声明一个处理器对象

    public DownLoadUtil(Context context, String url, String name) {
        this.mContext = context;
        downloadAPK(url, name);
        this.name = name;
        mHandler = new UiHandler(context);
    }

    //下载apk
    private void downloadAPK(String url, String name) {
        //创建下载任务
        DownloadManager.Request request = new DownloadManager.Request(Uri.parse(url));
        //移动网络情况下是否允许漫游
        request.setAllowedOverRoaming(false);
        //在通知栏中显示，默认就是显示的
        request.setNotificationVisibility(DownloadManager.Request.VISIBILITY_VISIBLE);
        request.setTitle("通知标题，随意修改");
        request.setDescription("新版***下载中...");
        request.setVisibleInDownloadsUi(true);

        //设置下载的路径
        File file = new File(mContext.getExternalFilesDir(Environment.DIRECTORY_DOWNLOADS), name);
        request.setDestinationUri(Uri.fromFile(file));
        pathstr = file.getAbsolutePath();
        //获取DownloadManager
        if (mDownloadManager == null)
            mDownloadManager = (DownloadManager) mContext.getSystemService(Context.DOWNLOAD_SERVICE);
        //将下载请求加入下载队列，加入下载队列后会给该任务返回一个long型的id，通过该id可以取消任务，重启任务、获取下载的文件等等
        if (mDownloadManager != null) {
            mDownloadId = mDownloadManager.enqueue(request);
            starts();
        }

        //注册广播接收者，监听下载状态
        mContext.registerReceiver(receiver,
                new IntentFilter(DownloadManager.ACTION_DOWNLOAD_COMPLETE));
    }

    //广播监听下载的各个状态
    private BroadcastReceiver receiver = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            checkStatus();
        }
    };

    //检查下载状态
    private void checkStatus() {
        DownloadManager.Query query = new DownloadManager.Query();
        //通过下载的id查找
        query.setFilterById(mDownloadId);
        Cursor cursor = mDownloadManager.query(query);
        if (cursor.moveToFirst()) {
            int status = cursor.getInt(cursor.getColumnIndex(DownloadManager.COLUMN_STATUS));
            switch (status) {
                //下载暂停
                case DownloadManager.STATUS_PAUSED:
                    break;
                //下载延迟
                case DownloadManager.STATUS_PENDING:
                    break;
                //正在下载
                case DownloadManager.STATUS_RUNNING:
                    break;
                //下载完成
                case DownloadManager.STATUS_SUCCESSFUL:
                    //下载完成安装APK
                    installAPK();
                    cursor.close();
                    break;
                //下载失败
                case DownloadManager.STATUS_FAILED:
                    Toast.makeText(mContext, "下载失败", Toast.LENGTH_SHORT).show();
                    cursor.close();
                    mContext.unregisterReceiver(receiver);
                    break;
            }
        }
    }

    private void installAPK() {
        setPermission(pathstr);
        Intent intent = new Intent(Intent.ACTION_VIEW);
        // 由于没有在Activity环境下启动Activity,设置下面的标签
        intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        //Android 7.0以上要使用FileProvider
        if (Build.VERSION.SDK_INT >= 24) {
            File file = (new File(pathstr));
            //参数1 上下文, 参数2 Provider主机地址 和配置文件中保持一致   参数3  共享的文件
            Uri apkUri = FileProvider.getUriForFile(mContext, "com.flash.fileprovider", file);
            //添加这一句表示对目标应用临时授权该Uri所代表的文件
            intent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);
            intent.setDataAndType(apkUri, "application/vnd.android.package-archive");
        } else {
            intent.setDataAndType(Uri.fromFile(new File(Environment.DIRECTORY_DOWNLOADS, name)), "application/vnd.android.package-archive");
        }
        mContext.startActivity(intent);
    }

    //修改文件权限
    private void setPermission(String absolutePath) {
        String command = "chmod " + "777" + " " + absolutePath;
        Runtime runtime = Runtime.getRuntime();
        try {
            runtime.exec(command);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private void starts() {
        new Thread() {
            @Override
            public void run() {
                super.run();
                boolean isFinished = false;
                // 创建一个下载查询对象，按照下载编号进行过滤
                DownloadManager.Query down_query = new DownloadManager.Query();
                // 设置下载查询对象的编号过滤器
                down_query.setFilterById(mDownloadId);
                // 向下载管理器发起查询操作，并返回查询结果集的游标
                Cursor cursor = mDownloadManager.query(down_query);
                while (cursor.moveToNext()) {
                    //Toast.makeText(MainActivity.this,"进入了while循环",Toast.LENGTH_LONG).show();
                    int nameIdx = cursor.getColumnIndex(DownloadManager.COLUMN_LOCAL_FILENAME);
                    int uriIdx = cursor.getColumnIndex(DownloadManager.COLUMN_LOCAL_URI);
                    int totalSizeIdx = cursor.getColumnIndex(
                            DownloadManager.COLUMN_TOTAL_SIZE_BYTES);
                    int nowSizeIdx = cursor.getColumnIndex(
                            DownloadManager.COLUMN_BYTES_DOWNLOADED_SO_FAR);
                    // 根据总大小和已下载大小，计算当前的下载进度
                    final int progress = (int) (100 * cursor.getLong(nowSizeIdx) / cursor.getLong(totalSizeIdx));
                    // Toast.makeText(MainActivity.this,progress,Toast.LENGTH_LONG).show();
                    if (cursor.getString(uriIdx) == null) {
                        break;
                    }
                    Message msg = new Message();
                    msg.what = MSG_SHOW;
                    Bundle bundle = new Bundle();
                    bundle.putInt("progress", progress);
                    msg.setData(bundle);//mes利用Bundle传递数据
                    mHandler.sendMessage(msg);

                    // Android7.0之后提示COLUMN_LOCAL_FILENAME已废弃
                    if (Build.VERSION.SDK_INT < Build.VERSION_CODES.N) {
                        pathstr = cursor.getString(nameIdx);
                    } else {
                        // 所以7.0之后要先获取文件的Uri，再根据Uri获取文件路径
                        String fileUri = cursor.getString(uriIdx);
                        pathstr = Uri.parse(fileUri).getPath();
                    }
                    if (progress == 100) { // 下载完毕
                        isFinished = true;
                    }
                }
                cursor.close(); // 关闭数据库游标
                if (!isFinished) { // 未完成，则继续刷新
                    // 延迟100毫秒后再次启动下载进度的刷新任务
                    mHandler.postDelayed(this, 100);
                } else { // 已完成
                    Message msg = new Message();
                    msg.what = MSG_COMPLETED;
                    Bundle bundle = new Bundle();
                    bundle.putString("path", pathstr);
                    msg.setData(bundle);//mes利用Bundle传递数据
                    mHandler.sendMessage(msg);
                }
            }
        }.start();
    }

    /**
     * 下载前先移除前一个任务，防止重复下载
     *
     * @param downloadId
     */
    public void clearCurrentTask(long downloadId) {
        DownloadManager dm = (DownloadManager) mContext.getSystemService(Context.DOWNLOAD_SERVICE);
        try {
            dm.remove(downloadId);
        } catch (IllegalArgumentException ex) {
            ex.printStackTrace();
        }
    }

    private static class UiHandler extends Handler {
        private WeakReference<Context> mWeakReference;

        UiHandler(Context context) {
            mWeakReference = new WeakReference<Context>(context);
        }

        @Override
        public void handleMessage(Message msg) {
            Context context = mWeakReference.get();
            if (context != null) {
                switch (msg.what) {
                    case MSG_SHOW:
                        int progress = msg.getData().getInt("progress", 0);
//                        updateView.setText("进度："+progress+"%");
                        break;
                    case MSG_COMPLETED:
                        String path = msg.getData().getString("path", "");
                        Toast.makeText(context, path, Toast.LENGTH_LONG).show();
                        break;
                    default:
                        break;
                }
            }
        }


    }

}


```

要注意的地方：
如果禁用了下载管理器，调用enqueue方法可能出现的错误：Cannot update URI: content://downloads/my_downloads/-1
下面的checkDownloadManagerEnable方法是用来检测下载管理器是否被禁用，如果禁用了主动开启。

```java
fun checkDownloadManagerEnable():Boolean {
       try {
           val state = mContext.packageManager.getApplicationEnabledSetting("com.android.providers.downloads")
           if (state == PackageManager.COMPONENT_ENABLED_STATE_DISABLED
                   || state == PackageManager.COMPONENT_ENABLED_STATE_DISABLED_USER
                   || state == PackageManager.COMPONENT_ENABLED_STATE_DISABLED_UNTIL_USED) {
               val packageName = "com.android.providers.downloads"
               try {
                   val intent = Intent(android.provider.Settings.ACTION_APPLICATION_DETAILS_SETTINGS)
                   intent.data = Uri.parse("package:$packageName")
                   mContext.startActivity(intent)
               } catch (e: ActivityNotFoundException) {
                   e.printStackTrace()
                   val intent = Intent(android.provider.Settings.ACTION_MANAGE_APPLICATIONS_SETTINGS)
                   mContext.startActivity(intent)
               }
               return false
           }
       } catch (e:Exception) {
           e.printStackTrace()
           return false
       }
       return true
   }
```
————————————————
使用方式参考：https://blog.csdn.net/u012861467/article/details/82808290


#### 2.需要注意的地方

1.我们为什么要使用DownloadManager

  1.它使用的是系统进程，系统服务。-----

  2.注意它的缺点------
  它无法暂停，只能remove
  应用无法控制，当开启下载后，基本上由系统管理。

2.原理分析

  这原理就不用分析了，总的说就是调用系统的服务来进行下载，和应用不在一个进程中;

  DownloadManager类中主要是数据操作（以应用为单位建表增删改查）和定义一套数据结构（数据协议）;

  具体怎么调用的，请看getSystemService（）方法，进程间通信基本上和AIDL的使用差不多吧，只不过它是系统级的;
  android系统的源码没去看，具体分析参考一下网上的资料吧

  1.https://www.jianshu.com/p/27f08de3ffa9

  2.https://www.jianshu.com/p/c9dc04af2f54
