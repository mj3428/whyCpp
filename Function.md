# 函数
## 函数声明
函数声明负责指定函数的名字、返回值的类型以及调用该函数所需的参数数量和类型。例如：
```
Elem* next_elem();  //  无须参数，返回Elem*
void exit(int); //  int 类型的参数，无返回值
double sqrt(double);  //  double类型的参数，返回double
```
参数传递的语义与拷贝初始化的语义完全一致。编译器检查实参的类型，如果需要的话还会执行隐式参数类型转换，例如:
```
double s2 = sqrt(2);  //  用实参double{2}调用函数sqrt()
double s3 = sqrt("three");  //  错误：函数sqrt()需要double类型的参数
```
