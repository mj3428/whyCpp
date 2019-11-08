# 容器与算法
## I/O
### 用户自定义类型的I/O
除了支持内置类型和标准库string的io之外，iostream库还允许程序员为自己的类型定义IO操作。例如考虑一个简单的类型Entry，我们用它来表示
电话簿中的一条记录:
```
struct Entry{
  string name;
  int number;
};
```

我们可以定义一个简单的输出运算符，以类似于初始化代码的形式{"name",number}来打印一个Entry:
```
ostream& operator<<(ostream& os,const Entry& e)
{
  return os << "{\"" << e.name << "\", "<<e.number<<"}";
}
```

一个用户自定义的输出运算符接受它的输出流为第一个参数，输出完毕后，返回此流的引用。  
对应的输入运算符要复杂得多，因为它必须检查格式是否正确并处理可能发生的错误:
```
istream& operator>>(istream& is,Entry& e)
  //  读取{"name",number}对。注意，正确的格式包含{""和}
{
  char c,c2;
  if(is>>c && c=='{'&&is>>c2 && c2==""){ //  以一个{"开始
    string name;  //  string的默认值是空字符串""
    while (is.get(c)&&c!="")  //  "之前的任何内容都是名字的一部分
      name+=c;
      
      if(is>>c && c==','){
        int number = 0;
        if(is>>number>>c && c=='}'){  // 读取数和一个}
          e = {name, number}; //  把读入的值赋予Entry对象
          return is;
        }
      }
    }
    is.setf(ios_base::failbit); //  将流状态置为fail
    return is;
}
```
输入运算符返回它所操作的istream对象的引用，该引用可用来检测操作是否成功。例如当用作一个条件时，is>>C表示“我们从is读取数据存入c的操作
成功了吗？”  
is>>c默认跳过空白符，而is.get(c)则不会，因此，上面的Entry的输入运算符忽略(跳过)名字字符串外围的空白符，但不会忽略其内部的空白符。
例如: `{"John Marwood Cleese", 123456    }` 和 `{"Michael Edward Palin", 987654}`  
我们可以用下面的代码从输入流读取这样的值对，然后存入Entry对象中:
```
for (Entry ee;cin>>ee;) // 从cin读取数据存入ee
  cout << ee << '\n'; // 将ee的值写入cout
```

则输出为:
```
{"John Marwood Cleese", 123456}
{Michael Edward Palin", 987654}
```

## 容器
### Vector
用一组值来初始化vector，当然,值的类型必须与vector元素类型吻合:
```
vector<Entry> phone_book = {
  {"David Hume",123456},
  {"Karl Popper",234567},
  {"Bertrand Arthur William Russell",345678}
};
```

我们可以通过下标运算符访问元素:
```
void print_book(const vector<Entry>& book)
{
  for (int i=0;i!=book.size();++i)
    cout << book[i] << '\n';
}
```

vector的所有元素构成了一个范围，因此我们可以对其使用for循环:
```
void print_book(const vecrtor<Entry>& book)
{
  for (const auto& x:book)
    cout << x << '\n';
}
```

定义一个vector时，为它设定一个初始大小（初始的元素数目）:
```
vector<int> v1 = {1,2,3,4}; //  size为4
vector<string> v2;  //  size为0
vector<Shape*> v3(23);  //  size为23;元素初值是nullptr
vector<double> v4(32,9.9);  //  size为32;元素初值是9.9
```

vector的初始大小随着程序的执行可以被改变。vector最常用的一个操作就是push_back(),它向vector末尾追加一个新元素,
从而将vector的规模增大1。
```
void input()
{
  for (Entry e; cin >> e;)
    phone_book.push_back(e);
}
```

在赋值和初始化时，vector可以被拷贝.例如:`vector<Entry> book2 = phone_book;`  
Vec继承了vector除下标运算符之外的所有内容，它重定义了下标运算符来进行范围检查。vector的at()函数同样负责下标操作，但它会在参数越界时
抛出一个类型为out_of_range的异常。  
对于一个Vec对象来说，越界访问会抛出一个用户可捕获的异常，例如:
```
void checked(Vec<ENtry>& book)
{
  try{
    book[book.size()] = {"Joe",999999}; //  会抛出一个异常
  }
  catch (out_of_range){
    cout << "range error\n";
  }
}
```

这段程序会抛出一个异常,然后将其捕获。如果用户不捕获异常,则程序会以一种定义良好的方式退出,而不是继续执行
或者以一种未定义的方式终止。一种尽量弱化未捕获异常影响try-块作为main()函数的函数体
