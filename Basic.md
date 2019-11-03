# 结构
构建一个新类型第一步是把所需的元素组织成一种数据结构
```
struct Vector{
  int sz; //元素数量
  double* elem; //指向元素指针
};
```

接着我们需要让V指向某些元素
```
void vector_init(Vector& v, int s)
{
  v.elem = new double[s]; //分配一个数组，它包含s个double值
  v.sz = s;
}
```

Vector&中的&符号指定我们通过非常量引用的方式传递v，这样vector_init()就能修改传入其中的向量了。  
而new运算符从一块名为自由存储的区域中分配内存。
**应用：** 
```
double read_and_sum(int s)
{
  Vector v;
  vector_init(v,s); // 为v分配s个元素
  for (int i=0;i!=s; ++i)
    cin>>v.elem[i]  // 读入元素
    
  double sum = 0;
  for (int i=0;i!=s;++i)
    sum += v.elem[i]; // 计算元素的和
  return sum;
}
```
