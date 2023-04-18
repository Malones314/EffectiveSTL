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
