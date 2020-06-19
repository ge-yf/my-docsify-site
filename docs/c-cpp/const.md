# const 关键字

## 作用

* 修饰变量，说明该变量不可以被改变；
* 修饰指针，分为指向常量的指针（pointer to const）和自身是常量的指针（常量指针，const pointer）；
* 修饰引用，指向常量的引用（reference to const），用于形参类型，即避免了拷贝，又避免了函数对值的修改；
* 修饰成员函数，说明该成员函数内不能修改成员变量。

## const指针与const引用

### const指针

指向常量的指针(pointer to const)(指针指向的地址可以改变，但是地址中的内容不能改变)：

```cpp
const int a = 1;
const int *p_a = &a;
```

指针常量(const pointer)(指针指向的地址不可以改变，但是地址中的内容可以改变)：

```cpp
int a = 1;
int * const p_a = &a;
```

指向常量的常量指针(const pointer that point to const)(地址和内容都不能改变)：

```cpp
const int a = 1;
const int * const p_a = &a;
```

### const引用

指向常量的引用(reference to const)(引用一个常量):

```cpp
const int a = 1;
const int & ra = a;
```

不存在const reference，因为reference本身就是const的。

const在变量名前，修饰变量(变量时const的)，在指针的后面时，修饰指针(指针是const的)。
