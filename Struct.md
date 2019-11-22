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
### struct的布局
