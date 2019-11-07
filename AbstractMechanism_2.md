# 抽象机制2
## 拷贝和移动
### 拷贝容器
当一个类作为资源句柄( resource handle)时,换句话说,当这个类负责通过指针访问一个对象时,采用默认的逐成员复制方式通常意味着错误。逐成员
复制将违反资源句柄的不变式。例如,下面所示的默认拷贝将产生Vector的一份拷贝,而这个拷贝所指向的元素与原来的元素是同一个  
```
void bad_copy(Vector v1)
{
  Vector v2 = v1; //  把v1的表现形式复制给v2
  v1[0] = 2;  //  v2[0]现在也是2了
  v2[1] = 3;  //  v1[1]现在也是3了
}
```

类对象的拷贝操作可以通过两个成员来定义：拷贝构造函数(copy constructor)与拷贝赋值运算(copy assignment):
```
class Vector{
private:
  double* elem; //  elem指向含有sz个double的数组
  int sz;
public:
  Vector(int s);  //  构造函数：建立不变式，请求资源
  ~Vector() {deletep[] elem;} //  析构函数:释放资源
  
  Vector(const Vector& a);  //  拷贝构造函数
  Vector& operator=(const Vector& a); //  拷贝赋值运算符
  double& operator[](int i);
  const double& operator[](int i)const;
  int size() const;
}
```

对于Vector来说，拷贝构造函数的正确定义应该首先为指定数量的元素分配空间，然后元素复制到空间中。这样在复制完成后，每个Vector就拥有自己的
元素副本了:
```
Vector::Vector(const Vector& a) //  复制构造函数
      :elem{new double[sz]},  //  为元素分配空间
      sz{a.sz}
{
  for (int i=0;i!=sz;++i) //  复制元素
    elem[i] = a.elem[i];
}
```

### 资源管理
通过定义构造函数、拷贝操作、移动操作和析构函数,程序员就能对受控资源(比如容器中的元素)的全生命周期进行管理。而且移动构造函数还允
许对象从一个作用域简单便捷 ,地移动到另一个作用域。采取这种方式,我们不能或不希望拷贝到作用域之外的对象就能简单高效地移动出去了。不妨以表示并发
活动的标准库thread和含有百万个double的Vector为例,前者“不能”执行拷贝操作,而后者我们“不希望”拷贝它。
```
std::vector<thread>my_threads;
Vector init(int n)
{
  thread t {heartbeat}; //  同时运行到heartbeat(在它自己的线程上)
  my_threads.push_back()  // 把t移动到my_threads
  //..其他初始化部分
  Vector vec(n);
  for (int i=0;i<vec.size();++i) vec[i]=777;
  return vec; //  把vec移动到init()之外
}
auto v = init(); // 启动heartbeat,初始化v
```
就像替换掉程序中的new和delete一样,我们也可以把指针转化为资源句柄。在这两种情况下,都将得到更简单也更易维护的代码,而且没什么额外的开
销。特别是我们能·实现强资源安全(strong resource safety),换句话说,对于一般概念上的资源,这种方法都可以消除资源泄漏。比如存放内存
的vector、存放系统线程的thread和存放文件句柄的fstream。  

