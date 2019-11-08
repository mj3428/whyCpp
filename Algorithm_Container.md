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
  if(is>>c & c=='{'&&is>>c2 && c2==""){ //  以一个{"开始
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
