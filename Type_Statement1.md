# 类型与声明
## 类型
### 布尔型
如果你既想使用{}初始化器列表防止窄化转换的发生，同时又确实想把int转换成bool，则可以显式声明如下:
```
void f(int i)
{
  bool b {i!=0};
}
```

在算术逻辑表达式和位逻辑表达式中, bool被自动转换成int,编译器在转换后的值上执行整数算术运算以及逻辑运算。如果最终的计算结果需要转换回bool,则
与之前介绍的一样, 0转换成false而非0值转换成true。  
如有必要，指针也能被隐式地转换成bool，其中，费控指针对应true，值为nullptr的指针对应false。例如：
```
void g(int *p)
{
  bool b = p; // 窄化成true或false
  bool b2 {p!=nullptr}; // 显式地检查指针是否为非空
  
  if (p){ // 等价于 p!=nullptr
    // ..
  }
}
```

与if (p!=nullptr)相比,我觉得if(p)更好,它不但简洁而且可以直接表达“p是否有效”的 "含义,使用if(p)也不太容易出错.  

### 字符类型
char16_t:该类型存放UTF-16等16位字符集  
char32_t:该类型存放UTF-32等32位字符集  
我们可以在字符类型上执行算术运算和为逻辑运算,例如:
```
void digits()
{
  for (int i=0;i!=10;++i)
    cout << static_cast<char>('0'+i);
}
```
上面的代码把10个阿拉伯数字输出到cout。字符字面值常量0,先转换成它对应的整数值, ,再与i相加;所得的int再转回char并被输出到cout。
'0'+i得到的结果本来是一个int,因此如果不加上static-cast的话,输出的结果将会是48, 49 ...而不是0, 1...

我们不能混用char\signed char\unsigned char这3种字符类型的指针,如:
```
void f(char c,signed char sc,unsigned char uc)
{
  char* pc = &uc; //错误:不存在对应的指针转换规则
  signed char* psc = pc;  //  错误:不存在对应的指针转换规则
  unsigned char* puc = pc;  //  错误:不存在对应的指针转换规则
  psc = puc;  //  错误:不存在对应的指针转换规则
}
```
3种char类型的变量可以互相赋值，但是把一个特别大的赋值给带符号的char是未定义的行为，例如:
```
void g(char c, signed char sc, unsigned char uc)
{
  c = 255;  //如果普通的char是带符号的且占8位，则该语句的行为依赖于具体实现
  c = sc; // OK
  c = uc; // 如果普通的char是带符号的且uc的值特别大，则该语句的行为依赖于具体实现
  sc = uc;  // 如果uc的值特别大，则该语句的行为依赖于具体实现
  uc = sc;  // OK转换成无符号类型
  sc = c; // 如果普通的char是无符号的且uc的值特别大,则该语句的行为依赖于具体实现
  uc = c; // ok 转换成无符号类型
}
```

### 整数类型
后缀U用于显式指定unsigned字面值常量，与之类似，后缀L用于显式指定long字面值常量。例如3U的类型时unsigned int而3L的类型时long int。  
多个后缀可以组合在一起使用，例如:`cout << 0xF0UL << '' << 0LU << '\n';`  
**前缀与后缀:**  
```
1UL //  unsigned long
2UL //  unsigned long
3ULL  //   unsigned long long
4LLU  //  unsigned long long
5LUL  //  错误
```
同样，后缀l和L也能用于表示浮点数字面值常量，表达的类型是long double
```
1L  // long int
1.0L  // long double
```

### void
从语法结构上来说,void属于基本类型。但是它只能被用作其他复杂类型的一部分，不存在任何void类型的对象。void有两个作用:一是作为函数的返回类型
用以说明函数不返回任何实际的值；二是作为指针的基本类型部分以表明指针所指对象的类型未知。
```
void x; //  错误:不存在void类型的对象
void& r;  //  错误：不存在void的引用
void f(); //  函数f不返回任何实际的值
void* pv; //  指针所指的对象类型未知
```
当我们声明一个函数时,必须指明返回结果的数据类型。从逻辑上来说,如果某个函数不返回任何值,也许我们会希望直接忽略掉返回值部分。
但其实这种想法并不可行,它会违反 C++的语法规则( §iso.A)。因此,我们使用void表示函数的返回值为空,此时void可以看成是一种“伪返回类型”  

### 类型尺寸
所有C++对象的尺寸都可以表示成char尺寸的整数倍,因此如果我们令char的尺寸 ,为1,则使用sizeof运算符就能得到任意类型或对象的尺寸。
下面是C++对于基本类型尺寸的一些规定:  
* 1=sizeof(char)≤sizeof(short)≤sizeof(int)≤sizeof(long)≤sizeof(long long)
* 1≤sizeod(bool)≤sizeof(long)
* sizeof(char)≤sizeof(wchar_t)≤sizeof(long)
* sizeof(float)≤sizeof(double)≤sizeof(long double)
* sizeof(N)=sizeof(signed N)=sizeof(unsigned N)  

其中,最后一行的N可以是char, short, int, long或者long long, C++规定char至少占8位, short至少占16位, long至少占32位。
char应该能存放机器字符集中的任意字符,它的实际类型依赖于实现并确保是当前机器上最适合保存和操作字符的类型。通常情况下, char占据一个8位的字节。
与之类似, int的实际类型也是依赖于实现的,并确保是当 ,前机器上最适合保存和操作整数的类型。int通常占据一个4字节(32位)的字。
上面这些假设是比较恰当的,但是我们很难做更多设定。比如,我们只能说char “通常”占据8位,因为确实也存在char占32位的机器。
又比如我们决不能假定int和指针的尺寸一样大,因为在很多机器上("64位体系结构”)指针的尺寸比整数大。  

如果你需要使用某种特定尺寸的整数类型(比如16位的整数),应该事先#include标准头文件中定义了很多类型,例如:
```
int16_t x {0xaabb}; //  2字节
int64_t xxxx {0xaaaabbbbccccdddd};  //  8字节
int_least16_t y;  //  至少2字节(与int类似)
int_least32_t yy; //  至少4字节(与long类似)
int_fast32_t z; //  是最快的整数类型，至少包含4个字节
```

### 对齐
对齐只有在涉及对象布局的问题中比较明显:有时候我们会让struct包含一些“空洞”以提升整齐程度。  
alignof()运算符返回实参表达式的对齐情况，例如:
```
auto ac = alignof('c'); //  char的对齐情况
auto ai = alignof(1); //  int的对齐情况
auto ad = alignof(2.0); //  double的对齐情况

int a[20];
auto aa = alignof(a); //  int的对齐情况
```

## 声明
在这种视角下,我们尽量用声明语句组成程序的接口。其中,同一个声明可以在不同文件中重复出现。负责申请内存空间的定义语句不属于接口。
```
char ch;  //  为一个char类型的变量分配内存空间并赋初值0
auto count = 1; //  为一个int类型的变量分配内存空间并赋初值1
const char* name = "Njal";  //  为一个指向char的指针分配内存空间
    //  为字符串字面值常量"Njal"分配内存空间
    //  用字符串字面值常量的地址初始化指针
struct Date {int d,m,y;}; //  Date是一个struct,它包含3个成员
int day(Date* p){return p->d;}  // day是一个函数，它执行某些既定的代码
using Point = std::complex<short>;  //  Point是类型std::complex<short>的别名
```

在上面这些声明语句中，只有3个不是定义:
```
double sqrt(double);  //  函数声明
extern int error_number;  //  变量声明
struct User;  //  类型名字声明
```

也就是说，要想使用它们对应的实体，必须先在其他某处进行定义，例如:
```
douyble sqrt(double d){/*...*/}
int error_number = 1;
struct User {/* ... */};
```
在同一实体的所有声明中，实体的类型必须保持一致，因此，下面的小片段包含两处错误：
```
int count;
int count;  //  错误:重定义
extern int error_number;
extern shor error_number; //  错误：类型不匹配
```
有的定义语句为它们定义的实体显式地赋值，例如:
```
struct Date{int d,m,y;};
using Point = std::complex<short>;  //  Point 是类型std::complex<short t>的别名
int day(Date* p){return p->d;}
const double pi {3.1415926535897};
```
### 声明的结构
* 可选的前置修饰符（比如static和virtual）
* 基本类型（比如vector<double>和const int）
* 可选的声明符，可包含一个名字（比如p[7],n和*(*)[]）
* 可选的后缀函数修饰符（比如const和noexcept）
* 可选的初始化器或函数体(比如={7,5,3}和{return x;})  

|**声明运算符**|||
| :--- | :--- | :--- |
|前缀|* |指针|
|前缀|* const |常量指针|
|前缀|* volatile |volatile指针|
|前缀|& |左值引用|
|前缀|&& |右值引用|
|前缀|auto |函数（使用后置返回类型）|
|后缀| [] |数组|
|后缀| () |函数|
|后缀| -> |从函数返回|  

后缀声明符的绑定效果比前缀声明符更紧密，所以char* kings[]是char指针的数组，而char(* kings)[]是指向char数组的指针

### 声明多个名字
在声明语句中，运算符只作用于紧邻的一个名字，对于后续的其他名字是无效的。例如：
```
int* p,y; //  准确的含义是int* p;int y;而非int* y;
int x, *q;  // int x;int* q;
int v[10], *pv; //  int v[10];int* pv;
```
### 名字
第一个字符必须是字母  
**一些有效的标识符如下:**
```
hello this_is_a_most_unusually_long_identifier_that_is_better_avoided
DEFINED foO bAr u_name HorseSense
var0 var1 CLASS _class ___
```

**无效的标识符:**
```
012   a fool  $sys  class 3var
pay.due foo~bar .name if
```
以下划线开头的非局部名字表示具体实现及运行时环境中的某些特殊功能,应用程序中不应该使用这样的名字。类似地包含双下划线（__） 的名字
和以下划线开头紧跟大写字母的名字（比如_Foo）都有特殊用途

### 作用域
* 局部作用域(local scope):函数或lambda表达式中声明的名字称为局部名字(local name)。局部名字的作用域从声明处开始，到声明语句所在的块结束
为止。其中块(block)是指用一对{}包围的代码片段。对于函数和lambda表达式最外层块来说，参数名字是其中的局部名字。  
* 类作用域(class scope)：如果某个类位于任意函数、类和枚举类或其他名字空间的外部，则定义在该类中的名字称为成员名字(member name)
或类成员名字(class member name)。类成员名字的作用域从类声明的{开始，到类声明结束为止。  
* 名字空间作用域(namespace scope)：如果某个名字空间位于任意函数，lambda表达式、类和枚举类或其他名字空间的外部，则定义在该名字空间中
的名字为名字空间成员名字(namespace member name)。名字空间成员名字的作用域从生命语句开始，到名字空间结束为止。名字空间名字能被其他翻译
单元访问。  
* 全局作用域(global scope):定义在任意函数、类、枚举类、和名字空间之外的名字称为全局名字(global name)。全局名字的作用域从声明处开始，
到声明语句所在的文件末尾为止。全局名字能被其他翻译单元访问。从技术上来说，全局名字空间也是一种名字空间。因此，我们可以把全局名字看成是一种
特殊的名字空间成员名字。  
* 语句作用域(statement scope)：如果某个名字定义在for\while\if\switch语句的()部分，则改名字位于语句作用域中。它的作用域范围从声明处开始，
到语句结束为止。语句作用域中的所有名字都是局部名字。  
* 函数作用域(function scope)：标签的作用域是从声明它开始到函数体结束  
