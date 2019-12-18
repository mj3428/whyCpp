# 异常处理
## 错误处理
### 异常
* 主调组件如果想处理某些失败的情形，可以把这些异常置于try块的catch从句中。  
* 被调组件如果无法完成既定的任务，可以用throw表达式抛出一个异常来说明这一情况。  
结构完整，能够说明异常处理的机制:
```
void taskmaster()
{
  try{
    auto result = do_task();
    // 使用result
  }
  catch (Some_error){
    //  执行do_task时发生错误:处理该问题
  }
}

int do_task()
{
  // ...
  if(/*能够执行该任务*/)
    return result;
  else
    throw Some_error{};
}
```
### 层次化错误处理
偶尔也需要从一种错误报告模式转换成另外一种。例如，我们可能会检查errno的值，在调用C语言库后抛出异常；反之，也可能从C++库返回C程序前捕获
异常并设置errno:
```
void callC()  //  在C++中调用C函数，把errno转换成throw
{
  errno = 0;
  c_function();
  if (errno){
    //  ..如果可能且必要的话，执行局部清理
    throw C_blewit(errno);
  }
}

extern "C" void call_from_C() noexcept  //  在C语言中调用一个C++函数，把throw转换成errno
{
  try{
    c_plus_plus_function();
  }
  catch (...){
    //  如果可能且必要的话，执行局部清理
    errno = E_CPLPLFCTBLEWIT;
  }
}
```
