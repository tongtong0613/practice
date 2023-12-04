## 单例模式

单例模式：一个类只能有一个实例化对象存在的模式

### 1.使用模块

python中模块是天然的单例模式，当一个模块被调用时，会生成对应的.pyc文件，接下来的每次使用都会自动读取.pyc文件里的内容，因此，要使用模块实现单例，只需这样做：

```python
class Singleton:
    def func(self):
        pass

singleton = Singleton()
```

在使用时只需引用该模块的singleton对象即可。

### 2.使用装饰器

```python
def singleton(cls):
    _instance = {}
    def wrapper(*args, **kwargs):
        if cls not in _instance:
            instance = cls(*args, **kwargs)
            _instance[cls] = instance
        return _instance[cls]
    return wrapper

@singleton
class A:
    a = 1
    def __init__(self, x):
        self.x = x

a1 = A(1)
a2 = A(2)
print(a1)
print(a2)
```

结果：
```
<__main__.A object at 0x0000019471949CC0>
<__main__.A object at 0x0000019471949CC0>
```

### 3.使用类

```python
class Singleton:
    def __init__(self, *args, **kwargs):
        pass

    @classmethod
    def instance(cls, *args, **kwargs):
        if not hasattr(Singleton, "_instance"):
            Singleton._instance = Singleton(*args, **kwargs)
        return Singleton._instance

s1 = Singleton.instance()
s2 = Singleton.instance()
print(s1)
print(s2)
```

结果：
```
<__main__.Singleton object at 0x0000016E499AB668>
<__main__.Singleton object at 0x0000016E499AB668>
```

上述方式实现的单例模式，在单线程时是可行的，但是在多线程时就会出现问题，假设上述的__init__()方法中有一些耗时的io操作，这里我们用sleep模拟一下：

```python
class Singleton:
    def __init__(self, *args, **kwargs):
        import time
        time.sleep(10)
```

再用多线程执行创建实例化对象的操作：
```python
import threading
def task(arg):
    job = Singleton.instance()
    print(job)
for i in range(10):
    t = threading.Thread(target=task, args=[i, ])
    t.start()
```

执行结果如下：
```
<__main__.Singleton object at 0x0000020FC8905B38>
<__main__.Singleton object at 0x0000020FC8905DA0>
<__main__.Singleton object at 0x0000020FC8905BE0>
<__main__.Singleton object at 0x0000020FC8905CC0>
<__main__.Singleton object at 0x0000020FC88F5CF8>
<__main__.Singleton object at 0x0000020FC88F5E80>
<__main__.Singleton object at 0x0000020FC88F5E10>
<__main__.Singleton object at 0x0000020FC88F5F98>
<__main__.Singleton object at 0x0000020FC88F5B70>
<__main__.Singleton object at 0x0000020FA791EA58>
```

使用装饰器也一样：
```python
import threading

def singleton(cls):
    _instance = {}
    def wrapper(*args, **kwargs):
        if cls not in _instance:
            instance = cls(*args, **kwargs)
            _instance[cls] = instance
        return _instance[cls]
    return wrapper

@singleton
class A:
    a = 1
    def __init__(self, x):
        self.x = x
        import time
        time.sleep(5)

def task():
    job = A(1)
    print(job)

for i in range(10):
    t = threading.Thread(target=task)
    t.start()
```

结果：
```
<__main__.A object at 0x000001ACECB45B70>
<__main__.A object at 0x000001ACECB459E8>
<__main__.A object at 0x000001ACECB45DA0>
<__main__.A object at 0x000001ACECB45C88>
<__main__.A object at 0x000001ACCBB8E2E8>
<__main__.A object at 0x000001ACECB35F98>
<__main__.A object at 0x000001ACECB35B70>
<__main__.A object at 0x000001ACECB35E80>
<__main__.A object at 0x000001ACECB35E10>
<__main__.A object at 0x000001ACECB35CF8>
```

从执行结果中我们可以发现：在多线程情况之下，通过上述方式创建的实例化对象，内存地址并不一样，这也就意味着我们并没有实现单例模式。那么，为什么这种单例模式的写法无法在多线程情况下奏效呢？

在搞清楚这个问题之前，我们需要先了解一点，即：任何数据实例化过程或者数据声明过程都是向系统申请内存空间，这些过程都是io，也即耗时操作。

再回到代码本身，在多线程情况下，当线程1运行完if not，但还没有执行实例化过程，此时如果CPU切换到线程2，线程2也执行到了if not，并完成了实例化过程，然后CPU再切换回了线程1，并继续执行了线程1的实例化过程，那么就是两个线程分别实例化了一次，也就不是单例了。

搞清楚上述方法无法满足多线程的原因之后，可以使用线程锁去解决上述问题：

```python
# 使用装饰器
import threading

def singleton(cls):
    _instance = {}
    _instance_lock = threading.Lock()
    def wrapper(*args, **kwargs):
        if cls not in _instance:
            with _instance_lock:
                if cls not in _instance:
                    instance = cls(*args, **kwargs)
                    _instance[cls] = instance
                return _instance[cls]
    return wrapper

@singleton
class A:
    a = 1
    def __init__(self, x):
        self.x = x
        import time
        time.sleep(5)

def task():
    job = A(1)
    print(job)

for i in range(10):
    t = threading.Thread(target=task)
    t.start()
```

```python
# 使用类
import threading


class Singleton:
    _instance_lock = threading.Lock()
    def __init__(self, *args, **kwargs):
        import time
        time.sleep(10)

    @classmethod
    def instance(cls, *args, **kwargs):
        if not hasattr(Singleton, "_instance"):
            with Singleton._instance_lock:
                if not hasattr(Singleton, "_instance"):
                    Singleton._instance = Singleton(*args, **kwargs)
        return Singleton._instance



def task(arg):
    job = Singleton.instance()
    print(job)

for i in range(10):
    t = threading.Thread(target=task, args=[i, ])
    t.start()
```

执行结果：

```
<__main__.Singleton object at 0x0000028918965A90>
<__main__.Singleton object at 0x0000028918965A90>
<__main__.Singleton object at 0x0000028918965A90>
<__main__.Singleton object at 0x0000028918965A90>
<__main__.Singleton object at 0x0000028918965A90>
<__main__.Singleton object at 0x0000028918965A90>
<__main__.Singleton object at 0x0000028918965A90>
<__main__.Singleton object at 0x0000028918965A90>
<__main__.Singleton object at 0x0000028918965A90>
<__main__.Singleton object at 0x0000028918965A90>
```

### 4. 基于__new__方法实现

实例化对象时，会先执行类的new方法（没有__new__方法时会默认执行object的__new__方法），然后再执行__init__方法初始化实例对象，那么我们可以通过定义__new__方法实现单例：

```python
import threading


class Singleton:
    _instance_lock = threading.Lock()

    def __init__(self, *args, **kwargs):
        pass

    def __new__(cls, *args, **kwargs):
        if not hasattr(Singleton, "_instance"):
            with Singleton._instance_lock:
                if not hasattr(Singleton, "_instance"):
                    Singleton._instance = object.__new__(cls)
        return Singleton._instance


def task(arg):
    job = Singleton()
    print(job)


for i in range(10):
    t = threading.Thread(target=task, args=[i, ])
    t.start()
```

执行结果：
```
<__main__.Singleton object at 0x00000206BC125B38>
<__main__.Singleton object at 0x00000206BC125B38>
<__main__.Singleton object at 0x00000206BC125B38>
<__main__.Singleton object at 0x00000206BC125B38>
<__main__.Singleton object at 0x00000206BC125B38>
<__main__.Singleton object at 0x00000206BC125B38>
<__main__.Singleton object at 0x00000206BC125B38>
<__main__.Singleton object at 0x00000206BC125B38>
<__main__.Singleton object at 0x00000206BC125B38>
<__main__.Singleton object at 0x00000206BC125B38>
```

### 5.基于metaclass（元类）实现

关于`元类（type）`：这里元类的概念，我们可以理解为创建类的类。当我们创建一个类时，会执行type的`__init__()`方法，使用类名+()创建类的实例化对像时，会调用元类type的`__call__()`方法。

关于`__call__()`：该方法的功能类似于在类中重载()运算符，使得类的实例化对象可以像调用普通函数那样，以对象名+()的形式调用，普通类的对象基于类，类基于元类，都是如此。

__call__的使用：
```python
class Foo:
    def __call__(self, *args, **kwargs):
        print('执行Foo类中的__call__')

obj = Foo()
print('创建实例化对象obj')
obj()
```

执行结果：
```
创建实例化对象obj
执行Foo类中的__call__
```

元类的使用：
```python
class FooType(type):
    def __call__(cls, *args, **kwargs):
        print('执行元类中的__call__')

    def __init__(cls, *args, **kwargs):
        print('执行元类中的__init__')
        super().__init__(*args, **kwargs)


class Foo(metaclass=FooType):
    def __call__(self, *args, **kwargs):
        print('执行Foo类中的__call__')


print('创建一个类')
obj = Foo()
```

执行结果：
```
执行元类中的__init__
创建一个类
执行元类中的__call__
```

因此可以利用元类的__call__方法来实现单例模式：
```python
import threading


class Singleton(type):
    _instance_lock = threading.Lock()
    def __call__(cls, *args, **kwargs):
        if not hasattr(cls, '_instance'):
            with Singleton._instance_lock:
                if not hasattr(cls, '_instance'):
                    cls._instance = super(Singleton, cls).__call__(*args, **kwargs)
        return cls._instance


class Foo(metaclass=Singleton):
    def __init__(self):
        import time
        time.sleep(5)


def task():
    obj = Foo()
    print(obj)

for i in range(10):
    t = threading.Thread(target=task)
    t.start()
```

执行结果：
```
<__main__.Foo object at 0x000001C16B475C18>
<__main__.Foo object at 0x000001C16B475C18>
<__main__.Foo object at 0x000001C16B475C18>
<__main__.Foo object at 0x000001C16B475C18>
<__main__.Foo object at 0x000001C16B475C18>
<__main__.Foo object at 0x000001C16B475C18>
<__main__.Foo object at 0x000001C16B475C18>
<__main__.Foo object at 0x000001C16B475C18>
<__main__.Foo object at 0x000001C16B475C18>
<__main__.Foo object at 0x000001C16B475C18>
```