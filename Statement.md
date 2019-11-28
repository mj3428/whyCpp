# 语句
## 语句概述
分号本身也是一条语句，即空语句(empty statement)  
花括号({})括起来的一个可能为空的语句序列称为块(block)或者符合语句(compound statement)。块中声明的名字的作用域到块的末尾就结束了  
声明(declaration)是一条语句，没有复制语句或过程调用语句；赋值和函数调用不是语句，它们是表达式。  
## 声明作为语句
一个声明就是一条语句。除非变量被声明成static，否则在控制线程传递给当前声明语句的同时执行初始化器。允许把声明当成一条语句使用的目的
是尽量减少由未初始化变量造成的程序错误，并且让代码的局部性更好。在绝大多数情况下，如果没有为变量找到一个合适的值，暂时不要声明它。例如：
```
void f(vector<string>& v, int i, const char* p)
{
  if (p==nullptr) return;
  if (i<0 || v.size()<=i)
    error("bad index");
  string s = v[i];
  if (s ==p ){
    //...
  }
  //...
}
```

对于用户自定义的数据类型，先确定一个合适的初始化器再定义变量能获得更好的程序性能。例如：
```
void use()
{
  string s1;
  s1 = "The best is the enemy of the good"
  //..
}
```

这段代码先把s1默认初始化成空字符串再对它赋值，这么做显然不如直接给定的值初始化效率:`string s2 {"Voltaire"};`  
声明一个缺少初始化器的变量，常常是因为我们需要一条专门的语句给变量赋值。其中一种情况是等待用户输入数据:
```
void input()
{
  int buf[max];
  int count = 0;
  for (int i;cin>>i;){
    if (i<0) error("unexpected negative value");
    if (count==max) error("buffer overflow");
    buf[count++] = i;
  }
}
```

## 选择语句
### switch语句
switch语句在一组候选项中进行选择。case标签中出现的表达式必须是整型或枚举类型的常量表达式。在同一个switch语句中，一个值最多被case标签使用
一次。  
switch语句可以用一组if语句等价地替换，例如:
```
switch (val) {
case 1:
  f();
  break;
case 2:
  g();
  break;
default:
  h();
  break;
}
```
可以将它等价转换为:
```
if (val==1)
  f();
else if (val==2)
  g();
else
  h();
```
但switch更清晰  
### 条件中的声明
在条件中声明变量一方面逻辑性更强，另一方面也让代码显得更紧凑。  
条件中的声明语句只能声明并初始化一个变量或const  

## 循环语句
### 范围for语句
简单case:
```
int sum(vector<int>& v)
{
  int s = 0;
  for (int x:v)
    s+=x;
  return s;
}
```
for (int x:v)读作“对于范围v中的每个元素x”，或者干脆说“对于v中分的每个x”.程序从头到尾依次访问v的全部元素。  
冒号之后的表达式必须是一个序列，换句话说，如果我们对它调用v.begin()和v.end()或者是begin(v)和end(v)，得到的应该是迭代器。  
- 编译器首先尝试寻找并使用成员begin和end。如果找到了begin和end，但是它们不能表示一个范围(begin有可能是变量而非函数)，则当前的
  范围for是错误的
- 如果没有找到，则编译器继续在外层作用域寻找begin/end成员。如果找不到或者找到的不能用（比如begin不接受当前序列类型的实参），则范围for是
  错误的  

如果想在范围for循环中修改元素的值，则应该使用元素的引用。如果，我们用下面的代码vector的每个元素都加1:
```
void incr(vector<int>& v)
{
  for (int& x:v)
    ++x;
}
```
引用还可以用于尺寸较大的元素，特别是当直接拷贝元素的值代价昂贵时。例如：
```
template<class T> T accum(vector<T>& v)
{
  T sum = 0;
  for (const T& x:v)
    sum += x;
  return sum;
}
```

### goto语句
goto可以用来跳出嵌套的循环或者switch语句(break只能跳出最内层的循环或者switch语句)，这是它为数不多的有意义的用法之一。例如：
```
void do_something(int i, int j)
  //  操作一个名为nm的二维矩阵
{
  for (i = 0;i!=n;++i)
    for (j = 0; j!=m; ++j)
      if (nm[i][j] == a)
        goto found;
  //  无关代码
  // ...
  found:
    // nm[i][j] == a
}
```
这个goto直接跳过了(退出了)整个循环。它既不会开启另一次循环,也不会进入到一个新作用域中。因此, goto的这种用法几乎
不会给程序员带来任何麻烦和困扰。
