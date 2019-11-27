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
