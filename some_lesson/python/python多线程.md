## 多线程

线程是操作系统能够进行运算调度的最小单位。它被包含在进程之中，是进程中的实际运作单位。一条线程指的是进程中一个单一顺序的控制流，一个进程中可以并发多个线程，每条线程并行执行不同的任务，多线程就是在一个进程中的多个线程，如果使用多线程默认开启一个主线程，按照程序需求自动开启多个线程(也可以自己定义线程数)。

### 多线程知识点

1. Python 在设计之初就考虑到要在解释器的主循环中，同时只有一个线程在执行，即在任意时刻，只有一个线程在解释器中运行。对Python 虚拟机的访问由全局解释器锁（GIL）来控制，正是这个锁能保证同一时刻只有一个线程在运行。
2. 多线程共享主进程的资源，所以可能还会改变其中的变量，这个时候就要加上线程锁，每次执行完一个线程在执行下一个线程。
3. 因为每次只能有一个线程运行，多线程怎么实现的呢？Python解释器中一个线程做完了任务然后做IO(文件读写)操作的时候，这个线程就退出，然后下一个线程开始运行，循环之。
4. 当你读完上面三点你就知道多线程如何运行起来，并且知道多线程常用在那些需要等待然后执行的应用程序上(比如爬虫读取到数据，然后保存的时候下一个线程开始启动)也就是说多线程适用于IO密集型的任务量（文件存储，网络通信）。
5. 注意一点，定义多线程，然后传递参数的时候，如果是有一个参数就是用args=（i，）一定要加上逗号，如果有两个或者以上的参数就不用这样。

### 代码实例

**1.多线程核心用法**

```python
import threading

def loop():
    print('thread %s is running' % threading.currentThread().name)
    n = 0
    while n < 10:
        n = n + 1
        print('%s >>> %s' % (threading.currentThread().name, n))
    print('thread %s end' % threading.currentThread().name)

print('thread %s is running' % threading.currentThread().name)

t = threading.Thread(target=loop, name='子线程')
t.start()
t.join()
print('thread %s end' % threading.currentThread().name)
```

执行结果：
```
thread MainThread is running
thread 子线程 is running
子线程 >>> 1
子线程 >>> 2
子线程 >>> 3
子线程 >>> 4
子线程 >>> 5
子线程 >>> 6
子线程 >>> 7
子线程 >>> 8
子线程 >>> 9
子线程 >>> 10
thread 子线程 end
thread MainThread end
```

**2.线程锁**

```python
import threading

def loop():
    l.acquire()
    print('thread %s is running' % threading.currentThread().name)
    n = 0
    while n < 10:
        n = n + 1
        print('%s >>> %s' % (threading.currentThread().name, n))
    print('thread %s end' % threading.currentThread().name)
    l.release()

print('thread %s is running' % threading.currentThread().name)

t = threading.Thread(target=loop, name='子线程')
l = threading.Lock()
t.start()
t.join()
print('thread %s end' % threading.currentThread().name)
```

使用线程锁后，程序按照一个一个有序执行。其中lock还有Rlock的方法，RLock允许在同一线程中被多次acquire。而Lock却不允许这种情况。否则会出现死循环，程序不知道解哪一把锁。注意：如果使用RLock，那么acquire和release必须成对出现，即调用了n次acquire，必须调用n次的release才能真正释放所占用的锁。

**3. join()方法的使用**

在多线程中，每个线程自顾执行自己的任务，当最后一个线程运行完毕后再退出，所以这个时候如果你要打印信息的话，会看到打印出来的信息错乱无章，有的时候希望主线程能够等子线程执行完毕后在继续执行，就是用join()方法。

```python
import threading
import time

t0 = time.time()
def cs1():
    s_time = time.time()
    for i in range(3):
        time.sleep(0.3)
        print(i, time.time()-s_time)
        print(threading.currentThread().name)

def cs2():
    for i in range(6, 9):
        print(i)
        print(threading.currentThread().name)

threads = []
t1 = threading.Thread(target=cs1)
t2 = threading.Thread(target=cs2)
threads.append(t1)
threads.append(t2)

for x in threads:
    x.start()
    x.join()
print('use time.{}'.format(time.time()-t0))
```

执行结果：
```
0 0.3001973628997803
Thread-1
1 0.6003985404968262
Thread-1
2 0.9005956649780273
Thread-1
6
Thread-2
7
Thread-2
8
Thread-2
use time.0.9015929698944092
```

如果不`x.join()`，此时打印信息就会错乱无章：
```
6
Thread-2
7
Thread-2
8
Thread-2
use time.0.000997304916381836
0 0.3011956214904785
Thread-1
1 0.6023943424224854
Thread-1
2 0.902768611907959
Thread-1
```

关于setDaemon()的概念就是：主线程A中，创建了子线程B，并且在主线程A中调用了B.setDaemon(),这个的意思是，把主线程A设置为守护线程，这时候，要是主线程A执行结束了，就不管子线程B是否完成,一并和主线程A退出.这就是setDaemon方法的含义，这基本和join是相反的。此外，还有个要特别注意的：必须在start() 方法调用之前设置，如果不设置为守护线程，程序会被无限挂起。

```python
import threading
import time

t0 = time.time()
def cs1():
    s_time = time.time()
    for i in range(3):
        time.sleep(0.3)
        print(i, time.time()-s_time)
        print(threading.currentThread().name)

def cs2():
    for i in range(6, 9):
        print(i)
        print(threading.currentThread().name)

threads = []
t1 = threading.Thread(target=cs1)
t2 = threading.Thread(target=cs2)
threads.append(t1)
threads.append(t2)

for x in threads:
    x.setDaemon(True)
    x.start()
print('use time.{}'.format(time.time()-t0))
```

执行结果：
```
6
Thread-2
7
Thread-2
8
Thread-2
use time.0.0009682178497314453
```

**4.线程锁之信号Semaphore**

类名：BoundedSemaphore。这种锁允许一定数量的线程同时更改数据，它不是互斥锁。比如地铁安检，排队人很多，工作人员只允许一定数量的人进入安检区，其它的人继续排队。

```python
import threading
import time

def run(n):
    sema.acquire()
    print('run thread: %s' % n)
    time.sleep(1)
    sema.release()


sema = threading.BoundedSemaphore(3)
for i in range(9):
    t = threading.Thread(target=run, args=(i, ))
    t.start()
```

运行后，可以看到3个一批的线程被放行。


**5.线程锁之事件Event**

事件线程锁的运行机制：
全局定义了一个Flag，如果Flag的值为False，那么当程序执行wait()方法时就会阻塞，如果Flag值为True，线程不再阻塞。这种锁，类似交通红绿灯（默认是红灯），它属于在红灯的时候一次性阻挡所有线程，在绿灯的时候，一次性放行所有排队中的线程。

事件主要提供了四个方法set()、wait()、clear()和is_set():
>调用clear()方法会将事件的Flag设置为False。
>调用set()方法会将Flag设置为True。
>调用wait()方法将等待“红绿灯”信号。
>is_set():判断当前是否"绿灯放行"状态

```python
import threading
import time
event = threading.Event()
# 定义一个事件的对象
def lighter():
    green_time = 5
    # 绿灯时间
    red_time = 5
    # 红灯时间
    event.set()
    # 初始设为绿灯
    while True:
        print("\33[32;0m 绿灯亮...\033[0m")
        time.sleep(green_time)
        event.clear()
        print("\33[31;0m 红灯亮...\033[0m")
        time.sleep(red_time)
        event.set()

def run(name):
    while True:
        if event.is_set():
        # 判断当前是否"放行"状态
            print("一辆[%s] 呼啸开过..." % name)
            time.sleep(1)
        else:
            print("一辆[%s]开来，看到红灯，无奈的停下了..." % name)
            event.wait()
            print("[%s] 看到绿灯亮了，瞬间飞起....." % name)

if __name__ == '__main__':
    light = threading.Thread(target=lighter,)
    light.start()
    for name in ['奔驰', '宝马', '奥迪']:
        car = threading.Thread(target=run, args=(name,))
        car.start()
```

运行结果：
```
绿灯亮...
一辆[奔驰] 呼啸开过...
一辆[宝马] 呼啸开过...
一辆[奥迪] 呼啸开过...
一辆[宝马] 呼啸开过...
一辆[奥迪] 呼啸开过...
一辆[奔驰] 呼啸开过...
一辆[奥迪] 呼啸开过...
一辆[宝马] 呼啸开过...
一辆[奔驰] 呼啸开过...
一辆[奥迪] 呼啸开过...
一辆[宝马] 呼啸开过...
一辆[奔驰] 呼啸开过...
一辆[奥迪] 呼啸开过...
一辆[宝马] 呼啸开过...
一辆[奔驰] 呼啸开过...
 红灯亮...
一辆[奥迪]开来，看到红灯，无奈的停下了...
一辆[宝马]开来，看到红灯，无奈的停下了...
一辆[奔驰]开来，看到红灯，无奈的停下了...
 绿灯亮...
[奔驰] 看到绿灯亮了，瞬间飞起.....
一辆[奔驰] 呼啸开过...
[宝马] 看到绿灯亮了，瞬间飞起.....
[奥迪] 看到绿灯亮了，瞬间飞起.....
```

**6.线程锁之条件Condition**

Condition称作条件锁，依然是通过acquire()/release()加锁解锁。

>wait([timeout])方法将使线程进入Condition的等待池等待通知，并释放锁。使用前线程必须已获得锁定，否则将抛出异常。
>notify()方法将从等待池挑选一个线程并通知，收到通知的线程将自动调用acquire()尝试获得锁定（进入锁定池），其他线程仍然在等待池中。调用这个方法不会释放锁定。使用前线程必须已获得锁定，否则将抛出异常。
>notifyAll()方法将通知等待池中所有的线程，这些线程都将进入锁定池尝试获得锁定。调用这个方法不会释放锁定。使用前线程必须已获得锁定，否则将抛出异常。

```python
import threading
import time

num = 0
con = threading.Condition()

class Foo(threading.Thread):
    def __init__(self, name, action):
        super(Foo, self).__init__()
        self.name = name
        self.action = action

    def run(self):
        global num
        con.acquire()
        print('开始执行%s' % self.name)
        while True:
            if self.action == 'add':
                num += 1
            elif self.action == 'reduce':
                num -= 1
            else:
                exit(1)
            print('当前num为：%s' % num)
            if num == 5 or num == 0:
                print('暂停执行%s' % self.name)
                con.notify()
                con.wait()
                print('开始执行%s' % self.name)
        con.release()

if __name__ == '__main__':
    a = Foo('线程A', 'add')
    b = Foo('线程B', 'reduce')
    a.start()
    b.start()
```

执行结果：
```
开始执行线程A
当前num为：1
当前num为：2
当前num为：3
当前num为：4
当前num为：5
暂停执行线程A
开始执行线程B
当前num为：4
当前num为：3
当前num为：2
当前num为：1
当前num为：0
暂停执行线程B
开始执行线程A
当前num为：1
当前num为：2
当前num为：3
当前num为：4
当前num为：5
```

```python
def aa():
    print('thread_aa get')
    cond.acquire()
    time.sleep(5)
    print('thread_aa finished first job')
    cond.wait()  # 释放对锁的控制，好让别的进程acquire到
    # 同时阻塞在此，等待得到锁的那个进程的先cond.notify后(cond.wait或cond.release)
    print('thread_aa get again')
    time.sleep(5)
    print('thread_aa finished all work')
    cond.notify()  # 告诉刚才释放锁的那个进程等通知吧,等我要么wait要么release
    cond.release()  # 释放锁


def bb():
    cond.acquire()
    print('thread_bb get')
    cond.notify()  # 告诉刚才释放锁的那个进程等通知吧,等我要么wait要么release
    print('依然是我bb在运行')
    time.sleep(5)
    print('thread_bb finished first job')
    cond.wait()  # 释放对锁的控制，而后被阻塞的aa就可以执行了
    # 同时阻塞在此，等待得到锁的aa的先cond.notify后(cond.wait或cond.release)
    print('thread_bb get again')
    print('thread_bb finished all works')
    cond.release()


a = threading.Thread(target=aa)
a.start()
time.sleep(2)
b = threading.Thread(target=bb)
b.start()
```

运行结果：
```
thread_aa get
thread_aa finished first job
thread_bb get
依然是我bb在运行
thread_bb finished first job
thread_aa get again
thread_aa finished all work
thread_bb get again
thread_bb finished all works
```

**7.定时器**
定时器Timer类是threading模块中的一个小工具，用于指定n秒后执行某操作。

```python
from threading import Timer

def hello():
    print('Hello World')

t = Timer(2, hello)
t.start()
```

**8.通过with语句使用线程锁**

类似于上下文管理器，所有的线程锁都有一个加锁和释放锁的动作，非常类似文件的打开和关闭。在加锁后，如果线程执行过程中出现异常或者错误，没有正常的释放锁，那么其他的线程会造到致命性的影响。通过with上下文管理器，可以确保锁被正常释放。其格式如下：

```python
with some_lock:
    # 执行任务
```

这相当于：
```python
some_lock.acquire()
try:
    # 执行任务..
finally:
    some_lock.release()
```

**9.threading 的常用属性**

>current_thread()    返回当前线程
active_count()  返回当前活跃的线程数，1个主线程+n个子线程
get_ident() 返回当前线程
enumerater()    返回当前活动 Thread 对象列表
main_thread()   返回主 Thread 对象
settrace(func)  为所有线程设置一个 trace 函数
setprofile(func)    为所有线程设置一个 profile 函数

**10.线程池**

```python
from concurrent.futures import ThreadPoolExecutor

executor = ThreadPoolExecutor(3)
# 实例化线程池对象，开启3个线程
def fun(a,b):
    print(a,b)
    return a**b
# 定义一个函数
executor.submit(fun,2,5) # y运行结果：2,5
# 这是调用与开启线程
res=executor.submit(fun,5,2)
print(res.result()) # 运行结果: 25
# 如果要有很多参数传入进行运算
res2 = executor.map(fun,[1,2,3,4],[2,3,5,6])
print([x for x in res2])
```

执行结果：
```
2 5
5 2
25
1 2
2 3
3 5
4 6
[1, 8, 243, 4096]
```

```python
from concurrent.futures import ThreadPoolExecutor
import time

def func(n):
    print(n)
    time.sleep(1)

with ThreadPoolExecutor(max_workers=4) as executor:
    executor.map(func, range(1,8))
```

执行结果：
```
1
2
3
4
5
6
7
```

**submit与map的区别**
- submit返回的是一个futures的对象，使用.result()才能获取他的运行结果
- submit接受的对象是函数加一个固定的参数
- map返回的是所有线程执行完毕后返回的结果
- map接受的对象是函数加一个传入函数的集合列表
- 他们都能提前获取先执行完的结果
- map比submit简单好用
- map返回的结果是按照列表传入参数的顺序返回结果，submit返回结果是哪个线程先执行完就返回哪个线程的结果

### 生产者和消费者模式

**案例一**

```python
import queue
import time
import threading
q = queue.Queue(10)

def get(i):
    # 这个函数用来生产任务，接受参数i，也可以不传入参数
    while 1:
        time.sleep(2)
        # 这里可以做一些动作，比如过去网站的网址之类的
        q.put(i)
        # 然后把得到的数据放在消息队列中
def fun(o):
    # 这个函数用来处理任务，必须要接受参数
    q.get(o)
    # 得到获取接受来的参数
    print(o*10)
    # 然后对获取的参数作处理，我这里仅仅打印数据乘以10


for i in range(10):
    # 生产任务启动，有10个任务量要产生
    t1 = threading.Thread(target=get, args=(i,))
    t1.start()
time.sleep(2)
for o in range(10):
    # 处理任务启动
    t = threading.Thread(target=fun, args=(o,))
    t.start()
```

执行结果：
```
0
10
20
30
40
50
60
70
80
90
```

**案例二**

```python
import time
import queue
import threading

q = queue.Queue(10)     # 生成一个队列，用来保存“包子”，最大数量为10

def productor(i):
    # 厨师不停地每2秒做一个包子
    while True:
        q.put("厨师 %s 做的包子！" % i)
        time.sleep(1)

def consumer(j):
    # 顾客不停地每秒吃一个包子
    while True:
        print("顾客 %s 吃了一个 %s"%(j,q.get()))
        time.sleep(10)

# 实例化了3个生产者（厨师）
for i in range(3):
    t = threading.Thread(target=productor, args=(i,))
    t.start()
# 实例化了10个消费者（顾客）
for j in range(10):
    v = threading.Thread(target=consumer, args=(j,))
    v.start()
```

执行结果：
```
顾客 0 吃了一个 厨师 0 做的包子！
顾客 1 吃了一个 厨师 1 做的包子！
顾客 2 吃了一个 厨师 2 做的包子！
顾客 4 吃了一个 厨师 1 做的包子！
顾客 3 吃了一个 厨师 2 做的包子！
顾客 5 吃了一个 厨师 0 做的包子！
顾客 6 吃了一个 厨师 2 做的包子！
顾客 8 吃了一个 厨师 0 做的包子！
顾客 7 吃了一个 厨师 1 做的包子！
顾客 9 吃了一个 厨师 2 做的包子！
```

Queue是python标准库中的线程安全的队列（FIFO）实现,提供了一个适用于多线程编程的先进先出的数据结构，即队列，用来在生产者和消费者线程之间的信息传递。

基本语法：
> queue.Queue(self, maxsize=0)

- obj.put(item,block,timeout)来增加线程队列，将item 放入队列，如果block设置为True（默认为True）时，队列满时，调用者将被阻塞，否则抛出异常，timeout 设置阻塞超时
- obj.get()来取出线程队列，
- obj.objsize():返回队列的大小
- obj.full():如果队列满了返回True,否则为False
- obj.empty():如果队列为空，返回True,否则为False