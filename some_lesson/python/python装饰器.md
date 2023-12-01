## 装饰器

Python 对某个对象是否能通过装饰器（` @decorator`）形式使用只有一个要求：decorator 必须是一个“`可被调用（callable）的对象`”。

Python中的可调用对象主要有以下几种：
1. 函数
2. 实现了__call__函数的类
3. 偏函数

因此装饰器的种类也主要由以上几种可调用对象进行区分。

### 1. 不带参数的函数装饰器

装饰器放在一个函数开始定义的地方，它就像一顶帽子一样戴在这个函数的头上，和这个函数绑定在一起。在我们调用这个函数的时候，第一件事并不是执行这个函数，而是将这个函数做为参数传入它头顶上这顶帽子，这顶帽子我们称之为装饰器。

下面是一个简单的不带参数的函数装饰器，不传参的装饰器，只能对被装饰函数，执行固定逻辑。add函数被traceroute装饰器装饰，因此add函数作为一个参数传入traceroute函数。

```python
import sys
from functools import wraps

def traceroute(fn):
    @wraps(fn)
    def wrapper(*args, **kwargs):
        ret = fn(*args, **kwargs)
        filename = './traceroute.txt'
        with open(filename, 'a') as f:
            f.write("--- Current function is: {}\n".format(fn.__name__))
            f.write("--- Args: {}\n".format([x for x in args[1:]]))
            f.write("--- Kwargs: {}\n".format({key:value for key, value in kwargs.items()}))
            f.write("--- Function belongs to class: {}\n".format(args[0].__class__.__name__))
            f.write("--- Current function from: {}\n".format(sys._getframe().f_code.co_filename))
            f.write("--- Called by function: {}\n".format(sys._getframe(1).f_code.co_name))
            f.write("--- Called at line: {}\n".format(sys._getframe().f_back.f_lineno))
            f.write("--- Called from line: {}\n".format(sys._getframe().f_back.f_code.co_filename))
            f.write("\n")
        return ret
    return wrapper

class Trace():
    @traceroute
    def add(self, a, b):
        print(a + b)

if __name__ == '__main__':
    t = Trace()
    t.add(2, 3)
```

### 2. 带参数的函数装饰器

装饰器本身是一个函数，做为一个函数，如果不能传参，那这个函数的功能就会很受限，只能执行固定的逻辑。这意味着，如果装饰器的逻辑代码的执行需要根据不同场景进行调整，若不能传参的话，我们就要写两个装饰器，这显然是不合理的。

以下是一个带参数的函数装饰器，相对于不带参数的函数装饰器，多了一层嵌套，且最外层函数接受的参数不再是被装饰的函数，而是装饰器的参数：

```python
def say(country):
    def wrapper(func):
        def deco(*args, **kwargs):
            if country == 'china':
                print('你好')
            elif country == 'us':
                print('hello')
            else:
                return
            func(*args, **kwargs)
        return deco
    return wrapper

@say('china')
def a():
    pass

@say('us')
def b():
    pass
```

### 3. 不带参数的类装饰器

基于类装饰器的实现，必须实现 `__call__` 和 `__init__`两个内置函数。 `__init__` ：接收被装饰函数 `__call__` ：实现装饰逻辑。

```python
class Logger1(object):
    def __init__(self, func):
        self.func = func

    def __call__(self, *args, **kwargs):
        print("[INFO] the function {func}() is running".format(func=self.func.__name__))
        return self.func(*args, **kwargs)

class Say():
    @Logger1
    def say1(self, some):
        print('say %s' % some)

if __name__ == '__main__':
    s = Say()
    s.say1('111')
```

### 4. 带参数的类装饰器

带参数和不带参数的类装饰器有很大的不同。`__init__` ：不再接收被装饰函数，而是接收传入参数。 `__call__` ：接收被装饰函数，实现装饰逻辑。

```python
class Logger2(object):
    def __init__(self, level='INFO'):
        self.level = level

    def __call__(self, func):
        def wrapper(*args, **kwargs):
            print('[{level}] the function {func}() is running'.format(level=self.level, func=func.__name__))
            func(*args, **kwargs)
        return wrapper

class Say():
    @Logger2(level='DEBUG')
    def say2(self, some):
        print('say %s' % some)

if __name__ == '__main__':
    s = Say()
    s.say2('222')
```

### 5. 使用偏函数与类实现装饰器

偏函数也是callable对象，因此其也可作为装饰器使用。如下所示，DelayFunc 是一个实现了__call__的类，delay返回一个偏函数，在这里delay 就可以做为一个装饰器。


```python
class DelayFunc:
    def __init__(self, duration, func):
        self.duration = duration
        self.func = func

    def __call__(self, *args, **kwargs):
        print(f"Waiting {self.duration} second...")
        time.sleep(self.duration)
        return self.func(*args, **kwargs)


def delay(duration):
    return functools.partial(DelayFunc, duration)

@delay(2)
def cal(a, b):
    print(a + b)

if __name__ == '__main__':
    cal(5, 6)
```

### 6. 装饰类的装饰器

装饰器用在类上，并不是很常见，但只要熟悉装饰器的实现过程，就不难以实现对类的装饰。在下面这个例子中，装饰器就只是实现对类实例的生成的控制，以保证其符合单例模式。

```python
def singeton(cls):
    _instance = {}

    def get_instance(*args, **kwargs):
        cls_name = cls.__name__
        if cls_name not in _instance:
            instance = cls(*args, **kwargs)
            _instance[cls_name] = instance
        return _instance[cls_name]
    return get_instance

@singeton
class User:
    def __init__(self, name):
        self.name = name

if __name__ == '__main__':
    u = User('tom')
    u = User('jack')
```

