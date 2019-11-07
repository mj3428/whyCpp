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

