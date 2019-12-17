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
### 
