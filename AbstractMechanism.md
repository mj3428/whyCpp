# 抽象机制
## 类
### 具类下的一种算术类型
一种“经典的用户自定义算术类型”是complex
```
class complex{
  double re,im; //  表现形式：两个双精度浮点数
public:
  complex(double r, double i):re{r},im{i}{} //  用两个标量构建该复数
  complex(double r):re{r}, im{0}{}  // 用一个标量构建该复数
  complex() :re{0}, im{0}{} //  默认的复数是{0,0}
  
  double real() const{return re;}
  void real(double d){re=d;}
  double imag() const{return im;}
  complex& operator+=(complex z){re+=z.re,im+=z.im;return *this;} //  加到re和im上后返回
  complex& operator-=(complex z){re-=z.re,im-=z.im;return *this;}
  complex& operatpr*=(complex); //   在类外的某处进行定义
  complex& operatpr/=(complex); //   在类外的某处进行定义
}
```
complex必须足够高效，否则依然没有实用价值。这意味着我们应该将简单的操作设置成内联的。也就是说，在最终生成的机器代码中，一些简单的操作（如
构造函数、+=和imag()等）不能以函数调用的方式实现。定义在类内部的函数默认是内联的。一个工业级的complex（就像标准库中的那个一项）必须精心实现，
并且恰当地使用内联。  
无须实参就可以调用的构造函数称为默认构造函数，complex()是complex的默认构造函数。通过定义默认构造函数，可以有效防止该类型的对象未初始化。  
在负责返回复数实部和虚部的函数中,const说明附表示这两个函数不会修改所调用的对象。  
很多有用的操作并不需要直接访问complex的表现形式，因此它们的定义可以与雷达定义分离开来:  
```
complex operator+(complex a,complex b){return a+=b;}
complex operator-(complex a,complex b){return a-=b;}
complex operator-(complex a){return {-a.real(),-a.imag()};} //    一元负号
complex operator*(complex a,complex b){return a*=b;}
complex operator/(complex a,complex b){return a/=b;}
```

以传值方式传递实参实际上是把一份副本传递给函数，因此我们修改形参（副本）不会影响主调函数的实参，并可以将结果作为返回值。  
我们可以像下面这样使用complex
```
void f(complex z)
{
  complex a{2.3}; //  用2.3构建出{2.3,0.0}
  complex b {1/a};
  complex c {a+z*complex{1,2.3}};
  if(c!=b)
    c = -(b/a)+2*b;
}
```
容器( container)是指一个包含若干元素的对象,因为Vector的对象都是容器,所以我们称Vector是一种容器类型而,
它还是存在一个致命的缺陷:它使用new分配了元素但是从来没有释放这些元素。这显然是个糟糕的设计,因为尽管 C++定义了一个垃圾回收的
接口,可将未使用的内存提供给新对象,但C++并不保证垃圾收集器总是可用的。在某些情况下你不能使用回收功能,而且有的时候出于逻辑或性能的
考虑你宁愿使用更精确的资源释放控制。因此,我们迫切需要一种机制以确保构造函数分配的内存一定会被销毁,这种机制就叫做析构函数(destructor):  
```
class Vector{
private:
  double* elem; //  elem指向一个包含sz个double的数组
  int sz;
public:
  Vector(int s):elem{new double[s]}, sz{s}  //  构造函数:请求资源
  {
    for (int i=0;i!=s;++i) elem[i]=0; //  初始化元素
  }
}
  
```
