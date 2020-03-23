# 字符串、向量和数组
## 命名空间的using声明
使用std::cin,作用域操作符(::)的含义是：编译器应从操作符左侧名字所示的作用域中寻找右侧那个名字。因此，std::cin的意思就是要使用
命名空间std中的名字cin.  
当然使用using符号更好。就无须专门的前缀也能使用所需的名字了。  
一旦声明了using语句，就可以直接访问命名空间中的名字:
```cpp
# include <iostream>
using std::cin;

int main()
{
  int i;
  cin >> i; //  正确:cin和std::cin含义相同
  cout << i;    //  错误:没有对应的using声明，必须使用完整的名字
  std::cout << i; //  正确:显式地从std中使用cout
  return 0;
}
```
## 标准库类型string
string初始化表达方式  

|初始化string对象的方式|
|:--- |
|string s1  默认初始化,s1是一个空串|
|string s2(s1)  s2是s1的副本|
|string s3("value") s3是字面值"value"的副本，除了字面值最后的那个空字符外|
|string s3 = "value"  等价于上面的示例，s3是字面值"value"的副本|
|string s4(n, 'c') 把s4初始化为由连续n个字符c组成的串|  

**例题:**  
```cpp
string s;
cout << s[0] << endl;
```
此做法不合理，因为s[0]对应是空字符串 不能对空字符串使用下标
