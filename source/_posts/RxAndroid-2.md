---
title: RxJava 学习笔记 2
date: 2016-06-20 15:32:31
tags: 
	- Android
categories: Android
---

### RxJava 的适用场景和使用方式
#### 1、与 Retrofit 的结合
Retrofit 是 Square 的一个著名的网络请求库。没有用过 Retrofit 的可以选择跳过这一小节也没关系，我举的每种场景都只是个例子，而且例子之间并无前后关联，只是个抛砖引玉的作用，所以你跳过这里看别的场景也可以的。
<!-- more -->
Retrofit 除了提供了传统的 `Callback` 形式的 API，还有 RxJava 版本的 `Observable` 形式 API。下面我用对比的方式来介绍 Retrofit 的 RxJava 版 API 和传统版本的区别。

以获取一个 User 对象的接口作为例子。使用Retrofit 的传统 API，你可以用这样的方式来定义请求：
``` java
	@FormUrlEncoded
    @POST("web/app.php/funmi/user")
    void getUser(@Field("userName") String userName,
               @Field("password") String password, @Field("stype") int stype,
               Callback<User> callback);
```

在程序的构建过程中， Retrofit 会把自动把方法实现并生成代码，然后开发者就可以利用下面的方法来获取特定用户并处理响应：
``` java
	getUser(userName,password,stype, new Callback<User>() {
	    @Override
	    public void success(User user) {
	        userView.setUser(user);
	    }

	    @Override
	    public void failure(RetrofitError error) {
	        // Error handling
	        ...
	    }
	};
```

而使用 RxJava 形式的 API，定义同样的请求是这样的：
``` java
	@FormUrlEncoded
    @POST("web/app.php/funmi/user")
    Observable<BaseEntity<User>> getUser(@Field("userName") String userName,
                                              @Field("password") String password, @Field("stype") int stype);
```
使用的时候是这样的：
``` java
	getUser(userName,password,stype)
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(new Observer<User>() {
	        @Override
	        public void onNext(User user) {
	            userView.setUser(user);
	        }
	
	        @Override
	        public void onCompleted() {
	        }
	
	        @Override
	        public void onError(Throwable error) {
	            // Error handling
	            ...
	        }
    });
```
再举一个例子：假设 `/user` 接口并不能直接访问，而需要填入一个在线获取的 `token` ，代码应该怎么写？

`Callback` 方式，可以使用嵌套的 `Callback`：
``` java
	@GET("web/app.php/funmi/token")
	public void getToken(Callback<String> callback);

	@GET("web/app.php/funmi/user")
	public void getUser(@Query("token") String token, @Query("userId") String userId, Callback<User> callback);

	...

	getToken(new Callback<String>() {
	    @Override
	    public void success(String token) {
	        getUser(token, userId, new Callback<User>() {
		            @Override
		            public void success(User user) {
		                userView.setUser(user);
		            }
		
		            @Override
		            public void failure(RetrofitError error) {
		                // Error handling
		                ...
		            }
		        };
		    }
	
		    @Override
		    public void failure(RetrofitError error) {
		        // Error handling
		        ...
		    }
	});
```
而使用 RxJava 的话，代码是这样的：
``` java
	@GET("web/app.php/funmi/token")
	public Observable<String> getToken();

	@GET("web/app.php/funmi/user")
	public Observable<User> getUser(@Query("token") String token, @Query("userId") String userId);

	...

	getToken()
	    .flatMap(new Func1<String, Observable<User>>() {
	        @Override
	        public Observable<User> onNext(String token) {
	            return getUser(token, userId);
	        })
	    .observeOn(AndroidSchedulers.mainThread())
	    .subscribe(new Observer<User>() {
	        @Override
	        public void onNext(User user) {
	            userView.setUser(user);
	        }
	
	        @Override
	        public void onCompleted() {
	        }
	
	        @Override
	        public void onError(Throwable error) {
	            // Error handling
	            ...
	        }
    });
```
用一个 `flatMap()` 就搞定了逻辑，依然是一条链。看着就很爽，是吧？

#### 2、 RxBinding
[RxBinding](https://github.com/JakeWharton/RxBinding) 是 Jake Wharton 的一个开源库，它提供了一套在 Android 平台上的基于 RxJava 的 Binding API。所谓 Binding，就是类似设置 `OnClickListener` 、设置 `TextWatcher` 这样的注册绑定对象的 API。

举个设置点击监听的例子。使用 `RxBinding` ，可以把事件监听用这样的方法来设置：
``` java
	Button button = ...;
	RxView.clickEvents(button) // 以 Observable 形式来反馈点击事件
	    .subscribe(new Action1<ViewClickEvent>() {
	        @Override
	        public void call(ViewClickEvent event) {
	            // Click handling
	        }
    });
```
看起来除了形式变了没什么区别，实质上也是这样。甚至如果你看一下它的源码，你会发现它连实现都没什么惊喜：它的内部是直接用一个包裹着的 `setOnClickListener()` 来实现的。然而，仅仅这一个形式的改变，却恰好就是 `RxBinding` 的目的：扩展性。通过 `RxBinding` 把点击监听转换成 `Observable` 之后，就有了对它进行扩展的可能。扩展的方式有很多，根据需求而定。一个例子是前面提到过的 `throttleFirst()` ，用于去抖动，也就是消除手抖导致的快速连环点击：
``` java
	RxView.clickEvents(button)
    .throttleFirst(500, TimeUnit.MILLISECONDS)
    .subscribe(clickAction);
```
#### 3、各种异步操作
前面举的 `Retrofit` 和 `RxBinding` 的例子，是两个可以提供现成的 `Observable` 的库。而如果你有某些异步操作无法用这些库来自动生成 `Observable`，也完全可以自己写。例如数据库的读写、大图片的载入、文件压缩/解压等各种需要放在后台工作的耗时操作，都可以用 `RxJava` 来实现，有了之前几章的例子，这里应该不用再举例了。

#### 4、RxBus
`RxBus` 名字看起来像一个库，但它并不是一个库，而是一种模式，它的思想是使用 `RxJava` 来实现了 `EventBus` ，而让你不再需要使用 `Otto` 或者 GreenRobot 的 `EventBus`。至于什么是 RxBus，可以看这篇文章。顺便说一句，Flipboard 已经用 `RxBus` 替换掉了 `Otto` ，目前为止没有不良反应。


#### 部分内容摘自 [扔物线](http://gank.io/post/560e15be2dca930e00da1083)