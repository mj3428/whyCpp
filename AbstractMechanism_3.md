# 抽象机制3
## 模板
### 参数化类型
对于我们之前使用的double向量，只要将其改为template并且用一个形参替换掉特定类型double,就能将其泛化成任意类型的向量了。例如:
```
template<typename T>
class Vector{
private:
  T* elem;  //  elem指向含有sz个T类型元素的数组
  int sz;
public:
  Vector(int s);  //  构造函数：建立不变式，获取资源
  ~Vector(){delete[] elem;}   //  析构函数：释放资源
  //...拷贝和移动操作...
  T& operator[](int i);
  const T& operator[](int i) const;
  int size() const{return sz;}
};
```

### 函数模板
用模板能参数化标准库中的很多类型和算法。例如，下面这段程序可以计算任意容器中元素的和:
```
template<typename Container,typename Value>
Value sum(const Container& c,Value v)
{
  for (auto x:c)
    v+=x;
  return v;
}
```
模板参数Value和函数参数v使得调用者可以指定累加器（用于求和的变量）的类型和初始值：
```
void user(Vector<int>& vi, std::list<double>& ld, std::vector<complex<double>>& vc)
{
  int x = sum(vi,0);  //  求整数向量的和(累加整数)
  double d = sum(vi,0,0); //  求整数向量的和(累加浮点数)
  double dd = sum(ld,0,0);  //  求浮点数列表的和
  auto z = sum(vc,complex<double>{}); //  求complex<double>向量的和
                                      //  初始值是{0.0,0.0}
}
```
将一些int值累加到double变量中的做法让我们可以得体地处理超出int表示范围的数值。请注意sum<T,V>的模板实参类型是如何根据函数实参
推断出来的。幸运的是,我们无须显式地指定这些类型。

### 函数对象
模板的一个特殊用途是函数对象( function object,有时也称为函子functor),我们可以像调用函数一样使用函数对象。例如:
```
template<typename T>
class Less_than{
  const T val;  //  待比较的值
public:
  Less_than(const T& v):val(v){}
  bool operator()(const T& x) const{return x<val;}  //  调用运算符
};
```
其中，名为operator()的函数实现了“函数调用”“调用”或“应用”运算符()  
我们能为某些实参类型定义Less_than类型的命名变量:
```
Less_than<int>lti{42};   // lti(i)将使用<比较i和42(i<42)
Less_than<string>lts{"Backus"}; // lts(s)将使用<比较s和“Backus”(s<"Backus")
```
接下来就能像调用函数一样调用该对象了：
```
void fct(int n,const string & s)
{
  bool b1 = lti(n); //  如果n<42则为真
  bool b2 = lts(s); //  如果s<"Backus"则为真
}
```

### 可变参数模板
定义模板时，可以令其接受任意数量任意类型的实参，这样的模板称为可变参数模板
```
template<typename T,typename... Tail>
void f(T head,Tail... tail)
{
  g(head);  //  对head做某些操作
  f(tail...); // 再次处理tail
}

void f(){}  // 不执行任何操作
```
实现可变参数模板的关键时：当你传给它多个参数时，谨记把第一个参数和其他参数区分对待。此处，我没首先处理第一个参数(head)，然后使用剩余参数
(tail)递归地调用f()。省略号...表示列表的“剩余部分”。最终tail将变空，我们需要另外一个独立的函数来处理它。  
调用f()的形式如下:
```
int main()
{
  cout << "first:";
  f(1,2.2,"hello");
  cout << "\nsecond:"
  f(0.2,'c',"yuck!",0,1,2);
  cout << "\n";
}
```
上面的程序首先调用f(1,2.2,"hello"),然后调用f(2.2,"hello"),接着调用f("hello")，最终调用f().g(head)又会做上面呢？显然，在一个真是的程序
中，它将完成我们希望对每个实参执行的操作。例如，我们想要输出实参(这里是head):
```
template<typename T>
void g(T x)
{
  cout << x <<"";
}
```
而输出的内容为 `first:1 2.2 hello` 和 `second: 0.2 c yuck! 0 1 2`
