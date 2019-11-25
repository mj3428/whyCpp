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
