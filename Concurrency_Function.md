# 并发与实用功能
## 并发
### 任务和thread
我们可以构造一个std::thread并将任务作为它的实参。这里的任务是以函数或函数对象形式出现的:
```
void f(): //  函数
struct F{
  void operator()();  // F调用运算符
};

void user()
{
  thread t1 {f};  // f()在独立的线程中执行
  therad t2 {F()} //  F()()在独立的线程中执行
  
  t1.join();  // 等待t1完成  
  t2.join();  //  等待t2完成
}
```
join()保证我们在线程完成后才退出user(),其中"join"的意思是“等待线程结束"。一个程序的所有线程共享单一地址空间。在这一点上线程与
进程不同,进程间通常不直接共享数据。由于共享单一地址空间,因此线程间可通过共享对象相互通信。通常通过锁或其他防止数据竞争(对变量的
不受控制的并发访问)的机制来控制线程间通信。  
### 传递参数
任务通常需要处理数据，我们可以将数据（或指向数据的指针或引用）作为参数参数给任务，例如：
```
void f(vector<double>& v);  // 处理v的函数

struct F{ // 处理v的函数的对象
vector<double>& v;
F(vector<double>& vv) :v{vv}{}
void operator()(); // 调用运算符
};

int main()
{
  vector<double> some_vec{1,2,3,4,5,6,7,8,9};
  vector<double> vec2{10,11,12,13,14};
  
  thread t1 {f,some_vec}; //  f(some_vec)在一个独立线程中执行
  thread t2 {F{vec2}};  // F(vec2)()在一个独立线程中执行
  
  t1.join();
  t2.join();
}
```

显然, F{vec2)将一个指向参数(一个向量)的引用保存在F中。F现在就可以使用向量了,并希望在它运行的时候其他任务不会访问vec2————将
vec2以传值方式传递就可以消除这个风险。  
上面代码用{f,some-vec)初始化一个线程,它使用了thread的可变参数模板构造函数,接受一个任意的参数序列,编译器检查第一个参数(函数或函
数对象)是 ,否可用后续的参数来调用,如果检查通过,就构造一个必要的函数对象并传递给线程。因此, F::operator()与f()执行相同的算法,两个任
务的处理大致相同:它们都为thread构造了一个函数对象来执行任务。  

### 共享数据
有时任务间需要共享数据。此时数据访问必须进行同步,以确保在同一时刻至多有一个任务能访问数据。有经验的程序员可能认为这是
一种简单化的方法(例如,很多任务同时读取不变的数据是没有任何问题的),但无论如何,确保在同一时刻至多有一个任务可以访问给定的对象是很有意义的  
解决此问题的基础是"互斥对象"mutex。thread使用lock()操作来获取一个互斥对象:
```
mutex m; // 控制共享数据访问的mutex
int sh; // 共享的数据

void f()
{
  unique_lock<mutex> lck {m}; // 获取mutex
  sh+=7; // 处理共享数据
} // 隐式释放mutex
```

标准库提供了一个同时获取多个锁的操作，可以解决访问多个资源带来的死锁问题:
```
void f()
{
  // ...
  unique_lock<mutex> lck1 {m1,defer_lock};  // 推迟加锁:还为尝试获取mutex
  unique_lock<mutex> lck2 {m2,defer_lock};
  unique_lock<mutex> lck3 {m3,defer_lock};
  //...
  lock(lck1,lck2,lck3); // 获取全部三个锁
  //...处理共享数据..
} // 隐式释放所有mutex
```
lock()调用只有在获取了全部mutex实参后才会继续执行，当它持有mutex时，绝不会阻塞("睡眠")，当然也就不会导致死锁。unique_lock的析构函数
保证了当thread离开作用域时mutex会被释放.  
**async()**  
如需启动异步运行的任务，我们可以使用async()：
```
double comp4(vector<double>& v)
  // 如果v足够大，则创建很多任务
{
  if (v.size()<10000) return accum(v.begin(),v.end(),0.0);
  
  auto v0 = &v[0];
  auto sz = v.size();
  
  auto f0 = async(accum,v0,v0+sz/4,0.0);  //  第一个四分之一
  auto f1 = async(accum,v0+sz/4,v0+sz/2,0.0);  //  第二个四分之一
  auto f2 = async(accum,v0+sz/2,v0+sz*3/4,0.0); // 第三个四分之一
  auto f3 = async(accaum,v0+sz*3/4,v0+sz,0.0);  //  第四个四分之一
  
  return f0.get()+f1.get()+f2.get()+f3.get(); //  收集并组合结果
}
```
async()将一个函数调用的“调用部分”和“获取结果部分”分离开来,并将这两部分与任务的实际执行分离开来。通过使用async(),你不必再操心
线程和锁,而只需考虑可能异步执行的任务。这种做法显然受到了限制:不要试图对共享资源且需要用锁机制的任务使用async()————使用async(),你甚至
不知道要使用多少个thread,因为这是由async()来决定的,它根据它所了解的调用发生时系统可用资源量来确定使用多少个thread。
例如, async()会先检查有多少可用核(处理器)再确定启动多少thread.  

## 正则表达式
正则表达式最常见的应用场景是在数据流中搜索符合某一模式的字符串：
```
int lineno = 0;
for (string line;getline(cin,line);){ //  把数据读入缓冲区
  ++lineno;
  smatch matches; //  用于存放匹配的字符串
  if (regex_search(line,matches,pat)) //  在一行字符串中查找符合模式pat的字串
    cout << lineno << ":" << matches[0] << '\n';
```

`regex_search(line,matches,pat)`负责在读入的line中查找所有符合模式pat的子串.如果找到了，把它们存入matches中;如果没找到,
regex_search(line,matches,pat)返回false。matches的类型是smatch，其中字符"s"表示"子(sub)"或"字符串(string)"，所以smatch就是一个
包含string类型匹配子串的vector.代码中的第一个元素matches[0]是匹配得到的结果。  

## 数学计算
### 随机数
随机数生成器包括两部分:  
1. 一个引擎(engine)，负责生成一组随机值或者伪随机值
2. 一种分布(distribution)，负责把引擎产生的值映射到某个数学分布上。
常用的分布包括uniform_int_distribution（生成的所有整数概率相等）、normal_distribution（正态分布，又名“铃铛曲线”）和
exponential_distribution（指数增长），它们的范围各不相同。例如:
```
using my_engine = default_random_engine;  // 引擎类型
using my_distribution = uniform_int_distribution<>; //  分布类型

my_engine re {};  //  默认引擎
my_distribution one_to_six{1,6};  // 该分布把随机数映射到1~6的范围
auto die = bind(one_to_six,re); // 得到一个随机数生成器

int x = die();  //  掷骰子：x得到的值位于1~6之间
```
标准库函数bind()生成一个函数对象,它会把第2个参数(re)作为实参绑定到第一个·参数( oneto-six函数对象)的调用中。因此,调用die()
等价于调用one to_six(re)
