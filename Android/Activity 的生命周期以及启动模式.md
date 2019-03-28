# Activity 的生命周期以及启动模式

## 生命周期

### 典型情况下的生命周期

- onCreat : Activity正在被创建，在这个方法中做一些初始化工作，比如加载布局、初始化控件等。
- onRestart : Activity正在重新启动，一般执行在onPause、onStop之后，Activity由不可见变为了可见状态。
- onStart : Activity正在被启动，执行在onCreat之后，可见但是不能交互。
- onResume : Activity已经启动，可见可交互，Activity显示到前台。
- onPause : Activity正在停止，在onStop之前执行。
- onStop : Activity即将停止，可以做一些轻量级的回收工作，不能太耗时。
- onDestory : Activity即将被销毁，生命周期中的最后一个回调，可以做一些回收工作和最终的资源释放。


####关于onStart   onResume  onPause  onStop  的问题

在实际过程中，onStart和onResume 、onPause和onStop  看起来的确差不多，我们只用保留其中的一对就可以了。但是这两对回调分别表示的不同的意义，onStart和onStop是从Activity是否可见的角度回调的，而onPause和onResume是从Activity是否位于前台来回调。

### 异常情况下的生命周期

- #### 资源相关的配置发生改变导致的Activity被杀死并重新创建

  当系统配置发生改变的时候，Activity会被销毁，其onPause 、onStop、onDestory均会被调用，由于Avtivity是被异常终止的，系统会调用onSaveInstanceState来保存Activity的状态。这个方法调用在onStop之前，但是和onPause没有既定的时序关系。在Activity被重新创建后，系统会调用onRestoreInstanceState方法，并且把销毁时onSaveInstanceState保存的Bundle对象作为参数传给onCreate和onRestoreInstanceState，从时序上来说，onRestoreInstanceState调用在onStart之后。

  Activity在发生异常进行重建的时候，会默认保存当前的视图结构，系统的工作流程大致为：Activity---->onSaveInstanceState---->window---->顶层ViewGroup(DecorView)---->子元素---->子元素

  系统只有在即将销毁并且有可能再次展示的情况下（异常终止）才会调用onSaveInstanceState，当其正常销毁的时候不会调用。

- #### 资源内存不足导致低优先级的Activity被杀死

  - 前台Activity——正在和用户交互的优先级最高
  - 可见但非前台的Activity——Dialog下的Activity可见 但是不能和用户进行交互
  - 后台Activity——已经被暂停的Activity，比如执行了onStop，优先级最低

当系统配置中的某一项内容发生改变时，若不想系统重新创建Activity,可以给Activity指定configChanges属性。

- local——设备的本地位置发生了变化，一般指的是切换了系统语言
- orientation——屏幕的方向发生了改变，一般是旋转了手机屏幕
- keyboardHidden——键盘的可访问性发生了改变，比如用户调出了键盘

## 启动模式

### Activity 的 LaunchMode

- standard ：标准模式，每次启动一个Activity都会创建一个新的实例，不管这个实例是否存在。它的onCreate、onStart、onResume 都会被调用，典型的多实例实现，一个任务栈中可以有多个实例，每个实例也可以属于不同的任务栈，睡启动了Activity,那么这个Activity就运行在启动它的那个Activity的栈中。
- singleTop ：栈顶复用模式，如果新的Activity已经位于任务栈的顶部，那么该Activity不会被重新创建，意味着它的onCreate、 onStart 不会被系统调用。同时，它的onNewIntent方法会被调用，通过此方法的参数我们可以获取当前请求的信息。如果新的Activity不在任务栈的栈顶，那么新的Activity还会被重新创建。
- singleTask ：栈内复用模式，一种单实例模式。只要Activity在一个栈中存在，那么多次启动此Activity都不会创建新的实例，和singleTop一样，系统也会回调其onNewIntent。
  - S1：ABC        D被启动，所需任务栈S2，所以系统会先创建S2，然后再创建D的实例并将其入栈到S2
  - S1：ABC        D被启动，所需任务栈为S1，系统会创建D的实例并将其入栈到S1。
  - S1：ADBC      D被启动，所需任务栈为S1，D不会被重新创建，并且会把D切换到栈顶，singTask默认有clearTop的效果，会导致D上面的Activity全部出栈，最终情况为：S1：AD。
- singleInstance ：单实例模式，一种加强的singleTask模式，具有此种模式的Activity只能单独的位于一个任务栈中。

#### 给Activity指定启动模式

- AndroidMennifest中给Activity设置LunchMode
- 通过Intent设置标志位为Activity指定启动模式

> 第二种的方式的优先级要高于第二种

### Activity 的Flags

Activity的Flags有很多，标记位的作用很多，有的可以设定Activity的启动模式，有的还可以影响Activity的运行状态。

常用的标记位：

- FLAG_ACTIVITY_NEW_TASK

- FLAG_ACTIVITY_SINGLE_TOP

- FLAG_ACTIVITY_CLEAR_TOP

- FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS

  具有这个标记的Activity不会出现在历史Activity列表中

### IntentFilter 的匹配规则

1. #### action 的匹配规则

   Intent中的action 必须能够和过滤规则总的action匹配即action的字符串值完全一样。一个过滤规则中可以有多个action，只要Intent中的action能够和过滤规则的任何一个action相同即可匹配成功。如果Intent中没有指定action，则匹配失败。

2. #### category 的匹配规则

   Intent中如果含有category,不论几个，都必须是过滤规则中已经存在的，则可以匹配成功。Intent中可以没有category，因为系统在调用startActivity或者startActivityForResult的时候会默认加上“android.intent.category.DEFAULT”，这个category。为了我们能接收隐式调用，就必须在intent-filter中指定“android.intent.category.DEFAULT”这个category。

3. #### data 的匹配规则

   data匹配规则和action 类似，它也要求Intent中必须包含有data数据，并且data数据能够完全匹配过滤规则中的某一个data。

   > - URI的默认值为content和file
   > - 要为Intent指定完整的data，必须要调用setDataAndType
   > - Service尽量使用显式调用方式来启动服务
   > - 可以使用PackageManager 或者Intent 的resolveActicity方法，如果找不到匹配的Activity就会返回null。
   > - PackageManager还提供了queryIntentActivities方法，返回所有匹配成功的Activity。

