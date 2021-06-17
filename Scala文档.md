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

