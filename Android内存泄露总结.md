# Android 内存泄露总结

> 简单的讲就是，该被释放的对象没有被释放，一直被某个或某些实例所持有却不再被使用导致GC不能回收。
### JAVA 内存分配策略
Java程序运行时的内存分配策略有3种：
- **静态分配**
- **栈式分配**
- **堆式分配**

三种存储策略使用的内存空间分别是：

- **静态存储区**

 主要存放`静态数据`、`全局static数据`、`常量`，这块内存在程序编译时就已分配好，并且在程序整个运行期间都存在。
 
 
- **栈区**

  当方法被执行时，方法体内的局部变量都在栈上创建，并在方法执行结束时，这些局部变量所持有的内存将会自动释放。

- **堆区**

又称动态内存分配，通常指在程序运行时直接`new`出来的内存，这部分内存在不使用时将会由java垃圾回收器来负责回收。


### 栈与堆的区别

在方法体内定义的`局部变量`，一些基本类型的变量和对象的引用变量都是在方法的栈内存中分配的，当在一段方法块中定义一个变量时，Java就会在栈中为该变量分配内存空间，当超过该变量的作用域后，该变量也就无效了，分配给他的内存空间也将被释放掉，该内存空间可以被重新使用。
<br>
堆内存用来存放所有由`new`创建的对象 ( 包括该对象其中的所有成员变量 ) 和数组。在堆中分配的内存，将由Java垃圾回收器来自动管理，在堆中产生了一个数组或者对象后，还可以在栈中定义一个特殊的变量，这个变量的取值等于数组或者对象在堆内存中的首地址，这个特殊的变量就是我们上面说的`引用变量`,我们可以通过这个引用变量来访问堆中的对象和数组。

举个例子:

```
public class Sample {
	
	public void method() {
		int s2 = 1;
		Sample mSample2 = new Sample();
	}
}
```

> Sample 类的局部变量 `s2` 和 引用变量 `mSample2` 都是存在于栈中，但`mSample2`指向的对象是存放于堆上的；

**结论**

<br>
局部变量的`基本数据类型`和`引用`存储于栈中，`引用的对象实体`存储于堆中。
—— 因为它们属于方法中的变量，生命周期随方法而结束。
<br>
成员变量全部存储于堆中 ( 包括`基本数据类型`，`引用`和`引用的对象实体` ) 
——  因为它们属于类，类对象终究是要被`new`出来使用的。


### Java 是如何管理内存

Java的内存管理就是对象的分配和释放问题。在Java中，程序员需要通过关键字`new`为每个对象申请空间，所有的对象都在堆(Heap)中分配空间。另外，对象的释放是由GC决定和执行的。在Java中，内存的分配是由程序完成的，而内存的释放是由GC完成的。简化了程序员的工作，加重了JVM的工作，这也是Java运行速度较慢的原因，因为GC为了能够正确释放对象，GC必须监控每一个对象的运行状态，包括对象的申请、引用、被引用、赋值等，GC都需要监控。

为了更好理解GC的工作原理，我们可以考虑将对象考虑为有向图的顶点，将引用关系考虑为有向边，有向边从引用者指向被引对象，另外，每个线程对象可以作为一个图的起始顶点，例如大多数程序从main进程开始执行，那么该图就是以main进程顶点开始的一颗树。在这个有向图中，根顶点可达的对象都是有效对象，GC将不回收这些对象，反之，不可达则将被GC回收。

### 什么是Java中的内存泄露

在Java中，内存泄露就是存在一些被分配的对象，这些对象有下面两个特点：
- 这些对象是可达的，即在有向图中，存在通路可以与其相连；
- 这些对象都是无用的，即程序以后不会再使用这些对象；

如果对象满足这2个条件，这些对象就可以判定为Java中的内存泄露，这些对象不会被GC回收，然而它却占用内存。

对于我们来说，GC基本是透明的，不可见的，可运行GC的函数 `System.gc()`，但该函数不保证JVM的垃圾收集器一定会执行。因为不同JVM实现者可能使用不同的算法管理GC，通常，GC的线程优先级较低，JVM调用GC的策略也有很多种，有的内存使用达到一定程度，GC才开始工作，也有定时执行的。有的平缓执行GC，有的中断式执行GC。GC的执行影响应用程序的性能。例如对于基于Web的实时系统，如网络游戏等，用户不希望GC突然中断应用程序执行而进行垃圾回收，那么我们需要调整GC的参数，让GC能够通过平缓的方式释放内存，例如将垃圾回收分解为一系列小步骤执行：
给出一个内存泄露的例子：

    List v = new ArrayList(10);
    for (int i = 1; i < 10; i++) {
        Object o = new Object();
        v.add(o);
        o = null;     
    }

> 在这个例子中，我们循环申请Object对象，并将所申请的对象放入一个List中，如果我们仅仅释放引用本身，那么List仍然引用该对象，所以这个对象对GC来说是不可回收的，因此，如果对象加入到List后，还需要从List中删除，最简单的方式就是将List对象设置为NULL；

### Android中常见的内存泄露汇总

-  **集合类泄露**

> 集合类如果仅仅有添加元素的方法，而没有相应的删除机制，导致内存被占用。如果这个集合类是全局变量，比如类中的`静态属性`，全局性的`Map`等即有静态引用或final一直指向它，那么没有相应的删除机制，很可能导致集合所占用的内存只增不减；

- **单例造成的内存泄露**

> 由于单例的静态特性使得其生命周期跟应用的生命周期一样长，所以如果使用不恰当的话，很容易造成内存泄露；

比如下面一个典型例子:

    public class AppManager {
	    private static AppManager instance;
	    private Context context;
	    
	    private AppManager(Context context) {
		    this.context = context;
	    }
	    
		public static AppManager getInstance(Context context){
			if(instance == null){
				synchronized(AppManager.class) {
					if(instance == null) {
						instance = new AppManager(context);
					}
				}
			}
			return instance;
		}

    }

这是一个普通的单例模式，当创建这个单例的时候，需要传入一个`Context`，所以这个`Context`的生命周期的长度至关重要：
 - 如果此时传入的是Application的Context，因为Application的生命周期就是整个应用的生命周期，所以没有问题；

 - 如果此时传入的是Activity的Context，当这个Context所对应的Activity退出时，由于该Context的引用被单例对象所持有，其生命周期等于整个应用的生命周期，所以当前Activity退出时它的内存并不会被回收，这就造成内存泄露。

正确的方式应该改为下面格式：

    public class AppManager {
	    private static AppManager instance;
	    private Context context;
	    private AppManager(Context context){
		    //使用Application的context
		    this.context = context.getApplicationContext();
	    }
		
		public static AppManager getInstance(Context context){
			if(instance == null) {
				synchronized(AppManager.class) {
					if(instance == null) {
						instance = new AppManager(context);
					}
				}
			}
			return instance;
		}
    }

或者在`Application`中添加一个静态方法 `getContext()` 返回`Applicaton`的`Context`；



- **非静态内部类创建静态实例造成的内存泄露**

有的时候我们可能会在启动频繁的Activity中，为了避免重复创建相同的数据资源，可能会出现这种写法：

    public class MainActivty extends AppCompatActivity {
	    private static TestResource mResource = null;
		
		@Override
		protected void onCreate(Bundle savedInstanceState){
			super.onCreate(savedInstanceState);
			setContentView(R.layout.activity_main);
			
			if(mResource == null){
				mResouce = new TestResource();
			}

			//...
		}

		class TestResource{
			//...
		}
	}

这样就在`Activity`内部创建了一个非静态内部类的对象，每次启动`Activity`时都会使用该单例的数据，这样虽然避免了资源的重复创建，不过这种写法却会造成内存泄露，因为非静态内部类默认会持有外部类的引用，而该非静态内部类又创建了个静态的实例，该实例的生命周期和应用一样长，这就导致了该静态实例一直会持有该`Activity`的引用，导致Activity的内存资源不能正常回收；

正确的做法是；

> 将该内部类设为静态内部类或将该内部类抽取出来封装成一个单例，如果需要使用Context，推荐使用Application的Context，当然Application的Context不是万能的，所以也不能随便使用，对于有些地方则必须使用Activity的Context；

- **匿名内部类**

android开发经常会继承实现`Activity`、`Fragment`、`View`；此时你使用了匿名类，并被异步线程持有了，那要小心了，如果没有任何措施这样一定会导致内存泄露；

    public class MainActivity extends Activity {
	    ...
		
		Runnable ref1 = new MyRunnable();
		Runnable ref2 = new Runnable(){
			@Override
			public void run(){
				
			}
		}

		...
    }

`ref1` 和 `ref2` 的区别是，`ref2`使用了匿名内部类，引用内存结构：

    ref1 = MyRunnable (id = 830042475848)
    ref2 = MainActivity$1 (id = 83004247656)
	    this$0 = MainActivity (id = 830042474408)

可以看到，`ref1`没什么特别的；
但`ref2`这个匿名类的实现对象里面多了个引用；
`this$0` 这个引用指向`MainActivity.this`, 也就是说当前`MainActivity`实例会被`ref2`持有，如果将这个引用再传入一个异步线程，此线程和`Activity`生命周期不一致的时候，就造成`Activity`的泄露。


-  **Handler 造成的内存泄露**

`Handler`的使用造成的泄露最为常见，平时处理网络任务或封装一些请求回调等`api`都应该会借助Handler来处理，对于Handler的使用代码编写一不规范即有可能造成内存泄露；如下示例:

    public class MainActivity extends AppCompatActivity {
	    private Handler mHandler = new Handler() {
		    @Override
		    public void handleMessage(Message msg){
			    //...
		    }
	    }

		@Override
		protected void onCreate(Bundle savedInstanceState){
			super.onCreate(savedInstanceState);
			setContentView(R.layout.activity_main);
			loadData();
		}
	
		private void loadData(){
			//...request
			Message message = Message.obtain();
			mHandler.sendMessage(message);
		}
    }

这种创建`Handler`的方式会造成内存泄露，由于`mHandler`是`Handler`的非静态匿名内部类的实例，所以它持有外部类`Activity`的引用，我们知道消息队列是在一个`Looper`线程中不断轮询处理消息，那么当这个`Activity`退出时消息队列中还有未处理的消息或正在处理的消息，而消息队列中的`Message`持有`mHandler`实例的引用，`mHandler`又持有`Activity`的引用，所以导致`Activity`的内存资源无法及时回收，引发内存泄露，所以另一种做法：

    public class MainActivity extends AppCopatActivity {
	    private MyHandler mHandler = new MyHandler(this);
	    private TextView mTextView;
	    private static class MyHandler extends Handler{
		    private WeakReference reference;
		    public MyHandler(Context context){
			    reference = new WeakReference<>(context);
		    }

			@Override
			public void handleMessage(Message msg) {
				MainActivity activity = (MainActivity)reference.get();
				if(activity != null){
					activity.mTextView.setText("");
				}
			}
	    }

		@Override
		protected void onCreate(Bundle savedInstanceState){
			super.onCreate(savedInstanceState);
			setContentView(R.layout.activity_main);
			loadData();
		}
	
		private void loadData(){
			//...request
			Message message = Message.obtain();
			mHandler.sendMessage(message);
		}

		@Override
		protected void onDestroy(){
			super.onDestroy();
			mHandler.removeCallbackAndMessages(null);
			
		}
    }

创建一个静态`Handler`内部类，然后对`Handler`持有的对象使用弱引用，这样在回收时也可以回收`Handler`持有的对象，这样虽然避免了`Activity`泄露，不过`Looper`线程的消息队列中还是可能会有待处理的消息，所以我们在`Activity`的`onDestroy`或者`onStop`方法时移除消息队列中的消息。

使用`mHandler.removeCallbackAndMessages(null)`是移除消息队列中的所有消息和所有`Runnagle`；

下面几个方法都可以移除Message:

> public final void removeCallbacks(Runnable r);

> public final void removeCallbacks(Runnable r,Object token);

> public final void removeCallbacksAndMessages(Object token);

> public final void removeMessages(int what);

> public final void removeMessages(int what,Object object);

- **线程造成的内存泄露**

对于线程造成的内存泄露，也是比较常见的，异步任务和`Runnable`都是一个匿名内部类，因此它们对当前`Activity`都有一个隐式引用，如果`Activity`在销毁之前，任务还未完成，那么将导致`Activity`的内存资源无法回收，造成内存泄露，正确的做法还是使用静态内部类的方式。如下：

    static class MyAsyncTask extends AsyncTask {
	    private WeakReference weakReference;

		public MyAsyncTask(Context context){
			weakReference = new WeakReference<>(context);
	    }

		@Override
		protected Void doInBackground(Void... params){
			SystemClock.sleep(10000);
			return null;
		}

		@Override
		protected void onPostExecute(Void aVoid){
			super.onPostExecute(aVoid);
			MainActivity activity = (MainActivity)weakReference.get();
			if(activity != null){
				//...
			}
		}
    }

	static class MyRunnable implements Runnable{
		@Override
		public void run(){
			SystemClock.sleep(10000);
		}
	}

	//----------
	new Thread(new MyRunnagle()).start();
	new MyAsyncTask(this).execute();

这样就避免了`Activity`的内存资源泄露，当然在`Activity`销毁时候也应该取消相应的任务`AsyncTask :: cancel()`,避免任务在后台执行浪费资源；


- **尽量避免使用static成员变量**

如果成员变量被声明为`static`,那么我们都知道其生命周期将与整个app进程周期一样。

这会导致一系列问题，如果你的app进程设计上是长驻内存的，那即使app切到后台，这部分内存也不会被释放，按照现在手机app内存管理机制，占内存较大的后台进程将优先回收，如果此app做过进程保活，那会造成app在后台频繁重启，当手机安装了你参与开发的app以后一夜时间手机被消耗空了电量，流量，你的APP不得不被用户卸载或者静默。

这里修复的方法是：

> 不要在类初始时化静态成员，可以考虑`Lazy`初始化。

- **资源未关闭造成的内存泄露**

对于使用了`BraodcastReceiver`、`ContentObserver`、`File`、`游标Cursor`、`Stream`、`Bitmap`等资源的使用，应该在`Activity`销毁时及时关闭或者注销，否则这些资源将不会被回收，造成内存泄露。

- **一些不良代码造成的内存压力**

有些代码并不造成内存泄露，但是它们，或是对没使用的内存没进行有效及时的释放，或是没有有效的利用已有的对象而是频繁的申请新内存。比如：

> `Bitmap `没调用 `recycle()`方法，对于`Bitmap`对象在不使用时，我们应该先调用`recycle()`释放内存，然后将它设置为null；
> 
> 构造`Adapter`时，么有使用缓存`convertView`,每次都在创建新的`convertView`,这里推荐使用`ViewHolder`;


### 工具分析

#### 使用`LeakCanary`检测`Android`的内存泄露

#### [LeakCanary 中文使用说明](https://www.liaohuqiu.net/cn/posts/leak-canary-read-me/)

#### Demo接入

在`build.gradle`中加入引用，不同的编译使用不同的引用；

    dependencies {
	debugCompile 'com.squareup.leakcanary:leakcanary-android:1.3'
	releaseCompile 'com.squareup.leakcanary:leakcanary-android-no-op:1.3'
	}
#### 如何使用

使用`RefWatcher`监控那些本该被回收的对象。

    RefWatcher refWatcher = {...}
    //监控
    refWatcher.watch(schrodingerCat);
`LeakCanary.install()`会返回一个预定义的`RefWatcher`,同时也会启用一个`ActivityRefWatcher`,用于自动监控调用`Activity.onDestroy()`之后泄露的`Activity`.

在`Application`中进行配置:

    public class ExampleApplication extends Application{
		private RefWatcher refWatcher;

		public static RefWatcher getRefWatcher(Context context){
			ExampleApplication application = (ExampleApplication)context.getApplicationContext();
			return application.refWatcher;
	    }

		@Override
		public void onCreate(){
			super.onCreate();
			refWatcher = LeakCanary.install(this);
			
		}
    }
    
使用`RefWatcher` 监控`Fragment`：


    public abstract class BaseFragment extends Fragment{
	    @Ovirride
	    public void onDestroy(){
		    super.onDestroy();
		    RefWatcher refWatcher = Examplepplication.getRefWatcher(getActivity());
		    refWatcher.watch(this);
	    }
    }

使用`RefWatcher`监控`Activity`:

    public class MainActivity extends AppCompatActivity{
	    .....
	    @Override
	    protected void onCreate(Bundle savedInstanceState){
		    super.onCreate(savedInstanceState);
		    setContentView(R.layout.activity_main);
		    //在自己的应用初始Activity中加入如下两行代码
		    RefWatcher refWatcher = ExampleApplication.getRefWatcher(this);
		    refWatcher.watch(this);
	    }
    }


#### Android Monitor Memory

Android Studio 自带的内存监视，可观察应用内存占用，运行应用一段时间如果内存占用持续升高，有可能存在内存泄露。

#### Android Device Monitor

SDK 的 `Device Monitor`是分析应用内存分配情况的好工具。

- Heap
  可查看堆内存，使用：选中进程，点击`update heap`,点击`Cause GC`即可显示该进程的内存情况，以后每次GC都会更新，也可手动`Cause GC`


### 总结
- 对`Activity`等组件的引用应该控制在`Activity`的生命周期之内，如果不能就考虑使用`getApplicationContext`或者`getApplication`，以避免`Activity`被外部长生命周期的对象引用而泄露。
- 尽量不要在静态变量或者静态内部类中使用非静态成员变量（包括`context`）,即使要使用，也要考虑适时把外部成员变量置空，也可以在内部类中使用弱引用来引用外部类的便量；
- 对于生命周期比`Activity`长的内部类对象，并且内部类中使用了外部类的成员变量，可以这样做避免内存泄露
    - 将内部类改为静态内部类
    - 静态内部类中使用弱引用来引用外部类的成员变量
- `Handler`的持有的引用对象最好使用弱引用，资源释放时也可以清空`Handler`里面的消息，比如在`Activity onStop 或 onDestroy`的时候，取消掉该`Handler`对象的`Message`和`Runnable`;
- 在`Java`的实现过程中，也要考虑其对象释放，最好的方法时在不使用某对象时，显示的将此对象置为`null`，比如使用完`Bitmap`后先调用`recycle()`，再赋值为`null`,清空对图片等资源有直接引用或者间接引用的数组（使用`array.clear();array = null`）等，最好遵循谁创建谁释放原则。
- 正确关闭资源，对于使用了`BroadcastReceiver,ContentObserver,File,Cursor,Stream,Bitmap`等资源的使用，应该在`Activity`销毁时及时关闭或注销。
- 保持对对象生命周期的敏感，特别注意单例，静态对象，全局性集合等的生命周期。
