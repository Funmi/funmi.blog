---
title: RxJava 学习笔记 1
date: 2016-06-07 15:40:53
tags: 
   - Android
categories: Android
---

### RxJava 是什么？
我的理解就是观察者模式，RxJava 最核心的东西就是 **Observable** （`被观察者`） 和 **Observer** (`观察者`) 。

**Observable** （`被观察者`） 会发出数据，而与之相对的 **Observer** (`观察者`) 则会通过订阅**Observable**（`被观察者`） 来进行观察。

**Observer**可以在**Observable**发出数据、报错或者声明没有数据可以发送时进行相应的操作。这三个操作被封装在**Observer**接口中，相应的方法为`onNext()`，`onError()`和`onCompleted()`。

### RxJava的优点
所有的逻辑业务都封装成一条链式结构，使其看起来很 **简洁**。
<!-- more -->
				
  ![](http://ww4.sinaimg.cn/large/52eb2279jw1f2rx409pcnj2044048mx5.jpg)

#### 加载Assets文件夹下的图片

	private String[] imgPath = null;

    private void loadImage() {

        try {
            //这里注意  只能返回  Assets 下某一个文件夹下的所有文件，不能返回 Assets下所有文件
            imgPath = getAssets().list("ssdk");
        } catch (IOException e) {
            e.printStackTrace();
        }
        if (null == imgPath && imgPath.length == 0) {
            Log.e(tag, "图片地址  为空");
            return;
        }
        new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < imgPath.length; i++) {
                    //一个坑，要把文件夹的路径也加上
                    InputStream is = null;
                    Bitmap bitmap = null;
                    try {
                        is = getAssets().open("ssdk/" + imgPath[i]);
                        bitmap = BitmapFactory.decodeStream(is);
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                    final ImageView imageView = new ImageView(mContext);
                    // 设置当前图像的图像（position为当前图像列表的位置）
                    imageView.setLayoutParams(new LinearLayout.LayoutParams(
                            200, 200));
                    // 设置Gallery组件的背景风格
                    imageView.setImageBitmap(bitmap);
                    runOnUiThread(new Runnable() {
                        @Override
                        public void run() {
                            rootView.addView(imageView);
                        }
                    });

                }
            }
        }).start();

    }

#### 加载Assets文件夹下的图片 使用 RxJava
	private void loadImage() {
        String[] imgUrl = null;
        try {
            //这里注意  只能返回  Assets 下某一个文件夹下的所有文件，不能返回 Assets下所有文件
            imgUrl = getAssets().list("ssdk");
        } catch (IOException e) {
            e.printStackTrace();
        }
        if (null == imgUrl && imgUrl.length == 0) {
            Log.e(tag, "图片地址  为空");
            return;
        }

        Observable.from(imgUrl)
                .map(new Func1<String, Bitmap>() {
                    @Override
                    public Bitmap call(String s) {
                        Bitmap bitmap = null;
                        try {
                            //一个坑，要把文件夹的路径也加上
                            InputStream is = getAssets().open("ssdk/" + s);
                            bitmap = BitmapFactory.decodeStream(is);
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                        if (null == bitmap) {
                            return null;
                        } else {
                            return bitmap;
                        }
                    }
                })
                .subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Observer<Bitmap>() {
                    @Override
                    public void onCompleted() {

                    }

                    @Override
                    public void onError(Throwable e) {

                    }

                    @Override
                    public void onNext(Bitmap bitmap) {
                        if (null == bitmap) {
                            Log.e(tag, "bitmap为空");
                            return;
                        }
                        ImageView imageView = new ImageView(mContext);
                        // 设置当前图像的图像（position为当前图像列表的位置）
                        imageView.setLayoutParams(new LinearLayout.LayoutParams(
                                200, 200));
                        // 设置Gallery组件的背景风格
                        imageView.setImageBitmap(bitmap);
                        rootView.addView(imageView);
                    }
                });
    }

看起来使用RxJava 代码量还变多了，但是逻辑更加简洁了，从上到下，一步一步，逻辑非常明确，没有嵌套，一眼明了。

#### RxJava 基本实现
##### 创建 Observer (观察者)
`Observer` 即观察者，它决定事件触发的时候将有怎样的行为。 RxJava 中的 `Observer` 接口的实现方式：

	Observer<String> observer = new Observer<String>() {
	    @Override
	    public void onNext(String s) {
	        Log.d(tag, "Item: " + s);
	    }
	
	    @Override
	    public void onCompleted() {
	        Log.d(tag, "Completed!");
	    }
	
	    @Override
	    public void onError(Throwable e) {
	        Log.d(tag, "Error!");
	    }
	};

除了 `Observer` 接口之外，RxJava 还内置了一个实现了` Observer` 的抽象类：`Subscriber`。 `Subscriber` 对 `Observer` 接口进行了一些扩展，但他们的基本使用方式是完全一样的：

	Subscriber<String> subscriber = new Subscriber<String>() {
	    @Override
	    public void onNext(String s) {
	        Log.d(tag, "Item: " + s);
	    }
	
	    @Override
	    public void onCompleted() {
	        Log.d(tag, "Completed!");
	    }
	
	    @Override
	    public void onError(Throwable e) {
	        Log.d(tag, "Error!");
	    }
	};

不仅基本使用方式一样，实质上，在 RxJava 的 `subscribe` 过程中，`Observer` 也总是会先被转换成一个 `Subscriber` 再使用。所以如果你只想使用基本功能，选择 `Observer` 和 `Subscriber` 是完全一样的。它们的区别对于使用者来说主要有两点：

1. `onStart()`: 这是 `Subscriber` 增加的方法。它会在 `subscribe` 刚开始，而事件还未发送之前被调用，可以用于做一些准备工作，例如数据的清零或重置。这是一个可选方法，默认情况下它的实现为空。需要注意的是，如果对准备工作的线程有要求（例如弹出一个显示进度的对话框，这必须在主线程执行）， `onStart()` 就不适用了，因为它总是在 `subscribe` 所发生的线程被调用，而不能指定线程。要在指定的线程来做准备工作，可以使用 `doOnSubscribe()` 方法，具体可以在后面的文中看到。
2. `unsubscribe()`: 这是 `Subscriber` 所实现的另一个接口 `Subscription` 的方法，用于取消订阅。在这个方法被调用后，`Subscriber` 将不再接收事件。一般在这个方法调用前，可以使用 `isUnsubscribed()` 先判断一下状态。 `unsubscribe()` 这个方法很重要，因为在 `subscribe()` 之后， `Observable` 会持有 `Subscriber` 的引用，这个引用如果不能及时被释放，将有内存泄露的风险。所以最好保持一个原则：要在不再使用的时候尽快在合适的地方（例如 `onPause()` `onStop()` 等方法中）调用 `unsubscribe()` 来解除引用关系，以避免内存泄露的发生。

##### 创建 Observable (被观察者)

`Observable` 即被观察者，它决定什么时候触发事件以及触发怎样的事件。 RxJava 使用 `create()` 方法来创建一个 `Observable` ，并为它定义事件触发规则：

	Observable observable = Observable.create(new Observable.OnSubscribe<String>() {
	    @Override
	    public void call(Subscriber<? super String> subscriber) {
	        subscriber.onNext("Hello");
	        subscriber.onNext("Hi");
	        subscriber.onNext("Aloha");
	        subscriber.onCompleted();
	    }
	});

可以看到，这里传入了一个 `OnSubscribe` 对象作为参数。`OnSubscribe` 会被存储在返回的 `Observable` 对象中，它的作用相当于一个计划表，当 `Observable` 被订阅的时候，`OnSubscribe` 的 `call()` 方法会自动被调用，事件序列就会依照设定依次触发（对于上面的代码，就是观察者`Subscriber` 将会被调用三次 `onNext()` 和一次 `onCompleted()`）。这样，由被观察者调用了观察者的回调方法，就实现了由被观察者向观察者的事件传递，即观察者模式。

`create()` 方法是 RxJava 最基本的创造事件序列的方法。基于这个方法， RxJava 还提供了一些方法用来快捷创建事件队列，例如：

* `just(T...)`: 将传入的参数依次发送出来。

		Observable observable = Observable.just("Hello", "Hi", "Aloha");
		// 将会依次调用：
		// onNext("Hello");
		// onNext("Hi");
		// onNext("Aloha");
		// onCompleted();

* `from(T[])` / `from(Iterable<? extends T>)` : 将传入的数组或 `Iterable` 拆分成具体对象后，依次发送出来。

		String[] words = {"Hello", "Hi", "Aloha"};
		Observable observable = Observable.from(words);
		// 将会依次调用：
		// onNext("Hello");
		// onNext("Hi");
		// onNext("Aloha");
		// onCompleted();

上面 `just(T...)` 的例子和 `from(T[])` 的例子，都和之前的 `create(OnSubscribe)` 的例子是等价的。

#### Subscribe (订阅)
创建了 `Observable` 和 `Observer` 之后，再用 `subscribe()` 方法将它们联结起来，整条链子就可以工作了。代码形式很简单：

		observable.subscribe(observer);
		// 或者：
		observable.subscribe(subscriber);

> 有人可能会注意到， `subscribe()` 这个方法有点怪：它看起来是『`observalbe` 订阅了` observer` / `subscriber`』而不是『`observer` / `subscriber` 订阅了 `observalbe`』，这看起来就像『杂志订阅了读者』一样颠倒了对象关系。这让人读起来有点别扭，不过如果把 API 设计成 `observer.subscribe(observable)` / `subscriber.subscribe(observable)` ，虽然更加符合思维逻辑，但对流式 API 的设计就造成影响了，比较起来明显是得不偿失的。

#### 例子
a. 打印字符串数组

		String[] names = ...;
		Observable.from(names)
    	.subscribe(new Action1<String>() {
        @Override
        public void call(String name) {
            Log.d(tag, name);
        }
   		});

b. 由 id 取得图片并显示

	int drawableRes = ...;
	ImageView imageView = ...;
	Observable.create(new OnSubscribe<Drawable>() {
	    @Override
	    public void call(Subscriber<? super Drawable> subscriber) {
	        Drawable drawable = getTheme().getDrawable(drawableRes));
	        subscriber.onNext(drawable);
	        subscriber.onCompleted();
	    }
		}).subscribe(new Observer<Drawable>() {
	    @Override
	    public void onNext(Drawable drawable) {
	        imageView.setImageDrawable(drawable);
	    }
	
	    @Override
	    public void onCompleted() {
	    }
	
	    @Override
	    public void onError(Throwable e) {
	        Toast.makeText(activity, "Error!", Toast.LENGTH_SHORT).show();
	    }
	});

正如上面两个例子这样，创建出 `Observable` 和 `Subscriber` ，再用 `subscribe()` 将它们串起来，一次 RxJava 的基本使用就完成了。非常简单。
然而，

![](http://ww3.sinaimg.cn/mw1024/52eb2279jw1f2rx4ddcncj2046053gll.jpg)

在 RxJava 的默认规则中，事件的发出和消费都是在同一个线程的。也就是说，如果只用上面的方法，实现出来的只是一个同步的观察者模式。观察者模式本身的目的就是『后台处理，前台回调』的异步机制，因此异步对于 RxJava 是至关重要的。而要实现异步，则需要用到 RxJava 的另一个概念：` Scheduler` 。

#### 线程控制 —— Scheduler

#####  Scheduler 的 API 

在RxJava 中，`Scheduler` ——调度器，相当于线程控制器，RxJava 通过它来指定每一段代码应该运行在什么样的线程。RxJava 已经内置了几个 `Scheduler` ，它们已经适合大多数的使用场景：

* `Schedulers.immediate()`: 直接在当前线程运行，相当于不指定线程。这是默认的 `Scheduler`。
* `Schedulers.newThread()`: 总是启用新线程，并在新线程执行操作。
* `Schedulers.io()`: I/O 操作（读写文件、读写数据库、网络信息交互等）所使用的 Scheduler。行为模式和 `newThread()` 差不多，区别在于 `io()` 的内部实现是是用一个无数量上限的线程池，可以重用空闲的线程，因此多数情况下 `io()` 比 `newThread()` 更有效率。不要把计算工作放在 `io()` 中，可以避免创建不必要的线程。
* `Schedulers.computation()`: 计算所使用的 `Scheduler`。这个计算指的是 CPU 密集型计算，即不会被 I/O 等操作限制性能的操作，例如图形的计算。这个 `Scheduler` 使用的固定的线程池，大小为 CPU 核数。不要把 I/O 操作放在 `computation()` 中，否则 I/O 操作的等待时间会浪费 CPU。
* 另外， Android 还有一个专用的 `AndroidSchedulers.mainThread()`，它指定的操作将在 Android 主线程运行。

有了这几个 `Scheduler` ，就可以使用 `subscribeOn()` 和 `observeOn()` 两个方法来对线程进行控制了。 `subscribeOn()`: 指定 `subscribe()` 所发生的线程，即 `Observable.OnSubscribe` 被激活时所处的线程。或者叫做事件产生的线程。 ` observeOn()`: 指定 `Subscriber` 所运行在的线程。或者叫做事件消费的线程。

文字叙述总归难理解，上代码：

	Observable.just(1, 2, 3, 4)
	    .subscribeOn(Schedulers.io()) // 指定 subscribe() 发生在 IO 线程
	    .observeOn(AndroidSchedulers.mainThread()) // 指定 Subscriber 的回调发生在主线程
	    .subscribe(new Action1<Integer>() {
	        @Override
	        public void call(Integer number) {
	            Log.d(tag, "number:" + number);
	        }
	    });

上面这段代码中，由于 `subscribeOn(Schedulers.io())` 的指定，被创建的事件的内容 1、2、3、4 将会在 IO 线程发出；而由于 `observeOn(AndroidScheculers.mainThread())` 的指定，因此 `subscriber` 数字的打印将发生在主线程 。事实上，这种在 `subscribe()` 之前写上两句 `subscribeOn(Scheduler.io())` 和 `observeOn(AndroidSchedulers.mainThread())` 的使用方式非常常见，它适用于多数的 『后台线程取数据，主线程显示』的程序策略。

而前面提到的由图片 id 取得图片并显示的例子，如果也加上这两句：

		int drawableRes = ...;
		ImageView imageView = ...;
		Observable.create(new OnSubscribe<Drawable>() {
		    @Override
		    public void call(Subscriber<? super Drawable> subscriber) {
		        Drawable drawable = getTheme().getDrawable(drawableRes));
		        subscriber.onNext(drawable);
		        subscriber.onCompleted();
		    }
		})
		.subscribeOn(Schedulers.io()) // 指定 subscribe() 发生在 IO 线程
		.observeOn(AndroidSchedulers.mainThread()) // 指定 Subscriber 的回调发生在主线程
		.subscribe(new Observer<Drawable>() {
		    @Override
		    public void onNext(Drawable drawable) {
		        imageView.setImageDrawable(drawable);
		    }
		
		    @Override
		    public void onCompleted() {
		    }
		
		    @Override
		    public void onError(Throwable e) {
		        Toast.makeText(activity, "Error!", Toast.LENGTH_SHORT).show();
		    }
	});

那么，加载图片将会发生在 IO 线程，而设置图片则被设定在了主线程。这就意味着，即使加载图片耗费了几十甚至几百毫秒的时间，也不会造成丝毫界面的卡顿。

#### 变换

终于要到牛逼的地方了，不管你激动不激动，反正我是激动了。

RxJava 提供了对事件序列进行变换的支持，这是它的核心功能之一，也是大多数人说『RxJava 真是太好用了』的最大原因。所谓变换，就是将事件序列中的对象或整个序列进行加工处理，转换成不同的事件或事件序列。概念说着总是模糊难懂的，来看 API。

##### (1) `map()`

首先看一个 `map()` 的例子：

	Observable.just("images/logo.png") // 输入类型 String
    .map(new Func1<String, Bitmap>() {
        @Override
        public Bitmap call(String filePath) { // 参数类型 String
            return getBitmapFromPath(filePath); // 返回类型 Bitmap
        }
    })
    .subscribe(new Action1<Bitmap>() {
        @Override
        public void call(Bitmap bitmap) { // 参数类型 Bitmap
            showBitmap(bitmap);
        }
    });

这里出现了一个叫做 `Func1` 的类。它和 `Action1` 非常相似，也是 RxJava 的一个接口，用于包装含有一个参数的方法。 `Func1` 和 `Action` 的区别在于， `Func1` 包装的是有返回值的方法。另外，和 `ActionX` 一样， `FuncX` 也有多个，用于不同参数个数的方法。`FuncX` 和 `ActionX` 的区别在 `FuncX` 包装的是有返回值的方法。

可以看到，`map()` 方法将参数中的 `String` 对象转换成一个 `Bitmap` 对象后返回，而在经过 `map()` 方法后，事件的参数类型也由 `String` 转为了 `Bitmap`。这种直接变换对象并返回的，是最常见的也最容易理解的变换。不过 RxJava 的变换远不止这样，它不仅可以针对事件对象，还可以针对整个事件队列，这使得 RxJava 变得非常灵活。我列举几个常用的变换：

* `map()`: 事件对象的直接变换，具体功能上面已经介绍过。它是 RxJava 最常用的变换。 `map()` 的示意图：
![](http://ww1.sinaimg.cn/mw1024/52eb2279jw1f2rx4fitvfj20hw0ea0tg.jpg)

* `flatMap()`:这是一个很有用但 **非常难理解** 的变换，因此我决定花多些篇幅来介绍它。 首先假设这么一种需求：假设有一个数据结构『学生』，现在需要打印出一组学生的名字。实现方式很简单：

		Student[] students = ...;
		Subscriber<String> subscriber = new Subscriber<String>() {
		    @Override
		    public void onNext(String name) {
		        Log.d(tag, name);
		    }
		    ...
		};
		Observable.from(students)
	    .map(new Func1<Student, String>() {
	        @Override
	        public String call(Student student) {
	            return student.getName();
	        }
	    })
	    .subscribe(subscriber);
		
很简单。那么再假设：如果要打印出每个学生所需要修的所有课程的名称呢？（需求的区别在于，每个学生只有一个名字，但却有多个课程。）首先可以这样实现：

	Student[] students = ...;
	Subscriber<Student> subscriber = new Subscriber<Student>() {
	    @Override
	    public void onNext(Student student) {
	        List<Course> courses = student.getCourses();
	        for (int i = 0; i < courses.size(); i++) {
	            Course course = courses.get(i);
	            Log.d(tag, course.getName());
	        }
	    }
	    ...
	};
	Observable.from(students)
    .subscribe(subscriber);

依然很简单。那么如果我不想在 `Subscriber` 中使用 for 循环，而是希望 `Subscriber` 中直接传入单个的 `Course` 对象呢（这对于代码复用很重要）？用 `map()` 显然是不行的，因为 `map()` 是一对一的转化，而我现在的要求是一对多的转化。那怎么才能把一个 `Student` 转化成多个 `Course` 呢？

这个时候，就需要用 `flatMap()` 了：

	Student[] students = ...;
	Subscriber<Course> subscriber = new Subscriber<Course>() {
	    @Override
	    public void onNext(Course course) {
	        Log.d(tag, course.getName());
	    }
	    ...
	};
	Observable.from(students)
    .flatMap(new Func1<Student, Observable<Course>>() {
        @Override
        public Observable<Course> call(Student student) {
            return Observable.from(student.getCourses());
        }
    })
    .subscribe(subscriber);

从上面的代码可以看出， `flatMap()` 和 `map()` 有一个相同点：它也是把传入的参数转化之后返回另一个对象。但需要注意，和 `map()` 不同的是， `flatMap()` 中返回的是个 `Observable` 对象，并且这个 `Observable` 对象并不是被直接发送到了 `Subscriber` 的回调方法中。 `flatMap(` 的原理是这样的：

1. 使用传入的事件对象创建一个 `Observable` 对象；
2.  并不发送这个 `Observable`, 而是将它激活，于是它开始发送事件；
3.  每一个创建出来的 `Observable` 发送的事件，都被汇入同一个 `Observable` ，而这个 `Observable` 负责将这些事件统一交给 Subscriber 的回调方法。这三个步骤，把事件拆成了两级，通过一组新创建的 `Observable` 将初始的对象『铺平』之后通过统一路径分发了下去。而这个『铺平』就是 `flatMap`() 所谓的 `flat`。

`flatMap()` 示意图：

![](http://ww1.sinaimg.cn/mw1024/52eb2279jw1f2rx4i8da2j20hg0dydgx.jpg)

##### 线程控制：Scheduler
除了灵活的变换，RxJava 另一个牛逼的地方，就是线程的自由控制。

因为 `observeOn()` 指定的是 `Subscriber` 的线程，而这个 `Subscriber` 并不是（严格说应该为『不一定是』，但这里不妨理解为『不是』）`subscribe()` 参数中的 `Subscriber` ，而是 `observeOn()` 执行时的当前 `Observable` 所对应的 `Subscriber` ，即它的直接下级 `Subscriber` 。换句话说，`observeOn()` 指定的是它之后的操作所在的线程。因此如果有多次切换线程的需求，只要在每个想要切换线程的位置调用一次 `observeOn()` 即可。上代码：

	Observable.just(1, 2, 3, 4) // IO 线程，由 subscribeOn() 指定
	    .subscribeOn(Schedulers.io())
	    .observeOn(Schedulers.newThread())
	    .map(mapOperator) // 新线程，由 observeOn() 指定
	    .observeOn(Schedulers.io())
	    .map(mapOperator2) // IO 线程，由 observeOn() 指定
	    .observeOn(AndroidSchedulers.mainThread) 
	    .subscribe(subscriber);  // Android 主线程，由 observeOn() 指定

如上，通过` observeOn()` 的多次调用，程序实现了线程的多次切换。

不过，不同于 `observeOn()` ， `subscribeOn()` 的位置放在哪里都可以，但它是只能调用一次的。

##### 延伸：doOnSubscribe()
然而，虽然超过一个的 `subscribeOn()` 对事件处理的流程没有影响，但在流程之前却是可以利用的。

在前面讲 `Subscriber` 的时候，提到过 `Subscriber` 的 `onStart()` 可以用作流程开始前的初始化。然而 `onStart()` 由于在 `subscribe()` 发生时就被调用了，因此不能指定线程，而是只能执行在 `subscribe()` 被调用时的线程。这就导致如果 `onStart()` 中含有对线程有要求的代码（例如在界面上显示一个 `ProgressBar`，这必须在主线程执行），将会有线程非法的风险，因为有时你无法预测 `subscribe()` 将会在什么线程执行。

而与 `Subscriber.onStart()` 相对应的，有一个方法 `Observable.doOnSubscribe()` 。它和 `Subscriber.onStart()` 同样是在 `subscribe()` 调用后而且在事件发送前执行，但区别在于它可以指定线程。默认情况下， `doOnSubscribe()` 执行在 `subscribe() `发生的线程；而如果在 `doOnSubscribe()` 之后有 `subscribeOn()` 的话，它将执行在离它最近的 `subscribeOn()` 所指定的线程。

示例代码：

	Observable.create(onSubscribe)
	    .subscribeOn(Schedulers.io())
	    .doOnSubscribe(new Action0() {
	        @Override
	        public void call() {
	            progressBar.setVisibility(View.VISIBLE); // 需要在主线程执行
	        }
	    })
	    .subscribeOn(AndroidSchedulers.mainThread()) // 指定主线程
	    .observeOn(AndroidSchedulers.mainThread())
	    .subscribe(subscriber);

如上，在`doOnSubscribe()`的后面跟一个 `subscribeOn()` ，就能指定准备工作的线程了。

#### 部分内容摘自 [扔物线](http://gank.io/post/560e15be2dca930e00da1083)