---
layout:     post
title:      "Android 系统缓存扫描与清理方法分析"
subtitle:   " \"这篇笔记记录Android系统缓存扫描及清理所遇到的问题...\""
date:       2016-10-2 00:30:00
author:     "RyanYans"
header-img: "img/post-bg-freecache.jpg"
catalog: true
tags:
    - Android
    - FreeCache
    - 反射
---


本文记录的是我对 Android 的「系统缓存」及其扫描和清理方法的探索与理解。

本文讲述内容的完整代码实例见 https://github.com/mzlogin/CleanExpert。

## 系统缓存的定义

如下是我捏造的非官方定义：

系统缓存： Android APP 在运行过程中保存在手机内置和外置存储上的缓存文件总和。

## 系统缓存的组成

先说结论：

「系统缓存」由所有已安装应用的 /data/data/packagename/cache 文件夹
和 /sdcard/Android/data/packagename/cache 文件夹组成。

如下是原理分析，不感兴趣的可以直接跳到下一节。

我们先来看一个熟悉的界面：

![Step3_0](http://mazhuang.org/images/posts/android/installed-app-details.png)

这是手机的「设置」——「应用」里的已安装应用的详情页，这里面会显示缓存的大小，而且提供了清理缓存的功能，这就是我们做「系统缓存」清理想做的事情。

这里显示的大小是如何计算出来的，它实际上的文件组成是怎么样的呢？可以从 Android 系统自带的 Settings APP 的源码中找到答案。


## 系统缓存大小的计算

首先我们需要了解：

*  「内部缓存」由所有已安装应用的 /data/data/packagename/cache 文件夹组成。
* 「外部缓存」由所有已安装应用的 /sdcard/Android/data/packagename/cache 文件夹组成。

当我们要想计算系统缓存大小的时候下意识的想法就是，用代码计算一下这两个文件夹的大小不就行了？

事实证明这个想法只是 too young, sometimes naive.

/data/data/packagename/cache 文件夹每个应用访问属于自己的无压力，但其它应用是没有权限读取的，如果是做本应用内的缓存清理，那事情就简单了，直接算一算就好了。

如果是要做针对所有应用的缓存清理功能，那就得另想办法了。

**这里分了两种情况：能获取 root 权限和不能获取 root 权限。我们这里先讨论非 root 权限的系统缓存计算和清理，root 权限的情况在后文会有说明。**

既然直接计算文件夹大小的方法行不通了，那就仍然重复上面的故事，参考 Settings APP 的做法吧。

### Settings 计算缓存大小的方法

Settings APP 使用了 PackageManager.getPackageSizeInfo 方法来做此事，难道 so easy？屁颠屁颠去查了一下 Android API，发现 PacakgeManager 的文档中压根就没有出现 getPackageSizeInfo 的身影，好吧这是一个不对外公开的 API。

但是区区困难怎么拦得住一颗改变世界的心？对付隐藏 API 我们还有反射大法。

我们回顾一下 Settings APP 里的做法：

	class BackgroundHandler extends Handler {
	    ......
	    final IPackageStatsObserver.Stub mStatsObserver = new IPackageStatsObserver.Stub() {
	                public void onGetStatsCompleted(PackageStats stats, boolean succeeded) {
	                    ......
	                                    entry.cacheSize = stats.cacheSize;
	                                    ......
	                                    entry.externalCacheSize = stats.externalCacheSize;
	                    ......
	                }
	            };
	    ......
	    @Override
	    public void handleMessage(Message msg) {
	        ......
	        switch (msg.what) {
	            ......
	            case MSG_LOAD_SIZES: {
	                synchronized (mEntriesMap) {
	                    ......
	                                mPm.getPackageSizeInfo(mCurComputingSizePkg, mStatsObserver);
	                    ......
	                }
	            } break;
	        }
	    }
	    ......
	}

这个类定义在文件 packages/apps/Settings/src/com/android/settings/applications/ApplicationsState.java 中。

**这里有两个问题需要解决：**

1. getPackageSizeInfo 方法是一个 @hide 方法，需要通过反射来调用。

从 PackageManager.java 文件的 getPackageSizeInfo 方法定义处可知，它需要 GET_PACKAGE_SIZE 权限，幸运的是，从 frameworks/base/core/res/AndroidManifex.xml 文件里可知，该权限的 Protection level 为 normal，是可以正常声明的。

	<!-- Allows an application to find out the space used by any package. -->
	<permission android:name="android.permission.GET_PACKAGE_SIZE"
	    android:permissionGroup="android.permission-group.SYSTEM_TOOLS"
	    android:protectionLevel="normal"
	    android:label="@string/permlab_getPackageSize"
	    android:description="@string/permdesc_getPackageSize" />

这段代码定义在文件 frameworks/base/core/res/AndroidManifex.xml 中。

2. 传给 getPackageSizeInfo 方法的第二个参数类型 IPackageStatsObserver 是在 android.content.pm 包下，需要自已通过 aidl 方式定义。

### 计算缓存大小的实现

解决步骤：

1. 在自己的工程的 src/main 目录下创建包目录结构 aidl/android/content/pm。

（注：这是使用 Android Studio 的默认做法，使用 Eclipse 默认在 src 目录下创建包目录结构 android/content/pm。）

将 Android 源码 frameworks/base/core/java/android/content/pm 目录下的 IPackageStatsObserver.aidl 与其依赖的 PackageStats.aidl 拷贝到上面一步创建的目录里。

根据 frameworks/base/core/java/android/content/pm/PackageManager.java 的 getPackageSizeInfo 接口上面的注释可知，需要在 AndroidManifest.xml 里声明需要 GET_PACKAGE_SIZE 权限。

<uses-permission android:name="android.permission.GET_PACKAGE_SIZE" />


获取 QQ 的系统缓存大小的示例代码：

		public void someFunc() {
		    IPackageStatsObserver.Stub observer = new PackageSizeObserver();
		    getPackageInfo("com.tencent.mobileqq", observer);
		}
		
		public void getPackageInfo(String packageName, IPackageStatsObserver.Stub observer) {
		    try {
		        PackageManager pm = ContextUtil.applicationContext.getPackageManager();
		        Method getPackageSizeInfo = pm.getClass()
		                .getMethod("getPackageSizeInfo", String.class, IPackageStatsObserver.class);
		
		        getPackageSizeInfo.invoke(pm, packageName, observer);
		    } catch (NoSuchMethodException e ) {
		        e.printStackTrace();
		    } catch (IllegalAccessException e) {
		        e.printStackTrace();
		    } catch (InvocationTargetException e) {
		        e.printStackTrace();
		    }
		}
		
		private class PackageSizeObserver extends IPackageStatsObserver.Stub {
		    @Override
		    public void onGetStatsCompleted(PackageStats packageStats, boolean succeeded)
		            throws RemoteException {
		        if (packageStats == null || !succeeded) {
		        } else {
		            AppEntry entry = new AppEntry();
		            entry.packageName = packageStats.packagename;
		            entry.cacheSize = packageStats.cacheSize + packageStats.externalCacheSize;
		            // do something else，比如把 entry 通过消息发送给需要的地方，或者添加到你的列表里
		        }
		    }
		}

获取一个应用的缓存的问题解决了，获取所有应用的系统缓存也就是遍历系统已安装应用，然后挨个调用 getPackageInfo 的事儿了。

例：

		private void initData() {
	        new Thread(new Runnable() {
	            @Override
	            public void run() {
	                mPackageManager = getPackageManager();
	                List<ApplicationInfo> installedApplications = mPackageManager.getInstalledApplications(PackageManager.GET_UNINSTALLED_PACKAGES);
	                pb_cacheclean.setMax(installedApplications.size());
	                int index = 1;
	
	                for (ApplicationInfo info : installedApplications) {
	                    try {
	                        Thread.sleep(20+ new Random().nextInt(50));
	                        getPackageCache(info);
	                        pb_cacheclean.setProgress(index++);
	
	                        Message msg = Message.obtain();
	                        msg.what = CHECK_APP_CACHE;
	                        msg.obj = info.packageName;
	                        mHandler.sendMessage(msg);
	
	                    } catch (InterruptedException e) {
	                        e.printStackTrace();
	                    }
	                }
	                Message msg = Message.obtain();
	                msg.what = SCAN_FINISH;
	                mHandler.sendMessage(msg);
	            }
	
	        }).start();
	    }
	
	    
	
	
	    private void getPackageCache(final ApplicationInfo info) {
	
	        mPm = getPackageManager();
	        //引入IPackageStatsObserver.stub 类的aidl文件，并实现 onGetStatsCompleted 方法
	        IPackageStatsObserver.Stub mStatsObserver = new IPackageStatsObserver.Stub() {
	
	            public void onGetStatsCompleted(PackageStats stats, boolean succeeded) {
	
	                Message msg = Message.obtain();
	                msg.what = ADD_CACHE_VIEW;
	                AppCacheViewInfo viewInfo = new AppCacheViewInfo();
	                //缓存大小的过程
	                if (stats.cacheSize > 0) {
	                    String cacheSize = Formatter.formatFileSize(getApplicationContext(), stats.cacheSize);
	                    viewInfo.setName(info.loadLabel(mPackageManager).toString());
	                    viewInfo.setPkgName(info.packageName);
	                    viewInfo.setCachaSize(cacheSize);
	                    viewInfo.setIcon(info.loadIcon(mPackageManager));
	                    msg.obj = viewInfo;
	                    mHandler.sendMessage(msg);
	                }
	            }
	        };
	
	        //函数被隐藏，需要通过反射获取方法
		//        mPm.getPackageSizeInfo("com.android.browser", mStatsObserver);
	        try {
	            Class<?> clazz = Class.forName("android.content.pm.PackageManager");
	            Method method = clazz.getMethod("getPackageSizeInfo", String.class, IPackageStatsObserver.class);
	            method.invoke(mPm, info.packageName, mStatsObserver);
	
	        } catch (Exception e) {
	            e.printStackTrace();
	        }
	    }

	class AppCacheViewInfo {
		        private String name;
		        private String pkgName;
		        private Drawable icon;
		        private String cachaSize;
		
		        public String getName() {
		            return name;
		        }
		
		        public void setName(String name) {
		            this.name = name;
		        }
		
		        public String getPkgName() {
		            return pkgName;
		        }
		
		        public void setPkgName(String pkgName) {
		            this.pkgName = pkgName;
		        }
		
		        public Drawable getIcon() {
		            return icon;
		        }
		
		        public void setIcon(Drawable icon) {
		            this.icon = icon;
		        }
		
		        public String getCachaSize() {
		            return cachaSize;
		        }
		
		        public void setCachaSize(String cachaSize) {
		            this.cachaSize = cachaSize;
		        }
		    }


## 系统缓存的清理

既然借鉴 Settings APP 的做法如此好使，在做缓存清理时我们当然故伎重施。我们先来看看它是怎样清理某一个应用的缓存的。

### Settings 清理缓存的方法

在 InstalledAppDetails.java 里能根据名称找到对应「清除缓存」按钮相关的代码：

	public class InstalledAppDetails extends Fragment
	 implements View.OnClickListener, CompoundButton.OnCheckedChangeListener,
	        ApplicationsState.Callbacks {
	    ......
	    private Button mClearCacheButton;
	    ......
	    class ClearCacheObserver extends IPackageDataObserver.Stub {
	        public void onRemoveCompleted(final String packageName, final boolean succeeded) {
	            final Message msg = mHandler.obtainMessage(CLEAR_CACHE);
	            msg.arg1 = succeeded ? OP_SUCCESSFUL:OP_FAILED;
	            mHandler.sendMessage(msg);
	         }
	     }
	    ......
	    @Override
	    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
	        ......
	        mClearCacheButton = (Button) view.findViewById(R.id.clear_cache_button);
	        ......
	    }
	    ......
	    private void refreshSizeInfo() {
	        ......
	        if (cacheSize <= 0) {
	            mClearCacheButton.setEnabled(false);
	        } else {
	            mClearCacheButton.setEnabled(true);
	            mClearCacheButton.setOnClickListener(this);
	        }
	    }
	    ......
	    public void onClick(View v) {
	        ......
	        } else if (v == mClearCacheButton) {
	            // Lazy initialization of observer
	            if (mClearCacheObserver == null) {
	                mClearCacheObserver = new ClearCacheObserver();
	            }
	            mPm.deleteApplicationCacheFiles(packageName, mClearCacheObserver);
	        }
	        ......
	    }
这个类定义在文件 packages/apps/Settings/src/com/android/settings/applications/InstalledAppDetails.java 中。

是不是很熟悉？是不是很激动？是不是觉得顶多再次祭出反射大法就能继续拯救世界了？先冷静一下，看看 frameworks/base/core/java/android/content/pm/PackageManager.java 文件里 deleteApplicationCacheFiles 方法上面的注释。

	/**
	 * Attempts to delete the cache files associated with an application.
	 * Since this may take a little while, the result will
	 * be posted back to the given observer.  A deletion will fail if the calling context
	 * lacks the {@link android.Manifest.permission#DELETE_CACHE_FILES} permission, if the
	 * named package cannot be found, or if the named package is a "system package".
	 *
	 * ......
	 *
	 * @hide
	 */
没错它又是一个 @hide 方法，关键是它需要 DELETE_CACHE_FILES 权限，而该权限的相关声明如下：

	<!-- Allows an application to delete cache files. -->
	<permission android:name="android.permission.DELETE_CACHE_FILES"
	    android:label="@string/permlab_deleteCacheFiles"
	    android:description="@string/permdesc_deleteCacheFiles"
	    android:protectionLevel="signature|system" />

这段声明定义在 frameworks/base/core/res/AndroidManifest.xml 中。

它的 protectionLevel 为 signaturesystem，系统应用或者与系统采用相同签名的应用才能获得此权限。

此路不通。

新的发现

那就继续想其它办法了。frameworks/base/core/java/android/content/pm/PackageManager.java 里提供了很多实用的功能，比如上面的系统缓存的大小计算以及清理都是它里面声明的方法，仔细看一下它里面声明的其它方法还真是有发现：

	/**
	 * Free storage by deleting LRU sorted list of cache files across
	 * all applications. If the currently available free storage
	 * on the device is greater than or equal to the requested
	 * free storage, no cache files are cleared. If the currently
	 * available storage on the device is less than the requested
	 * free storage, some or all of the cache files across
	 * all applications are deleted (based on last accessed time)
	 * to increase the free storage space on the device to
	 * the requested value. There is no guarantee that clearing all
	 * the cache files from all applications will clear up
	 * enough storage to achieve the desired value.
	 * @param freeStorageSize The number of bytes of storage to be
	 * freed by the system. Say if freeStorageSize is XX,
	 * and the current free storage is YY,
	 * if XX is less than YY, just return. if not free XX-YY number
	 * of bytes if possible.
	 * @param observer call back used to notify when
	 * the operation is completed
	 *
	 * @hide
	 */
public abstract void freeStorageAndNotify(long freeStorageSize, IPackageDataObserver observer);
从解释来看它是用来在必要时释放所有应用的缓存所占空间的，在 Android 源码里搜索一下它被调用的地方，有一处是在 frameworks/base/services/java/com/android/server/DeviceStorageMonitorService.java 中，大致的逻辑是在系统空间不够的时候，提示用户清理系统缓存。

我们来看看这个方法实际做了什么事情：

	public class PackageManagerService extends IPackageManager.Stub {
	    ......
	    public void freeStorageAndNotify(final long freeStorageSize, final IPackageDataObserver observer) {
	        mContext.enforceCallingOrSelfPermission(
	                android.Manifest.permission.CLEAR_APP_CACHE, null);
	        // Queue up an async operation since clearing cache may take a little while.
	        mHandler.post(new Runnable() {
	            public void run() {
	                mHandler.removeCallbacks(this);
	                int retCode = -1;
	                retCode = mInstaller.freeCache(freeStorageSize);
	                if (retCode < 0) {
	                    Slog.w(TAG, "Couldn't clear application caches");
	                }
	                if (observer != null) {
	                    try {
	                        observer.onRemoveCompleted(null, (retCode >= 0));
	                    } catch (RemoteException e) {
	                        Slog.w(TAG, "RemoveException when invoking call back");
	                    }
	                }
	            }
	        });
	    }
	    ......
	}

这个类定义在文件 frameworks/base/services/java/com/android/server/pm/PackageManagerService.java 中。

也就是说，这个方法的注释里没有提及它需要申请什么权限，但事实上它是需要 CLEAR_APP_CACHE 权限的。

该权限的相关声明：

<!-- Allows an application to clear the caches of all installed
     applications on the device.  -->
<permission android:name="android.permission.CLEAR_APP_CACHE"
    android:permissionGroup="android.permission-group.SYSTEM_TOOLS"
    android:protectionLevel="dangerous"
    android:label="@string/permlab_clearAppCache"
    android:description="@string/permdesc_clearAppCache" />
这段声明定义在 frameworks/base/core/res/AndroidManifest.xml 中。

虽然它的 protectionLevel 是 dangerous，但是好歹还是能用的。

另外，跟踪实际执行清理过程的 retCode = mInstaller.freeCache(freeStorageSize); 这一行实际是通过给 /dev/socket/installd 发送 freecache freeStorageSize 来完成清理过程，最终调用到如下函数：

int free_cache(int64_t free_size)
{
    ......
    char datadir[PKG_PATH_MAX];
    avail = disk_free();
    ......
    if (avail >= free_size) return 0;

    if (create_persona_path(datadir, 0)) { // /data/data
        ......
    }

    d = opendir(datadir);
    ......
    dfd = dirfd(d);

    while ((de = readdir(d))) {
        if (de->d_type != DT_DIR) continue;
        name = de->d_name;
        ......
        subfd = openat(dfd, name, O_RDONLY | O_DIRECTORY);
        if (subfd < 0) continue;

        delete_dir_contents_fd(subfd, "cache");
        close(subfd);

        avail = disk_free();
        if (avail >= free_size) {
            closedir(d);
            return 0;
        }
    }
    ......
}
这个函数定义在文件 frameworks/base/cmds/installd/Commands.c 中。

实际就是遍历 /data/data 下的所有文件夹，依次删除它们下面的 cache 子目录，直到磁盘的可用空间大于需要的空间为止。

也就是说，freeStorageAndNotify 只是删除了「内部缓存」，扩展存储上的「外部缓存」需要我们另外处理。


## 清理缓存的实现

参考 frameworks/base/services/java/com/android/server/DeviceStorageMonitorService.java 中对 freeStorageAndNotify 的相关调用，最后我们的实现步骤如下：

在自己的工程的 src/main 目录下创建包目录结构 aidl/android/content/pm。

（注：这是使用 Android Studio 的默认做法，使用 Eclipse 默认在 src 目录下创建包目录结构 android/content/pm。）

将 Android 源码 frameworks/base/core/java/android/content/pm 目录下的 IPackageDataObserver.aidl 拷贝到上面一步创建的目录里。

在 AndroidManifest.xml 里声明 CLEAR_APP_CACHE 权限。

    <uses-permission android:name="android.permission.CLEAR_APP_CACHE" />

通过反射调用 freeStorageAndNotify 方法，第一个参数给它一个足够大的值，它就会帮我们清理掉所有应用的缓存了。

	public static void freeAllAppsCache(final Handler handler) {
	
	    Context context = ContextUtil.applicationContext;
	
	    File externalDir = context.getExternalCacheDir();
	    if (externalDir == null) {
	        return;
	    }
	
	    PackageManager pm = context.getPackageManager();
	    List<ApplicationInfo> installedPackages = pm.getInstalledApplications(PackageManager.GET_GIDS);
	    for (ApplicationInfo info : installedPackages) {
	        String externalCacheDir = externalDir.getAbsolutePath()
	                .replace(context.getPackageName(), info.packageName);
	        File externalCache = new File(externalCacheDir);
	        if (externalCache.exists() && externalCache.isDirectory()) {
	            deleteFile(externalCache);
	        }
	    }
	
	    try {
	        Method freeStorageAndNotify = pm.getClass()
	                .getMethod("freeStorageAndNotify", long.class, IPackageDataObserver.class);
	        long freeStorageSize = Long.MAX_VALUE;
	
	        freeStorageAndNotify.invoke(pm, freeStorageSize, new IPackageDataObserver.Stub() {
	            @Override
	            public void onRemoveCompleted(String packageName, boolean succeeded) throws RemoteException {
	                Message msg = handler.obtainMessage(JunkCleanActivity.MSG_SYS_CACHE_CLEAN_FINISH);
	                msg.sendToTarget();
	            }
	        });
	    } catch (NoSuchMethodException e) {
	        e.printStackTrace();
	    } catch (IllegalAccessException e) {
	        e.printStackTrace();
	    } catch (InvocationTargetException e) {
	        e.printStackTrace();
	    }
	}
	
	public static boolean deleteFile(File file) {
	    if (file.isDirectory()) {
	        String[] children = file.list();
	        for (String name : children) {
	            boolean suc = deleteFile(new File(file, name));
	            if (!suc) {
	                return false;
	            }
	        }
	    }
	    return file.delete();
	}

**备注：** 经测试该方法在 Android 6.0 版本和部分 5.0+ 版本上已经失效，Android 源码里已经给 freeStorageAndNotify 方法声明添加了 @SystemApi 注释（开始添加了 @PrivateApi，后修改为 @SystemApi），见「添加」和「修改」两次提交，而且 CLEAR_APP_CACHE 方法的权限已经由 dangerous 改成了 system signature，已经无法通过反射来正常调用，会产生 java.lang.reflect.InvocationTargetException，所以在这些版本上需要另想办法了，StackOverflow 上的一个相关讨论链接：What’s the meaning of new @SystemApi annotation, any difference from @hide?。

## 有 root 权限的系统缓存计算与清理

如果能获取到 root 权限，/data/data 目录的访问限制也就不再是问题，计算缓存大小和清理缓存也就不用再受上面说的方法与权限的限制了，而且能做一些没有 root 权限的情况下做不到的事情，比如：

清理单个应用的缓存。

列出应用的缓存文件列表供用户选择性清理。

实现思路很简单粗暴（如下思路未写实例验证）：

**思路一**  

通过 su 命令获取一个有 root 权限的 shell，然后通过与它交互来获取缓存文件夹的大小或清理缓存，比如让它执行命令 du -h /data/data/com.trello/cache 就能获取到 trello 的「内部缓存」大小，让它执行 rm -rf /data/data/com.trello/cache 就能删除 trello 的「内部缓存」。

注：du 命令行与参数在不同 ROM 下的不一致，所以并不推荐此做法。

**思路二**  

或者，也可以做一个原生程序专门来负责缓存计算与清理，通过 su 命令获取有 root 权限的 shell，再用 shell 创建该原生程序进程，它继承 shell 的 root 权限，然后它就可以计算缓存大小与清理缓存，再将结果上报给 APP 进程。


----------  

##### 学习笔记，如有谬误，敬请指正。

