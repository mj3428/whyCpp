#结构
struct是任意类型元素的集合。
```
struct Address{
const char* name; //  "Jim Dandy"
int number; //  61
const char* street; //  "South St"
cosnt char* town; //  "New Providence"
char state[2];  //  'N''J'
const char* zip;
};
```
我们通常使用->运算符(struct指针解引用)访问结构的内容，例如:
```
void print_addr(Address* p)
{
  cout << p -> name <<'\n'
       << p -> number << '' << p -> street << '\n'
       << p -> town << '\n'
       << p -> state[0] << p ->state[1] << '' << p ->zip <<'\n';
}
```
如果p是一个指针，则p ->m等价于`(*p).m`  
结构类型的对象可以被赋值、作为实参传入函数，或者作为函数的结果返回。例如:
```
Address current;
Address set_current(Address next)
{
  address prev = current; 
  current = next;
  return prev;
}
```
### 结构与数组
很自然地，我们可以构建struct的数组，也可以让struct包含数组，例如:
```
struct Point{
  int x,y
};

Point point[3]{{1,2},{3,4},{4,5}}
int x2 = pointt[2].x;

struct Arrray{
  Point elem[3];
};

Array points2 {{1,2},{3,4},{4,5}};
int y2 = points2.elem[2].y;
```
把内置数组置于struct的内部意味着我们可以把该数组当成一个对象来使用：我们可以在初始化（包括函数传参及函数返回）和赋值时直接拷贝struct。例如:
```
Array shift(Array a, Point p)
{
  for (int i=0; i!=3; ++i){
    a.elem[i].x += p.x;
    a.elem[i].y += p.y;
  }
  return a;
}
Array ax = shift(points2,{10,20});
```

我们基于array继续编写下面的程序：
```
struct Point{
  int x, y
};

using Array = array<Point,3>; // 包含3个Point的array
Array points {{1,2},{3,4},{4,5}};
int x2 = points[2].x;
int y2 = points[2].y;

Array shift(Array a, Point p)
{
  for (int i=0; i!=a.size();++i){
    a[i].x += p.x;
    a[i].y += p.y;
  }
  return a;
}

Array ax = shift(points,{10,20});
```

### 类型等价
对于两个struct来说，即使它们的成员相同，它们本身仍是不同的类型。例如:
```
struct S1 {int a;};
struct S2 {int a;};
```
S1和S2是两种类型，因此：
```
S1 x;
S2 y=x; //  错误：类型不匹配
```
struct本身的类型与其成员的类型不能混为一谈，例如:
```
S1 x;
int i = x;  //  错误:类型不匹配
```
在程序中，每个struct只能有唯一的定义  
