# Android 设计模式与最佳实践

schedule to read 8.26-9.1

tag 8.26

## 工厂模式：

1. 用途：借助通用接口将逻辑和使用分开

2. 举例：我们定义面包作为接口，不同种类的面包例如法棍🥖作为接口面包的实现类，我们将这些实现类封装在工厂的统一函数中，有

   ``` java
   interface Bread{
       fun name()
   }
   class Baguette:Bread{
       @override fun name(){
           // to do 
       }
   }
   class BreadFactory{
       fun getBread(BreadType:String){
           val bread=when(BreadType){
               baguette -> Baguette()
               //...
               // in java, it equals with
               // Bread bread=new Baguette
           }
           
       }
   }
   ```

3. 更进一步，我们有抽象工厂，也就是比工厂模式更高一级的工厂，换言之，是制造工厂的工厂，我们从工厂生成器出发，使用抽象工厂类来决定实际创建的工厂类（这里我有一点疑惑，为什么需要这个AbstractFactory，我现在的想法是，这个抽象工厂实际上用于限制能产生的工厂的种类，并规定了每种工厂应该实现的方法，但是对于这种独立的工厂，会产生冗余，因为一个厂实际有效的只是一个必要的方法。进而继续思考，同样是制作面包，如果我们增加耦合性，比如说我们一个工厂自己生产以法棍加奶油组合的面包，一个生产以全麦加牛油果🥑为组合的面包，我们的`getFactory`现在的选择标准是工厂的编号，那么抽象工厂中的方法就都发挥了各自的用处）

   ``` kotlin
   class FactoryGenerator{
       fun getFactory(String factoryName):AbstractFactory{
           when(factoryName){
               breadFactory ->BreadFactory()
               fillFactory -> FillFactory()
           }
           
       }
   }
   abstract class AbstractFactory{
       fun Bread getBread(s:String)
       fun Fill getFill(s:String)
   }
   class BreadFactory:AbstractFactory{
       @override fun getBread(s:String):Bread{
           //to do
       }
       
       @override fun getFill(s:String):Fill{
        //null
       }
   
   }
   ```

4. kotlin 在使用的时候可以发现，如果接口仅仅是获得变量的信息，那么其实是没有必要通过接口来实现的，因为data class 已经具备了获取变量的功能

## 适配器模式：

1.  使用举例：RecycleView 的 adapter
2.  用途：接口不兼容
3.  关键：**包装旧接口，实现新接口**

``` kotlin
interface old{
    fun oldfunction()
    fun function()
}
class oldImplement:old{
    String oldVariable
    @override oldfunction(){
        return oldvariable
    }
    @override function(){
        //to do
    }
    
}
interface new{
    fun function()
}
class adapter(old:oldImplement):new{
    latinit var old:oldImplement
    //other variable  
    init{
        old=old
    }
    @override function(){
        //to do
    }
    
}

```

## 过滤器模式

1. 用途：设定标准，过滤对象
2. 产生过滤器对象

``` kotlin
interface filter{
    List<T> meetCriteria(List<T>)
}
class Filter1:filter{
    List<T>afterfilter
    @override meetCriteria(origin:List<T>){
        //operate the origin add the satisfied one to the afterfilter
        return afterfiler
    }
}
class Filter2:filter{
    List<T>afterfilter
    @override meetCriteria(origin:List<T>){
        //operate the origin add the satisfied one to the afterfilter
        return afterfiler
    }
}
// and 执行两边的filter
class FilterAnd(filter1:Filter1,filter2:Filter2):filter{
     @override meetCriteria(origin:List<T>){
         List<T>firstFilter=filter1.meetCriteria(origin)
         return filter2.meetCriteria(firstFilter)
    
}
// or 
class FilterOr(filter1:Filter1,filter2:Filter2):filter{
     @override meetCriteria(origin:List<T>){
     List<T>firstFilter=filter1.meetCriteria(origin)
         List<T>secondFilter=filter2.meetCriteria(origin)
         for(item in secondFilter){
             if(!firstFilter.contain(item)){
firstFilter.add(item)}
         }
         return firstFilter
     }
}
    
//使用: Filter1().meetCriteria(origin)
```



tag 8.27

## 装饰者模式

1. 用途：向现有类添加新的功能、属性
2. 例子：自选咖啡选项时，我们可能加奶、加浓缩、选杯的大小,这三种本质上都是装饰者的扩展，我们可以实现**管接**调用
3. 本质：**多态**  `Bread b=new Bagel() `

```kotlin
abstract  class Bread {
    open lateinit var description:String
    open var kcal:Int = 0
}
class Bagel : Bread() {
    init {
        description="Bagel"
        kcal=250
    }
}
class Bun: Bread() {
    init {
        description="Bun"
        kcal=150
    }
}
abstract class BreadDecoration:Bread() {
    // 所有的处理都是从这里扩展开
    abstract override var description:String
    abstract override var kcal:Int
}
class Butter(bread:Bread):BreadDecoration(){
    private var br:Bread = bread
    override var description: String=br.description+" butter"
    override var kcal: Int=br.kcal+50
    }
class Toast(bread:Bread):BreadDecoration(){
    private var br:Bread = bread
    override var description: String=br.description+" Toast"
    override var kcal: Int=br.kcal/2
}
fun main(){
    var b=Bagel()
    println(b.description+" "+b.kcal)
    var butter=Butter(b)
    println(butter.description+" "+butter.kcal)
    var toastAndbutter=Toast(butter)
    println(toastAndbutter.description+" "+toastAndbutter.kcal)
}
```



## 建造者模式

1. 用途：小对象构建大对象

   



## 组合模式

1. 用途：通过可能完全相同的代码处理单个组件或者组件的集合，从而完成层次的构建
2. 核心在于对于集合组件的处理，实现的是视图结构的复用：

``` kotlin
interface Component{
    fun add(component:Component)
    fun inflate()
}

class Leaf(var name:String):Component{
    override fun add(component: Component) {
    }

    override fun inflate() {
        println(name)
    }
}
class Composite(var name:String):Component{
    var list=ArrayList<Component>()
    override fun add(component: Component) {
        list.add(component)
    }

    override fun inflate() {
        for(component in list){
            component.inflate()
        }
    }
}
class Builder{
    val single1=Leaf("single1")
    val single2=Leaf("single2")
    val single3=Leaf("single3")
    fun layout1():Component{
        val composite1=Composite("conmposite1")
        composite1.add(single1)
        composite1.add(single2)
        return composite1
    }
    fun layout2():Component{
        val composite2=Composite("conmposite2")
        composite2.add(layout1())
        composite2.add(single3)
        return composite2
    }
}
fun main(){
    Builder().layout1().inflate()
    Builder().layout2().inflate()
}

```



## 观察者模式

1. 核心：主体接口，观察者接口

2. 观察者接口干什么：定义我的响应方法 ；取消注册的逻辑

   主题接口干什么：1. 注册/取消注册观察者实例（列表）的方法定义 2. 通知观察者 

3. 实例：主体类：observable 接口类：observer



## 行为型模式

1. 访问者模式：

   可以将进程用于一组不相关的对象，而不必去担心对象差异

   优势：访问者和访问者至今啊完全独立互不影响

```kotlin
各类商品具有接收访问者访问的功能 accept(visitor)，我们把这个具有的功能封装成接口，将商品作为实现这个接口的实例； 收银台是访问者接口的一个具体实例，作为收银台，我们可以去访问商品
interface Item{
    fun accept(visitor:Visitor):Double
}
class Can(var name:String,var price:Double):Item{
    override fun accept(visitor: Visitor):Double {
        return visitor.visit(this)
    }
}
class Bread(var name:String,var price:Double):Item {
    override fun accept(visitor: Visitor): Double {
        return visitor.visit(this)
    }
}
interface Visitor{
    fun visit(can: Can):Double
    fun visit(bread: Bread):Double
}
class Checkout:Visitor{
    override fun visit(can: Can): Double {
        println(can.name)
        return can.price
    }

    override fun visit(bread: Bread): Double {
        println(bread.name)
        return bread.price
    }
}
fun getPrice(lists:ArrayList<Item>):Double{
    var checkout=Checkout()
    var sum:Double=0.0
    for(item in lists){
        sum+=item.accept(checkout)
    }
    return sum
}
fun main(){
    var list= arrayListOf<Item>(Can("fishCan",18.2),Bread("raw Bread",29.0))
    println(getPrice(list))
}
```

2. 状态模式:

   有限状态机的实现：A----------->B（with the input of a）

   ``` kotlin
   interface Status{
       fun excute(context:Context,input:String)
   }
   //define two statuses A,B that inplement the status interface, implement the excute function depending on the input parameter.
   class Context(var state: Status){
    	 fun excute(context:Context,input:String){
            status.excute(this,input)
        }
   }
   fun main(){
       var context=Context(A())
       context.excute(input) //实际调用的是A()的excute
   }
   ```

   

