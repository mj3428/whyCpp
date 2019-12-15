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


### 局部变量
如果我没把局部变量声明成static，则在函数的所有调用中都会讲使用唯一的一份静态分配的对象，该对象在线程第一次达到它的定义出时被初始化，例如：
```
void f(int a)
{
  while (a--){
    static int n = 0; //  只初始化一次
    int x = 0;  //  每次调用f()都会初始化
    
    cout << 'n == ' << n++ << ",x == " << x++ << '\n';
  }
}

int main()
{
  f(3);
}

结果:
 n == 0, x==0
 n == 1, x==0
 n == 2, x==0
```

C++实现必须用某种雾锁机制确保局部static变量的初始化能被正确执行。递归地初始化一个局部static变量将产生未定义的结果。例如:
```
int fn(int n)
{
  static int n1 = n;  //  ok
  static int n2 = fn(n-1)+1;  //  未定义的
  return n;
}
```

## 参数传递
编译器负责检查实参的类型是否与对应的形参类型吻合，并在必要的时候执行标准类型转换或者用户自定义的类型转换。除非形参是引用，其他情况
下传入函数的是实参的副本。例如:
```
int* find(int* first, int* last,int v)  //  在[first:last)的范围内寻找v
{
  while (first!=last && *first!=v)
    ++first;
  return first;
}

void g(int* p, int* q)
{
  int* pp = find(p,q,'x');
  // ...
}
```
主调函数提供的实参p不会被find()函数修改，发生变化的是find()函数中与p对应的副本first。该指针是以传值方式传入函数的

### 引用参数
应尽量避免修改引用类型的实参，因为这么做会困扰程序读者。但遇到大对象，引用传递比值传递更有效。此时，我们应该将该引用类型的参数声明成
const的，以表明我们之所以使用引用只是处于效率上的考虑，而并非想让函数修改对象的值:
```
void f(const Large& arg)
{
  // 不允许修改"arg"的值
  // (除非显式使用类型转换)
}
```

如果在函数的声明中有某个引用参数未被指定const，则我们倾向与认为函数将修改改参数的值:`void g(Large& arg); //  倾向于认为g()会修改arg的值`
类似地，指针类型的参数被声明成const意味这该指针所指对象的值不会被函数改变，例如:
```
int strlen(const char*);  //  c风格字符串中的字符数量
char* strcpy(char* to, const char* from); //  复制一个C风格的字符串
int strcmp(const char*, const char*); //  比较两个C风格的字符串
```
