---
layout: post
title:  Android Jetpack -- Navigation
date:   2019-12-19 10:46:54
categories: Android
tags: Android Jetpack navigation
mathjax: true
---

### 前言 ###

**Navigation** 用于管理 APP 页面跳转导航，切换 fragment 更加直观，可视化界面展示 fragment 的切换流程图



> 注意：目前 Jetpack 架构组件需要运行在 Android Studio 3.2 以上

> 以下部分内容摘抄至官方文档



**Navigation** 组件功能：

1. 自动处理 Fragment 碎片事务
2. 默认处理返回键和上一步的操作
3. 动画和过渡的默认行为
4. 默认页面以及下一层的链接操作
5. 用很少的代码去实现导航界面（例如导航抽屉和底部导航）
6. 导航切换界面之间的参数传递
7. 在 Android Studio 中一个可视化的 Navigation 操作界面 

**Navigation** 组件主要包括的三个部分：

1. Navigation Graph (一种新版的XML文件)，是一个集中包含了所有 Navigation 相关的信息的一种新版的XML文件。它包括 app 中的所有界面（Activity或Fragment），以及用户可以通过 app 访问的可能路径。
2. NavHostFragment （Layout XML view），这是一个添加到布局中的特殊部件。avHostFragment 通过 navGraph 与 navigation 导航编辑器进行关联。
3. NavController(Kotlin/Java object),这是一个跟踪导航图中当前位置的对象，当您在导航图中移动时，它协调交换 navhostfragment 中的目标内容。

当你导航时，你将使用导航控制器对象，告诉它你想去哪里，或者你想在你的导航图中走什么路径。然后，导航控制器将在 NavHostframent 中显示适当的目的地。


下面已**底部导航切换 fragment** 为例子，来简单讲诉 Navigation 基础使用方法


----------
### Demo ###
> **[传送！传送！传送！](https://github.com/lian-ma/JetpackDemo)**

**所需依赖**

```java
implementation 'androidx.navigation:navigation-fragment:2.2.0-rc03'
implementation 'androidx.navigation:navigation-ui:2.2.0-rc03'
```

**相关配置**

fragment 的跳转是通过 xml 文件配置，比平常的 FragmentManager 管理 fragment 方便的多，上面的组件功能已经说的很明确啦！

- *新建三个 fragment 和一个 activity ，名字随意，oneFragment、twoFragment、threeFtagment、MainActivity*

- *创建名为 navigation 的文件夹，路径为 app > src > main > res > navigation，在 navigation 文件夹下面新建 xxx.xml 文件，以下示例代码：*
	``` java
	<?xml version="1.0" encoding="utf-8"?>
	<navigation
	    xmlns:android="http://schemas.android.com/apk/res/android"
	    xmlns:app="http://schemas.android.com/apk/res-auto"
	    xmlns:tools="http://schemas.android.com/tools"
	    android:id="@+id/nav_graph_main"
	    app:startDestination="@id/oneFragment">//startDestination 最直观的翻译“出发地”,就是设置默认的起始页
		/**
		 * name 为 fragment 路径
		 * layout 为 fragment 的 layout 文件
		 **/
	    <fragment
	        android:id="@+id/oneFragment"
	        android:name="com.smodock.jetpackdemo.fragment.OneFragment"
	        android:label="fragment_one"
	        tools:layout="@layout/fragment_one">
			/**
			 * destination 指定跳转的 fragment
			 * enterAnim 指定进入动画 
			 * exitAnim 指定退出动画
			 * popEnterAnim 指定弹入动画
			 * popExitAnim 指定弹出动画
			 **/
	        <action
	            android:id="@+id/action_oneFragment_to_twoFragment2"
	            app:destination="@id/twoFragment"
	            app:enterAnim="@anim/slide_in_right"/>
	    </fragment>
	    <fragment
	        android:id="@+id/twoFragment"
	        android:name="com.smodock.jetpackdemo.fragment.TwoFragment"
	        android:label="fragment_two"
	        tools:layout="@layout/fragment_two" >
	        <action
	            android:id="@+id/action_twoFragment_to_threeFragment"
	            app:destination="@id/threeFragment"
	            app:enterAnim="@anim/slide_in_right"/>
	    </fragment>
	    <fragment
	        android:id="@+id/threeFragment"
	        android:name="com.smodock.jetpackdemo.fragment.ThreeFragment"
	        android:label="fragment_three"
	        tools:layout="@layout/fragment_three">
	        <action
	            android:id="@+id/action_threeFragment_to_oneFragment"
	            app:destination="@id/oneFragment"
	            app:enterAnim="@anim/slide_in_right"/>

	    </fragment>
	</navigation>
	```
	**顺便附上代码中提到的从右进入的动画配置代码**
	```java
	<?xml version="1.0" encoding="utf-8"?>
	<set xmlns:android="http://schemas.android.com/apk/res/android">
	    <translate
	        android:duration="600"
	        android:fromXDelta="100%"
	        android:toXDelta="0" />
	</set>
	```
	
	> 注意：上诉代码可通过前面列出的第7条功能自动生成
	> 
	> 以下就是通过点击 `+` 把三个fragment添加进来，然后进行手动连线，最后生成上诉代码，是不是很简单明了！
	
	
	![design.png](https://i.loli.net/2019/12/24/3DKrvwlkFb5VCam.png)

- *接下来，回到咱们 activity 的 layout 文件中进行配置，以下贴出 MainActivity 的 layout 代码：*


	``` java
	<?xml version="1.0" encoding="utf-8"?>
	<layout
	    xmlns:android="http://schemas.android.com/apk/res/android"
	    xmlns:app="http://schemas.android.com/apk/res-auto"
	    xmlns:tools="http://schemas.android.com/tools">
	    <androidx.constraintlayout.widget.ConstraintLayout
	        android:layout_width="match_parent"
	        android:layout_height="match_parent"
	        tools:context=".MainActivity">
			/**
			 * id 必须要有
			 * name 固定填写 NavHostFragment，类似打个标签，指定他是 NavHostFragment
			 * defaultNavHost 是否拦截返回键，true 则帮你维护页面栈，按返回键直接返回上一个页面，false 则直接退出当前 activity 
			 * navGraph 指定上面咱们创建的 navigation 导航文件
			 **/
	        <fragment
	            android:id="@+id/nav_host_fragment"
	            android:name="androidx.navigation.fragment.NavHostFragment"
	            android:layout_width="0dp"
	            android:layout_height="0dp"
	            app:defaultNavHost="true"
	            app:layout_constraintBottom_toBottomOf="parent"
	            app:layout_constraintLeft_toLeftOf="parent"
	            app:layout_constraintRight_toRightOf="parent"
	            app:layout_constraintTop_toTopOf="parent"
	            app:navGraph="@navigation/nav_graph_main" />
	    </androidx.constraintlayout.widget.ConstraintLayout>
	</layout>
	```

- *xml都配置好啦！下面回到 fragment 一句话跳转 fragment，下面贴出两种跳转方式的代码：*

	``` java
	/**
	 *R.id.action_oneFragment_to_twoFragment2 是上诉 navigation 导航配置文件内 action 标签的id
	 **/
	//第一种
	binding.oneTv.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View view) {
            Navigation.findNavController(view).navigate(R.id.action_oneFragment_to_twoFragment2);
        }
     });
	//第二种
	binding.oneTv.setOnClickListener(Navigation.createNavigateOnClickListener(R.id.action_oneFragment_to_twoFragment2));
	```
	*createNavigateOnClickListener 点进去查看源码，发现他也只是做了和第一种方法一样的操作，以下为此方法源码：*

	``` java
    public static View.OnClickListener createNavigateOnClickListener(@IdRes final int resId,
            @Nullable final Bundle args) {
        return new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                findNavController(view).navigate(resId, args);
            }
        };
    }
	```
- *三个 fragment 都同上，那么怎么传值呐，下面介绍两种传值方法*
	
	1.传统的传值方式 **Bundle**，createNavigateOnClickListener 与 navigate 都提供了接收 bundle 的重载方法。
	
	```java
	//发送方
	Bundle bundle = new Bundle();
    bundle.putString("message", "我是传过来的值");
    Navigation.findNavController(view).navigate(R.id.action_oneFragment_to_twoFragment2, bundle);
	//binding.oneTv.setOnClickListener(Navigation.createNavigateOnClickListener(R.id.action_oneFragment_to_twoFragment2, bundle));
	// 接收方
	Bundle bundle = getArguments();
    if (bundle != null) {
        String str = bundle.getString("message", "没有传值");
        binding.twoMessageTv.setText(str);
    }
	```
	2.navigation 官方提供的传值方法
	- 在工程的 build.gradle 文件内配置如下：
	```java
	dependencies {
		classpath "android.arch.navigation:navigation-safe-args-gradle-plugin:1.0.0-rc02"
    }
	```
	- 在 Moudle 下的 build.gradle 文件内配置如下：
	```java
	apply plugin: 'androidx.navigation.safeargs'
	```

	- 传统的传值方式需要知道key,而 navigation 会根据导航文件的参数配置生成规则为：`fragment类名 + Args` 的类文件，以下附上 navigation 导航配置文件加了 argument 标签的部分代码 argument 标签在 fragment 标签内。
	
		```java
		<fragment
		    android:id="@+id/oneFragment"
		    android:name="com.smodock.jetpackdemo.fragment.OneFragment"
		    android:label="fragment_one"
		    tools:layout="@layout/fragment_one">
		    <action
		        android:id="@+id/action_oneFragment_to_twoFragment2"
		        app:destination="@id/twoFragment"
		        app:enterAnim="@anim/slide_in_right"/>
		    <argument
		        android:name="message"
		        app:argType="string"
		        android:defaultValue="我是默认值" />
		</fragment>
		```
	- fragment 中 通过初始化 `xxxArgs.Builder` set 上面配置的 argument 的 name，Buider 内部静态类里面通过映射上面配置文件自动生成对应 `name` 字段内容的 `get 与 set` 方法，以下附上传值与取值的代码：
	
		```java
		//发送方
		OneFragmentArgs.Builder builder = new OneFragmentArgs.Builder();
	    Bundle bundle = builder.setMessage("我是另一种方法转过来的值").build().toBundle();
	    Navigation.findNavController(view).navigate(R.id.action_oneFragment_to_twoFragment2, bundle);
	
		//接收方
		Bundle bundle = getArguments();
	    if (bundle != null) {
	        String str = OneFragmentArgs.fromBundle(bundle).getMessage();
	        binding.twoMessageTv.setText(str);
	    }
		```

- *现在基本的跳转 fragment 大家都会啦，那回到上面提到的 Navigation 组件功能的第5条，用底部导航为例子，毕竟在日常需求中底部导航基本遍布大部分的 APP 内，官方提出了 `BottomNavigationView` 与 `NavigationUI` 搭配的一款超简洁的底部导航栏。*
	
	1. 添加 `BottomNavigationView` 的依赖
	```java
	implementation 'com.google.android.material:material:1.2.0-alpha02'
	```
	2. 如 navigation 需要创建导航文件， `BottomNavigationView` 需要创建菜单文件，创建名为 menu 的文件夹，路径为 app > src > main > res > menu，在 navigation 文件夹下面新建 xxx.xml 文件，以下示例代码：
	
		```java
		<?xml version="1.0" encoding="utf-8"?>
		<menu xmlns:android="http://schemas.android.com/apk/res/android">
			/**
			 * id 为 navigation 导航文件内 fragment 标签设置的id
			 * icon 为任意 icon 图片
			 * title 为导航栏 icon 下面的文字
			 **/
		    <item
		        android:id="@id/oneFragment"
		        android:icon="@drawable/ic_launcher_background"
		        android:title="One" />
		    <item
		        android:id="@id/twoFragment"
		        android:icon="@drawable/ic_launcher_background"
		        android:title="Two" />
		    <item
		        android:id="@id/threeFragment"
		        android:icon="@drawable/ic_launcher_background"
		        android:title="Three" />
		</menu>
		```

	3. activity 的 layout 文件内添加 `BottomNavigationView` 的使用，在原基础下添加如下代码：
	
		```java
		<fragment
	        android:id="@+id/nav_host_fragment"
	        android:name="androidx.navigation.fragment.NavHostFragment"
	        android:layout_width="0dp"
	        android:layout_height="0dp"
	        app:defaultNavHost="false"
	        app:layout_constraintBottom_toBottomOf="parent"
	        app:layout_constraintLeft_toLeftOf="parent"
	        app:layout_constraintRight_toRightOf="parent"
	        app:layout_constraintTop_toTopOf="parent"
	        app:navGraph="@navigation/nav_graph_main" />
		//以下为新添加的代码
		//menu 与上面我们新建的绑定
	    <com.google.android.material.bottomnavigation.BottomNavigationView
	        android:id="@+id/bottom_nav_view"
	        android:layout_width="match_parent"
	        android:layout_height="wrap_content"
	        app:menu="@menu/bottom_nav_menu"
	        app:layout_constraintBottom_toBottomOf="parent"
	        android:background="#ffffff"
	        app:itemTextColor="@color/colorAccent"/>
		```
	
	4. 接下来 activity 内已非常简洁的代码将 `NavController` 与 `BottomNavigationView` 绑定
	
		```java
		NavController navController = Navigation.findNavController(this, R.id.nav_host_fragment);
	    NavigationUI.setupWithNavController(binding.bottomNavView, navController);
		```
	**效果图如下**
	
		![navigation.png](https://i.loli.net/2019/12/24/ODX4il1GZMS8jL2.png)

	
### Happy Ending！ ###

### 总结 ###

通过这个小Demo 个人感觉 Navigation 还是填补了原生 fragment 管理上的缺失，使跳转与回退变得非常简洁，但是用到底部导航后发现，通过底部tab切换 fragment 除了初始页面，其他页面是会每次重新创建，需结合ViewModel 方能达到最佳效果。















