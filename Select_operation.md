# 选择适当的操作
## 自由存储
new分配的对象“位于自由存储之上”（或者说“在堆上”或“在动态内存中”）  
如果某一类型含有默认构造函数，则我们可以省略掉初始化器。但是对内置类型这么做的话，其变量将会处于未初始化的状态。例如:
```
auto pc = new complex<double>;  //  该复数被初始化为{0,0}
auto pi = new int;  //  该 int 未被初始化
```
代码生成器先使用expr()创建的Enode，然后将其删除：
```
void generate(Enode* n)
{
  switch(n->oper){
  case Kind::plus:
    //  使用n
    delete n; //  从自由存储中删除一个Enode
  }
}
```
对于一个用new创建的对象来说，我们必须用delete显式地将它销毁，否则将一直存在。只有将它销毁了，它占用的空间才能被其他new使用。  
### 内存管理
内存问题:  
- 对象泄漏(leaked object):使用new,但是忘了用delete释放掉分配的对象.
- 提前释放(premature deletion)：在尚有其他指针指向该对象并且后续仍会使用该对象的情况下过早地delete
- 重复释放(double deletion):同一对象被释放两次，两次调用对象的析构函数  

对象泄露是一种潜在的严重错误,因为它可能会令程序面临资源耗尽的情况。与之相比,提前释放更容易造成恶果,因为指向“已删对象”的指针所指的可能
已经不是一个有效的对象了(此时读取的结果很可能与预期不符),又或者该内存区域已经存放了其他对象(此时对该区域执行写入操作将会影响本来无关的对象)。
下面是一段非常糟糕的代码:
```
int* p1 = new int{99};
int* p2 = p1; //  存在潜在的麻烦
delete p1;  // 此时，p2所指的不再是一个有效对象
p1 = nullptr; //  造成代码安全的错觉
char* p3 = new char{'x'}; // 此时，p3有可能指向了p2所指的内存区域
*p2 = 999;  //  改代码可能造成错误
cout << *p3 << '\n' //  输出的内容可能不是x
```
重复释放的问题在于资源管理通常无法追踪资源的所有者。例如:
```
void sloppy() //  非常糟糕的代码
{
  int* p = new int[1000]; //  请求内存
  //...使用 *p ...
  delete[] p; //  释放内存
  // ...完成一些其他操作...
  delete[] p; //  此时，sloppy()已经不拥有*p了
}
```
在执行第2个delete[]的时候，*p对应的内存区域可能已经被重新分配了，此时重新分配的内容可能会受到影响。如果把该段实例代码中的Int
替换成string的话，我们就能看到string的析构函数试图先读取一块已经被释放并且可能已被其他代码重写的内存区域，然后再delete这块区域，这显然
是错误的。通常情况下，重复释放属于未定义的行为，将产生不可预知的结果，甚至引发程序灾难。  
避免方法:
1. 除非万不得已不要把对象放在自由存储上，有限使用作用域内的变量
2. 当你在自由存储上构建对象时，把它的指针放在一个管理器对象中，此类对象通常含有一个析构函数，可以确保释放资源

另外一个比较深入的例子是用于资源管理的“智能指针”（Unique_ptr和share_ptr）例如:
```
void f(int n)
{
  int* p1 = new int[n]; //  存在潜在的风险
  unique_ptr<int[]> p2 {new int[n]};
  // ...
  if (n%2) throw runtime_error("odd");
  delete[] p1;  //  程序有可能运行不到此处
}
```
对于f(3)来说，p1所指的内存发生了泄漏，但是p2所指的内存以隐式的方式正确释放了。  
关于new和delete,我的经验是应该尽量确保“没有裸new",即,令 new位于构造函数或类似的函数中, delete位于析构函数中,由它们提供内存管理的策略。
此外, new常用作资源句柄的实参。  
### 数组
new 还能用来创建对象的数组，例如:
```
char* save_string(const char* p)
{
  char* s = new char[strlen(p)+1];
  strcpy(s,p);  //  从p拷贝到s
  return s;
}

int main(int argc,char* argv[])
{
  if (argc < 2) exit(1);
  char* p = save_string(argv[1]);
  // ...
  delete[] p;
}
```
"普通"delete用于删除单个对象，delete[]负责删除数组  
除非必须直接使用char*,否组一般情况下，标准库string是更好的选择，它可以简化save_string():
```
string save_string(const char* p)
{
  return string{p};
}

int main(int argc,char* argv[])
{
  if (argc < 2) exit(1);
  string s = save_string(argv[1]);
}
```
delete和delete[]必须清楚分配的对象有多大，才能准确地释放new分配的空间。
