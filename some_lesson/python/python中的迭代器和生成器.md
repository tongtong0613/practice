### 1.迭代器

**可迭代对象**：迭代是指重复反馈过程，每一次的迭代都会得到一个结果，又是下一次迭代的开始。在python中，一个对象只要是实现了`__iter__() `或`__getitem__()`方法，就被称为可迭代对象。

python中的可迭代对象有字符串、列表、元组、字典、文件；自定义的类若是实现了`__iter__() `或`__getitem__()`方法，则也是可迭代对象。

**迭代器**：调用可迭代对象的`__iter__()` 方法，返回得到的就是一个迭代器，迭代器用于迭代可迭代对象中的每一个元素。

迭代器有两个方法：`__iter__() `方法，`__next__() `方法，调用迭代器的`__iter__()` 方法，返回的依旧是迭代器对象，即将自身返回。不断的调用`__next__() `方法，则会逐个返回可迭代对象中的元素。

在获取可迭代对象的迭代器之后，不断调用迭代器的`__next__() `方法，以遍历其中的所有元素，当全部遍历完成后，再次调用`__next__()` 方法，就会抛出 `StopIteration` 异常。

```python
with open(file='./w.txt', mode='r', encoding='utf-8') as f:
    file_iter = f.__iter__()
    print(file_iter.__next__(), end='')
    print(file_iter.__next__(), end='')
    print(file_iter.__next__(), end='')
    print(file_iter.__next__(), end='')
```

结果：
```
{'fault': 'The zone does not exists!', 'Detail': 'no detail'}
{'fault': 'Parameter error !', 'Detail': 'no detail'}
{'ret': '0'}
Traceback (most recent call last):
  File "C:/Users/root/Desktop/HelloWorld/test/script/test.py", line 6, in <module>
    print(file_iter.__next__(), end='')
StopIteration

Process finished with exit code 1
```

`for item in Iterable` 循环会调用 in 后面对象的 `__iter__()` 方法，得到迭代器，然后自动的，不断的 调用迭代器的`__next__()`方法，得到的返回值 赋值给 for 前面的item 变量，这样依次循环；直到调用`__next__()`方法时报错（StopIteration异常），for循环会自动捕获异常，然后循环结束。

```python
with open(file='./w.txt', mode='r', encoding='utf-8') as f:
    file_iter = f.__iter__()
    for item in file_iter:
        print(item, end='')
```

结果：
```
{'fault': 'The zone does not exists!', 'Detail': 'no detail'}
{'fault': 'Parameter error !', 'Detail': 'no detail'}
{'ret': '0'}

Process finished with exit code 0
```

`Iterable`, `Iterator` 用于判断一个对象是不是可迭代对象或者是不是迭代器：

```python
from collections import Iterable, Iterator

f = open(file='./w.txt', mode='r', encoding='utf-8')
l = [1, 2, 3]

print(isinstance(f, Iterable))
print(isinstance(f, Iterator))
print(isinstance(l, Iterable))
print(isinstance(l, Iterator))
```

结果：
```
True
True
True
False
```

**迭代器的特性**:

	总结一下，迭代器有以下 2 个特性：
	1）提供了一种不依赖于索引的取值方式
	2）惰性计算，节省内存

	这里的节省内存是指 迭代器在迭代过程中，不会一次性把可迭代对象的所有元素都加载到内存中，仅仅是在迭代至某个元素时才加载该元素，而在这之前或之后，元素可以不存在或者被销毁。这个特点就使得迭代器适合用于遍历一些巨大的或是无限的集合。

	迭代器的优缺点
	1）取值不如按照索引取值来的方便（索引可以直接定位某一个值，迭代器不行，只能一个一个地取下去）。
	2）迭代器只能往后迭代，不能回退（执行__next__() 方法 只能向后，不能向前）。
	3）无法获取迭代器的长度。


### 2.生成器

**生成器函数和生成器**：
生成器函数：函数体内包含有`yield`关键字，该函数执行的结果就是一个生成器（`generator`）。

生成器具有 `__next__() `方法 和`__iter__()`，所以生成器就是一种迭代器。调用生成器函数后，返回`generator`对象，然后通过调用 `__next__()` 方法不断获得下一个返回值。

```python
def foo():
    print('step1 ---')
    yield 1
    print('step2 ---')
    yield 2
    print('step3 ---')
    yield 3
    print('step4 ---')


f = foo()
print(next(f))
print(f.__next__())
print(next(f))
print(f.__next__())
```

结果：
```
step1 ---
1
step2 ---
2
step3 ---
3
step4 ---
Traceback (most recent call last):
  File "C:/Users/root/Desktop/HelloWorld/test/script/test.py", line 15, in <module>
    print(f.__next__())
StopIteration
```

由于生成器也是一种迭代器，因此同样可以使用`for item in generator` 循环会调用in后面对象的` __iter__() `方法，得到生成器，然后自动的，不断的调用迭代器的`__next__()`方法，得到的返回值赋值给 for前面item变量，这样依次循环；直到调用`__next__()`方法时报错（`StopIteration`异常），for循环会自动捕获异常，然后循环结束。

```python
def foo():
    print('step1 ---')
    yield 1
    print('step2 ---')
    yield 2
    print('step3 ---')
    yield 3
    print('step4 ---')


f = foo()
for item in f:
    print(item)
```

结果：
```
step1 ---
1
step2 ---
2
step3 ---
3
step4 ---
```

yield功能总结：

	1）与return类似，都可以返回值，但不一样的地方在于yield返回多次值，而 return 只能返回一次。
	2）为函数封装好了__iter__() 和 __next__() 方法，把函数的执行结果做成了迭代器。
	3）遵循迭代器的取值方式（obj.__next__()），这样的操作会触发函数的执行，函数暂停与再继续的状态都是由 yield 保存，（暂停于yield处，下一次的 __next__() 方法执行后，会从 yield 处继续往下执行）。

```python
def foo():
    print('start...')
    while True:
        x = yield
        print(x)

f = foo()
next(f)
f.send(2)
f.send(8)
next(f)
```

结果：
```
start...
2
8
None
```

这里生成器的`send()`方法会先将值传递给`yield`，然后由`yield`赋值给`x`，赋值完成之后，继续往下执行，直到再一次遇到`yield`。所以`send`的作用和`next`方法相同，还多了一个赋值功能。

`send`的2个作用：
1. 传值给yield，然后由yield传递给变量（若没有传值给yield，则yield会将None赋值给变量）
2. 与next相同的功能

上述示例中使用send之前，必须对生成器来一个类似于初始化的操作：执行next或者send(None)。为了简化这个步骤，这里可以使用装饰器：

```python
def init(func):
    def wrapper(*args, **kwargs):
        f = func(*args, **kwargs)
        next(f)
        return f
    return wrapper

@init
def foo():
    print('start...')
    while True:
        x = yield
        print(x)

f = foo()
f.send('abc')
```

结果：
```
start...
abc
```

yield表达式一共有4种：
>1. yield exp，仅有返回值，exp可以是函数，表达式等
>2. s = yield exp，有返回值，且可以传入一个值存入s中
>3. s = yield，可传入一个值，没有返回值（返回为None）
>4. yield，不接受输入，也没有返回值（返回为None）

**生成器的应用**

python中的生成器（generator）通常用来实现协程，即在执行一个函数的过程中，中断当前函数，转而去执行别的函数。

执行完成之后，返回来继续当前函数的执行，整个过程在一个线程中完成；也可以换个角度来进行理解，即当前函数的循环执行会不断产生数据，将每一次产生的数据交由另一个函数做进一步的处理，处理完成之后返回，继续执行当前函数。

下面通过一个简单的生产者-消费者模型来说明：

```python
def init(func):
    def wrapper(*args, **kwargs):
        f = func(*args, **kwargs)
        next(f)
        return f
    return wrapper

@init
def consumer():
    res = ''
    while True:
        p = yield res
        if not p:
            continue
        print('consuming %s...' % p)
        res = 'ok'

def producer(c):
    for i in range(1, 5):
        print('producing %s...' % i)
        r = c.send(i)
        print('return status %s' % r)
    c.close()

c = consumer()
producer(c)
```

结果：
```
producing 1...
consuming 1...
return status ok
producing 2...
consuming 2...
return status ok
producing 3...
consuming 3...
return status ok
producing 4...
consuming 4...
return status ok
```

执行流程说明：
1. c = consumer()拿到的已经是初始化后的生成器（即生成器已经执行了一次next(c)）
2. 调用 produce()，生产数据之后，通过send(i)，将数据发送给consumer，并且切换到consumer执行；
3. consumer 通过yield获取数据，然后进行消费，最后通过yield把处理结果返回给produce；
4. produce获取consumer的处理结果之后，继续生产下一次的数据。