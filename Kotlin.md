# Kotlin 

2021/08/24

## Basic :

1. **所有东西都是对象**
2. 较小的类型**不能**隐式转换为较大的类型,只能向下兼容,需通过`toChar:Char`这样的显示转换
3. 位运算：中缀方式调用 （a or b）
   - and/ or/ xor（异或）/inv(非)
   - shl(有符号左移)/ shr/ ushr（无符号）

## Class:

1. 一个类可以有一个**主构造函数**以及一个或多个**次构造函数**，主构造函数没有任何注解或者可见性修饰符，可以省略这个 *constructor* 关键字，进而使用简洁的写法,次构造函数需要委托给主构造函数， 可以直接委托或者通过别的次构造函数间接委托,用`this` 即可调用次构造或者主构造; 初始构造块`init{}`用于实现主构造的逻辑与属性初始化器交织

2. 默认情况类为final，若要可继承：`open`

3. 派生类有主构造，必须初始化基类的构造函数，若没有，但是派生类想使用次构造,需要使用`super()`初始化基类,在重载的时候，如果要调用基类的方法，采用`super.funcionName()`,特殊的，当内部类中访问外部类的超类`super@Outer.funcionName()`,多继承的时候如果调用基类方法冲突，采用`super<baseName>.functionName()`的方式

   -----------------------------------------------------------------------

4. interface: 接口可以既包含抽象方法的声明也包含实现。与抽象类不同的是，接口无法保存状态

   ------------------------------------------------------------------

5. 可见性修饰符:有这四个可见性修饰符：`private`、 `protected`、 `internal` 和 `public`

   ### data class:

   1. 数据类不能是抽象、开放、密封或者内部的,换言之，抽象类不能继承
   2. 自动实现了对成员的`setter` & `getter`
   3. `hashCode`，`toString`和`equals`方法仅对数据类的**构造函数参数**起作用
   
   ### Inner class:
   
   1. Inner class 使用关键词 inner表示持有对外部类的引用,所以可以调用外部的变量和方法
   2. anonymous inner class: 



## Object

1. 对象表达式：”只是需要一个对象而已“，和 inner class 一样，可以访问来自包含它的作用域的变量

   eg. 等式右边为一个对象表达式

   ``` kotlin
   val ab: A = object : A(1), B {
       override val y = 15
   }

2. 对象声明：

   1. 单例模式：

   ``` kotlin
   object className:Base class{
       func1()
       func2()
   }
   ```

   2. 类内部的对象声明可以用 *companion* object ,直接使用类名调用
   3. @JvmField 修饰成员变量，@JvmStatic 修饰方法， 让java 调用的时候和kotlin保持一致

## 委托

   

1.  实现继承的一个很好的替代方式： 一个接口，一个类委托另一个类去实现这个接口，然后自己让那个实现类的对象来初始化自己，并且可以覆盖方法

2. 委托属性：一个类的某个属性值不是在类中直接进行定义，而是将其托付给一个代理类，从而实现对该类的属性统一管理

   ``` kotlin
   class Example {
       var p: String by Delegate()
   }
   ```

   - 要求： 

     - 委托的这个类需要具有`getValue`,`setValue`方法，因为变量的`get`和`set`会传给这两个方法

       ``` kotlin
       // use the ReadWriteProperty interface to simpily,coz it is the basic library in kotlin and has implement the getValue and setValue so we just need to override it
       fun resourceDelegate(): ReadWriteProperty<Any?, Int> =
           object : ReadWriteProperty<Any?, Int> {
               var curValue = 0 
               override fun getValue(thisRef: Any?, property: KProperty<*>): Int = curValue
               override fun setValue(thisRef: Any?, property: KProperty<*>, value: Int) {
                   curValue = value
               }
           }
       
       val readOnly: Int by resourceDelegate()  // ReadWriteProperty as val
       var readWrite: Int by resourceDelegate()
       
       //if not write in this way 
       //we need "operator fun" to write this two functions
       ```

       

   Kotlin 标准库为几种有用的委托提供了工厂方法：

   	1. [`lazy()`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/lazy.html) 是接受一个 lambda 并返回一个 `Lazy <T>` 实例的函数，由于是接收一个参数，所以按照下面函数规则，可以直接将函数体放在{}中,保证了变量初始化一次，而不用自己去实现线程不安全的逻辑，因为lazy会在第一次get到值后执行lambda语句，并存储值，之后的get访问得到的直接是值而不执行语句。
   	2. 解析 JSON 或者做其他“动态”事情的应用中：

   ``` kotlin
   class User(val map: Map<String, Any?>) {
       val name: String by map
       val age: Int     by map
   }
   ```

   

   

   


## Function:



1. 函数类型 （A, B）-> C ;

2. Lambda 表达式：一种函数类型实例化的常见方式

   完整语法形式：

   ``` kotlin
   val sum: (Int, Int) -> Int = { x: Int, y: Int -> x + y }
   ```

   函数类型往往可以推测从而**省略**

3. 高阶函数（higher-order function）：使用函数作为参数的函数

   ``` kotlin
   fun h-o-f (para1:type,funcName:funcType) :Returntype { // funcType见1
       LOGIC
   }
   ```

   如果函数的最后一个参数是函数，那么作为相应参数传入的 lambda 表达式可以放在圆括号之外,语法也称为*拖尾 lambda 表达式*, 也就是说可以放在花括号中，如果该 lambda 表达式是调用时唯一的参数，那么圆括号可以完全省略

4. 高阶函数的底层=>内联函数：

   inline function: 做了什么事？ 

   lambda表达式找到inline，类似覆盖方法，或者说，给方法赋予实际意义，形成的最后return的结果返回到调用点上，消除产生的开销（若在java中会使用匿名类而造成开销） 

   

5. 匿名函数：省略函数名其余与正常函数相同



## Coroutines

协程允许我们通过单线程去模拟多线程的编程效果

[link](https://mp.weixin.qq.com/s?__biz=MzIzNTc5NDY4Nw==&mid=2247483860&idx=1&sn=d8a4441912d0d1eee189d97506bb4689&chksm=e8e0f844df977152652d69a3b4cc3cd1d1a148609f4295b6142e6d577156b76905e1cb6b95be&token=1091218095&lang=zh_CN#rd)

1. 协程域：

   1. ```
      runBlocking{} :
      coroutineScope{}:
      和runBlocking{}一样会等待其协程体以及所有子协程结束。而 coroutineScope 只是挂起当前协程
      上下文+launch{} //GlobalScope.launch 顶层的直接封装好了
      开启子协程，子协程所在的协程域如果结束，那么子协程也会结束
      
      ```

2. 关于这块有很多疑惑：

   1. 关于 coroutineScope 以及 接口CoroutineScope

   ``` kotlin
   suspend fun <R> coroutineScope(block: suspend CoroutineScope.() -> R): R
   ```

   CoroutineScope 只是一个接口，持有coroutineContext 信息，这里面包含后台job的信息，函数coroutineScope 创造了一个CoroutineScope**接口实例**，并且从外部继承了coroutineContext，但是重载了job信息

   2. 关于launch ：,是CoroutineScope的扩展函数只是fetch CoroutineScope中的context，得到协程返回对 job 的*引用*，值得注意的是，launch只能执行逻辑，而不能返回结果，若想返回结果则需要 async

   3. async{}

      async 返回一个 Deferred,这代表了一个将会在稍后提供结果的 promise。你可以使用 `.await()` 在一个延期的值上得到它的最终结果,

      可以通过传入start参数，指定为惰性启动，从而在await的时候才会去并发执行

   4. 关于 coroutineScope 和 runBlocking

      同：产生协程构造域整个域内执行完毕才继续往下执行
   
      异：coroutineScope只会挂起外部的协程，而runBlocking会挂起外部的线程，所以runBlocking不能再主线程上使用；除此之外，runBlocking不需要在 suspend 或在coroutine中创建,只是为了桥连挂起代码在main中的**测试**而使用
   
   5. 域构造器 withContext与三种参数：
   
      Dispatchers.Default：低并发，计算密集型
   
      Dispatchers.IO:高并发，会出现长时间堵塞
   
      Dispatchers.Main：Android中使用
   
   6. 协程上下文：
   
      啥是协程上下文：各种不同元素的集合，主元素是协程中的 **Job**
   
      由此我们引出Job的调度器： launch 和 async 接收一个可选的调度器参数作为context，而withContext必要提供调度器

## 一些遍历（语法糖）

 1. with : 

    使用场合：对一个对象实例调用多个方法 

    ```kotlin
    with(obj){
    	//提供obj的上下文 直接调用obj的函数
        value // return
    }
    ```

2. apply ：

   使用场合：配置对象的属性

   ``` kotlin
   obj.apply{
       //提供obj的上下文 
   }//返回的值为obj
   ```

3. use:
