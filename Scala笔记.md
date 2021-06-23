# 创始人

联邦理工学院的马丁·奥德斯基（Martin Odersky）于2001年开始设计Scala。

# Scala与Java

https://www.processon.com/diagraming/60cb70d607912975024fb081



# 函数柯力化&闭包

**说明**

函数柯里化：将一个接收多个参数的函数转化成一次接受一个参数的函数过程，可以简单的理解为一种特殊的参数列表声明方式。



```scala
object TestFunction {
  val sum = (x: Int, y: Int, z: Int) => x + y + z
  val sum1 = (x: Int) => {
​    y: Int => {
​      z: Int => {
​        x + y + z
​      }
​    }
  }
  val sum2 = (x: Int) => (y: Int) => (z: Int) => x + y + z
  def sum3(x: Int)(y: Int)(z: Int) = x + y + z
  def main(args: Array[String]): Unit = {
​    sum(1, 2, 3)
​    sum1(1)(2)(3)
​    sum2(1)(2)(3)
​    sum3(1)(2)(3)
      //用 =>连接每个参数等价于直接写多个括号传入参数
  }
}
```



闭包：就是**一个函数**和与**其相关的引用环境（变量）**组合的一个**整体**(实体)



```scala
//外部变量
var z: Int = 10
//闭包
def f(y: Int): Int = {
​      z + y
}
```



# 控制抽象

Scala中可自定义流程控制语句，即所谓的控制抽象。

案例：定义如下控制结构

scala中，代码块（block），可视为无参函数，作为 =>Unit类型的参数值。

```scala
object TestBlock {

  def loop(n: Int)(op: => Unit): Unit = {//传入的参数是一个无参函数op: => Unit，就是下面的println("test")
    if (n > 0) {
      op
      loop(n - 1)(op)
    }
  }

  def main(args: Array[String]): Unit = {
    loop(5) {
      println("test")
    }

  }
```



#  特质（Trait）-Java中的接口

Scala中，采用特质trait（特征）来代替接口的概念

多个类具有相同的特征（特征）时，就可以将这个特质（特征）独立出来，采用关键字trait声明

Scala中的trait中即可以有抽象属性和方法，也可以有具体的属性和方法，一个类可以混入（mixin）多个特质。



对比Java**

**Java 7**，那么接口中可以包含的内容有：

​	常量

​	抽象方法

**Java 8**，还可以额外包含有：

​	默认方法

​	静态方法

**Java 9**，还可以额外包含有：

​	私有方法

****

Scala引入trait特征，第一可以替代Java的接口，第二个也是对单继承机制的一种补充。



## **特质叠加**

scala可以多实现,如果子类实现了多个trait,这多个trait中都一个同名方法,参数列表也一样,此时子类的对象在调用该同名方法的时候默认会报错。可以在子类中通过重写同名方法解决报错问题。

子类重写同名方法之后,如果需要在方法体中调用父trait的同名方法,可以通过 super.同名方法/super[特质名].同名方法 的形式调用
	super.同名方法 调用的继承顺序最后一个trait的同名方法
	super[特质名].同名方法 调用的指定trait的同名方法
	如果子类继承的多个父trait都有一个共同的父trait,而且在方法中都有通过super调用同名方法,此时方法的调用顺序是**按照继承顺序从右向左**开始调用

## 特质的自身类型（Self Type）

提醒子类在继承/实现trait的时候必须提前继承/实现指定class/trait

《Programming in Scala》一书对“自身类型”（Self Type）的解释是：
“A self type of a trait is the assumed type of this,the receiver, to be used within the trait. Any concrete class that mixes in the trait must ensure that its type conforms to the trait’s self type.”

一个特质的“自身类型”是这个特质要求的“this”指针（引用）的“实际”类型，它会作为声明的“那个类型”的实例在这个特质中使用，从而可以让这个特质轻松地调用“那个类型”的字段和方法。所以，任何混入这个特质的具体类必须要同时保证它还是这个特质“自身类型”声明的“那个类型”。


案例：

```scala
trait Car {
    this: Engine => // self-type
    def drive() {
        start()
        println("Vroom vroom")
    }
    def park() {
        println("Break!")
        stop()
    }
}

val myCar = new Car extends DieselEngine

```

`this:Engine =>`就是我们说的自身类型声明，这告诉编译器，所有继承Car的具体类必须同时也是一个Engine，因为Car的业务代码里已经把this当成一个Engine在使用使用了（调用了它的`start`和`stop`方法），如果具体类无法满足Car的这一要求，编译就将失败。



## “蛋糕模式”（Cake Pattern）

蛋糕模式的思路是：假如A依赖B，我们用一个特质把被依赖方B包裹起来，我们可以叫它BComp,再用一个特质把依赖A方包裹起来，我们可以叫它AComp,我们会把AComp的自身类型声明为BComp, 这样我们可以在AComp中自由引用BComp的所有成员，这样从形式上就实现了把B注入到A的效果。此外，两个Comp都要有一个抽象的val字段来代表将来被实例化出来的A或B。最后就是粘合各个组件，这需要第三个类，它同时继承Acomp和Bcomp，然后重写Comp里要求的val字段（在Scala里除了重写方法，还可以重写字段），来实例化A和B，这样，一切就都粘合并实例化好了。回到我们的例子，看看第三版的实现吧，基于蛋糕模式的标准实现：

```scala
trait EngineComponent {

    trait Engine {
        private var running = false
        def start(): Unit = { /* as before */ }
        def stop(): Unit = {/* as before */ }
        def isRunning: Boolean = running
        def fuelType: FuelType
    }

    protected val engine : Engine

    protected class DieselEngine extends Engine {
        override val fuelType = FuelType.Diesel
    }

}

trait CarComponent {

    this: EngineComponent => // gives access to engine

    trait Car {
        def drive(): Unit
        def park(): Unit
    }

    protected val car: Car

    protected class HondaCar extends Car {
        override def drive() {
            engine.start()
            println("Vroom vroom")
        }
        override def park() { … }
    }
}

//tie them all together
object App extends CarComponent with EngineComponent with FuelTankComponent with GearboxComponent {
    override protected val engine = new DieselEngine()
    override protected val fuelTank = new FuelTank(capacity = 60)
    override protected val gearBox = new FiveGearBox()
    override val car = new HondaCar()
}

App.car.drive()

App.car.park()

```

最后生产出的这个App.car是一辆Honda，柴油发动机，60升的油箱，5级变速。这一版实现繁杂了很多，但是有这样几个重点：

​	HondaCar在实现过程中使用到了Engine,但是它即没有继承Engine也没实例化一个Engine字段，这和传统的依赖注入在效果上是无差别的，实际上就是实现了把Engine注入到HondaCar的目标。

​	粘合互相依赖的组件的过程发生在App的定义中。所有的组件都预留了protected val的字段，留待组装粘合的时候实例化。

​	主动要去依赖其他组件的组件必定要将依赖的组件声明成自身类型，以便在组件内部自由引用被依赖组件的成员和方法。

### 蛋糕模式总结


蛋糕模式完全依赖语言自身的特性，没有外部框架依赖，类型安全，可以获得编译期的检查。但缺点也是很明显的，代码复杂，配置不灵活。就个人而言，不太会选择使用。

