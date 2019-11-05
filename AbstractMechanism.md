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
