# 联合
union是一种特殊的struct，它的所有成员都分配在同一个地址空间上。因此，一个union实际占用的空间大小与其最大的成员一样。自然地，在同一时刻union
只能保存一个成员的值。例如，考虑一个符号表项存放名字和值对的情况:
```
enum Type {str,num}:
struct Entry{
  char* name;
  Type t;
  char* s;  //  如果 t==str,使用s
  int i;  //  如果 t==num，使用i
};

void f(Entry* p)
{
  if(p->t == str)
    cout << p->s;
  //...
}
```
在这个例子中，成员s和i永远不会被同时使用，因此空间被浪费了。我们可以把它们指定成union的成员，以解决上述问题:
```
union Value{
  char* s;
  int;
}
```

union是一种特殊的struct，而struct是一种特殊的class。然而，很多提供给类的功能与联合无关，因此对union施加了一些限制:  
- union不能含有虚函数
- union不能含有引用类型的成员
- union不能含有基类
- 如果union的成员含有用户自定义的构造函数、拷贝操作、移动操作或者易购函数，则此类函数对于union来说被delete掉了。换句话说，union类型的对象不能含有
  这些函数
- 在union的所有成员中，最多只能有一个成员包含类内初始化器
- union不能被用作其他类的基类  

### 匿名union
