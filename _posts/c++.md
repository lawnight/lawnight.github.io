

## 运算符重载和自动类型转换
### 运算符重载

运算符重载可以定义成成员函数和全局函数。内置类型表达式的运算符是不能改的，用户自定义类型可以重载运算符。

运算符重载不能改变运算符的参数个数和优先级。运算符重载本质上也是函数调用，是一种语法糖。



### 全局操作符重载和友元

当左侧的运算符是别的对象的时候，可以重载全局运算符。并且在声明中标识friend，表示获得对类私有成员的访问权。`<<`左移操作符，经常被重载为表示输出的意思。

```C++
class A{
    int data[5];
    friend void g(A&,int);//全局函数
    friend void Y::f(x&);//成员函数
    friend class Z;
    friend ostream& operator<<(ostream& os,const A&a);//全局函数
}

ostream& operator<<(ostream& os,const A&a){
    for(int i=0;i<5;i++>){
        os<<5;
    }
    return os;
}

uint32 PackedHeader = 0;	
Reader << PackedHeader;
```
### 自动类型转换

表达式或函数调用在面对不合适的类型，编译器会尝试进行类型转换。有两种方式：指定类型的构造函数和重载的运算符。

```c++
class Pack{
    Handle(int*){}
}
void incomming(Pack p){}

int* Data = (int*)InData;
incomming(Data, Count)//自动先调用Pack(data)构造函数，再传参数。方式一

class newPack{
    int* data;
    operator Pack() const{
        return Pack(data)
        };
}
newPack pack
incomming(pack);//调用newPack的Pack()转换需要的类型。方式二
```

## 模板

模板分为类模板和函数模板。模板让我们不用手动复制代码，编译器帮我们为不同类型复制代码，并修改类名或函数名。模板有三种类型的参数：类型 ；编译时参数；其它模板。并且都可以指定模板值。
```c++
template<class T=int,int size=100> class Array{
    T data[size]
    void push();
}
//typename和class在这里是等价的
template<typename T ,template<class> class seq =Array> class Container{
    //当模板参数是其它模板时，需要对它实例化
    Array<T> array;
    //这里typename是告诉编译器，InterType是类型T的嵌入类型。
    typename T::InterType type;
}
//如果在头文件外定义类成员函数，也需要加上模板声明
template<class T=int,int=100>
void Array:push(){

}

```

模板的实例化都是编译期间，不同的值实例化，会产生不同的类。例如Array<int,10>和Array<int,20>是不同的类。函数模板的实例化可以不用显式指定类型，编译器根据传入的参数推导出类型。在多个重载的函数模板中，会选择特化程度最高的函数来进行特化。
### 模板特化
特化（specialization）模板，可以为模板提供代码来使其特化。显式特化可以特化所有或者部分的模板参数，但都应该实现之前定义的全部接口。。如下特化了Array模板在持有bool类型时，可以用位操作来优化性能。这提供了一种“重载”类模板的方式
```c++
//全特化
template<> class Array<bool,100>{
    BitSet bitset;
    //实现其它方法
}

//半特化
template<int size=100> class Array<bool,size>{
    BitSet bitset;
    //实现其它方法
}
```
### 其它
模板有十分多的新奇用法。模板的特化甚至能在编译时实现循环和if-else的判定。提供了编译时编程的可能性。
```c++
template<int n> struct Factorial{
    enum{val = Factorial<n-1>::val*n};
};
template<> struct Factorial<0>{
    enum{val = 1};
};
int main(){
    cout << Factoral<12>::val << enl; //会在编译期间算出常数479001600
}
```

C++11的新特性--可变模版参数。并且用模板的方式递归调用展开参数包 。
```c++
template<typename... ParamTypes>
static void SendParams(FControlChannelOutBunch& Bunch, ParamTypes&... Params) {}

template<typename FirstParamType, typename... ParamTypes>
static void SendParams(FControlChannelOutBunch& Bunch, FirstParamType& FirstParam, ParamTypes&... Params)
{
    Bunch << FirstParam;
    SendParams(Bunch, Params...);
}
```

解析模板编译器会做很多推导，当有二义性的时候，就会编译报错。