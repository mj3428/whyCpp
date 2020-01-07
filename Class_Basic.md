# 类基础
简要概括:
* 一个类就是一个用户自定义类型
* 一个类由一组成员构成。最常见的成员类别是数据成员和成员函数。
* 成员函数可定义初始化（创建）、拷贝、移动和清理（析构）等语句。 
* 对对象使用.(点)访问成员，对指针使用->（箭头）访问成员。 
* 可以为类定义运算符，如+、！和[]。
* 一个类就是一个包含其成员的名字空间。
* public成员提供类的接口，private成员提供实现细节。
* struct是成员默认为public的class。  

## class和struct
`class X {...};`称为类定义，它定义了一个名为X的类型。由于历史原因，类定义常常被称为类声明(class declaration)。这样叫他的原因是，与
其他法并非定义的C++声明类似，我们可以在不同源文件中使用#include重复定义而不会违反单一定义规则。  
struct就是一个成员默认为公有的类，即`struct S{/*...*/}`如果认为一个类是“简单数据结构”，更喜欢使用struct,如果认为一个类是“具有不变式的
真正类型”，会使用class。即使是对struct而言，构造函数和访问函数也是非常有用的，但它们只是一种简写而非不变式的保证。  
class的成员默认是私有的。  

## 构造函数
使用像init()这样的函数为类对象提供初始化功能既不优雅也容易出错。构造函数的本质是构造一个给定类型的值。其显著特征是与类具有相同的名字。例如:
```
class Date{
  int d, m, y;
public:
  Date(int dd, int mm, int yy); //  构造函数
  // ...
};
```