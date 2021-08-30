# Hello Android



## Android developer 上一些摘取：

[The link](https://developer.android.com/guide/components/activities/intro-activities)

### Activity

1. 为什么选择 Activity 而不是一般的`main()`作为启动方式：

    用户与应用的互动并不总是在同一位置开始，而是经常以不确定的方式开始，Activity 充当了应用与用户互动的入口点  

2. manifest 中关于 Activity 的描述与类的关系：

   ``` xml
    <application ... >
             <activity android:name=".MainActivity" />
             ...
   </application ... >
   ```

   通过 `android:name` 唯一描述 Activity. 与 java 包下源代码中的 Activity 类 `class MainActivity : AppCompatActivity()` （继承兼容的 Activity 类）相互对应

3. 从启动 Activity 时的 intent 过滤器的设置扩展开来:

   1.  intent 对于 Activity 作用：在 Activity 的实现类中执行 `startActivity`  或者 `startActivityForResult(Intent, int)` ，（插一句，int 标识符不是全局标识符,主要用于消除来自同一 Activity 的多次调用之间的歧义）这个指令实际上是发送一个调用指令给操作系统的 ActivityManager （维护一个非特定应用独享的**后退栈**） ，由这个 ActivityManager 来创建实例并执行这个实例的 `onCreate()`,因此 ActivityManager 决定何时调用哪个 Activity ，而哪个 Activity 是由 intent 携带的信息去决定的。

   2.  显式调用：`Intent(Context, Class)` 构造函数分别为应用和组件提供 `Context` 和 `Class` 对象，直接指定需要调用的class.

      隐式调用：告诉 OS 你要的 Action、数据类型( mimeType / scheme )、category(在哪个**环境**中才能被激活) 、数据位置(URI)，让 OS 匹配符合条件的部件. 为了接收隐式调用，在 manifest 文件中通过以下来响应.

      ``` xml
      <activity ...>
          <intent-filter>
              <action ...></action>
              <category ...></category>
              <data ...></data>
          </intent-filter>
      </activity>
      ```

      

   3. 由上述可知：Intent 是一个消息传递对象,我们设置一个 Intent 对象，将一些描述和一些附加信息放入( `putExtra(key,map)`), 这一信息模块被传递到 OS，描述筛选出符合接收这个对象的 Activities , 由用户选择做出决定，附加信息通过 `getIntent` 取得，这样就实现了活动之间的跳转以及数据在不同活动之间的传递.

4. Activity 生命周期:

   

   <img src="pic/activity life span.jpg" alt="alt " style="zoom: 33%;" />

   其中：

   1. `onCreate `:  绑定数组到列表( ViewModel )；渲染布局( 	`setContentView()`)

   2. 对于数据的恢复和存储：

      对于 Activity 的结束，我们从两个角度去区分：一是用户主动关闭（返回按钮），二是系统（旋转发生的设备配置变更）

      - 轻量数据存储：在 Activity 结束前，会调用`onSaveInstanceState` 将一些键值对放到 Bundle 中，Bundle 需要在主线程上进行序列化处理并占用系统进程内存，`onCreate()`和 `onRestoreInstanceState()`都可以恢复，后者可以处理一些没用的内容，需要注意的是，**当用户显式关闭 Activity 时，或者在其他情况下调用 `finish()` 时，系统不会调`onSaveInstanceState()`**
      - 通过 viewModel 保存UI 解决旋转时候恢复问题

### Fragment

* **作用**：控制器对象，管理UI, 可执行 Activity 委派的任务

​	需由 Activity 托管，放置在 Activity 的视图层级中，本质上，Fragment 是作为一种特殊的 UI 片段（因为具有和 Activity 一样可以实现逻辑的功能，

​	Fragment 显示的两种方式：
​		1. 静态：在 Activity 的 xml 中添加`<fragment />` 用`android:name`指代 自定义的 Fragment 类
  		2. 动态显示

* **流程（动态）**：

   ​	1. 定义 UI xml视图

   ​	2. 定义`Fragment()` 实现类, 类中通过`onCreateView` 渲染 UI 视图

   ​	3. 在 Activity 的视图布局中为 fragment 设置一个容器视图( FrameLayout )

   ​	4. Activity 权限很高，因为 fragment 交由 Activity 托管，所以 Activity 有 FragmentManager 类，可通过 supportFragmentManager 直接得到，这个 Manager 开启事务 ，add (视图容器，`Fragment()` 实现类 ) ，实现绑定.

* 关于 FragmentManager :

   ​		上述流程中提及， FragmentManager 在 Activity 中可以访问，在 Fragment 中同样可以. 因为 Fragment 也能够托管一个或多个子 Fragment，具体方法有：`getChildFragmentManager()`、`getParentFragmentManager()`，每一个有托管 Fragment 的 Activity 或是 Fragment 都有 manager 来管理 children .例如，在

   Activity 上层的 Fragment 通过 `getParentFragmentManager()` 可以得到 Activity **层级**的 Manager

   ​		在[文档](https://developer.android.com/guide/fragments/fragmentmanager#dependencies)中提及的 Factory 后续补充，应该非常重要，处理的主要问题是发生配置变换重建时，默认的工厂只加载无参构造函数，而导致参数丢失的问题.

* Fragment 生命周期： 

     ​	总体上看，Fragment 生命周期和 Activity 对应，因为 Fragment 代表 Activity 工作，所以要反映 Activity 的状态.在`oncreate()` 之前，`onAttach()` 被执行完成托管的过程。

     

     <img src="pic/fragment-view-lifecycle.png" alt="alt " style="zoom: 67%;" />

     

### 数据库存储 ROOM

<img src="pic/room_architecture.png" alt="alt " style="zoom:50%;" />

1. 使用流程：
   1. 将 model 定义为实体类，model 的每个属性都是表头(字段)
   2. 建立数据库类，继承`RoomDatabase()`，注解`@Database` 参数传入定义的实体(准确说应该是实体类集合)，版本号
   3. 新建类，为除基本数据类型之外的类型添加类型转换器(@TypeConverter ),传入注解`TypeConverter ` 的参数中，加到数据库内
   4. 定义数据库访问对象(DAO)接口，定义所需的对数据库的操作(`@Query("SQL")`)
   5. 建立仓库模式(单例类)，需要具有两功能：一是初始化形成实例，二是读取仓库实例
   6. 在仓库中`Room.databaseBuilder().build()` 传入`context`、数据库类、数据库文件名,将（ DAO ）中定义的注释SQL语句下函数简化
   7. 将仓库的初始化放入 application 中，数据库应该具有最长的生命周期，application 满足条件
   
2. LiveData 和 Room 配合使用

   - 原因：主线程上不允许访问数据库，因为主线程运行着更新 UI 的代码,相比较访问数据库耗时且会导致应用无响应的状态. LiveData 具有跨线程沟通的能力

   - 什么是 LiveData ：可观察的数据存储器类，可用于任何数据的封装容器，只会将更新通知给活跃的观察者

   - MutableLiveData && LiveData：

     ​	1.MutableLiveData的父类是LiveData

     ​	2.LiveData在实体类里可以通知指定某个字段的数据更新.	

     ​	3.MutableLiveData则是完全是整个实体类或者数据类型变化后才通知.不会细节到某个字段

   - LiveData 使用：

     	1. 在 ViewModel 中将数据封装
     	2. 在主程序初始化的函数中定义 observer，`onChanged()` 方法负责响应 LiveData 的数据变化
     	3. LiveData 对象使用 `observe()`方法将  Observer 对象附加,其中`observe()`方法的第一个参数为 Observer 的生命周期，一般与当前组件一致，第二个参数为 Observer

### [retrofit](https://square.github.io/retrofit/)











### Broadcast 

1. 接收广播：

   方法1：xml 文件中静态注册 `<receiver>` 指定接收的 intent => 实现在注册中的 `BroadcastReceiver()`的继承类，重载接收到广播的逻辑

   方法2： 上下文注册，只要应用在运行，就会收到广播：调用`registerReceiver(BroadcastReceiver, IntentFilter)` 来注册接收器：`unregisterReceiver(android.content.BroadcastReceiver)`注销接收器

   在接收广播（`onReceive()`）过程中,整个进程被认为使前台进程，在`onReceive()` 执行完毕后，系统会将其进程视为低优先级进程，随时可能终止进程来回收内存，要执行长时间运行的工作

2. 发送广播：

   广播三种方式：

   			1. `sendOrderedBroadcast(Intent, String)`顺序接收中途可以终止
   			2.  `sendBroadcast(Intent)`常规广播
   			3. `LocalBroadcastManager.sendBroadcast(Intent)` 广播发送给与发送器位于同一应用中的接收器

   方式1、2可以携带`Manifest.permission.`加入权限





## Material Design











## 属性动画



