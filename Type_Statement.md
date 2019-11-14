# 类型与声明
## 类型
### 布尔型
如果你既想使用{}初始化器列表防止窄化转换的发生，同时又确实想把int转换成bool，则可以显式声明如下:
```
void f(int i)
{
  bool b {i!=0};
}
```

在算术逻辑表达式和位逻辑表达式中, bool被自动转换成int,编译器在转换后的值上执行整数算术运算以及逻辑运算。如果最终的计算结果需要转换回bool,则
与之前介绍的一样, 0转换成false而非0值转换成true。  
如有必要，指针也能被隐式地转换成bool，其中，费控指针对应true，值为nullptr的指针对应false。例如：
```
void g(int *p)
{
  bool b = p; // 窄化成true或false
  bool b2 {p!=nullptr}; // 显式地检查指针是否为非空
  
  if (p){ // 等价于 p!=nullptr
    // ..
  }
}
```

与if (p!=nullptr)相比,我觉得if(p)更好,它不但简洁而且可以直接表达“p是否有效”的 "含义,使用if(p)也不太容易出错.  

### 字符类型
char16_t:该类型存放UTF-16等16位字符集  
char32_t:该类型存放UTF-32等32位字符集  
我们可以在字符类型上执行算术运算和为逻辑运算,例如:
```
void digits()
{
  for (int i=0;i!=10;++i)
    cout << static_cast<char>('0'+i);
}
```
上面的代码把10个阿拉伯数字输出到cout。字符字面值常量0,先转换成它对应的整数值, ,再与i相加;所得的int再转回char并被输出到cout。
'0'+i得到的结果本来是一个int,因此如果不加上static-cast的话,输出的结果将会是48, 49 ...而不是0, 1...

我们不能混用char\signed char\unsigned char这3种字符类型的指针,如:
```
void f(char c,signed char sc,unsigned char uc)
{
  char* pc = &uc; //错误:不存在对应的指针转换规则
  signed char* psc = pc;  //  错误:不存在对应的指针转换规则
  unsigned char* puc = pc;  //  错误:不存在对应的指针转换规则
  psc = puc;  //  错误:不存在对应的指针转换规则
}
```
3种char类型的变量可以互相赋值，但是把一个特别大的赋值给带符号的char是未定义的行为，例如:
```
void g(char c, signed char sc, unsigned char uc)
{
  c = 255;  //如果普通的char是带符号的且占8位，则该语句的行为依赖于具体实现
  c = sc; // OK
  c = uc; // 如果普通的char是带符号的且uc的值特别大，则该语句的行为依赖于具体实现
  sc = uc;  // 如果uc的值特别大，则该语句的行为依赖于具体实现
  uc = sc;  // OK转换成无符号类型
  sc = c; // 如果普通的char是无符号的且uc的值特别大,则该语句的行为依赖于具体实现
  uc = c; // ok 转换成无符号类型
}
```

### 整数类型
后缀U用于显式指定unsigned字面值常量，与之类似，后缀L用于显式指定long字面值常量。例如3U的类型时unsigned int而3L的类型时long int。  
多个后缀可以组合在一起使用，例如:`cout << 0xF0UL << '' << 0LU << '\n';`  
**前缀与后缀:**  
```
1UL //  unsigned long
2UL //  unsigned long
3ULL  //   unsigned long long
4LLU  //  unsigned long long
5LUL  //  错误
```
同样，后缀l和L也能用于表示浮点数字面值常量，表达的类型是long double
```
1L  // long int
1.0L  // long double
```

### void
从语法结构上来说,void属于基本类型。但是它只能被用作其他复杂类型的一部分，不存在任何void类型的对象。void有两个作用:一是作为函数的返回类型
用以说明函数不返回任何实际的值；二是作为指针的基本类型部分以表明指针所指对象的类型未知。
```
void x; //  错误:不存在void类型的对象
void& r;  //  错误：不存在void的引用
void f(); //  函数f不返回任何实际的值
void* pv; //  指针所指的对象类型未知
```
当我们声明一个函数时,必须指明返回结果的数据类型。从逻辑上来说,如果某个函数不返回任何值,也许我们会希望直接忽略掉返回值部分。
但其实这种想法并不可行,它会违反 C++的语法规则( §iso.A)。因此,我们使用void表示函数的返回值为空,此时void可以看成是一种“伪返回类型”  

### 类型尺寸
所有C++对象的尺寸都可以表示成char尺寸的整数倍,因此如果我们令char的尺寸 ,为1,则使用sizeof运算符就能得到任意类型或对象的尺寸。
下面是C++对于基本类型尺寸的一些规定:  
* 1=sizeof(char)≤sizeof(short)≤sizeof(int)≤sizeof(long)≤sizeof(long long)
* 1≤sizeod(bool)≤sizeof(long)
* sizeof(char)≤sizeof(wchar_t)≤sizeof(long)
* sizeof(float)≤sizeof(double)≤sizeof(long double)
* sizeof(N)=sizeof(signed N)=sizeof(unsigned N)  

其中,最后一行的N可以是char, short, int, long或者long long, C++规定char至少占8位, short至少占16位, long至少占32位。
char应该能存放机器字符集中的任意字符,它的实际类型依赖于实现并确保是当前机器上最适合保存和操作字符的类型。通常情况下, char占据一个8位的字节。
与之类似, int的实际类型也是依赖于实现的,并确保是当 ,前机器上最适合保存和操作整数的类型。int通常占据一个4字节(32位)的字。
上面这些假设是比较恰当的,但是我们很难做更多设定。比如,我们只能说char “通常”占据8位,因为确实也存在char占32位的机器。
又比如我们决不能假定int和指针的尺寸一样大,因为在很多机器上("64位体系结构”)指针的尺寸比整数大。  
