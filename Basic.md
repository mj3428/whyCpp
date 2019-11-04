# 基础
## 结构
构建一个新类型第一步是把所需的元素组织成一种数据结构
```
struct Vector{
  int sz; //元素数量
  double* elem; //指向元素指针
};
```

接着我们需要让V指向某些元素
```
void vector_init(Vector& v, int s)
{
  v.elem = new double[s]; //分配一个数组，它包含s个double值
  v.sz = s;
}
```

Vector&中的&符号指定我们通过非常量引用的方式传递v，这样vector_init()就能修改传入其中的向量了。  
而new运算符从一块名为自由存储的区域中分配内存。
**应用：** 
```
double read_and_sum(int s)
{
  Vector v;
  vector_init(v,s); // 为v分配s个元素
  for (int i=0;i!=s; ++i)
    cin>>v.elem[i]  // 读入元素
    
  double sum = 0;
  for (int i=0;i!=s;++i)
    sum += v.elem[i]; // 计算元素的和
  return sum;
}
```

## 类
类含有一些列成员，可能是数据、函数或者类型。类的public成员定义该类的接口，private成员则只能通过接口访问。例如：
```
class Vector{
public:
  Vector(int s) :elem{new double[s]}, sz{s}{} // 构建一个Vector
  double& operator[](int i){ return elem[i];} // 通过下标访问元素
  int size(){return sz;}
private:
  double* elem; //指向元素的指针
  int sz; //元素的数量
};
```

**解析:**  
* 与类名同名的函数成为构造函数，作用是初始化类的对象。而上述的Vector(int)的构造函数里面int指定元素的数量。  
* `:elem{new double[s]},sz{s}`首先从自由空间获取s个double类型的元素，并用一个指向这些元素的指针初始化elem；然后用s初始化sz。  
* 访问元素的功能是由一个下标函数提供的的，这个函数名为operator[]，它的返回值是对相应元素的引用(double&);  

## 枚举
```
enum class Color{red, blue, green};
enum class Traffic_light{green, yellow, red};

Color col = Color::red;
Traffic_light light = Traffic_light::red;
```
enum class中class是指明了枚举是强类型的  
变量名相同，所以枚举的目的是为了防止冲突；**比如:**  
```
Color x = red;  // 错误:哪个red?
Color y = Traffic_light::red;  // 错误:这个red不是color的对象
Color z = Color::red;  //正确
```

并且不能隐式混用类型
```
int i = Color::red; //错误:Color::red不是整型
Color c = 2;  //错误:2不是一个Color对象
```
## 模块化
### 分离编译
一般情况下，我们把描述接口的声明放置在一个特定的文件中，文件命往往指示模块的预期用途。
```
class Vector{
public:
  Vector(int s) 
  double& operator[](int i)
  int size(){return sz;}
private:
  double* elem; //elem指向一个数组，该数组包含sz个double
  int sz; 
};
```
这段声明被置于Vector.h中，我们称这种文件为头文件，用户将其包含(include)进程序以访问接口。**例如：**  
// user.cpp
```
# include "Vector.h"  //获得Vector接口
# include <cmath> //获得标准库数学函数接口，其中含有sqrt()
using namespace std;
double sqrt_sum(Vector& v)
{
  double sum = 0;
  for (int i=0;i!=v.size();++i)
    sum+=sqrt(v[i]);  // 平方根的和
  return sum;
}
```
