## 运算符重载

### 1. 基础知识

运算符重载，意味着在某个类的方法中拦截内置的操作，当类的实例出现在内置操作中时，Python会自动调用你的方法，并且你的方法的返回值会作为相应操作的结果。

1. 运算符重载让类拦截常规的Python操作

2. 类可重载所有Python表达式运算符

3. 类也可重载打印、函数调用、属性访问等内置运算

4. 重载使类实例的行为更接近内置类型

5. 重载是通过在一个类中提供特殊名称的方法来实现的

下面是一个简单的构造函数`__init__`和表达式`__sub__`的运算符重载:
```python
class Number:
    def __init__(self, data):
        self.data = data

    def __sub__(self, other):
        return Number(self.data-other)

X = Number(5)
Y = X - 2
print(Y.data)
```
执行结果：
```
3
```

### 2. 索引和分片：`__getitem__`和`__setitem__`

当实例X出现在X[i]这样的索引运算中时，Python会调用这个实例继承的`__getitem__`方法，把X作为第一位参数传入，并且将方括号内的索引值传递给第二个参数:

```python
class Indexer:
    def __getitem__(self, item):
        return item ** 2

X = Indexer()
print(X[1])
print(X[2])
```

执行结果：
```
1
4
```

`__getitem__`是查看类属性的值，可以通过索引，而`__setitem__`是更新类属性的值，主要是针对有键值对的:

```python
class Indexer:
    data = [1, 2, 3]
    def __getitem__(self, item):
        return self.data[item]

    def __setitem__(self, key, value):
        self.data[key] = value

X = Indexer()
print(X[1])
X[1] = 22
print(X[1])
```

执行结果：
```
2
22
```

### 3. 索引迭代：`__getitem__`

如果一个类实现了`__getitem__`方法，那么它就是可迭代对象，可以通过`list()`函数直接查看实例中的内容：
```
class Indexer:
    data = [1, 2, 3]
    def __getitem__(self, item):
        return self.data[item]

X = Indexer()
print(list(X))
```

执行结果：
```
[1, 2, 3]
```

### 4. 可迭代对象：`__iter__`和`__next__`

虽然上面说的`__getitem__`可用，但是Python中所有的迭代上下文都会先尝试`__iter__`方法，再尝试`__getitem__`,也就是说我们应该优先调整`__iter__`方法。

通过`__iter__`和`__next__`方法的结合使用，可以实现类实例的迭代：

```python
class Square:
    def __init__(self, start, stop):
        self.value = start - 1
        self.stop = stop

    def __iter__(self):
        return self

    def __next__(self):
        if self.value == self.stop:
            raise StopIteration
        self.value += 1
        return self.value ** 2

s = Square(1, 3)
iter_obj = iter(s)
print(next(iter_obj))
print(next(iter_obj))
print(next(iter_obj))
print(next(iter_obj))
```

执行结果：
```
1
4
9
16
Traceback (most recent call last):
  File "C:/Users/root/Desktop/HelloWorld/test/script/test.py", line 21, in <module>
    print(next(iter_obj))
  File "C:/Users/root/Desktop/HelloWorld/test/script/test.py", line 11, in __next__
    raise StopIteration
StopIteration
```

### 5. 属性访问：`__getattr__`和`__setattr__`

`__getattr__`是访问属性，`__setattr__`是修改属性。

先创建一个简单的类，如果要访问不存在的属性，系统会出现提示信息:

```python
class Empty:
    def __getattr__(self, item):
        if item == 'age':
            return 40
        else:
            raise AttributeError(item)

X = Empty()
print(X.age)
print(X.name)
```

执行结果：
```
40
Traceback (most recent call last):
  File "C:/Users/root/Desktop/HelloWorld/test/script/test.py", line 10, in <module>
    print(X.name)
  File "C:/Users/root/Desktop/HelloWorld/test/script/test.py", line 6, in __getattr__
    raise AttributeError(item)
AttributeError: name
```

对于类，如果你直接添加属性和值是可以的，但是可以通过__setattr__来屏蔽:

```python
class Empty:
    def __getattr__(self, item):
        if item == 'age':
            return 40
        else:
            raise AttributeError(item)

    def __setattr__(self, key, value):
        if key == 'age':
            self.__dict__[key] = value + 10
        else:
            raise AttributeError(key + ' not allowed')

X = Empty()
print(X.age)
X.age = 50
print(X.age)
X.name = 'tom'
```

执行结果：
```
40
60
Traceback (most recent call last):
  File "C:\Program Files\JetBrains\PyCharm 2020.3.4\plugins\python\helpers\pydev\pydevd.py", line 1477, in _exec
    pydev_imports.execfile(file, globals, locals)  # execute the script
  File "C:\Program Files\JetBrains\PyCharm 2020.3.4\plugins\python\helpers\pydev\_pydev_imps\_pydev_execfile.py", line 18, in execfile
    exec(compile(contents+"\n", file, 'exec'), glob, loc)
  File "C:/Users/root/Desktop/HelloWorld/test/script/test.py", line 18, in <module>
    X.name = 'tom'
  File "C:/Users/root/Desktop/HelloWorld/test/script/test.py", line 12, in __setattr__
    raise AttributeError(key + ' not allowed')
AttributeError: name not allowed
```

### 6. 字符串显示：`__repr__`和`__str__`

`__str__`：会首先被打印操作和str内置函数尝试（print运行的内部等价形式），它通常应当返回一个用户友好的显示。

`__repr__`：用于所有其他的场景，如果没有定义`__str__`则打印操作也会使用`__repr__`，也就是说可以用于任何地方。

如果没有定义__str__和__repr__的情况下：

```python
class Text:
    def __init__(self, data):
        self.data = data
        
X = Text(5)

In[1]:X
Out[2]: <__main__.Text at 0x17a75271898>
In[2]:print(X)
Out[2]:<__main__.Text object at 0x0000017A75271898>
```

直接调用实例或者打印实例，都是显示一串代码，它是说明该实例的内存地址。如果定义`__repr__`：

```python
class Text:
    def __init__(self, data):
        self.data = data

    def __repr__(self):
        return '[R : %s]' % self.data

X = Text(5)

In[1]:X
Out[2]: [R : 5]
In[2]:print(X)
Out[2]:[R : 5]
```

接着定义`__str__`：
```python
class Text:
    def __init__(self, data):
        self.data = data

    def __repr__(self):
        return '[R : %s]' % self.data
    
    def __str__(self):
        return '[S : %s]' % self.data

X = Text(5)
In[1]:X
Out[2]: [R : 5]
In[2]:print(X)
Out[2]: [S : 5]
```

### 7. 加法：`__radd__`和`__iadd__`

常规定义的__add__一般都是右侧加法，也就是相加的对象在实例的右侧，如果是左侧，系统会提示错误:

```python
class Adder:
    def __init__(self, value=0):
        self.value = value

    def __add__(self, other):
        return self.value + other

X = Adder(5)
print(X+2)
print(2+X)
```

执行结果：
```
7
Traceback (most recent call last):
  File "C:/Users/root/Desktop/HelloWorld/test/script/test.py", line 10, in <module>
    print(2+X)
TypeError: unsupported operand type(s) for +: 'int' and 'Adder'
```

通过定义`__radd__`来解决这个问题:
```python
class Adder:
    def __init__(self, value=0):
        self.value = value

    def __add__(self, other):
        return self.value + other

    def __radd__(self, other):
        return self.__add__(other)

X = Adder(5)
print(X+2)
print(2+X)
```

执行结果：
```
7
7
```
`+=`这种操作可以通过`__iadd__`来定义：

```python
class Adder:
    def __init__(self, value=0):
        self.value = value

    def __add__(self, other):
        return self.value + other

    def __radd__(self, other):
        return self.__add__(other)

    def __iadd__(self, other):
        self.value = self.value + other
        return self

X = Adder(5)
X += 3
print(X.value)
```

执行结果：
```
8
```

### 8. 调用表达式：`__call__`

通过定义__call__可以实现实例不同的调用:

```python
class Caller:
    def __call__(self, *args, **kwargs):
        print('Called: ', args, kwargs)

C = Caller()
C()
C(1, 2, 3)
C(1, 2, 3, x=4, y=5)
```

执行结果：
```
Called:  () {}
Called:  (1, 2, 3) {}
Called:  (1, 2, 3) {'x': 4, 'y': 5}
```

### 9. 比较运算：`__lt__`、`__gt__`和其他方法

```python
class C:
    def __init__(self):
        self.data = 'tom'

    def __lt__(self, other):
        return self.data < other

    def __gt__(self, other):
        return self.data > other

c = C()
print(c > 'haha')
print(c < 'haha')
```

执行结果：
```
True
False
```

### 10. 布尔测试：`__bool__`和`__len__`

Python会首先尝试__bool__来获取一个直接的布尔值，如果没有该方法，就退而求其次的尝试__len__来根据对象的长度确定真值。

系统默认实例对象是真的，如果需要我们特殊的处理，可以通过这个定义不一样的结果，先看系统默认的情况：

```python
class T:
    pass

t = T()
print(bool(t))
```

执行结果：
```
True
```

定义新的结果：

```python
class T:
    def __bool__(self):
        return False

t = T()
print(bool(t))
```

执行结果：
```
False
```

### 11. 对象析构函数：`__del__`

例创建的时候会调用__init__来创建，如果实例空间被回收时，就会调用__del__,一般来说，系统是自动调用完成的，在此我们可以了解这个如何运作的，析构函数一般不常用，但是很重要:

```python
class Life:
    def __init__(self, name='unknown'):
        print('Hello', name)
        self.name = name

    def live(self):
        print(self.name)

    def __del__(self):
        print("Bye", self.name)

l = Life('tom')
l.live()
l = '123'
```

执行结果：
```
Hello tom
tom
Bye tom
```