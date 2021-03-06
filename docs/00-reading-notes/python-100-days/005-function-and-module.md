# 函数和模块

## 定义函数

Python使用`def`关键字来定义函数，和变量一样每个函数也有一个名字，而且命名规则跟变量的命名规则是一致的。在函数名后面的圆括号中可以放置传递给函数的参数，这一点和数学上的函数非常相似，程序中函数的参数就相当于是数学上说的函数的自变量，而函数执行完成后我们可以通过`return`关键字来返回一个值，这相当于数学上说的函数的因变量。


```python
"""
求阶乘

说明： 
Python的math模块中其实已经有一个factorial函数了，
事实上要计算阶乘可以直接使用这个现成的函数而不用自己定义。

下面例子中的一些函数在Python中也都是现成的，
我们这里是为了讲解函数的定义和使用才把它们又实现了一遍，
实际开发中不建议做这种低级的重复性的工作。
"""

def factorial(num):
    result = 1
    for n in range(1, num+1):
        result *= n
    return result

m = int(input('m = '))
print(factorial(m))
```

## 函数的参数

函数是绝大多数编程语言中都支持的一个代码的"构建块"，但是Python中的函数与其他语言中的函数还是有很多不太相同的地方，其中一个显著的区别就是Python对函数参数的处理。在Python中，**函数的参数可以有默认值，也支持使用可变参数**，所以Python并不需要像其他语言一样支持函数的重载，因为我们在定义一个函数的时候可以让它有多种不同的使用方式，下面是两个小例子。


```python
from random import randint

def roll_dice(n =2):
    total = 0
    for _ in range(n):
        total += randint(1, 6)
    return total

def add(a = 0, b = 0, c = 0):
    return a + b + c

print(roll_dice())   # 如果不指定参数，默认摇2个骰子
print(roll_dice(1))  # 摇1个骰子
print(add(1, 2))     # 1+2
print(add(1, 2, 3))  # 1+2+3
print(add(c = 1, a = 2, b = 3)) # 传递参数时，可以不按照设定的顺序
```


```python
"""
可变参数
"""

def add(*args):
    total = 0
    for val in args:
        total += val
    return total

#调用add时，可以传入任意数量的参数
print(add(1, 2))
print(add(1, 2, 3))
print(add(1, 2, 3, 4))
print(add(1, 2, 3, 4, 5))
```

## 用模块管理函数

为了解决命名冲突，python引入了模块的概念。

Python中每个文件就代表了一个模块（module），在不同的模块中可以有同名的函数，在使用函数的时候我们通过`import`关键字导入指定的模块就可以区分到底要使用的是哪个模块中的函数。

`module1.py`
```python
def foo():
    print('hello, world!')
```

`module2.py`
```python
def foo():
    print('goodbye, world!')
```

`test.py`
```python
from module1 import foo

# 输出hello, world!
foo()

from module2 import foo

# 输出goodbye, world!
foo()
```

也可以按照如下所示的方式来区分到底要使用哪一个`foo`函数。

`test.py`
```python
import module1 as m1
import module2 as m2

m1.foo()
m2.foo()
```

如果将代码写成了下面的样子，那么程序中调用的是最后导入的那个`foo`，因为后导入的`foo`覆盖了之前导入的`foo`。

`test.py`
```python
from module1 import foo
from module2 import foo

# 输出goodbye, world!
foo()
```

`test.py`
```python
from module2 import foo
from module1 import foo

# 输出hello, world!
foo()
```

需要说明的是，如果我们导入的模块除了定义函数之外还中有可以执行代码，那么Python解释器在导入这个模块时就会执行这些代码。为了避免这种情况，最好将这些执行代码放入如下所示的条件中，这样的话除非直接运行该模块，if条件下的这些代码是不会执行的，因为只有直接执行的模块的名字才是"`__main__`"。

`moudule3.py`
```python
def foo():
    pass

def bar():
    pass

# __name__是Python中一个隐含的变量，它代表了模块的名字
# 只有被Python解释器直接执行的模块的名字才是"__main__"
if __name__ == '__main__':
    print('call foo()')
    foo()
    print('call bar()')
    bar()
```

`test.py`
```python
import module3

# 导入module3时 不会执行if条件中的代码，因为模块的名字是"module3"而不是"__main__"
```

### 变量作用域问题

```python
def foo():
    b = 'hello'

    # Python中可以在函数内部再定义函数
    def bar():
        c = True
        print(a)
        print(b)
        print(c)

    bar()
    # print(c)  # NameError: name 'c' is not defined


if __name__ == '__main__':
    a = 100
    # print(b)  # NameError: name 'b' is not defined
    foo()
```

上面的代码能够顺利的执行并且打印出100、hello和True，但我们注意到了，在`bar`函数的内部并没有定义a和b两个变量，那么`a`和`b`是从哪里来的。我们在上面代码的`if`分支中定义了一个变量`a`，这是一个全局变量（global variable），属于**全局作用域**，因为它没有定义在任何一个函数中。在上面的`foo`函数中我们定义了变量`b`，这是一个定义在函数中的局部变量（local variable），属于**局部作用域**，在`foo`函数的外部并不能访问到它；但对于`foo`函数内部的`bar`函数来说，变量`b`属于**嵌套作用域**，在`bar`函数中我们是可以访问到它的。`bar`函数中的变量`c`属于局部作用域，在`bar`函数之外是无法访问的。事实上，Python查找一个变量时会按照“局部作用域”、“嵌套作用域”、“全局作用域”和“内置作用域”的顺序进行搜索，前三者我们在上面的代码中已经看到了，所谓的**内置作用域**就是Python内置的那些标识符，我们之前用过的`input`、`print`、`int`等都属于内置作用域。

再看看下面这段代码，我们希望通过函数调用修改全局变量a的值，但实际上下面的代码是做不到的。


```python
def foo():
    a = 200
    print(a)  # 200


if __name__ == '__main__':
    a = 100
    foo()
    print(a)  # 100
```

在调用`foo`函数后，我们发现`a`的值仍然是100，这是因为当我们在函数`foo`中写`a = 200`的时候，是重新定义了一个名字为`a`的局部变量，它跟全局作用域的`a`并不是同一个变量，因为局部作用域中有了自己的变量`a`，因此`foo`函数不再搜索全局作用域中的`a`。如果我们希望在`foo`函数中修改全局作用域中的`a`，代码如下所示。


```python
def foo():
    global a
    a = 200
    print(a)  # 200


if __name__ == '__main__':
    a = 100
    foo()
    print(a)  # 200
```

使用`global`关键字来指示`foo`函数中的变量`a`来自于全局作用域，如果全局作用域中没有`a`，那么下面一行的代码就会定义变量`a`并将其置于全局作用域。同理，如果我们希望函数内部的函数能够修改嵌套作用域中的变量，可以使用`nonlocal`关键字来指示变量来自于嵌套作用域。

在实际开发中，我们应该尽量减少对全局变量的使用，因为全局变量的作用域和影响过于广泛，可能会发生意料之外的修改和使用，除此之外全局变量比局部变量拥有更长的生命周期，可能导致对象占用的内存长时间无法被垃圾回收。事实上，减少对全局变量的使用，也是降低代码之间耦合度的一个重要举措，同时也是对迪米特法则的践行。减少全局变量的使用就意味着我们应该尽量让变量的作用域在函数的内部，但是如果我们希望将一个局部变量的生命周期延长，使其在定义它的函数调用结束后依然可以使用它的值，这时候就需要使用闭包，这个我们在后续的内容中进行讲解。
