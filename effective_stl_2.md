01. 仔细选择你需要的容器

  * 序列容器: vector, list, deque
    vector是默认的标准序列容器, 当需要对序列频繁插入删除是则用list, 当大部分插入在头和尾部是则用deque. 
  * 关联容器: set, map, multiset, multimap (非标准关联容器:hash_set等)

基于连续内存的容器, 插入或删除元素一般需要移动已有的数据块;
基于节点的容器, 插入或删除元素只影响节点指向的指针;
你需要在任意位置插入吗? 如果是, 则选用序列容器, 关联容器做不到;
需要关心元素在容器中的位置吗? 如果不, 可用散列容器, 尽量避免用散列容器;
你要选用哪一类迭代器? 如果必须是随机迭代器, 则必须用vector,  list , deque
容器中数据的内存布局需要与C兼容吗? 需要则用vector;
查找速度很重要吗? 则用散列容器或排序的vector容器或关联容器;
你需要插入事务性语义吗? 如果是, 则需要选择list容器;
你要把迭代器的失效减到最少吗? 应该用基于节点的容器, 一般来说连续内存插入节点会使得整个容器失效.

02. 不要对"容器无关代码"抱有幻想.

不同容器是不同的, 并不是设计为互相容易的替换, STL建立在泛化的基础上, 数组泛华为容器, 函数泛华为算法, 指针泛华为迭代器.
设计为替换容器的常用方法是: 封装再封装, 最简单的方法是通过对容器和迭代器的类型使用typedef.
```
class Widget{}
typedef vector<Widget> WidgetContainer; 
typedef WidgetContainer::iterator WCIterator; 
WidgetContainer cw; 
Widget bestWidget; 
WCIterator i=find(cw, begin(), cw.end(), bestWidget); 
```

03. 使容器中对象的拷贝操作轻量而正确

容器中的对象是对象的拷贝, 如果拷贝过程很昂贵, 则把对象放进容器将会出现性能瓶颈.
由于继承的存在, 拷贝会导致分割, 如果你以基类对象建立的容器, 往里面插入派生类对象, 则派生类的派生部门会删除. 
一般解决办法是: 建立指针容器而不是对象容器

04. 用empty来代替size()来检查是否为0

因为对于标准容器, empty是常数时间, 对于像list的实现, size花费线性时间

05. 尽量用区间成员函数代替它们的单元素函数兄弟

大多STL程序员过度使用copy, 几乎所有的区间被插入迭代器指定的copy的使用都可以调用区间成员函数来代替, 基本上容器提供区间构造, 区间插入, 区间删除, 区间赋值

06. 当容器容纳指向通过new分配的对象指针是, 则需要自己控制delete指针指向的对象. 

当你调用for_each算法是, 你需要把delete转入一个函数对象中. 
```
struct DeleteObject: 
{
    Template<typename T>
    void operator()( const T* ptr)const
    { 
       delete ptr; 
    }
}
for_each( vec.begin(),  vec.end(),  DeleteObject()); 
```
或者用boost库的shared-ptr
```
void dosomething（）
{
   typedef boost::shared-ptr<Widget> SPW; 
   vector<SPW> vwp; 
   for( ......)
     vwp.push_back( SPW( new Widget)); 
}
```

07. 永远不要建立auto-ptr容器

08. 删除元素时候仔细选择, 不同的容器也不同

如果是连续容器, 最好是用erase_remove惯用法.
```
c.erase( remove( c.begin(), c.end(), 1963), c.end());
```
如果是list, 则用其成员函数
```
c.remove(1963);
```
如果是关联容器, 则没有remove成员函数, 应该调用erase成员函数

如何在循环内做某些事情?
如果是标准序列容器, 写一个循环遍历容器元素, 每当调用erase时记得用它的返回值更新迭代器;
如果是标准关联容器, 写一个循环遍历容器元素, 当你把迭代器传给erase时候后置递增它;

09. 尽量使用vector和string来代替动态分配数组

10. 使用reserve来避免不必要的重新分配

11. 使用交换技巧来修正容器中过多的容量

```
string  s; 
// 使其变大, 然后删除所有
string(s).swap(s);   
vector<Contestant>().swap();
```

12 不能使用vector<bool>容器, 其替代品是bitset
13 了解相等于等价的区别 find算法中寻找元素是基于operator==, 而set::insert对相同的定义是等价, 等价是基于在一个有序区间中对象值的相对位置, 基于operator<

14 为指针的关联容器指定比较类型 
比如用set<string *> ssp;  
是这样的简写set<string*, less<string*>, allocator<string*>> ssp; 
如果想要string*指针以字符串值确定顺序被存储在set中, 就必须提供自己改写的比较仿函数类, 
Struct StringPtrLess:
Public binary_function<const string*, const string*, bool>
{
bool operator()(const string* ps1, const string * ps2)const
{
    Return *ps1<*ps2; 
}
}
Typdef set<string, StringPtrLess> StringPtrSet; 
StringPtrSet ssp; 
15:避免原地修改set和multiset的键
替换方法:EmpIdSet se; 
Employee selectedId; 
EmpIdSet::iterator i=se.find( selectedId);  //第一步找到要改变的元素
If(  i!= se.end())
{ 
   Employee e(*i); //第二步, 拷贝这个元素
   se.erase(i++); //删除这个元素, 自增保持迭代器有效
   e.setTitle("corportate "); //修改副本  
   se.insert(i, e);  //插入新值
} 
15: 考虑有序的vector代替关联容器
平衡二叉树设计为应用于进行一些插入, 然后一些查找, 然后可能再进行一些插入, 然后也许一些删除, 然后再来一些查找, 然后更多的插入或删除, 然后更多的查找, 关键特征是插入, 删除, 查找混合在一起. 
当应用中出现三个截然不同的阶段时候（建立, 查找, 重组）
vector<Widget> vw;  // 代替set<Widget>
... // 建立阶段:很多插入, 
// 几乎没有查找
sort(vw.begin(),  vw.end(), DataCompare());  // 结束建立阶段. （当
// 模拟一个multiset时, 你可能更喜欢用stable_sort
// 来代替; 参见条款31. ）
Widget w;  // 用于查找的值的对象
... // 开始查找阶段
if (binary_search(vw.begin(),  vw.end(),  w, DataCompare()))... // 通过binary_search查找
vector<Widget>::iterator i =
lower_bound(vw.begin(),  vw.end(),  w, DataCompare());  // 通过lower_bound查找
if (i != vw.end() && !(w < *i))... // 条款19解释了
// “!(w < *i)”测试
pair<vector<Widget>::iterator, 
vector<Widget>::iterator> range =equal_range(vw.begin(),  vw.end(),  w, DataCompare());  // 通过equal_range查找
if (range.first != range.second)...
... // 结束查找阶段, 开始
// 重组阶段
sort(vw.begin(),  vw.end());  // 开始新的查找阶段...

当使用vector来模拟map<K,  V>时, 保存在vector中数据的类型将是pair<K,  V>, 而不是pair<const K,  V>. 
你需要为你的pair写一个自定义比较函数, 因为pair的operator<作用于pair的两个组件
typedef pair<string,  int> Data;  // 在这个例子里
// "map"容纳的类型
class DataCompare { // 用于比较的类
public:
bool operator()(const Data& lhs,  // 用于排序的比较函数
const Data& rhs) const
{ return keyLess(lhs.first,  rhs.first);  // keyLess在下面}
bool operator()(const Data& Ihs,  // 用于查找的比较函数
const Data::first_type& k) const // （形式1）
{   return keyLess(lhs.first,  k); }
bool operator()(const Data::first_type& k,  // 用于查找的比较函数
const Data& rhs) const // （形式2）
{return keyLessfk,  rhs.first); }
private:
bool keyLess(const Data::first_type& k1,  // “真的”
const Data::first_type& k2) const // 比较函数
{  return k1 < k2; }
}; 

16 理解map::operator[]和map-insert之间的含义
  给定map<K,  V> m;   这个表达式
m[k] = v; 
检查键k是否已经在map里. 如果不, 就添加上, 以v作为它的对应值. 如果k已经在map里, 它的关联值被更新成v. 设计为简化”添加或更新”功能

对效率的考虑, 当给map添加一个元素时, 我们断定insert比operator[]好
高效的”添加或更新”功能, 想象一个像这样的调用接口
iterator affectedPair = // 如果键k不再map m中; 高效地
efficientAddOrUpdate(m,  k,  v);  // 把pair(k,  v)添加到m中; 否则
// 高效地把和k关联
// 的值更新为v. 返回一个
// 指向添加或修改的
// pair的迭代器
template<typename MapType,  // map的类型
typename KeyArgType,  // KeyArgType和ValueArgtype
typename ValueArgtype> // 是类型参数
typename MapType::iterator
   efficientAddOrUpdate(MapType& m, const KeyArgType& k, const ValueArgtype& v)
{
typename MapType::iterator Ib = // 找到k在或应该在哪里; 
m.lower_bound(k);  // 为什么这里需要”typename”
if(Ib != m.end() && // 如果Ib指向一个pair
!(m.key_comp()(k,  Ib->first))) { // 它的键等价于k...
Ib->second = v;  // 更新这个pair的值
return Ib;  // 并返回指向pair的
} // 迭代器
else{
typedef typename MapType::value_type MVT; 
return m.insert(Ib,  MVT(k,  v));  // 把pair(k,  v)添加到m并
} // 返回指向新map元素的
} // 迭代器

17 确保目标区间足够大
int transmogrify(int x);  // 这个函数从x
// 产生一些新值
vector<int> values; 
... // 把数据放入values
vector<int> results;  // 把transmogrify应用于
transform(values.begin(),  values.end(),  // values中的每个对象
results.end(),  // 把这个返回的values
transmogrify);  // 附加到results这段代码有bug！
因为在*results.end()以及后面没有对象, 如果往后面插入对象的话, 需要加入back_inserter, 或前插front_inserter, 或者中间插入inserter

18 了解你排序需求
如果你有一个Widget的vector, 你想选择20个质量最高的Widget发送给你最忠实的客户, 你需要做的只是排序以鉴别出20个最好的Widget, 剩下的可以保持无序. 
bool qualityCompare(const Widget& lhs,  const Widget& rhs)
{
// 返回lhs的质量是不是比rhs的质量好
}
...
partial_sort(widgets.begin(),  // 把最好的20个元素
widgets.begin() + 20,  // （按顺序）放在widgets的前端
widgets.end(), 
qualityCompare); 
如果你需要的只是任意顺序的20个最好的Widget, 则调用nth_element
如果你需要把标准序列容器的元素或数组分隔为满足和不满足某个标准, 你大概就要找partition或stable_partition. 
● 如果你的数据是在list中, 你可以直接使用partition和stable_partition, 你可以使用list的sort来代替sort和stable_sort. 如果你需要partial_sort或nth_element提供的效果, 你就必须间接完成这个任务. 
排序的资源需求(空间和时间复杂度)比较 
partition<stable_partition<nth_element<partial_sort<sort<stable_sort
19 提防在指针容器中使用类remove算法
如果你无法避免在那样的容器上使用remove, 排除这个问题一种方法是在应用erase-remove惯用法之前先删除指针并设置它们为空, 然后除去容器中的所有空指针:
class Widget{
public:
...
bool isCertified() const;  // 这个Widget是否通过检验
...
}; 
vector<Widget*> v;  // 建立一个vector然后用
... // 动态分配的Widget
v.push_back(new Widget);  // 的指针填充
void delAndNullifyUncertified(Widget*& pWidget) // 如果*pWidget是一个
{ // 未通过检验Widget, 
if (!pWidget->isCertified()) { // 删除指针
delete pWidget;  // 并且设置它为空
pWidget = 0; 
}
}

for_each(v.begin(),  v.end(),  // 把所有指向未通过检验Widget的
delAndNullifyUncertified);  // 指针删除并且设置为空
v.erase(remove(v.begin(),  v.end(),  // 从v中除去空指针
static_cast<Widget*>(0)),  // 0必须映射到一个指针, 
v.end());  // 让C++可以
// 正确地推出remove的
// 第三个参数的类型

智能指针解法 shared_ptr
template<typename T> // RCSP = “引用计数
class RCSP { ...};  // 智能指针”
typedef RCSP< Widget> RCSPW;  // RCSPW = “RCSP to Widget”
vector<RCSPW > v;  // 建立一个vector, 用动态
... // 分配Widget的
v.push_back(RCSPW(new Widget));  // 智能指针填充它
...
v.erase(remove_if(v.begin(),  v.end(),  // erase未通过检验的
not1 (mem_fun(&Widget::isCertified))),  // Widget的指针
v.end()); 
19 仿函数设计用于值传递
20 用纯函数用做判断式
判断式必须返回值为bool值, 纯函数是返回值只依赖于参数的函数, 
一个判断式类是一个仿函数类, 它的operator()函数是一个判断式

21 使仿函数类可以适配
正规方法是从一个基类, 或, 更精确地说, 一个基结构, 继承它们. operator()带一个实参的仿函数类, 要继承的结构是std::unary_function. operator()带有两个实参的仿函数类, 要继承的结构std::binary_function. 

如:
template<typename T>
class MeetsThreshold: public std::unary_function<Widget,  bool>{
private:
const T threshold; 
public:
MeetsThreshold(const T& threshold); 
bool operator()(const Widget&) const; 
...
}; 
struct PtrWidgetNameCompare:
public std::binary_function<const Widget*,  const Widget*,  bool> {
bool operator()(const Widget* lhs,  const Widget* rhs) const; 
}; 

22. 了解使用ptr_fun、mem_fun和mem_fun_ref的原因
