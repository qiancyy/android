# Android 设计模式与最佳实践

schedule to read 8.26-9.6

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

