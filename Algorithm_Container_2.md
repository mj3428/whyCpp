# 容器和算法2
## 算法
### 使用迭代器
```
template<typename C,typename V>
vector<typename C::iterator> find_all(C& c,V v) //  在容器c中查找值v出现的所有位置
{
  vector<typename C::iterator> res;
  for(auto p = c.begin();p!=c.end();++p)
    if(*p==v)
      res.push_back(p);
    return res;
}
```
这里的typename必不可少，它通知编译器C的itertor是一种类型，而非某种类型的值，比如说整数7。我们可以通过引入一个类型别名Itertor来隐藏
这些实现细节
```template<typename T>
using Iterator<T> = typename T::iterator; //  T的迭代器

template<typename C, typename V>
vector<Iterator<C>>find_all(C&c, V v) //  在C中查找v出现的所有位置
{
  vector<Iterator<C>> res;
  for (auto p = c.begin();p!=c.end();++p)
    if (*p==v)
      res.push_back(p);
    return res;
}
```

现在就可以编写下面的代码来完成一些搜索任务了：
```
void test()
{
  string m {"Mary had a littile lamb"};
  for (auto p: find_all(m,'a'))
    if (*p!='a')
      cerr << "string bug!\n";
  
  list<double> Id {1.1,2.2,3.3,1.1};
  for (auto p:find_all(Id,1.1))
    if (*p!=1.1)
      cerr << "list bug!\n";
      
  vector<string> vs {"red","blue", "green", "green", "orange", "green"};
  for (auto p: find_all(vs,"green"))
    if (*p!="green")
      cerr << "vector bug!\n";
  for (auto p: find_all(vs,"green"))
    *p = "vert";
}
```
