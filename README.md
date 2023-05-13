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
## 4.20
1. 字符串常见的4种实现方式
```cpp
1.1  string对象包括：
Allocator
大小
容量
指针；指针指向-->RefCnt(引用计数)+值
```
```cpp
1.2 string对象包括：
指针；指针指向-->大小，容量，RefCnt，其他数据，指针(指向-->值);
```
```cpp
1.3 stirng对象包括：
指针；指针指向-->大小，容量，RefCnt，值，与值的可共享性有关的数据
```
```cpp
1.4
当string容量小于15时
string对象包括：
Allocator，值，大小，容量

当string容量大于15时
string对象包括：
Allocator，大小，容量，未用空间，指针；指针指向-->值
```
2. 只有当字符串被频繁拷贝时，引用计数才有用，不常拷贝的内存不值得引用计数

3. decltype经验
```cpp
C++11

int x = 3;
decltype (x) y = 4;
int foo( int a, int b);
decltype( foo( 1, 2)) y = 3;  //y为int，y=4
```
```cpp
C++14

int x = 3;
decltype( auto) y = x;
int foo( int a, int b);
decltype( auto) y = foo( 1, 2); //y为int，值为foo的返回值
```
4. 将vector传到C API中
对于```vector<int> v```,希望将v传入```void dosomething( const int* pInt, size_t numberInts)```中
```cpp
if( !v.empty())
  dosomething( &v[0], v.size());
```
不能传```v.begin()```因为```v.begin()```返回的是迭代器，而非首元素, 可以传```&*v.begin()```
将string传到C API中
对于```string s```,希望将ｓ传入```void dosomething( const char* pString);```使用c_str成员函数
```cpp
dosomething( s.c_str());
```
使用C API初始化vector
```CPP
size_t fillArray(double* pArray, isze_t arraySize);
vector<double>vd( maxNumber);
vd.resize( fillArray(&vd[0], vd.size())); //使用fillArry向vd中写入数据，再把vd大小改成fillArray所写入的元素个数
```
同理可以完成对```vector<char>```的数据写入，再用```string s(vc.begin(), vc.begin()+charNumber);```完成s的构建
同城都是先写入vector再通过vector完成对于STL容器的写入
5.　使用```swap```除去多余容器
把容器从之前的大容量缩减到当前所需要的容量(称为shrink to fit)
```cpp
vector<container>(containers).swap(containers);
  //创建临时对象，是containers的拷贝，vector的拷贝构造函数只会拷贝需要的大小，
  //故此临时对象中没有多余容量，再与containers做swap操作，此时临时对象容量变为之前
  //containers容量。在语句尾，临时对象被析构,释放之前container所占据的内存
string也可以使用这种技巧
```
swap时不光两个容器内容交换，同时迭代器，指针，引用都交换(string除外)，迭代器，指针，引用依旧可用，并且指向同样的元素，只不过元素已经在另一个容器中
6. 避免使用```vector<bool>```
```vector<bool>```中并非存储了```bool```，存储的是```bool```的紧凑表示，一个```“bool”```仅仅占用一个二进制位
```vecotr<bool>::operator[]```返回一个代理对象
可以使用```deque<bool>```或```bitset```代替```vector<bool>```
## 4.21
1. 要为包含指针的关联容器指定比较类型
  比较模板
```cpp
struct DerederenceLess{
  template<typename ptrType>
  bool operator()( ptrType p1, ptrType p2) const {
    return *p1 < *p2;
  }
}
```
2. 总是让比较函数在等值时返回false
3. ```map```, ```multimap```的键无法更改，```set```, ```multiset```的键可以更改但不提倡，除非能保证容器依旧有序
4. 考虑用排序的```vector```来代替关联容器(排序的```vector```消耗更少的内存 )，用```vector```代替```map```时，用```pair<K,V>```而非```pair< const K, V>```
5. 当你很在乎效率时，谨慎决定选择map::operator[]或map::insert

## 4.29
1. 减少混用不同类型迭代器的机会，尽量使用iterator来代替const_iterator
2. 使用distance和advance来将容器的const_iterator转换成iterator
```cpp
具体实现：
advance( i, distance<const_iterator>( i, const_i));   //const_i为const_iterator

//advance是一个函数模板，定义在<iterator>头文件中，用于将指定迭代器移动指定距离。

//distance 是一个函数模板，定义在 <iterator> 头文件中，用于计算两个迭代器之间的距离。
//distance必须要加上const_iterator，否则出现二义性，因为i是非const的iterator，
//distance不知道调用const版本的函数还是非const版本的函数
```
3. 使用reverse_iterator的base() 成员函数可以得到与之对应的iterator
reverse_iterator 类型的迭代器 rit，并通过 rit.base() 获取其所适配的迭代器，
```rbegin()```函数所返回的迭代器指向的是容器中最后一个元素，因此调用 ```base()``` 函数所得到的原始迭代器指向的是容器中最后一个元素的下一个位置。
```rend()```函数所返回的迭代器指向的是容器中第一个元素的前一个位置，因此调用 ```base()``` 函数所得到的原始迭代器指向的是容器中第一个元素的位置。
```base()```函数是得到当前reverse_iterator的下一个位置的iterator
```cpp
         rend  begin           rbegin  end          
          |     |                 |     |
          |     |                 |     |
vector: | 空 |  1  |  2  |  3  |  4  |  空  |
```
```cpp
    std::vector<int> v = { 1, 2, 3, 4, 5 };
    auto rit = v.rend( ); // 获取反向迭代器
    auto it = rit.base( );
    std::cout << *(it ) << std::endl; // 输出 1
    rit = v.rbegin( );
    it = rit.base( );
    std::cout << *(it - 1) << std::endl; //输出5
```
## 4.30
对于逐个字符的输入考虑使用istreambuf_iterator
```cpp
fileName.unsetf( ios::skipws);  //禁止忽略fileName中的空白字符
```
```istreambuf_iterator<char>```对象直接从输入的缓冲区中读取下一个字符（```istreambuf_iterator<char>```对象从一个输入流```istream s```中读取下一个字符的操作是通过```s.rdbuf()->sgetc()```完成的。相比于常规的读取方法，如 ```s.get()```、```s.peek()```（用于预读取流中的下一个字符，但并不实际从流中取出该字符） 或者 ```s >> c``` 等等，```s.rdbuf()->sgetc()``` 的优点在于可以直接读取缓冲区中的下一个字符，而无需进行字符流的解析和格式化，因此速度相对较快。）
```cpp
ifstream inputFile( "inputName.txt");
string fileData( ( istreambuf_iterator<char>( inputFile)), istreambuf_iterator<char>() );
```
## 5.4
### 1. 对区间进行操作
如果所使用的算法需要指定一个目标区间，那么必须要确保目标区间足够大，或者确保它会随着算法的运行而增大，在算法执行过程中如果要加大目标区间，要使用插入型迭代器(由ostream_iterator, back_inserter, front_inserter返回的迭代器)
### 2. 做好排序算法的选择
```cpp
//使用compareFunction排序好container的前20个元素，使得container的前20个元素是有序的
partial_sort( container.begin(), container.begin() + 20, container.end(), cmopareFunction);
//因为 partial_sort 函数只保证前 k 个元素是有序的，所以排序结果不一定是完全有序的。
//如果容器中元素的数量小于 k，则对整个容器进行排序。

//如果你想取前十个最大的数字，但是有 13 个数字是等价最大的，
//partial_sort 可能会选择其中的任意 10 个数字作为前十个最大的数字，
//而其他 3 个数字则可能出现在第 11 到第 13 个位置。
```
```cpp
//使用compareFunction排序好container的前20个元素，但container的前20个元素是无序的
nth_element( container.begin(), containber.begin() + 19, container.end(), compareFunction);

//得到前20个经过compareFunction排序后的元素(这20个元素无序)
//nth_element 算法会将序列分为两部分，前半部分的元素都小于等于第 n 个元素，
//而后半部分的元素都大于等于第 n 个元素。并且，第 n 个元素在排序后应该处于它的最终位置上。
//如果容器中元素的数量小于 k，则对整个容器进行排序，相当于使用 sort 函数。
//如果你想取前十个最大的数字，但是有 13 个数字是等价最大的，
//无法确定在具体情况下 nth_element 会选择哪个等价最大的元素作为第 n 个元素。
```
```cpp
//如果需要相对位置排序，可以使用stable_sort 
stable_sort( container.begin(), container.begin() + 20, container.end(), cmopareFunction);
//排序前20个元素，序列中相等的元素在排序后的序列中的相对位置不变。
//使用自定义的cmopareFunction排序函数
```
```cpp
partition算法把所有符合条件的元素放在区间前部
bool hasAcceptable( const myClass& c){
  ........
}
vecotr<myClass>::iterator ci = partition( vmyClass.begin(), vmyClass.end, hasAcceptable); 
//将所有满足条件的元素移到前面，返回第一个不满足条件的myClass
```
#### 总结：
对vector、string、deque或者数组中元素执行操作，
如果要对所有元素进行一次完全排序，使用sort或stable_sort
如果要对等价性最前面的n个元素排序，使用partial_sort
如果要找到第n个位置上的元素，或要找等价性最前面的n个元素且不必对这前n个排序，使用nth_element

如果需要将标准序列容器中的元素按照是否满足某个特定的条件区分开来，使用partition或stable_partition

如果数据在list中，仍然可以使用partition和stable_partition；可以使用list::sort代替sort；但是不能直接使用partition_sort或nth_element
一种做法是把list中的元素拷贝到提供随机访问迭代器的容器中
第二种是创建一个list::iterator容器，对该容器执行相对的算法，然后通过其中的迭代器让问list的元素
第三种做法是利用一个包含迭代器的有序容器种的信息，反复调用splice成员函数，将list种的元素调整到期望的目标位置

根据空间时间效率，这些排序算法从资源消耗从低到高排序如下：
1. partition
2. stable_partition
3. nth_element
4. partial_sort
5. sort
6. stable_sort

## 5.5
1. remove有时并不是真正意义上的删除
```cpp
remove把要删除的元素移到容器最后，返回最后一个不符合删除条件的元素的指针
eg：
vector <int> v;
v.reserve( 10);
for( int i = 0; i < 10; i++){
  v.push_back(i);
}
v[0] = v[1] = v[2] = 1;
auto newEnd(remove( v.begin(), v.end(), 1));
cout << v.size(); //输出10；
v.erase(newEnd, v.end()); //真正删除元素
cout << v.size(); //输出7；
```
2. 当容器中存放的是动态分配的对象的指针时，要避免使用remove和类似算法。可以考虑使用partition算法。正确的做法是先把要删除的指针内容free后置空，然后删除所有空指针。最方便的做法是使用智能指针。
