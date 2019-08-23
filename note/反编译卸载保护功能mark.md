
### 总结：
##### 该功能的主要原理应该是获取到手机系统管理安装卸载的系统级应用的“包名”，然后在跳转到该界面时打开锁屏界面。（应该与普通的applock功能原理相同，只不过是lock系统级界面）


```java
private static final List<String> f14474as = new ArrayList<String>() {
    {
        add("com.android.systemui");
        add("com.vivo.upslide");
        add("com.coloros.recents");
    }
};

/* renamed from: er */
private static final Map<String, String> f14475er = new HashMap<String, String>() {
    {
        put("com.android.packageinstaller", "com.android.packageinstaller.UninstallerActivity");
        put("com.google.android.packageinstaller", "com.android.packageinstaller.UninstallerActivity");
        put("com.samsung.android.packageinstaller", "com.android.packageinstaller.UninstallerActivity");
    }
};

public static boolean m13764fe() {//在小米华为和蓝绿厂的手机上不显示
    return !TextUtils.equals(Build.MANUFACTURER.toLowerCase(), "xiaomi") && !TextUtils.equals(Build.MANUFACTURER.toLowerCase(), "huawei") && !TextUtils.equals(Build.MANUFACTURER.toLowerCase(), "oppo") && !TextUtils.equals(Build.MANUFACTURER.toLowerCase(), "vivo");
}
```

### 主要逻辑的分析：

实现该逻辑的主要类是：LockAppActivity，LockAppFragment，dbe（这是个混淆后的adapter，是主页列表adapter），dbf是存储的数据类型，从上到下分别为包名（string），
应用名（string），类别（int），？（int）;

无论是应用卸载保护，应用界面保护又或者是小标题，都在同一个list中。用getItemType（）进行区分;

结合以上两条，所以代码中直接判断包名是否equals（）com.android.packageinstaller就能区分是哪个功能。


下面是锁屏弹出的逻辑：


```java
dbo.java:

private BroadcastReceiver f14460xz = new BroadcastReceiver() {
    public final void onReceive(Context context, Intent intent) {
        if ("android.intent.action.SCREEN_ON".equals(intent.getAction())) {
            return;
        }
        if ("android.intent.action.USER_PRESENT".equals(intent.getAction())) {
            dcf as = dcf.m13890as();
            if (!as.mo16618er() && as.f14588yr && as.f14578fe) {
                FingerprintActivity.m23711jd();
                dbv.m13817as().mo16550td();
            }
            C3134a.f11752as;
            String as2 = ckx.m11083as();
            if (TextUtils.isEmpty(as2) || !AppLockProvider.m18305xv(as2)) {//EXTRA_APP_IS_LOCKED?
                return;
            }
            if (!dbp.m13766jd() || !dbo.this.f14452hv.contains(as2)) {
                ApplicationInfo as3 = C4067a.f15536as.mo17099as(as2);
                if (as3 != null) {
                    String as4 = C4067a.f15536as.mo17100as(as3);
                    if (!TextUtils.isEmpty(as4)) {
                        dcf.m13890as().mo16617as(as4, as2, dbo.this.mo16496xv(as2)); //这个方法打开锁屏activity
                        dbo.this.mo16494er(as2);
                    }
                }
            }
        } else if ("android.intent.action.SCREEN_OFF".equals(intent.getAction())) {
            dcf as5 = dcf.m13890as();
            as5.f14578fe = true;
            as5.f14587yf = false;
            switch (dbo.this.f14461yf) {
                case 1:
                    for (String er : new ArrayList(Arrays.asList(dbo.this.f14453jd.mo14522as("PREF_KEY_DO_NOT_LOCK_BEFORE_SCREEN_OFF_PACKAGE_NAMES", "").split(";")))) {
                        dbo.this.f14453jd.mo14527er(er);
                    }
                    dbo.this.f14453jd.mo14530er("PREF_KEY_DO_NOT_LOCK_BEFORE_SCREEN_OFF_PACKAGE_NAMES", "");
                    return;
                case 2:
                    dbo.this.f14453jd.mo14530er("PREF_KEY_DO_NOT_LOCK_BEFORE_APP_CLOSE_PACKAGE_NAME", "");
                    return;
                default:
                    return;
            }
        }
    }
};
```
下面是启动锁屏的方法，方法中不断传递的2个string就是包名和应用名。
```java
dcf.java

public final void mo16617as(String str, String str2, int i) {
    this.f14585td = str;
    this.f14580hv = str2;
    this.f14581jd = i;
    this.f14584ss = this.f14589yt.get() && AppLockProvider.m18277jd(str2);
    new StringBuilder("LockProcessor performToLockApp() appLabel = ").append(str).append(" packageName = ").append(str2).append(" shouldDisguiseThisApp = ").append(this.f14584ss);
    if (mo16618er()) {
        this.f14575bh = false;
        HSApplication.m18054xv().startActivity(new Intent(HSApplication.m18054xv(), LockAppActivity.class).addFlags(872480768));
        this.f14583oi.postDelayed(new Runnable() {
            public final void run() {
                if (!dcf.this.f14575bh) {
                    dbo as = dbo.m13716as();
                    C3134a.f11752as;
                    String as2 = ckx.m11083as();
                    if (!TextUtils.isEmpty(as2)) {
                        switch (as.f14461yf) {
                            case 1:
                                if (as.f14453jd.mo14522as("PREF_KEY_DO_NOT_LOCK_BEFORE_SCREEN_OFF_PACKAGE_NAMES", "").contains(as2)) {
                                    int as3 = as.f14453jd.mo14520as(as2, 0);
                                    if (as3 < as.f14446bh) {
                                        Toast toast = new Toast(HSApplication.m18054xv());
                                        toast.setView(View.inflate(HSApplication.m18054xv(), R.layout.mc, null));
                                        toast.setDuration(0);
                                        toast.show();
                                    }
                                    as.f14453jd.mo14528er(as2, as3 + 1);
                                    return;
                                }
                                break;
                            case 2:
                                if (!TextUtils.equals(as.f14453jd.mo14522as("PREF_KEY_DO_NOT_LOCK_BEFORE_APP_CLOSE_PACKAGE_NAME", ""), as2)) {
                                    as.f14453jd.mo14530er("PREF_KEY_DO_NOT_LOCK_BEFORE_APP_CLOSE_PACKAGE_NAME", "");
                                    break;
                                } else {
                                    return;
                                }
                        }
                        if (!AppLockProvider.m18305xv(as2)) {
                            return;
                        }
                        if (TextUtils.equals(as2, as.f14454nf)) {
                            as.f14454nf = "";
                        } else if (as.f14458td.contains(as2)) {
                        } else {
                            if (!dbp.m13766jd() || !as.f14452hv.contains(as2)) {
                                ApplicationInfo as4 = C4067a.f15536as.mo17099as(as2);
                                if (as4 != null) {
                                    String as5 = C4067a.f15536as.mo17100as(as4);
                                    if (!TextUtils.isEmpty(as5)) {
                                        dcf.m13890as().mo16617as(as5, as2, as.mo16496xv(as2));
                                        as.mo16494er(as2);
                                    }
                                }
                            }
                        }
                    }
                }
            }
        }, 300);
    } else if (!dkw.m15033as()) {
        this.f14583oi.post(new Runnable() {
            public final void run() {
                if (dcf.this.f14574as != null) {
                    dcf.this.f14574as.mo16593er();
                }
            }
        });
    } else if (this.f14574as != null) {
        this.f14574as.mo16593er();
    }
}
```
启动锁屏流程的方法中的变量只有传入的应用信息，所以我认为实现原理相同。但是有意思的是判断是否支持卸载保护的方法没有解析出来，所以这里还需注意。

待续。。。
--------------------------
