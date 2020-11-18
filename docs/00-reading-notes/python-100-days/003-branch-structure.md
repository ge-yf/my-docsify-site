# 分支结构

## if语句

在Python中，要构造分支结构可以使用`if`、`elif`和`else`关键字。

和C/C++、Java等语言不同，Python中没有用花括号来构造代码块而是使用了缩进的方式来设置代码的层次结构，如果if条件成立的情况下需要执行多条语句，只要保持多条语句具有相同的缩进就可以了，换句话说连续的代码如果又保持了相同的缩进那么它们属于同一个代码块，相当于是一个执行的整体。

例如，分段函数求值:

$$f(x)=\begin{cases} 3x-5&\text{(x>1)}\\x+2&\text{(-1}\leq\text{x}\leq\text{1)}\\5x+3&\text {(x<-1)}\end{cases}$$


```python
"""
分段函数求值
        3x -5  (x > 1)
f(x) =  x + 2  (-1 <= x <= 1)
        5x + 3 (x < -1)
"""

x = float(input())


if x > 1:
    fx = 3 * x -5
elif x > -1:
    fx = x + 2
else:
    fx = 5 * x + 3

print(('fx = %d') % (fx))
```

分支可以嵌套，但是最好避免嵌套分支的情况。Python之禅中有这么一句话“Flat is better than nested.”，之所以提倡代码“扁平化”是因为嵌套结构的嵌套层次多了之后会严重的影响代码的可读性，所以能使用扁平化的结构时就不要使用嵌套。