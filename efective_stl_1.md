01. C++ C STL

C++为一个语言联邦, 是一个多重泛型编程语言, 支持过程形式, 面向对象形式, 函数形式, 泛型, 元编程.
C包括block, statement, preprocessor, built-in data types, arrays, pointers.
C++包括class包括构造, 析构函数, 封装 , 继承, 多态, virtual 函数.
STL是个模板程序库, 包括容器, 迭代器, 算法, 函数对象, 适配器, 这些组件通过迭代器紧密的联系在一起.
迭代器与函数对象都是在C指针之上塑造出来的, 所以传递的是pass-by-value.

02. 宏与内联

尽量用const , enum, inline替换#define, 也就是宁可用编译器替换预处理器.
#define定义的记号没有进入symbol table.
```
#define AUTHORNAME "Scoter meyer"
Const char* const authorname="Scoter meyer"; 
Const string authorname="Scoter meyer"; 
```
用#ifdef/#ifndef控制头文件重复包含. 

03. const

尽可能使用const, 它允许你指定语义约束, 编译器会强制实现这个约束.
const可施加于任何作用域的对象, 函数参数, 函数返回值, 成员函数. 
当const和non-const有等价实现, 用non_const调用const版本

```
char greeting[]="hello world"; 
const char* p=greeting;  //常量数据  看const在星号那边, 左表示被指物为常量. 
char* const p=greeting;  //常量指针   右边表示指针为常量. 
const vector<int>::iterator iter=vec.begin();  //iter作用像T* const
*iter=10;  // 正确
++iter;    // 错误
 vector<int>::const_iterator iter=vec.begin();  //iter作用像const T*
*iter=10;  错误
++iter; 正确
```

04. 确定对象使用前先被初始化

为内置型对象进行手工初始化, C++不保证初始化它们
构造函数最好用成员初始化列表, 初始化次序更声明次序相同
为免除”跨编译单元初始化次序问题”, 以local static对象替换non-local static对象

05. 编译器暗自为class创建default构造函数, 拷贝构造函数, 拷贝赋值操作符函数, 及析构函数. 

```
class Empty
{
public :
  Empty(){}; 
  Empty(const Empty&  rhs){}; 
  Empty& operator=(const Empty& rhs); 
  ~Empty(){}; 
};
//自己重新定义拷贝构造函数, 拷贝赋值操作符函数则涉及浅拷贝深拷贝问题. 
```

06. 若不想使用编译器自动生成函数, 就该明确拒绝它.

比如房子独一无二. 拒绝对象拷贝, 拷贝赋值, 声明为private.  或继承专门设计为阻止copying动作的base class
```
class Uncopyable{
public:
  Uncopyable（）{}
  ~Uncopyable（）{}
private:
  Uncopyable（const Uncopyable& ）; 
  Uncopyable& operator=（const Uncopyable& ）; 
}
class Homeforsale:private Uncopyable{}; 
```

07. 多态基类声明为virtual析构函数

基类指针指向派生类对象, 如果基类是non-virtual析构函数, 执行的时候是对象的.
derived成分没有销毁. 不具备多态性的类, 就不该声明为virtual析构函数.
实现virtural函数, 对象携带vptr虚拟指针指出要调用的函数, 虚拟指针指向虚拟表vtbl(每个带有虚拟函数的类都有个虚拟表), 当对象调用某个虚拟函数时, 实际被调用的函数时取决于该对象虚拟指针所指的虚拟表, 编译器在其中寻找合适的函数指针. 

08. 别让析构函数中突出异常, 也就是阻止异常从析构函数中传播出去.

09. 绝对不要在构造函数和析构函数中调用virtual函数, 因为这类调用从不下降至derived base

10. 令operator=返回一个reference to *this  //赋值连锁形式

11. 在operator=中处理自我赋值问题也就是证同测试, 或者采用copy-and-swap

```
if（this== &rhs）return *this;
```

12. 复制对象时不要忘了每一个成分, copy函数确保复制对象内的所有成分及所有基类成分

13. 以对象管理资源, 确保资源总是被释放, 获得资源立即放进管理对象, 管理对象用其析构函数确保资源被释放. 

资源取得时机便是初始化时机RAII
智能指针的析构函数自动对其所指向的对象调用delete, 注意别让多个auto-ptr指向同一对象, 通过拷贝构造函数或拷贝赋值函数复制智能指针, 会使其变为NULL, 所以STL容器不能装auto_ptr, 因为STL要求其元素有正常的复制行为.
常用的替代方案是"引用计数型智能指针"share_ptr, 可用于STL容器上.

14. 管理资源类中小心copying行为

当一个RAII对象复制, 会发生什么? 有以下几种可能:
  * 禁止复制 private继承uncopyable类
  * 对底层资源采用引用计数
  * 复制底部资源, 若字符串的深度拷贝
  * 转移底部资源拥有权.
  
15. 资源管理类中提供对原始资源的访问, 最好提高一个get函数让用户显示的调用

16. 成对的使用new和delete函数

17. 让接口设计容易被正确使用, 不易误用, 尽量将成员变量声明为private

18. 尽量要传递const 引用来代替传值, 通常比较高效, 但并不适合内置内型, 和STL迭代器和函数对象

19. 为什么带虚拟函数的类不能实例化?
