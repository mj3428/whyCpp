# 抽象机制1
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
  Vector(){delete[] elem;}  // 析构函数：释放资源
  
  double& operator[](int i);
  int size() const;
};
```

析构函数的命名规则是一个求补运算符~后接类的名字，从含以上来说它是构造函数的补充。Vector的构造函数使用new运算符从自由存储（也称为堆或
动态存储）分配一些内存空间，析构函数则使用delete运算符释放该空间以达到清理资源的目的。这一切都无须Vector的使用者干预，他们只需要像
使用普通的内置类型变量那样使用Vector对象就可以了。  
```
void fct(int n)
{
  Vector v(n);  // ...使用v...
  {
    Vector v2(2*n);
    //...使用v和v2...
  } //v2在此处被销毁
  // ...使用v...
} //  v在此处被销毁
```
### 具类里初始化容器
两种简洁途径:  
* 初始化器列表构造函数(Initializer-list constructor):使用元素的列表进行初始化;
* push_back():在序列的末尾添加一个新元素.
它们的声明形式如下所示:  
```
class Vector{
public:
  Vector(std::initializer_list<double>);  // 使用一个列表进行初始化
  // ...
  void push_back(double); // 在末尾添加一个元素，容器的长度加1
  // ...
};
```
其中，push_back()可用于添加任意数量的元素。例如:
```
Vector read(istream& is)
{
  Vector v;
  for (douoble d; is>>d;) // 将浮点值读入d
    v.push_back(d); // 把d加到v当中
  return v;
}
```
### 抽象类型
complex和Vector等类型之所以被称为具体类型(concrete type),是因为它们的表现形式属于定义的一部分。在这一点上,它们与内置类型很相似。
相反,抽象类型(abstract type)则将使用者与类的实现细节完全隔离开来。为了做到这一点,我们分离接口与表现形并且放弃了纯局部变量。  
首先，我们为Container类设计接口，Container类可以看成是比Vector更抽象的一个版本:
```
class Container{
public:
  virtual double& operatorp[](int) = 0; //纯虚函数
  virtual int size() const = 0; //常量成员函数
  virtual Container(){} //析构函数
};
```

关键字virtual的意思是“可能随后在其派生类中重新定义”。关键字virtual声明的函数称为虚函数。Container的派生类负责为Container接口提供
具体实现。看起来有点奇怪的=0说明该函数是纯虚函数，意味着Container的派生类必须定义这个函数。含有纯虚函数的类称为抽象类(abstract class)  
Container的用法是:  
```
void use(Container& c)
{
  const int sz = c.size();
  
  for (int i=0;i!=sz;++i)
    cout<<c[i]<<'\n';
}
```

一个容器为了实现抽象类Container接口所需的函数，可以使用具体类Vector:
```
class Vector_container: public Container{ // Vector_container实现了Container
  Vector v;
public:
  Vector_container(int s):v(s){}  //  含有s个元素的Vector
  Vector_container(){}
  
  double& operator[](int i){return v[i];}
  int size() const{return v.size();}
}
```
public可读作“派生自”或“是 的子类型”。我们说Vector_container类派生(derived)自Container类,而Container类是Vector-container类的
基类(base),还有另外一种叫法,分别把Vector_container和Container叫做子类(subclass)和超类(superclass)。派生类从它的基类继承成员,所
以我们通常把基类和派生类的这种关联关系叫做继承(inheritance)。  
成员operatorD()和size()覆盖(override)了基类Container中对应的成员。析构函数~Vector_container()则覆盖了基类的析构函数Container()。
注意,成员v的析构函数(~Vector())被其类的析构函数(~Vector-container())隐式调用.  
因为use()只知道Container的接口而不了解Vector_container，因此对于Container的其他实现，use()仍能正常工作。例如:
```
class List_container: public Container{// List_container实现了Container
  std::list<double> ld; //  一个double类型的标准库list
public:
  List_container(){}  //  空列表
  List_container(initializer_list<double>il):ld{il}{}
  ~List_container(){}
  double& operatorp[](int i);
  int size() const {return ld.size();}
};
double& List_container::operator[](int i)
{
  for (auto&:ld){
    if(i==0)return x;
    --i;
  }
  throw out_of_range("List container");
}
```

在上述代码中，类的表现形式是一个标准库list<double>。一般情况下，我们不会用list实现一个带下标的容器，毕竟list取下标的性能很难与vector相比。  
我们可以通过一个函数创建一个List_container，然后让use()使用它:  
```
void h()
{
  List_container lc ={1,2,3,4,5,6,7,8,9};
  use(lc);
}
```

这段代码的关键点是use(Container&)并不清楚它的实参是Vector_container, List_container,还是其他什么容器,它也根本不需要知道。
它只要了解Container定义的接口就可以了。因此,不论List_container的实现发生了改变还是我们使用了Container的一个全新派生类,
都需要i编译use(Container&)  
### 虚函数
进一步思考Cointainer的用法:
```
void use(Container& c)
{
  const int sz = c.size();
  for (int i=0;i!=sz;++i)
    cout << c[i] << '\n';
}
```

一个有趣的问题是:use()中的c[i]是如何解析到正确的operator[]()的？当h()调用品use()时，必须调用Lisr_container的operator[]();而当g()
调用use()时，必须调用Vector_container的operator[]()。要想达到这种效果，Container对象就必须包含一些有助于它在运行时选择正确函数的信息。
### 类层次
Container是一个非常简单的类层次的例子,所谓类层次(class hierarchy)是指通过派生(如:public)创建的一组类,在框架中有序排列。
我们使用类层次表示具有层次关系的概念,比如“消防车是卡车的一种,卡车是车辆的一种”以及“笑脸是一个圆,圆是一个形状”。  
例如，Circle类派生自Shape类。要想把上面这个简单的图例写成代码，我们首先需要说明一个类，令其定义所有这些形状的公共属性:
```
class Shape{
public:
  virtual Point center() const=0; //  纯虚函数
  virtual void move(Point to)=0;
  
  virtual void draw() const=0;  //  在当前“画布”上绘制
  virtual void rotate(int angle)=0;
  
  virtual ~Shape(){}  //  析构函数
  //...
};
```

基于上面的定义，我们就能编写函数令其操纵由形状指针组成的向量了:
```
void rotate_all(vector<Shape*>& v, int angle) //  将v的元素按照指定角度旋转
{
  for (auto p:v)
    p->rotate(angle);
}
```

要定义一种具体的形状，首先必须指明它是一个Shape,然后再规定其特有的属性（包括虚函数）：
```
class Circle:public Shape{
public:
  Circle(Point p, int rr);  //  构造函数
  Point center() const{return x;}
  void move(Point to){x=to;}
  
  void draw() const;
  void rotate(int){}  //  一个简单明了的示例算法
private:
  Point x;  //  圆心
  int r;  //  半径
}
```

Shape和Circle涉及的语法知识并不比Container和Vector_container多多少，但是我们可以继续构造:
```
class Simley: public Circle{  //  使用Circle作为笑脸的基类
public:
  Smile(Point p,int r):Circle{p,r},mouth{nullptr}{}
  ~Smiley()
  {
  delte mouth;
  for (auto p:eyes) deletep;
  }
  void move(Point to);
  void draw() const;
  void rotate(int);
  void add_eye(Shape* s){eye.push_back(s);}
  void set_mouth(Shape* s);
  virtual void wink(int i); //  眨眼数i
  // ...
private:
  vector <Shape*>eyes;  //  通常包含两只眼睛
  Shape* mouth;
};
```

成员函数push_back()把它的实参添加给vector(此处是eyes),每次将向量的长度加1.  
通过调用Smiley的基类(Circle)的draw()和Simley的成员(eyes)的draw()来定义Smiley::draw()：
```
void Smiley::draw()
{
  Circle::draw();
  for (auto p:eyes)
    p -> draw();
    mouth->draw();
}
```
