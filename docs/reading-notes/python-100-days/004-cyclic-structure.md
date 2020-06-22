# 循环结构

在Python中构造循环结构有两种做法，一种是`for-in`循环，一种是`while`循环。

## for-in循环

如果明确知道循环执行的次数或者要对一个容器进行迭代，推荐使用for-in循环。

例如下面代码中计算1~100求和的结果（$\displaystyle \sum \limits_{n=1}^{100}n$）


```python
"""
for-in循环实现1~100求和
"""

sum = 0
for i in range(101):   # range(x, y, step)构造一个从x到y -1的取值范围, 步长(增量)为step
    sum += i

print(('sum(1~100) = %d') % (sum))
```


```python
"""
for-in 循环实现1~100之间的偶数求和
"""

sum = 0
for i in range(2, 101, 2):   # 2, 4, 6, 8...
    sum += i

print(('sum(1~100) = %d') % (sum))
```

## while循环

要构造不知道具体循环次数的循环结构，推荐使用`while`循环。

`while`循环通过一个能够产生或转换出`bool`值的表达式来控制循环，表达式的值为`True`循环继续，表达式的值为`False`循环结束。


```python
"""
猜数字游戏

计算机出一个1~100之间的随机数由人来猜
计算机根据人猜的数字分别给出提示大一点/小一点/猜对了
"""

import random

answer = int(random.randint(1, 100))
counter = 0

while True:
    counter += 1
    num = int(input('Please input a number :'))
    
    if num > answer:
        print('smaller')
    elif num < answer:
        print('bigger')
    else:
        print('congraduations!')
        break
    
    if counter >5:
        print('failed!')
```

## 应用


```python
"""
正整数的反转
"""

num = int(input('Please input a positive integer:'))
result = 0

while num > 0:
    result *= 10
    result = (result * 10) + (num % 10)
    num //= 10

print(('result : %d') % (result))
```
