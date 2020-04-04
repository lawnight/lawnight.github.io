## 访问者模式

访问模式可以增加已有类新的操作方法而不需要改变已有类的代码。遵循了开放关闭原则。


```java
class Base{
    public accept(Vistor visitor){
        vistor.visit(this)
    }

Base o = new Base()
}
//然后通过继承vistor来提供不同的实现，已达到扩展object方法的目的。
o.accept(new Vistor1())
o.accept(new Vistor2())
```