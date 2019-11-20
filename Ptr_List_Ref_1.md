# 指针、数组与引用
在C++语言中存放及使用内存地址是通过指针和引用完成的
## 指针
对于类型T来说，T*是表示“指向T的指针”的类型。换句话说，T*类型的变量能存放T类型对象的地址，例如:
```
char c = 'a';
char* p = &c; //  p存放着c的地址，&是取地址运算符
```

对指针的一个基本操作是解引用，即引用地址所指的对象。这个操作也称为间接取值(indirection)。解引用运算符是个前置一元运算符，对应的符号是*,例如:
```
char c = 'a';
char* p = &c; //  p存放着c的地址，&是取地址运算符
char c2 = *p; //  c2 =='a'; *解引用运算符
```

指针p所指的对象是c，c中存储的值是'a'，因此我们把* p赋给c2等价于给c2赋值'a'。  
bool占用的内存空间至少和char一样多。如果想把更小的值存得更紧密，可以使用位逻辑操作、结构中的位域或者bitset  
符号 * 在用作类型名的后缀时表示"指向"的含义。如果我们想表示指向数组的指针或者指向函数的，需要使用稍微复杂一些的形式：
```
int* pi;  //  指向int的指针
char** ppc; //  指向字符指针的指针
int* ap[15];  //  ap是一个数组，包含15个指向int的指针
int (*fp)(char*); //  指向函数的指针，该函数接受一个char*实参，返回一个int
int* f(char*);  //  该函数接受一个char*实参，返回一个指向int的指针
```

### void* 
在某些偏向底层的代码中，我们偶尔需要在不知道对象确切类型的情况下，仅通过对象在内存中的地址存储或传递对象。此时，我们会用到void*.void* 的含义
是“指向未知类型对象的指针”。  
除了函数指针和指向类成员的指针,指向其他任意类型对象的指针都能被赋给一个void"类型的变量。此外,一个void"能被赋给另一个void",两个
" void* 能比较是否相等,我们还能把void"显式地转换成其他类型。因为编译器事实上并不清楚void"所指的对象到底是什么类型,所
以对它执行其他操作可能不太安全并且会引发编译器错误。要想使用void*,我们必须把它显式地转换成某一特定类型的指针。例如:
```
void f(int* pi)
{
  void* pv = pi;  //  ok:发生了从int*到void*的隐式类型转换
  *pv;  //  错误：不允许解引用void*
  ++pv; //  错误:不允许对void*执行递增操作(所指的对象尺寸未知)
  
  int* pi2 = static_cast<int*>(pv); //  显式转换回int*

  double* pd1 = pv; //  错误
  double* pd2 = pi; //  错误
  double* pd3 = static_cast<double*>(pv); //  不安全
}
```

### nullptr
字面常量nullptr表示空指针，即不指向任何对象的指针。我们可以把nullptr赋给其他任意指针类型，但是不能赋给其他内置类型:
```
int* pi = nullptr;
double* pd = nullptr;
int i = nullptr;  //  错误：i不是指针
```
nullptr只有一个，它可以用于任意指针类型。  
在C语言中，NULL通常是(void*)0,这种用法在C++中是非法的:`int* p = NULL; //  错误:不能把void*赋给int*`  

## 数组
假设有类型T，T[size] 的含义是“包含size个T类型元素的数组”。元素的索引值范围是0到size-1。例如:
```
float v[3]; //  包含3个float的数组，分贝是v[0],v[1],v[2]
char* a[32];  //  包含32个char指针的数组，依次是a[0]..a[31]
```

数组中的元素的数量（即数组的边界）必须是常量表达式。如果你希望边界可变，最好使用vector。例如:
```
void f(int n)
{
  int v1[n];  //  错误:数组的大小不是常量表达式
  vector<int> v2(n);  //  OK:包含n个int元素的vector
}
```

C++允许静态地分配数组空间，也允许在栈上或者在自由存储上分配数组空间。例如:
```
int a1[10]; //  静态存储中的10个int

void f()
{
  int a2 [20];  //  栈上的20个int
  int*p = new int[40];  //  自由存储上的40个int
  // ...
}
```
数组不能执行赋值操作，一旦需要，数组名就会隐式地转换成指向数组首元素的指针。特别要注意避免在接口中使用数组，因为数组名隐式转换成指针是
C代码和C风格的C++代码中很多错误的根源.如果是在自由存储上分配数组的，切记一定要在最后一次使用数组之后把对应的指针delete[]掉。如果
你是静态地分配数组或者是在栈上分配数组，一定不要delete[] 它。  
C++没有为整数提供内置的拷贝操作。不允许用一个数组初始化另一个数组（即使两个数组的类型完全一样也不行），因为数组不支持赋值操作:
```
int v6[8] = v5; //  错误：不允许拷贝数组（不允许把int*赋给数组）
v6 = v5 //  错误：不存在数组的赋值操作
```
如果你想给一组对象赋值，可以使用vector、array、或者valarry  
### 字符串字面值常量
字符串字面值常量是指双引号内的字符序列:`"this is a string"`  
字符串字面值常量实际包含的字符数量比它表现出来的样子多一个。它以一个取值为0的空字符'\0'结尾,例如:`sizeof("Bohr")==5`  
字符串字面值常量的类型是“若干个const字符组成的数组”，因此"Bohr"的类型是const char[5].  
有些语句存在不安全：
```
void f()
{
  char* p = "Plato";  //  错误，但是被c++11之前的代码接受
  p[4] = 'e'; //  错误：试图为常量赋值
}
```

如果我们希望字符串能被修改，最好把字符放在一个非常量的数组中：
```
void f()
{
  char p[] = "Zeno";  //  p是含有5个字符的数组
  p[0] = 'R'; //  OK
}
```

字符串字面值常量是静态分配的，因此函数返回字符串字面值常量是很安全的行为，不会有什么问题
```
const char* error_message(int i)
{
  return "range error";
}
```

空字符串记作一队紧挨着的双引号"",其类型是const char[1].空字符串中唯一的一个字符是结束符'\0'.  
### 大字符集
UTF-8字符串的结尾是'\0',UTF-16是u'\0',UTF-32是U'\0'  
英文字符串的表示方式有很多种，考虑以反斜线作为分隔符的文件名:
```
"folder\\file"  //  基于实现字符集的字符串
R"(folder\file)"  //  基于实现字符集的原始字符串
u8"folder\\file"  //  UTF-8字符串
u8R"(folder\file)"  //  UTF-8原始字符串
u"folder\\file" //  UTF-16字符串
uR"(folder\file)" //  UTF-16原始字符串
U"folder\\file" //  UTF-32字符串
UR"(folder\file)" //  UTF-32原始字符串
```
另外`\u`之后的十六进制数是一个Unicode编码点。编码点独立于编码方式，事实上，在不同编码方式下编码点的表现形式会有不同。  

## 数组中的指针
```
int v[] = {1,2,3,4};
int* p1 = v;  //  指向数组首元素的指针（隐式转换）
int* p2 = &v[0];  //  指向数组首元素的指针
int* p3 = v+4;  //  指向数组尾后位置的指针
```
令指针指向数组的最后一个元素的下一个位置（尾后位置）是有效的，这对于很多算法非常重要。不过，因为该指针事实上指向的并不是数组中的任何一个
元素，所以不能对它进行读写操作  
```
extern "C" int strlen(const char*)

void f()
{
  char v[] = "Annermarie"
  char* p = v;  //  char[]到char*的隐式类型转换
  strlen(p);
  strlen(v);  //  char[]到char*的隐式类型转换
  v = p;  //  错误:不允许给 数组直接赋值
}
```
两次调用传入标准库函数strlen()的值是相同的。唯一的问题是这种隐式类型转换无法避免,注定会发生。换句话说,我们不可能让函数接受整个数
组v。不过幸运的是,不存在从指针向数组的显式或隐式类型转换。  
### 数组漫游
我们就可以通过指向数组的指针加上一个索引值来访问数组元素，也可以通过直接指向数组元素的指针进行访问。
```
void fp(char v[])
{
  for (char* p = v;*p!=0;++p)
    use(*p);
}
```
前置* 运算符执行解引用运算，因此 `*p`是指针p所指的字符，++运算令p指向数组的下一个元素。  
内置数组的取下标操作是通过组合指针的+和 * 两种运算得到的， 对于内置数组a和数组范围之内的整数j，有下式成立：
`a[j] == *(&a[0]+j) == *(a+j) == *(j+a) == j[a]`  
人们常常会纠结于为什么alj==j[a],比如3["Texas"] == "Texas"[3]=='a',其实这种小聪明在实际的代码中并没有多少展示的空间。
上面这些等价关系属于非常底层的规则,并不适用于array和vector等标准库  
```
template<typename T>
int byte_diff(T* p, T* q)
{
  return reinterpret_cast<char*>(q)-reinterpret_cast<char*>(p);
}

void diff_test()
{
  int vi[10];
  short vs[10];
  cout << vi << '' << &vi[1] << '' << &vi[1]-&vi[0] << '' << byte_diff(&vi[0], &vi[1]) << '\n';
  cout << vs << '' << &vs[1] << '' << &vs[1]-&vs[0] << '' << byte_diff(&vs[0], &vs[1]) << '\n';
}
```

输出的结果是
```
0x7fffaef0 0x7fffaef4 1 4
0x7fffaedc 0x7fffaede 1 2
```

当计算两个指针p和q的差值(q-p)时,所得结果是序列`[p:9)`中的元素数量(一个整数),我们可以给指针加上一个整数或者从指针中减去一个整数,得到的结果
都是指针。
```
void f()
{
  int v1[10];
  int v2[10];
  
  int i1 = &v1[5]-&v1[3]; //  i1 = 2
  int i2 = &v1[5]-&v2[3]; //  结果是未定义的
  
  int* p1 = v2 + 2; //  p1 = &v2[2]
  int* p2 = v2 - 2; //  *p2是未定义的
}
```

因为数组的元素数量不一定能与数组本身存储在一起，所以数组不具有自解释性。当我们需要遍历一个数组并且不像C风格的字符串那样具有明确的终结符时，我
们必须以某种方式提供元素的数量。例如：
```
void fp(char v[], int size)
{
  for (int i=0;i!=size;++i)
    use(v[i]);  //  祈祷数组v至少包含size个元素，否则就会越界
  for (int x: v)
    use(x); //  错误:范围for循环对指针无效
    
  const int N = 7;
  char v2[N];
  for(int i=0;i!=N;++i)
    use(v2[i]);
  for (int x:v2)
    use(x); //  当已知数组的大小是可以使用范围for循环
}
```

### 传递数组
不能以值传递的方式直接把数组传给函数，我们通常传递的是指向数组首元素的指针。例如：
```
void comp(double arg[10]) //  arg的类型是double*
{
  for (int i=0;i!=10;++i)
    arg[i]+=99;
}

void f()
{
  double a1[10];
  double a2[5];
  double a3[100];
  
  comp(a1);
  comp(a2); //  严重错误
  comp(a3); //  只用到了前10个元素
};
```
它虽然能通过编译，但是调用comp(a2)试图向a2的合法边界之外的区域写入内容。此外，如果你期望数组以值传递的方式传给函数，恐怕也要大失所望了：
对arg[i]执行写操作实际上是直接向comp()的实参的元素写内容，而不是工作在该实参的一份副本上。  

编译器会因实参声明m[][] 非法而报错，因为多维数组的第二个维度必须是已知的，这样我们才能准确定位其中的元素。然而，表达式`m[i][j]` 会被编译器
理解成`*(*(m+i)+j)`.一种正确的解决方案是:
```
void print_mij(int* m, int dim1, int dim2)
{
  for (int i=0;i!=dim1;i++){
    for (int j=0;j!=dim2;j++)
      cout << m[i*dim2+j] << '\t';  //  有点儿难懂
    cout << '\n';
  }
}
```