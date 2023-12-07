## 封装、继承和多态

### 1. 封装

#### 封装的目的：
>封装数据：保护隐私
>封装方法：隔离复杂度（只保留部分接口对外使用）

#### 封装的方式：

**1.公有属性和方法**
公有属性和方法可以被类的外部访问和使用，不用添加任何特殊符号：

```python
class Student:
    public_attribute = 'this is a public attribute'
    def __init__(self):
        self.public_stu = 'tom'
    def public_method(self):
        print('this is a public method')

if __name__ == '__main__':
    s = Student()
    print(s.public_attribute)
    print(s.public_stu)
    s.public_method()
```

执行结果：
```
this is a public attribute
tom
this is a public method
```

**2.私有属性和方法**
在Python中，可以使用`_`和`__`来定义属性的访问级别。

以`_`开头的属性被视为受保护的属性，即它们不应该在类外部被直接访问。但是，它们可以在类的子类和内部方法中被访问。

以`__`开头的属性被视为私有属性，即它们不应该在类的外部被访问。但是，它们也可以在类的内部方法中被访问。

```python
class Student:
    def __init__(self, name, gender, age):
        self.name = name
        self._gender = gender
        self.__age = age


if __name__ == '__main__':
    s = Student('tom', 'female', '13')
    print('类外部调用受保护属性：', s._gender)
    s._gender = 'male'
    print('类外部调用并修改受保护属性：', s._gender)
    print('类外部调用私有属性：', s.__age)
    s.__age = 14
    print('类外部调用并修改私有属性：', s.__age)
```

执行结果：
```
类外部调用受保护属性： female
类外部调用并修改受保护属性： male
Traceback (most recent call last):
  File "C:/Users/root/Desktop/HelloWorld/test/script/test.py", line 13, in <module>
    print('类外部调用私有属性：', s.__age)
AttributeError: 'Student' object has no attribute '__age'
```

**3.属性装饰器和方法装饰器**

`属性装饰器`是一种特殊的函数，它可以被用来修改属性的访问方式。在Python中，我们可以使用`@property`和`@属性名.setter`装饰器来定义属性的访问器和修改器。


>@property装饰器用于定义属性的访问器，它会将属性定义为一个只读属性。当外部代码试图修改这个属性时，会触发一个AttributeError异常。
>@属性名.setter装饰器用于定义属性的修改器，它可以让外部代码修改这个属性的值。当外部代码试图读取这个属性时，会触发一个AttributeError异常。

`方法装饰器`是用来装饰类的方法的，它的作用是在不改变原方法定义的情况下，为方法添加一些额外的功能，比如权限检查、缓存等。常见的方法装饰器有静态装饰器 `@staticmethod`；类方法装饰器 `@classmethod`。

```python
class MyClass:
    def __init__(self, value):
        self.__value = value

    @property
    def value(self):
        return self.__value

    @value.setter
    def value(self, new_value):
        self.__value = new_value

    @staticmethod
    def my_static_method():
        print('this is a static method')

    @classmethod
    def my_class_method(cls):
        print(f'this is a class method of {cls.__name__}')

if __name__ == '__main__':
    my_object = MyClass(1024)
    print(my_object.value)
    my_object.value = 111
    print(my_object.value)
    my_object.my_static_method()
    my_object.my_class_method()
```

执行结果：
```
1024
111
this is a static method
this is a class method of MyClass
```

如果想要私有属性或者私有方法在类外部被调用，可以使用`_类名+私有属性/方法`的方式：

```python
class MyClass:
    def __init__(self, value):
        self.__value = value

    def __show(self):
        print(self.__value)

if __name__ == '__main__':
    my_object = MyClass(1024)
    print(my_object._MyClass__value)
    my_object._MyClass__show()
```

执行结果:
```
1024
1024
```

#### 父类与子类的封装：

子类的封装属性或方法不会覆盖父类的属性或方法：

```python
class A:
    __name = 'A'
    def get_x(self):
        print('父类A:', self.__name)

class A1(A):
    __name = '父类A的子类A1'
    def __say(self):
        print('子类A1的方法')
    

if __name__ == '__main__':
    obj = A1()
    obj.get_x()
```

执行结果：

```
父类A: A
```

### 2.继承

#### 继承的实现

python中的继承机制经常用于创建和现有类功能类似的新类，又或是新类只需要在现有类基础上添加一些成员（属性和方法），但又不想直接将现有类代码复制给新类。也就是说，当我们需要进行类的重复使用时，就可以使用这种继承机制来实现。

Python 中，实现继承的类称为子类，被继承的类称为父类；

子类继承父类时，只需在定义子类时，将父类（可以是多个）放在子类之后的圆括号里即可：
```python
class 类名(父类1, 父类2, ...)：
    #类定义部分
```

python中如果类没有指定继承某个类，那么就默认继承的是object类:
```python
class Person(object):
    def __init__(self,name,age):
        self.name=name
        self.age=age
    def info(self):
        print('姓名:',self.name,'年龄:',self.age)
# Student类
class Student(Person):
    def __init__(self,name,age,stu_no):
        super().__init__(name,age)
        self.stu_no=stu_no
# Teacher 类
class Teacher(Person):
    def  __init__(self,name,age,teach_year):
        super().__init__(name,age)
        self.teach_year=teach_year
# 创建学生对象和教师对象
stu1=Student('小明',17,1001)
teacher1=Teacher('张丽',30,6)   
stu1.info()
teacher1.info()
```
执行结果：
```
姓名: 小明 年龄: 17
姓名: 张丽 年龄: 30
```

如果子类对继承自父类的某个属性或方法不满意，可以在子类中对其（方法体）进行重新编写；子类重写后的方法中可以通过`super().xxx() `调用父类中被重写的方法：

```python
class Person(object):
    def __init__(self, name, age):
        self.name = name
        self.age = age
        
    def info(self):
        print('名字:', self.name, '年龄:', self.age)


class Student(Person):
    def __init__(self, name, age, stu_no):
        super().__init__(name, age)
        self.stu_no = stu_no
        
    #方法重写
    def info(self):
        super().info()  #调用父类的info()
        print('学号是：', self.stu_no)
        

class Teacher(Person):
    def __init__(self, name, age, teach_year):
        super().__init__(name, age)
        self.teach_year = teach_year
        
     #方法重写
    def info(self):
        super().info() #调用父类的info()
        print('教龄是：', self.teach_year)

stu1=Student('小华', 17, 1001)
teacher1 = Teacher('赵丽', 40, 10)
stu1.info()
print('\n---------------教职工-------------\n')
teacher1.info()
```

执行结果：
```
名字: 小华 年龄: 17
学号是： 1001

---------------教职工-------------

名字: 赵丽 年龄: 40
教龄是： 10
```

如果一个类是空类，但是继承了含有方法和属性的类，那么这个类也同样含有父类的方法和属性：
```python
class People:
    def __init__(self,name):
        self.name = name

    def say(self):
        print("People类", self.name)

class Animal:
    def __init__(self):
        self.name = Animal

    def say(self):
        print("Animal类", self.name)

#People类是Person父类中最近的父类，因此People中的name属性和say()方法会遮蔽 Animal 类中的
class Person(People, Animal):
    pass

zhangsan = Person('张三')
zhangsan.say()
```
执行结果：
```
People类 张三
```

可以看到，当Person同时继承People类和Animal类时，People类在前，因此如果People和Animal拥有同名的类方法，实际调用的是People类中的；由此也可以看出，python支持多继承。

#### object类

object类是所有类的父类 ，因此所有类都有object类的属性和方法：

```python
class Student:
    pass

stu = Student()
print(dir(stu))
print(stu)
```

执行结果：
```
['__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__']
<__main__.Student object at 0x00000253C2419CC0>
```

object有一个`__str__()`方法，用于返回一个对于对象的描述 ， 对应于内置函数str()经常用于print()方法，帮我们查看对象的信息 所以我们经常会对`__str__()`进行重写:

```python
class Student:
    def __init__(self, name, age):
        self.name = name
        self.age = age

    def __str__(self):
        return '名字是{}，年龄是{}'.format(self.name, self.age)

stu = Student('tom', 13)
print(stu)
print(type(stu))
```

执行结果：
```
名字是tom，年龄是13
<class '__main__.Student'>
```

#### Python3的继承机制

- 子类在调用某个方法或变量的时候，首先在自己内部查找，如果没有找到，则开始根据继承机制在父类里查找。
- 根据父类定义中的顺序，以深度优先的方式逐一查找父类！

### 3.多态

python是一种动态语言，其最明显的特征就是在使用变量时，无需为其指定具体的数据类型。这会导致一种情况，即同一变量可能会被先后赋值不同的类对象。类的多态特性，要满足以下 2 个前提条件：

- 继承：多态一定是发生在子类和父类之间；
- 重写：子类重写了父类的方法。

```python
class Animal:

    def kind(self):
        print("i am animal")


class Dog(Animal):

    def kind(self):
        print("i am a dog")


class Cat(Animal):

    def kind(self):
        print("i am a cat")


class Pig(Animal):

    def kind(self):
        print("i am a pig")

# 这个函数接收一个animal参数，并调用它的kind方法
def show_kind(animal):
    animal.kind()


d = Dog()
c = Cat()
p = Pig()

show_kind(d)
show_kind(c)
show_kind(p)
```

执行结果：
```
i am a dog
i am a cat
i am a pig
```

狗、猫、猪都继承了动物类，并各自重写了 kind方法。show_kind() 函数接收一个 animal 参数，并调用它的 kind 方法。可以看出，无论我们给animal传递的是狗、猫还是猪，都能正确的调用相应的方法，打印对应的信息。这就是多态。

实际上，由于 Python 的动态语言特性，传递给函数 show_kind() 的参数 animal 可以是 任何的类型，只要它有一个 kind() 的方法即可。动态语言调用实例方法时不检查类型，只要方法存在，参数正确，就可以调用。这就是动态语言的“鸭子类型”，它并不要求严格的继承体系，一个对象只要“看起来像鸭子，走起路来像鸭子”，那它就可以被看做是鸭子。





