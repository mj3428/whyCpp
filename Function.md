# 函数
## 函数声明
函数声明负责指定函数的名字、返回值的类型以及调用该函数所需的参数数量和类型。例如：
```
Elem* next_elem();  //  无须参数，返回Elem*
void exit(int); //  int 类型的参数，无返回值
double sqrt(double);  //  double类型的参数，返回double
```
参数传递的语义与拷贝初始化的语义完全一致。编译器检查实参的类型，如果需要的话还会执行隐式参数类型转换，例如:
```
double s2 = sqrt(2);  //  用实参double{2}调用函数sqrt()
double s3 = sqrt("three");  //  错误：函数sqrt()需要double类型的参数
```
### 函数定义
函数定义是特殊的函数声明，它给出了函数体的内容。例如:
```
void swap(int*, int*);  //  声明

void swap(int* p, int* q) //  定义
{
  int t = *p;
  *p = *q;
  *q = t;
}
```
函数的定义以及全部声明必须对应同一类型。不过，为了与C语言兼容，我们会自动忽略参数类型的顶层const。下面两条声明语句对应的同一个函数：
```
void f(int);  //  类型是void(int)
void f(const int);  //  类型是void(int)
```

### 返回值
下面的两个声明是等价的:
```
string to_string(int a);  //  前置返回类型
auto to_string(int a) -> string;  //  后置返回类型
```

后置返回类型的必要性源于函数模板声明，因为其返回类型是依赖于参数的。例如:
```
template<class T, class U>
auto product(const vector<T>& x, const vector<U>& y) -> decltype(x*y)
```

### constexpr函数
通常情况下，函数无法在编译时求值，因此也就不能在常量表达式中被调用。但是通过将函数指定为constexpr，我们就能向编译器传递这样的信息，即，
如果给定了常量表达式作为实参，则我们希望该函数能被用在常量表达式中。例如:
```
constexpr int fac(int n)
{
  return (n>1) ? n* fac(n-1):1;
}
constexpr int f9 = fac(9);
```
当constexpr出现在函数定义中时，它的含义是“如果给定了常量表达式作为实参，则该函数应该能用常量表达式中”。而当constexpr出现在对象定义中时，
它的含义是"在编译时对初始化器求值"。例如：
```
void f(int n)
{
  int f5 = fac(5); //  可能在编译时求值
  int fn = fac(n);  //  在运行时求值（n是变量）
  
  constexpr int f6 = fac(6);  //  必须在编译时求值
  constexpr int fnn = fac(n); //  错误：无法确保在编译时求值(n是变量)
  
  char a[fac(4)]; //  OK：数组的尺寸必须是常量，而fac()恰好是constexpr
  char a2[fac(n)];  //  错误：数组的尺寸必须是常量而n是一个变量
}
```
函数必须足够简单才能在编译时求值：constexpr函数包含一条独立的return语句，没有循环，也没有局部变量。同时，constexpr函数不能有副作用。
就是要纯函数（函数不能是void、不能有副作用、不能有if语句、不能有局部变量、不能有循环）
