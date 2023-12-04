## 闭包

### 1. 何为闭包

`闭包（Closure）`，又称`词法闭包（Lexical Closure）`或`函数闭包（function closures）`，是引用了自由变量的函数。这个被引用的自由变量将和这个函数一同存在，即使已经离开了创造它的环境也不例外。所以，有另一种说法认为闭包是由函数和与其相关的引用环境组合而成的实体。闭包在运行时可以有多个实例，不同的引用环境和相同的函数组合可以产生不同的实例。

`自由变量`：如果在一个代码块中使用了一个变量，而这个变量并没有被定义在该代码块中，那么该变量就被称为自由变量。

创建闭包的条件：
- 必须包含一个嵌套函数；
- 嵌套函数必须引用封闭函数中定义的值（自由变量）；
- 封闭函数必须返回嵌套函数。

### 2. 嵌套函数

在另一个函数中定义的函数称为嵌套函数，需要注意的是，嵌套函数能够访问封闭范围中的变量。在这里，inner()就是嵌套函数，它可以很容易地访问变量msg:

```python
def outer(msg):
    def inner():
        print(msg)
    return inner()

outer('hello')
```

结果：
```
hello
```

### 3. 定义闭包

对上述示例略作修改，让它直接返回函数对象，而不是调用嵌套函数：

```python
def outer(msg):
    def inner():
        print(msg)
    return inner

func = outer('hello')
print(func)
func()
```

结果：
```
<function outer.<locals>.inner at 0x000001DA3BE1A8C8>
hello
```

当外部函数outer(msg)被调用时，一个闭包inner()就形成了，并且它持有自由变量msg。这意味着，当函数outer(msg)的生命周期结束之后，变量msg的值依然会被记住：

```python
def outer(msg):
    def inner():
        print(msg)
    return inner

func = outer('hello')
print(func)
func()
del outer
func()
```

结果：
```
<function outer.<locals>.inner at 0x000001E43125A8C8>
hello
hello
```

### 4. 闭包的好处

- 闭包可以避免使用全局变量，并提供某种形式的数据隐藏；
- 当只有几个方法（通常只有一个）时，使用闭包比实现一个类更加简单；
- 闭包允许我们在其范围之外调用 Python 函数；
- python 中的装饰器广泛使用了闭包

如果要创建一个由不同参数构成的一系列函数（例如：平方和立方），使用传统方式，需要分别实现：

```python
def square(x):
    return x ** 2

def cube(x):
    return x ** 3

print(square(2))
print(cube(2))
```

结果：
```
4
8
```

换用闭包的话，仅需一个pow()就可以搞定了：

```python
def pow(y):
    def inner(x):
        return x ** y
    return inner

square = pow(2)
cube = pow(3)

print(square(2))
print(cube(2))
```

结果：
```
4
8
```

这样做的好处是：pow() 可以用来构建任何一个指数（1、2、3 …）。所有函数对象都有一个` __closure__ `属性，如果这个函数是一个闭包函数，那么该属性会返回一个由`cell对象`组成的元组。而`cell对象`有一个`cell_contents`属性，存储了闭包中的自由变量：

```python 
def pow(y):
    def inner(x):
        return x ** y
    return inner

cube = pow(3)

print(cube.__closure__)
print(cube.__closure__[0].cell_contents)
```

结果：
```
(<cell at 0x00000260775895B8: int object at 0x00000000638DC6F0>,)
3
```

这也解释了为什么局部变量在脱离函数之后，还可以在函数之外被访问，因为它存储在了闭包的`cell_contents`中。

### 5. nonlocal

```python
def outer():
    count = 0
    def inner():
        count += 1
        print('count:', count)
    return inner

f = outer()
f()
```

结果：
```
Traceback (most recent call last):
  File "C:/Users/root/Desktop/HelloWorld/test/script/test.py", line 9, in <module>
    f()
  File "C:/Users/root/Desktop/HelloWorld/test/script/test.py", line 4, in inner
    count += 1
UnboundLocalError: local variable 'count' referenced before assignment
```

因为count是一个不可变类型，当在内部范围对其重新分配时，它会被看作是一个新变量，由于它还没有被定义，所以会发生错误。

Python 3.x 引入了`nonlocal`关键字，用于标识外部作用域的变量：

```python
def outer():
    count = 0
    def inner():
        nonlocal count
        count += 1
        print('count:', count)
    return inner

f = outer()
f()
f()
```

结果：
```
count: 1
count: 2
```
