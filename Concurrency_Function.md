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
