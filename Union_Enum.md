# 联合
union是一种特殊的struct，它的所有成员都分配在同一个地址空间上。因此，一个union实际占用的空间大小与其最大的成员一样。自然地，在同一时刻union
只能保存一个成员的值。例如，考虑一个符号表项存放名字和值对的情况:
```
enum Type {str,num}:
struct Entry{
  char* name;
  Type t;
  char* s;  //  如果 t==str,使用s
  int i;  //  如果 t==num，使用i
};

void f(Entry* p)
{
  if(p->t == str)
    cout << p->s;
  //...
}
```
在这个例子中，成员s和i永远不会被同时使用，因此空间被浪费了。我们可以把它们指定成union的成员，以解决上述问题:
```
union Value{
  char* s;
  int;
}
```

union是一种特殊的struct，而struct是一种特殊的class。然而，很多提供给类的功能与联合无关，因此对union施加了一些限制:  
- union不能含有虚函数
- union不能含有引用类型的成员
- union不能含有基类
- 如果union的成员含有用户自定义的构造函数、拷贝操作、移动操作或者易购函数，则此类函数对于union来说被delete掉了。换句话说，union类型的对象不能含有
  这些函数
- 在union的所有成员中，最多只能有一个成员包含类内初始化器
- union不能被用作其他类的基类  

### 匿名union
匿名联合是一个对象而非一种类型，我们无须对象名就能直接访问它的成员。因此，我们使用匿名联合的成员的方式与使用类成员的方式完全一样，只要谨记
同一时刻只能使用union的一个成员就可以了。  

# 枚举
枚举(enumeration)类型用于存放用户指定的一组整数值。枚举类型的美中取值各自对应一个名字，我们把这些值叫做枚举值，例如`enum class Color {
red, green, blue};`；该段代码定义了一个名为Color的枚举类型，它的枚举值是red、green和blue。“一个枚举类型”简称“一个enum”。  
枚举类型分为两种:  
- enum class,它的枚举值名字（比如red）位于enum的局部作用域内，枚举值不会隐式地转换成其他类型
- "普通的enum"，它的枚举值名字与枚举类型本身位于同一个作用域中，枚举值隐式地转换成整数。  

## enum class
enum class是一种限定了作用域的强类型枚举，例如:
```
enum class Traffic_light {red, yellow, green};
enum class Warning {green, yellow, orange, red};  //  火警等级

Warning a1 = 7; //  错误:不存在int向Waring的类型转换
int a2 = green; //  错误:green位于它的作用域之外
int a3 = Warning::green;  // 错误：不存在Warning向int的类型个转换
Warning a4 = Warning::green;  // OK

void f(Traffic_light x)
{
  if (x==9){/* ... */}  //  错误: 9不是一个Traffic_light
  if (x==red){/* ... */}  //  错误: 当前作用域中没有red
  if (x==Warning::red){/* ... */} //  错误:x不是一个Warning
  if (x==Traffic_light::red){/* ... */} //  OK
}
```
两个enum的枚举值不会互相冲突，它们位于各自enum class的作用域中   
枚举常用一些整数类型表示,每个枚举值是一个整数。我们把用于表示某个枚举的类型称为它的基础类型( ,基础类型必须是一种带符号或无符号的
整数类型，默认是int，我们可以显式地指定:
```
enum class Warning : int {green, yellow, orange, red};  //  sizeof(Warning)==sizeof(int)
也可以用char代替int
enum class Warning: char{green,yellow, orange,red}; //  sizeof(Warning)==1
```

用Warning变量代替普通的int变量使得用户和编译器都能更好地理解该变量的真正用途。例如:
```
void f(Warning key)
{
  switch(key){
  case Warning::green:
    // ...相应的操作...
    break;
  case Warning::orange:
    // ...相应的操作...
    break;
  case Warning::red:
    // ...相应的操作...
    break;
    }
}
```
一各整数类型的值可以显式地转换成枚举类型。如果这个值属于枚举的基础类型的取值范围，则转换是有效的；否则，如果超出了合理的表示范围，则
转换的结果是未定义的。例如:
```
enum class Flag:char{x=1,y=2,z=4,e=8};
Flag f0 {}; //  f0的默认值是0
Flag f1 = 5;  //  类型错误：5不属于Flag类型
Flag f2 = Flag{5};  //  错误：不允许窄化转换成enum class类型
Flag f3 = static_cast<Flag>(5); //  "不近人情"的转换
Flag f4 = static_cast<Flag>(999); //  错误：999不是一个char类型的值（也许根本捕获不到）
```
最后一条赋值语句很好地展示了为什么不允许从整数到枚举类型的隐式转换，因为绝大多数整数值根本不在一枚举类型的合理表示范围之内。  
每个枚举值对应一个整数，我们可以显式地把这个整数抽取出来出来。例如:
```
int i = static_cast<int>(Flag::y);  //  i的值变为2
char c = static_cast<char>(Flag::e);  //  c的值变为8
```

### 普通的enum
普通的enum的枚举值位于enum本身所在的作用域中，它们隐式地转换成某些整数类型的值。
```
enum Traffic_light{red, yellow, green};
enum Warning {green, yellow, orange, red};  //  火警等级

//  错误:yellow被重复定义（取值相同）
//  错误:red被重复定义（取值不同）

Warning a1 = 7; //  错误:不存在int向Warning的类型转换
int a2 = green; //  ok:green位于其作用域中，隐式地转换成int类型
int a3 = Warning::green;  // ok:Warning向int的类型转换
Warning a4 = Warning::green;  //  ok

void f(Traffic_light x)
{
  if (x==9){/*...*/}  //  ok(但是Traffic_light并不包含枚举值9)
  if (x==red){/*...*/}  //  错误:作用域中有两个red
  if (x==Warning::red){/*...*/} //  ok(哎呦！)
  if (x==Traffic_light::red){/*...*/} //  ok
}
```

可以为普通的枚举指定基础类型，就像对enum class所做的一样。此时，允许先声明枚举类型，稍后再给出它的定义，例如:
```
enum Traffic_light:char{tl_red,tl_yellow,tl_green}; //  基础类型是char
enum Color_code:char; //  声明
void foobar(Color_code* p); //  使用声明
// ...
enum Color_code:char {red,yellow,green,blue}; //  定义
```
整数到普通enum的显式类型转换规则与转换为enum class的规则一样。稍有的一点区别是,当没有显式地指定基础类型时,除非该值位于
枚举类型的范围之内,否则转换的结果是未定义的。例如：
```
enum Flag {x=1,y=2,z=4,e=8};  //  范围0:15

Flag f0{};  //  f0的默认值是0
Flag f1 = 5;  //  类型错误：5不是一个Flag
Flag f2 = Flag{5};  //  错误：不存在int向Flag的显式类型转换
Flag f2 = static_cast<Flag>(5);  //  ok:5在Flag的取值范围之内
Flag f3 = static_cast<Flag>(z|e); //  12在Flag的取值范围之内
Flag f4 = static_cast<Flag>(99);  //  未定义的：99不在Flag的取值范围之内
```
因为普通的enum和其他基础类型之间存在隐式类型转换，所以我们不需要为他专门定义运算符|：z和e会自动转换成int，因此z|e能够正常求值。
对枚举类型求sizeof的结果等价于对其基础类型求sizeof的结果。
