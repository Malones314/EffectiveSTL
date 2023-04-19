# EffectiveSTL
## 4.18
1. 容器中数据布局若要兼容C，则只能选择```vector```
 
2. string 中用了引用计数技术，若想避免使用这种技术，可以考虑使用 ```vector<char>```

3. 基于节点的容器会使iterator、指针、引用变无效的次数最少，该类容器的插入和删除操作不会使iterator、指针、引用变得无效(除非指向了删除的元素)
4. 不要写独立于容器的代码
5. 两者对比，改进后的让以后更改容器类型变得容易得多，尤其是增加一个分配子时更加方便
```cpp
class widget{.....};
vector<widget> vw;
widget w;
vector<widget>::iterator i;
```
```cpp
class widget{.....};
typedef vector<widget> widgetContainer;
typedef widgetContainer::iterator WCIterator;
widgetContainer wc;
widget w;
WCIterator i;
```

6. STL的工作方式：
```insert```, ```push_back```等操作往容器里存入对象的方式为指定对象的拷贝
```front```, ```back```等取对象操作取的也是拷贝
所以要帮正容器中对象的拷贝正确高效

7. ```vector```，```string```，```deque```的插入、删除让其元素位置发生移动（通过拷贝）

8. STL的设计思想是避免创建不必要的对象，故要使用指针而非实例对象（首选智能指针）

9. ```if( c.size() == 0)``` 和 ```if( c.empty())```两者等价，但对于list来说```empty```使用了常数时间操作，而```size```使用线性时间，故首选```empty```检查容器是否为空
10. 想给元素一组全新的值可以使用assign
11. 让v1的内容和v2的后半段内容一样：
```cpp
v1.assign( v2.begin()+v2.size()/2, v2.end());
```
区间成员函数优先与之对应的单元素成员函数，要尽量避免使用显示的循环

## 4.19

1. 区间创建ctor
```cpp
container::container( InputIterator begin, InputIterator end);
```

2. 区间插入
```cpp
用于所有标准容器
iterator container::insert( iterator position, InputIterator begin, InputIterator end);
```
```cpp
用于关联式容器
void container::insert( InputIterator begin, InputIterator end);
```

3. 区间赋值
```cpp
void container::assign( InputIterator begin, InputIterator end);
```

4. 各种函数声明
```cpp
cpp尽可能的把语句解释为函数声明

int f( double d); //声明一个带 double 参数 返回int的函数f

int f( double (d)); //同上，d两边的括号被忽略

int f( double );    //同上，参数名省略

int g( dluble( *pf)()); //声明一个带 指向不带参数的函数的指针 参数 返回int的函数g

int g( double pf());  //同上，pf为隐式指针

int g( double ());  //同上，省略参数名称
```
```cpp
ifstream dataFile("ints.dat");
list<int> data( 
  istream_iterator<int>(dataFile),  
      //dataFile被看成参数变量名，括号被忽略，istream_iterator<int>为参数类型
  istream_iterator<int>());
      //看成不带参数名的函数指针
//data为函数名
```
```cpp
list<int> data( 
  (istream_iterator<int>(dataFile)),  
      //看成省去函数名的函数，传入的参数为ifstream类型的dataFile
  istream_iterator<int>());
      //看成不带参数名的函数指针
//data为变量，类型为list<int>
形参的声明用括号括起来是非法的，但是函数参数可以加上括号
```

5. for_each函数：
```cpp
template<class InputIterator, class UnaryFunction>
UnaryFunction for_each( InputIterator first, InputIterator last, UnaryFunction f);
```

6. 若容器中有通过```new```创建的指针，要在容器对象析构前delete指针，解决办法：
把delete变成一个函数对象：
```cpp
struct DeleteObject{
  template<typename T>
  void operator()( const T* ptr) const{
    delete ptr;
  }
};
```
之后可以通过for_each来删去创建的对象：
```cpp
void doSomething(){
  deque<specialString*> dss;
  for_each( dss.begin, dss.end(), DeleteObject());
}

```
上述问题最简单的解决方法为使用boost库中的智能指针：
```cpp
void doSomething(){
  typedef boost::shared_ptr<widget> spw;
  vector<spw> vwp;
  for( int i = 0; i < NUMBER; ++i)
    vwp.push_back( spw( new widget));
}
```

7. 切勿创建包含auto_ptr的容器对象

8. 关联容器内元素的删除使用erase
9. 序列容器内元素的删除用remove
10. 当容器删除一个元素对指向其的所有迭代器都无效，故一旦c.erase(i)返回，i变为无效值，循环中的i++无效
```cpp
应改进为：
for( container<int>::iterator i = c.begin(); i != c.end();){
  if( badValue(*i) == true)
    c.erase(i++);
  else
    ++i;
}
```
11. 分配子(allocator):封装STL容器在内存管理上的底层实现
12. 只有当考虑string中的引用计数的开销时会选择动态new的数组，也可以考虑vector<char>
13. 考虑使用reserve来避免不必要的重新分配
14. resize(n) 强迫容器改变那到包含n个元素的状态，若n大于size则尾部多的元素被析构，若大于size小于capacity则用默认的ctor创建元素，n大于capacity则先重新分配内存
15. reserve(n) 强迫容器把它的容量变为至少是n(n必须不小于当前的大小，若n小于当前的容量，vector什么也不做，string把容量减小到size和n中的较大值)通常会导致重新分配，应当尽早的使用reserve来避免重新分配
16.
```cpp
vector<int> v;
for( int i = 0; i < 1000; ++i)
  v.push_back(i);
```
```cpp
vector<int>v;
v.reserve(1000);
for( int i = 0; i < 1000; ++i){
  v.push_back(i);
}
```
上面的会在循环过程中进行多次重新分配内存，下面的在循环过程中不会重新分配内存
