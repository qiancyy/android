# Kotlin 

2021/08/24

## Basic :

1. **所有东西都是对象**
2. 较小的类型**不能**隐式转换为较大的类型,只能向下兼容,需通过`toChar:Char`这样的显示转换
3. 位运算：中缀方式调用 （a or b）
   - and/ or/ xor（异或）/inv(非)
   - shl(有符号左移)/ shr/ ushr（无符号）

## Class:

1. 一个类可以有一个**主构造函数**以及一个或多个**次构造函数**，主构造函数没有任何注解或者可见性修饰符，可以省略这个 *constructor* 关键字，进而使用简洁的写法,次构造函数需要委托给主构造函数， 可以直接委托或者通过别的次构造函数间接委托,用`this` 即可调用次构造或者主构造; 初始构造块`init{}`用于实现**主**构造的逻辑与属性初始化器交织

2. 继承与重写：

   1. kotlin 遵循默认关闭（closed）的原则，对应java 为final，若要可继承：`open`，对于抽象类默认是`open`

   2. 接口：**最高级别的抽象**

      接口可以有默认的方法实现，可以包含属性，但是属性没有`field`字段，这就意味着不具有状态，只能只读;那些没有方法体的方法都是隐式的abstract，有方法体的以及属性是隐式的open,意味着所有都可以被重写.

   3. 抽象类：**次高**级别的抽象

      和接口的区别在于可以包含具体的实现并且可以携带**状态**

3. 派生类有主构造，必须初始化基类的构造函数，若没有，但是派生类想使用次构造,需要使用`super()`初始化基类,在重载的时候，如果要调用基类的方法，采用`super.funcionName()`,特殊的，当内部类中访问外部类的超类`super@Outer.funcionName()`,多继承的时候如果调用基类方法冲突，采用`super<baseName>.functionName()`的方式

4. **可见性**修饰符:有这四个可见性修饰符：`private`、 `protected`、 `internal`(类似于包模块的可见性) 和 `public`

   ### data class:

   1. 数据类不能是抽象、开放、密封或者内部的,换言之，抽象类不能继承
   3. `hashCode`，`toString`和`equals`方法仅对数据类的**构造函数参数**起作用可以按照预想的按照值去判断工作，除此之外还自动生成了copy以及解构的`componentN`方法(和python 很像)`val (para1,para2, _)=obj1` 把obj的属性拆分出来不需要的使用下划线代替

   ### Inner class:

   1. Inner class 使用关键词 **inner**表示持有对外部类的引用,所以可以调用外部的变量和方法，这很重要。而嵌套类需要持有外部类的一个实例

      ```kotlin
      class outer{
          var outName:String="out"
          inner class INner{
              outName="inner"
      	}
      }
      ```

      



## Object

1. 对象表达式：”只是需要一个对象而已“，和 inner class 一样，可以访问来自包含它的作用域的变量 动态创建一个对象

   对象表达式的功能强大，可以重写某些成员的同时创建对象，这就使得这个对象变得像类一样。

   ``` kotlin
   setListener(object:Listener{
       override fun onClick():Boolean{
           
       }
   })
   // 只有一个抽象方法的接口(SAM接口)，可以直接将parameter传入lambda表达式简化,不用写重载函数 例如 listener{
   	view -> ...
   }
   ```

   

   eg. 等式右边为一个对象表达式

   ``` kotlin
   val ab: A = object : A(1), B {
       override val y = 15
   }

2. 对象声明 (object + 对象名)：不能像对象表达式一样放在等式的右侧了

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

   1. [lazy()](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/lazy.html) 
     是接受一个 lambda 并返回一个 `Lazy <T>` 实例的函数，由于是接收一个参数，所以按照下面函数规则，可以直接将函数体放在{}中,保证了变量初始化一次，而不用自己去实现线程不安全的逻辑，因为lazy会在第一次get到值后执行lambda语句，并缓存，之后的get直接返回缓存的值。

   2. Delegates.observable

     传入一个初始值和一个用于监听变化的函数lambda

   3. 解析 JSON 或者做其他“动态”事情的应用中：

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

   当Lambda表达式只有一个参数的时候可以在函数体中用`it`来表示参数

   整个过程为lambda表达式赋值给函数类型变量sum

3. 高阶函数（higher-order function）：

   ``` kotlin
   fun h-o-f (para1:type,funcName:funcType) :Returntype { // funcType见1
       LOGIC
   }
   ```

   **用函数作为参数去创造一个函数对象，然后再传入参数**

   如果函数的最后一个参数是函数，那么作为相应参数传入的 lambda 表达式可以放在圆括号之外,语法也称为*拖尾 lambda 表达式*, 也就是说可以放在花括号中，如果该 lambda 表达式是调用时唯一的参数，那么圆括号可以完全省略

4. 高阶函数的底层=>内联函数：

   为什么需要内联函数：每个lambda表达式都会创造一个对象，这就意味着消耗内存。

   inline function: 做了什么事？ 

   lambda表达式找到inline，类似覆盖方法，或者说，给方法赋予实际意义，形成的最后return的结果返回到调用点上，消除产生的开销（若在java中会使用匿名类而造成开销） 

   lambda 不允许从它封装好的函数中返回，但是如果lambda传入的是一个内联函数，允许`return `非局部返回

   

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

[link](https://youkmi.blog.csdn.net/article/details/78786646?utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.control&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromMachineLearnPai2%7Edefault-1.control)

 1. with : 

    使用场合：与apply 基本相同，唯一不同的是lambda最后一行作为返回

    ```kotlin
    with(obj){
    	//提供obj的上下文 直接调用obj的函数
        value // return
    }
    ```

2. apply ：

   使用场合：

   1. 配置对象的属性（初始化对象）
   2. 封装对同一对象的多次调用（无需再用it,因为已经提供了上下文的环境，所有的apply的操作都会作用再obj上，这也使得链式操作成为可能）

   ``` kotlin
   obj.apply{
       //提供obj的上下文 
   }//返回的值为obj
   ```

3. use:不是standard.kt的成员，保证调用者在执行完操作后关闭资源，仅仅作为closable的子类来使用`Reader` `SOcket`

4. let:

   使用场合：

    1. 限定变量作用域，it指代，只在lambda中使用.

    2. 仅当变量不为空的时候使用（判空）

    3. lambda最后一行作为返回

       

5. run: 和let类似，不同的是可以立即执行，内部使用this,不是it，可以看作with 和 let的结合

6. also: 

   使用场景：

   1. 执行验证或记录：also中的lambda相当于对函数的限制，例如`function().also{A(it)}` function 函数执行，并且执行lambda 传入it,A作为验证函数，如果符合条件才会真正返回给前面的变量
   2. interceptor
   
 7.  **sum up **

    apply, with ,run 具有潜在的特性，也就是，在引用对象的时候，使用的this,而不是it,(this提供的是对象类的环境，就像是对象的扩展类一样可以使用对象里的变量)，这是带接收者的lambda，见以下：

    `fun <T,R> T.run(block:T.()->R):R=block()`

    而像let使用的是it,只是参数lambda

    `fun<T,R> T.let(block:(T)->R):R=block(this)` 由于在这里，T作为唯一的参数被传入，所以在lambda中可以通过it获取



8.30《kotlin 移动应用开发笔记》

1. 中缀函数

   1. 常见：for循环中 `until` `downTo` `step`
   2. 特征：允许函数放在两个参数之间
   3. 自定义中缀函数：要求该函数是类成员函数或者是定义的扩展函数，并且只有一个额外参数，用`infix fun `修饰

2. 运算符函数：

   1. 与中缀表达式的唯一区别就是将`infix fun`转换为`operator fun `

3. 空安全：

   1. 当使用`val name:String?`表示定义了一个可空变量，这时对该变量的访问不再安全，因此在调用变量的时候采用`name?.`表示对name检查判空
   2. 快速空变量赋值：`?:`类似于三目表达式 `val len=name?.length ?:0`
   3. 强制不检查`!!`

4. 相等性检查：

   1. 通过`===`来检查引用相等，也就是两者是否指向的是同一个对象，`==`检查结构性相等，即值相同，在结构性检查`==`采用的是`equals`函数

5. 集合API（List,Set,Map）和高阶函数

   1. filter{} 留下{}满足条件的元素
   2. map映射函数,形成的是一个List
   3. groupBy{}形成一个map,group的条件转化为key
   4. associate{}两部分的数据形成关联，组成map
   5. fold(initial number){} 将集合转换为一个值

6. 惰性计算&惰性序列：一般的函数执行是采用广度的方式，比如说我们需要得到一个list的映射，那么我们一口气得到所有list的映射然后再执行下一步操作，而惰性序列是深度的方式执行,是一种高效的，只有在**需要**的时候（**最终操作**）才会对表达式进行计算的数据结构

   ```kotlin
   使用
   1. var sequence=sequenceOf()
   2. 转换：Collection.asSequence()
   3. 所有的操作都最后以最终操作结束,不然sequnce不执行最终操作有toList,toSet,toMutableList，first,max,last,min,joinToString
   4.使用take(n) drop(n)缩减数据的规模
   ```

   

7. 小知识：

   1. 扩展函数是在编译时运行（静态解析）而不是在运行时进行，所以不存在多态的情况

   

8.31《kotlin 移动应用开发笔记》

1. 类和对象的示例化：

   ``` kotlin
   //一般的语言的写法
   class Task(title:String){
       val t:String=title
   }
   //kotlin简化
   class Task(var/val title:String){
       //主构造函数加上val/var 修饰,就可以隐式的将这个参数作为类的属性
   }
   ```

   值得注意的是，我们讨论的是属性（property，类与外部接口的组成成分，可以访问field），而不是字段（field，存储实际的数据），这就意味着，我们被允许通过setter和getter方法访问这个字段的值 val的属性自动有getter,var的属性自动有getter和setter，我们如果需要修改默认的get和set 可以通过`get()``set()`对field进行操作

2. 进一步学习延迟初始化：

   为什么要延迟初始化：

   很多时候一个类中的对象都通过依赖注入的形式，由其他的类产生自己的对象

   ``` kotlin
   class Test{
       lateinit var car:Car
       fun setup{
           car=CarFactory.car() //Re-initialize the property
       }
   }
   ```

   1. 如果想在使用前查看是否已被初始化,可在 Test 类中使用`this::car.isInitialized`（::是函数指针引用）

3. 泛型函数的具体化：

   使用场景：希望访问泛型函数的泛型类型参数，然而JVM规定类型擦除，这就导致在运行的时候参数类型不会保留，但是通过inline内联可以将具体类型参数内联到内联函数处，从而可以不被擦除

   总之，原先类型信息::class.java不能够获得，需要通过反射，现在可以通过像泛型T一样去指定

   ```kotlin
   inline fun <refined T> Iterable<*>.filterByType():List<T>{
       return this.filter{it is T}.map{it as T}
   }
   //call
   List.filterByType<Int>()
   ```

4. 协变：对于只读集合子类型代替超类型，对于可变集合，无法协变，类型不安全

   (待补充)

5. 并发的补充：

   1. 协程始终应该式非阻塞的，所以我们不能在协程中调用runBlocking,
