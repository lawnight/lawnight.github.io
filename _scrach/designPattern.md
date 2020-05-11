简单常用的观察者，单列，工厂，适配器，模板，策略的设计模式就不提了，记录下比较容易忘掉的设计模式。

## 访问者模式

访问模式可以增加已有类新的操作方法而不需要改变已有类的代码。遵循了开放关闭原则。
特点

- 数据和算法分离。将自己委托给vistor来操作，`vistor.vist(this)`。
- 通过double dispatch 给一组继承类添加新的virtual functions。

当你对固定的一组famaly classes，你需要经常增加一组不同的操作时，可以考虑用访问者模式

以下面java代码为例，CarElement将操作派发给Vistor，所以vistor需要对每个具体的子类实现visit方法

![1](/assets/dp1.png)
最后可以对car定义不同的操作。
```java
//然后通过继承vistor来提供不同的实现，已达到扩展object方法的目的。
public class VisitorDemo {
    public static void main(final String[] args) {
        Car car = new Car();
        car.accept(new CarElementPrintVisitor());
        car.accept(new CarElementDoVisitor());
    }
}
```

scala是一门多范式编程语言，可以面向对象，也可以函数式编程。运行在java虚拟机上，和java代码兼容。其中的`implicit classes`，能够对已有的class增加新的方法。其实现方法并不是完全的访问者模式，但是有访问者的影子。定义新的方法，传入访问者对象，然后扩展访问者的操作方法。

```scala
//a.scala
object some {
    //对String增加新的方法
  implicit class improvement(s: String){
    def xxxx(): Unit ={
      println(s + "new add function")
    }
  }
}
//b.scala
//必须import
import some.improvement
object Main {
  def main(args: Array[String]): Unit = {
    "abc".xxxx();          //语法糖，实际就是调用some.improvement.xxx("abc")
  }
}
```



## 修饰器模式