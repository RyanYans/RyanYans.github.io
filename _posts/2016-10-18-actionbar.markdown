---
layout:     post
title:      "Android 系统状态栏沉浸式 && 顶部标题栏去除 "
subtitle:   " \"记录Android系统标题栏去除及沉浸式状态栏的解决方案...\""
date:       2016-10-18 00:30:00
author:     "RyanYans"
header-img: "img/post-bg-actionbar.jpg"
catalog: true
tags:
    - Android
    - Toolbar
    - Actionbar
    - 沉浸式
---


本文记录的是我对 Android 系统状态栏沉浸式解决方案探索与理解。

本文讲述内容的完整代码实例见 [https://github.com/RyanYans/adaptiveStatusBar](https://github.com/RyanYans/adaptiveStatusBar)

### 去除顶部标题栏  
直接上代码：
styles.xml：

			<!--安卓6.0去掉标题栏的方法-->
		    <style name="AppTheme.NoActionBar">
		        <item name="windowActionBar">false</item>
		        <item name="windowNoTitle">true</item>
		    </style>

然后在AndroidManifest.xml的application节点上添加属性：  

		android:theme="@style/AppTheme.NoActionBar"

自定义标题栏后, 效果如图：
![Toobar.PNG](https://ooo.0o0.ooo/2016/10/17/580505a311c5e.png)


### 状态栏沉浸式

#### 关于术语

* 沉浸式全屏模式  
隐藏status bar（状态栏）使屏幕全屏，让Activity接收所有的（整个屏幕的）触摸事件。

* 透明化系统状态栏  
透明化系统状态栏，使得布局侵入系统栏的后面，必须启用fitsSystemWindows属性来调整布局才不至于被系统栏覆盖。

貌似这里所谓透明化系统状态栏才是本菜想要的,不管了,现在开始一一试验,至于这概念理解的对不对,管他呢？那到底应该叫什么,那我就叫自适应状态栏,行不行？

#### 前提条件

让系统状态栏颜色随app主题颜色变化而变化这一设计,毫无疑问,也是向ios学习的:从android4.4开始引进的,并且在5.0进行了改进。因此,也只能将这一特性应用在android4.4以上的手机,无法做到全部适配。记得stormzhang(貌似是)曾说过:  

> 作为一个android程序员,还能有什么比做出ios风格的app更感到悲哀的呢?...

两种情况下的解决方案:

1. 使用toolbar  
这种方案相对简单,个人喜欢这种方案,本菜虽菜,但喜欢紧跟潮流。toolbar太好用了!

2. 不使用toolbar

#### 使用toolbar的解决方案

参照 [薄荷app的toolbar适配方案](http://stormzhang.com/android/2015/08/16/boohee-toolbar/)

其基本原理就是:
theme里添加style: <item name="android:windowTranslucentStatus"> true </item>后,包含toolbar的内容布局就可以扩展至系统状态栏,状态栏会覆盖在toolbar上,如果此时使用android:fitsSystemWindows="true",就可以调整内容布局(估计也是在根布局上加padding)恢复到原来位置.但是,上面的解决方案确是给toolbar加上一个padding-top="25dp",这样就可以做到系统状态栏的颜色和toolbar的颜色保持一致.具体方案可以参照上面的薄荷app的方案链接.  

----------
1. 引入v7包,并在布局里添加toolbar

		compile ‘com.android.support:appcompat-v7:22.2.1’

2. 在代码中设置透明化:

		if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
		 WindowManager.LayoutParams localLayoutParams = getWindow().getAttributes();
		 local LayoutParams.flags = (WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS | localLayoutParams.flags);
		}

3. 给toolbar加上padding-top,toolbar代码如下

		<android.support.v7.widget.Toolbar xmlns:android="http://schemas.android.com/apk/res/android"
		    xmlns:app="http://schemas.android.com/apk/res-auto"
		    android:id="@+id/toolbar"
		    android:layout_width="match_parent"
		    android:layout_height="25dp"
		    android:paddingTop="@dimen/toolbar_padding_top"
		    android:background="@color/primary_color"
		    app:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
		    app:popupTheme="@style/ThemeOverlay.AppCompat.Light" />

4. 其中android:paddingTop="@dimen/toolbar_padding_top"要在values中的styles文件里设为0dp,在values-v19的styles里设为25dp.

这样就可以达成了我们的目标,如果只是这样也就罢了,按照上面做就可以了,效果如图：

![GIF.gif](https://ooo.0o0.ooo/2016/10/18/5805993a360ed.gif)


不过，假如使用MD风格的DrawerLayout+NavigationView时,在android4.4的手机下,就会变这样了:

![Step3_0](http://upload-images.jianshu.io/upload_images/1839594-d8127e1d94db7806.jpg?imageMogr2/auto-orient/strip)

很明显,drawerlayout并没用被扩展至系统状态栏,但在android5.0以上效果还是可以的,这让我很奇怪,只能归咎于5.0的优化了

![Step3_1](http://upload-images.jianshu.io/upload_images/1839594-36846daa7defb203.jpg?imageMogr2/auto-orient/strip)

经过各种折腾终于想起来,可以把fitsSystemWindows的特性用在drawerlayout上试试,最后发现居然可以,最终将设置windowTranslucentStatus的代码调整如下:

#### 不使用toolbar的解决方案

不使用toolbar时,而是actionbar时,因为actionbar不好定制,所以无法采用上面那个方法,只能采用其它方法,这里的方案主要参考这里:Translucent System Bar 的最佳实践
这篇简书看的本菜晕乎乎的,仔细看下来,其实都是基于一个原理:
不管有没有actionbar,内容布局的背景颜色一律设为主题颜色,然后有actionbar的话,就将actionbar与内容布局的背景颜色同时设为主题颜色,然后,每个内容布局的根布局都要设上fitsSystemWindows="true"进行调整,感觉超麻烦有没有?

不说多少,简述步骤:

1. 在代码中设置透明化,步骤同上
2. 设置内容布局的根布局的背景颜色为主题颜色,同时设置fitsSystemWindows="true"
		
		<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
		 android:layout_width="match_parent"
		 android:layout_height="match_parent"
		 android:background="@color/colorPrimary"
		 android:fitsSystemWindows="true" 
		 android:orientation="vertical">

3. 在内容布局的下面再设置一层内容布局,设背景颜色为白色(或其它颜色):

		<LinearLayout    android:layout_width="match_parent"
		 android:layout_height="match_parent"
		 android:background="@color/c_light_white"
		 android:gravity="center"
		 android:orientation="vertical">

这种方案也完成了,看下效果:

![Step3_2](http://upload-images.jianshu.io/upload_images/1839594-dfdfbdf0c4a7489d.jpg?imageMogr2/auto-orient/strip)

可以看出,这种方案一般情况下,还是可行的,但是有三个问题:

1. 如果用上drawerlayout+navigationview,actionbar就会覆盖在侧边栏上(如上图),暂时未找到解决方案,但是我想说你都用drawerlayout+navigationview了,为何不用toolbar,因此这个问题应该不是问题,况且还可以使用其它的侧边栏实现方式,各位道友可以试试

2. 这种方案在每个根布局上都要设fitsSystemWindows="true"进行调整,当然上面也有优化方案,可仍然觉得很麻烦,

3. 每个根布局里都要多加的一层布局来覆盖根布局的背景主题颜色

**因此,这种方案的确不是上上之选.**

### 总结

* 本文主要在考虑使用标题栏(actionbar/toolbar)的情况下,做出的方案,当然你也可以自定义标题栏,或者不使用标题栏;其实都可以基于上面一样的道理。

* 在状态栏透明化的前提下,调整顶部view的padding-top,来达到状态栏自适应一体化的目的。

----------  

##### 学习笔记，如有谬误，敬请指正。

