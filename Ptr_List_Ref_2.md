# 指针、数组与引用
## 指针与const
* constexpr: 编译时求值  
* const:在当前作用域内，值不发生改变  

基本上，cosntexpr的作用是指示或确保在编译时求值，而const的主要任务是规定接口的不可修改性。  
很多对象的值一旦初始化就不会再改动：
- 使用符号化常量的代码比直接使用字面值常量的代码更易维护
- 我们经常通过指针读取数据，但是很少通过指针写入数据
- 绝大多数函数的参数只负责读取数据，很少写入数据  

为了表达一经初始化就不可修改的特性，我们可以在对象的定义中加上const关键字。
```
const int model = 90; //  model是一个const
const int v[] = {1,2,3,4};  //  v[i]是一个const
const int x;  //  错误:缺少初始化器
```
因为我们无法给const对象赋值，所以它必须初始化。一旦我们把某物声明成const,就确保它的值在其作用域内不会发生改变：
```
void f()
{
  model = 200;  //  错误
  v[2] = 3; //  错误
}
```
一个指针牵扯到两个对象:指针本身以及指针所指的对象。在指针的声明语句中“前置” const关键字将令所指的对象而非指针
本身成为常量。要想令指针本身成为常量,应该用`*const`代替普通的* .例如：
```
void f1(char* p)
{
  char s[] = "Gorm";
  
  const char* pc = s; //  指向常量的指针
  pc[3] = 'g';  //  错误:pc指向常量
  pc = p; //  ok
  
  char *const cp = s; //  常量指针
  cp[3] = 'a';  //OK
  cp = p; //  错误:cp是一个常量
  
  const char *const cpc = s;  //  指向常量的常量指针
  cpc[3] = 'a'; //  错误:cpc指向常量
  cpc = p;  //  错误:cpc本身是一个常量
}
```

声明运算符`*const`的作用是令指针本身称为常量。不存在形如`const*`的声明运算符，相反，出现在 * 前面的const是基本类型的一部分。例如：
```
char *const cp; //  指向char的常量指针
char const* pc; //  指向常量const的指针
const char* pc2;  //  指向常量char的指针
```

对于同一个对象来说,通过一个指针访问它时是常量并不妨碍在其他情况下它是个变量。这一点在涉及函数的实参时特别有用。我们可以把指
针类型的实参声明成const,这样就能阻止函数修改该指针所指的对象了。例如:
```
const char* strchr(const char* p, char c);  //  找到在字符串p中字符c第一次出现的位置
char* strchr(char* p,char c); //  找到在字符串p中字符c第一次出现的位置
```
第一个函数的参数是常量字符串，函数无权修改其中的元素；它的返回值是指向cosnt的指针，也不允许修改其所指的对象。第二个函数没有这些
限制。  
常量的地址不能被赋给某个不受限的指针，因为如果这样的话，用户有可能通过该指针修改对象的值，这显然不被允许的。例如:
```
void f4()
{
  int a = 1;
  const int c = 2;
  const int* p1 = &c; //  ok
  const int* p2 = &a; //  ok
  int* p3 = &c; //  错误：用const int* 初始化int*
  *p3 = 7;  //  试图改变c的值
}
```

## 引用
使用指针与使用对象名存在以下差别：  
* 语法形式不同，`*p` 和 p->m分别取代了obj和obj.m
* 同一个指针在不同时刻可以指向不同对象
* 使用指针要比指直接使用对象更小心：指针的值可能是nullptr,也可能指向一个我们并不想要的对象  

引用与指针的区别主要包括:  
* 访问引用与访问对象本身从语法形式上看是一样的
* 引用所引的永远是一开始初始化的那个对象
* 不存在“空引用”,我们可以认为引用一定对应着某个对象  

引用最重要的用途是作为函数的实参或返回值，此外，它也被用于重载运算符。例如:
```
template<class T>
class vector{
  T* elem;
public:
  T& operator[](int i){return elem[i];} //  返回元素的引用
  const T& operator[](int i) const {return elem[i];}  //  返回常量元素的引用
  
  void push_bback(const T& a);  //  通过引用传入待添加的元素
};

void f(const vector<double>& v)
{
  double d1 = v[1]; //  把v.operator[](1)所引的double的值拷给dl
  v[2] = 7; //  把7赋给v.operator[](2)所引的double
  
  v.push_back(d1);  //  给push_back()传入dl的引用
}
```

为了体现左值/右值以及const/非const的区别，存在三种形式的引用:  
* 左值引用(lvalue reference):引用那些我们希望改变值的对象
* const引用(const reference):引用那些我们不希望改变值的对象（比如常量）
* 右值引用(rvalue reference):引用对象的值在我们使用之后就无须保留了（比如临时变量）  

### 左值引用
在类型名字中，符号X&的意思是“X的引用”；
```
void f()
{
  int var = 1;
  int& r {var}; //  r和var对应同一个int
  int x = r;  //  x的值变为1
  r = 2;  //  var的值变为2
}
```

为了确保引用对应某个对象，我们必须初始化引用，例如:  
```
int var = 1;  
int& r1 {var};  //  ok:初始化r1
int& r2;  //  错误：缺少初始化器
extern int& r3; //  ok:r3在别处初始化
```

初始化引用和给引用赋值是完全不同的操作。除了形式上的区别外，事实上没有专门针对引用的运算符。例如:
```
void g()
{
  int var = 0;
  int& rr {var};
  ++rr; //  var的值加1
  int* pp = &rr;  // pp指向var
}
```
在这段代码中, ++rr的含义并不是递增引用r,相反它的作用是给rr所引的int (即var)加 1,因此,引用本身的值一旦经过初始化就不能再改变了;它
永远都指向一开始指定的对象。我们可以使用&rr得到一个指向rr所引对象的指针。但是我们既不能令某个指针指向引用,也不能定义引用的数组。
从这个意义上来说,引用不是对象。  

const T&的初始值不一定非得是左值，甚至可以不是T类型。
```
double& dr = 1; //  错误：此处需要左值
const double& cdr{1}; //  ok
```
后一条语句的初始化过程可以理解为:
```
double temp = double{1};  //  首先用给定的值创建一个临时变量
const double& cdr {temp}; //  然后用这个临时变量作为cdr的初始值
```

### 右值引用
C++之所以设计了几种不同形式的引用，是为了支持对象的不同用法：  
* 非const左值引用所引的对象可以由用户写入内容
* const左值引用所引的对象从用户的角度来看是不可修改的
* 右值引用对应一个临时对象，用户可以修改这个对象（通常确实会修改它），并且认定这个对象以后不会被用到了  

右值引用可以绑定到右值，但是不能绑定到左值。从这一点上来说，右值引用与左值引用正好相反。例如：
```
string var{"Cambridge"};
string f();

string& r1 {var}; //  左值引用，r1绑定到var左值上
string& r2 {f()}; //  左值引用，错误:f()是右值
string& r3 {"Princeton"}; //  左值引用，错误：不允许绑定到临时变量

string&& rr1 {f()}; //  右值引用，正确：rr1绑定到一个右值（临时变量）
string&& rr2 {var}; //  右值引用，错误: var是左值
string&& rr3 {"Oxford"};  //  rr3引用的是一个临时变量，它的内容是"Oxford"

const string cr1& {"Harvard"};  //  OK:创建一个临时变量，然后把它绑定到crl
```
声明符&&表示“右值引用”。我们不使用const右值引用，因为右值引用的大多数用法都是建立在能够修改所引对象的基础上的。const左值引用和右值引用
都能绑定右值，但是它们的目标完全不同：  
* 右值引用实现了一种“破坏性读取”，某些数据本来需要被拷贝，使用右值引用可以优化性能。  
* const左值引用的作用是保护参数内容不被修改

### 指针与引用
如果你想让某个名字永远对应同一个对象，应该使用引用。例如：
```
template<class T> class Proxy { //  Proxy引用初始化它的那个对象
  T& m;
public:
  Proxy(T& mm) :m{mm}{}

};

template<class T> class Handle{ //  Handle引用当前对象
  T* m;
public:
  Proxy(T* mm) :m {mm}{}
  void rebind(T* mm){m = mm;}
  // ...
};
```

如果你想自定义（重载）一个运算符，使之用于指向对象的某物，应该使用引用。例如:
```
Matrix operator+(coonst Matrix&, const Matrix&);  //  OK
Matrix operator-(coonst Matrix*, const Matrix*);  //  错误：不是童虎自定义类型参数

Matrix y, z;
Matrix x = y+z; //  OK
Matrix x2 = &y-&z;  //  难看且存在错误
```

C++不允许重新定义指针等内置类型的运算符含义。如果你想让一个集合中的元素指向对象，应该使用指针:
```
int x,y;
string& a1[] = {x,y}; //  错误:引用的数组
string* a2[] = {&x, &y};  //  OK
vector<string&> s1 = {x,y}; //  错误：引用的向量
vector<string*> s2 = {&x, &y};  //  OK
```

如果你需要表示“值空缺”，则应该使用指针。指针提供了nullptr作为“空指针”，但是并没有“空引用”与之对应。例如：
```
void fp(X* p)
{
  if (p==nullptr) {
    //  指针的值为空
  }
  else{
    //  使用 *p
  }
}

void fr(X& r) //  常规形式
{
  //  假定r合法 然后使用它
}
```
当确实需要的时候，也可以为特定的类型构造一个“空引用”，例如:
```
void  fr2(X& r)
{
  if (&r == &nullX){  //  或者是r == nullX
    //  引用为空
  }
  else{
    //使用r
  }
}
```

## 建议
1. 使用指针时越简单直接越好;
2. 不要对指针执行稀奇古怪的算术运算;
3. 注意不要越界访问数组,尤其不要在数组之外的区域写入内容;
4. 不要使用多维数组,用合适的容器替代它;
5. 用nullptr代替0和NULL;
6. 与内置的C风格数组相比,优先选用容器(比如vector, array和valarray);
7. 优先选用string,而不是以0结尾的char数组;
8. 如果字符串字面值常量中包含太多反斜线,则使用原始字符串;
9. const引用比普通引用更适合作为函数的实参;
10. 只有当需要转发和移动时才使用右值引用;
11. 让表示所有权的指针位于句柄类的内部;
12. 在底层代码之外尽量不要使用`void*`;
13. 用const指针和const引用表示接口中不允许修改的部分;
14. 引用比指针更适合作为函数的实参，不过当需要处理“对象缺失”的情况时例外;
