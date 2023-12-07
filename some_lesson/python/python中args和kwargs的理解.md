### *args

`∗`的作用：函数接受实参时，按顺序分配给函数形参，如果遇到带`*`的形参，那么就把还未分配出去的实参以元组形式打包（pack）,分配给那个带`*`的形参。

```python
def foo(a, b, *number):
    print('a:', a)
    print('b:', b)
    print('number:', number)
    for i in number:
        print(i)
    print(type(number))


foo(1, 2, 3, 4, 5)
```

执行结果：
```
a: 1
b: 2
number: (3, 4, 5)
3
4
5
<class 'tuple'>
```

`*`如果没有用在函数定义中，而是用在了函数调用中。`*`的作用是把打包了的实参（元组或列表），拆分（unpack）成单个的，依次赋值给函数的形参。

```python
def bar(a, b, c):
    print(a, b, c)

bar(*[1, 2, 3])
```

执行结果：
```
1 2 3 
```

### **kwargs

`∗∗`的作用同样也有两个,打包参数（pack）和拆分参数（unpack）。

但是区别还是有的，简单来说就是：

打包（pack）：`*args`是把多个位置参数打包成元组，`**kwargs`是把多个关键字参数打包成字典。
拆分（unpack）：`*args`是把打包了的参数拆成单个的，依次赋值给函数的形参，`**kwargs`是把字典的键值拆成单个的，依次赋值给函数的形参。

用`**`方式拆解字典给形参赋值时，需要字典的键名和函数形参一致，否则会报错。

```python
def bar(**number):
    print(number)

bar(a=1, b=2, c=3)
```

执行结果：
```
{'a': 1, 'b': 2, 'c': 3}
```

```python
def bar(a, b=2, c=3):
    print(a, b, c)

bar(**{'a': 5, 'b': 6, 'c':7})
```

执行结果：
```
5 6 7
```




