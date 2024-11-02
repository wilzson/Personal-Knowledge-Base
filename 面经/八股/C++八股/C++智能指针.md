# C++智能指针

智能指针是一种封装了原始指针的智能对象，它的使用方法类似于普通指针，但它可以自动管理所指向的资源，避免了常见的内存泄露和悬挂指针等问题。

## unique_ptr

unique_ptr是一种独占式智能指针，它确保了只有一个指针可以指向资源。可以通过move( )函数将资源的所有权转移给其他的unique_ptr对象

```c++
#include<memory>
#include<iostream>
using namespace std;
int main() {
    unique_ptr<int> p(new int (42));
    cout<<*p<< endl;
    int* q = p.release();
    p.reset(new int (43));
    cout<<*q<<"  "<<*p;
    int* r = p.get();
    cout<<r<<endl;
}

```



## shared_ptr

shared_ptr是一种共享式智能指针，它可以被多个指针共享同一个资源，资源的引用计数会被自动管理，当所有shared_ptr对象都不再需要该资源时，资源会自动被销毁。

```c++
#include<memory>
#include<iostream>
using namespace std;
int main() {
    shared_ptr<int> p(new int (42));
    shared_ptr<int> r = make_shared<int>(50); // 动态分配一个shared_ptr类型的数据
    cout<<*p<<endl;
    // 创建一个新的shared_ptr对象，与p共享同一个资源
    shared_ptr<int> q(p);
    shared_ptr<int> e(q);
    cout<<p.use_count()<<endl; // 返回p的引用计数
}

```

