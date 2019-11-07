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
