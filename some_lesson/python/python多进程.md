## 多进程

Python中的multiprocess提供了Process类，实现进程相关的功能。但是它基于fork机制，因此不被windows平台支持。想要在windows中运行，调用方法一定要加上：
>`if __name__ == '__main__':`

### 基础用法

多进程的使用方法和多线程使用方法基本一样，所以如果你会多线程用法多进程也就懂了，有一点要注意，定义多进程，然后传递参数的时候，如果是有一个参数就是用args=（i，）一定要加上逗号，如果有两个或者以上的参数就不用这样。

```python
import sys
import multiprocessing

def fun(i):
    print(sys.path)
    print(sys.version_info)
    print(sys.platform)

if __name__ == '__main__':
    m = multiprocessing.Process(target=fun,args=(1,))
    m.start()
```

执行结果：
```
['C:\\Users\\root\\Desktop\\HelloWorld\\test\\script', 'C:\\Users\\root\\Desktop\\HelloWorld', 'C:\\Users\\root\\Desktop\\HelloWorld\\Git', 'C:\\Users\\root\\Desktop\\HelloWorld\\test\\script\\trading_card', 'C:\\Program Files\\JetBrains\\PyCharm 2020.3.4\\plugins\\python\\helpers\\pycharm_display', 'C:\\Users\\root\\AppData\\Local\\Programs\\Python\\Python36\\python36.zip', 'C:\\Users\\root\\AppData\\Local\\Programs\\Python\\Python36\\DLLs', 'C:\\Users\\root\\AppData\\Local\\Programs\\Python\\Python36\\lib', 'C:\\Users\\root\\AppData\\Local\\Programs\\Python\\Python36', 'C:\\Users\\root\\AppData\\Roaming\\Python\\Python36\\site-packages', 'C:\\Users\\root\\AppData\\Local\\Programs\\Python\\Python36\\lib\\site-packages', 'C:\\Users\\root\\AppData\\Local\\Programs\\Python\\Python36\\lib\\site-packages\\hupulive-1.4-py3.6.egg', 'C:\\Users\\root\\AppData\\Local\\Programs\\Python\\Python36\\lib\\site-packages\\win32', 'C:\\Users\\root\\AppData\\Local\\Programs\\Python\\Python36\\lib\\site-packages\\win32\\lib', 'C:\\Users\\root\\AppData\\Local\\Programs\\Python\\Python36\\lib\\site-packages\\Pythonwin', 'C:\\Program Files\\JetBrains\\PyCharm 2020.3.4\\plugins\\python\\helpers\\pycharm_matplotlib_backend']
sys.version_info(major=3, minor=6, micro=2, releaselevel='final', serial=0)
win32
```

**构造方法:**

- Process([group [, target [, name [, args [, kwargs]]]]])
- group: 线程组，目前还没有实现，库引用中提示必须是None；
- target: 要执行的方法；
- name: 进程名；
- args/kwargs: 要传入方法的参数。

**实例方法:**

- is_alive()：返回进程是否在运行。
- join([timeout])：阻塞当前上下文环境的进程程，直到调用此方法的进程终止或到达指定的3. timeout（可选参数）。
- start()：进程准备就绪，等待CPU调度。
- run()：strat()调用run方法，如果实例进程时未制定传入target，这star执行t默认run()方法。
- terminate()：不管任务是否完成，立即停止工作进程。

**属性：**

- authkey
- daemon：和线程的setDeamon功能一样（将父进程设置为守护进程，当父进程结束时，子进程也结束）。
- exitcode(进程在运行时为None、如果为–N，表示被信号N结束）。
- name：进程名字。
- pid：进程号。

### 数据通信

ipc：就是进程间的通信模式，常用的一半是socke，rpc，pipe和消息队列等。

multiprocessing提供了threading包中没有的IPC(比如Pipe和Queue)，效率上更高。应优先考虑Pipe和Queue，避免使用Lock/Event/Semaphore/Condition等同步方式 (因为它们占据的不是用户进程的资源)。

**使用Array共享数据**

对于Array数组类，括号内的“i”表示它内部的元素全部是int类型，而不是指字符“i”，数组内的元素可以预先指定，也可以只指定数组的长度。Array类在实例化的时候必须指定数组的数据类型和数组的大小，类似temp = Array('i', 5)。对于数据类型有下面的对应关系：

>'c': ctypes.c_char, 'u': ctypes.c_wchar,
'b': ctypes.c_byte, 'B': ctypes.c_ubyte,
'h': ctypes.c_short, 'H': ctypes.c_ushort,
'i': ctypes.c_int, 'I': ctypes.c_uint,
'l': ctypes.c_long, 'L': ctypes.c_ulong,
'f': ctypes.c_float, 'd': ctypes.c_double

```python
from multiprocessing import Process
from multiprocessing import Array

def func(i, temp):
    temp[0] += 100
    print("进程%s " % i, ' 修改数组第一个元素后----->', temp[0])

if __name__ == '__main__':
    temp = Array('i', [1, 2, 3, 4])
    for i in range(10):
        p = Process(target=func, args=(i, temp))
        p.start()
```

执行结果：
```
进程1   修改数组第一个元素后-----> 101
进程0   修改数组第一个元素后-----> 201
进程2   修改数组第一个元素后-----> 301
进程3   修改数组第一个元素后-----> 401
进程4   修改数组第一个元素后-----> 501
进程6   修改数组第一个元素后-----> 601
进程7   修改数组第一个元素后-----> 701
进程5   修改数组第一个元素后-----> 801
进程8   修改数组第一个元素后-----> 901
进程9   修改数组第一个元素后-----> 1001
```

**使用Manager共享数据**

通过Manager类也可以实现进程间数据的共享，主要用于线程池之间通信，Manager()返回的manager对象提供一个服务进程，使得其他进程可以通过代理的方式操作Python对象。manager对象支持 list, dict, Namespace, Lock, RLock, Semaphore, BoundedSemaphore, Condition, Event, Barrier, Queue, Value ,Array等多种格式。

```python
from multiprocessing import Process
from multiprocessing import Manager

def func(i, dic):
    dic["num"] = 100 + i
    print('线程%s修改后内容为' %i, dic.items())

if __name__ == '__main__':
    dic = Manager().dict()
    for i in range(10):
        p = Process(target=func, args=(i, dic))
        p.start()
        p.join()
```

执行结果：
```
线程0修改后内容为 [('num', 100)]
线程1修改后内容为 [('num', 101)]
线程2修改后内容为 [('num', 102)]
线程3修改后内容为 [('num', 103)]
线程4修改后内容为 [('num', 104)]
线程5修改后内容为 [('num', 105)]
线程6修改后内容为 [('num', 106)]
线程7修改后内容为 [('num', 107)]
线程8修改后内容为 [('num', 108)]
线程9修改后内容为 [('num', 109)]
```

**使用queues的Queue类共享数据**

multiprocessing是一个包，它内部有一个queues模块，提供了一个Queue队列类，可以实现进程间的数据共享，如下例所示：

```python
import multiprocessing
from multiprocessing import Process
from multiprocessing import queues

def func(i, q):
    ret = q.get()
    print("进程%s从队列里获取了一个%s，然后又向队列里放入了一个%s" % (i, ret, i))
    q.put(i)

if __name__ == '__main__':
    lis = queues.Queue(20, ctx=multiprocessing)
    lis.put(0)
    for i in range(10):
        p = Process(target=func, args=(i, lis))
        p.start()
```

执行结果：
```
进程2从队列里获取了一个0，然后又向队列里放入了一个2
进程1从队列里获取了一个2，然后又向队列里放入了一个1
进程3从队列里获取了一个1，然后又向队列里放入了一个3
进程7从队列里获取了一个3，然后又向队列里放入了一个7
进程0从队列里获取了一个7，然后又向队列里放入了一个0
进程4从队列里获取了一个0，然后又向队列里放入了一个4
进程6从队列里获取了一个4，然后又向队列里放入了一个6
进程5从队列里获取了一个6，然后又向队列里放入了一个5
进程8从队列里获取了一个5，然后又向队列里放入了一个8
进程9从队列里获取了一个8，然后又向队列里放入了一个9
```

例如来跑多进程对一批IP列表进行运算，运算后的结果都存到Queue队列里面，这个就必须使用multiprocessing提供的Queue来实现

关于queue和Queue，在Python库中非常频繁的出现，很容易就搞混淆了。甚至是multiprocessing自己还有一个Queue类(大写的Q)和的Manager类中提供的Queue方法，一样能实现消息队列queues.Queue的功能，导入方式是from multiprocessing import Queue，前者Queue用于多个进程间通信，和queues.Queue()差不多，后者Manager().queue用于进程池之间通信。

**使用pipe实现进程间通信**

pipe只能适用于两个进程间通信,相当于queue的一个子集，只能服务两个进程，pipe的性能高于queue。

```python
from multiprocessing import Process, Pipe
import time

def produce(pipe):
    pipe.send('666')
    time.sleep(1)

def consume(pipe):
    print(pipe.recv())

if __name__ == '__main__':
    send_pi, recv_pi = Pipe()
    pro = Process(target=produce, args=(send_pi, ))
    con = Process(target=consume, args=(recv_pi, ))
    pro.start()
    con.start()
    pro.join()
    con.join()
```

执行结果：
```
666
```

### 进程锁

一般来说每个进程使用单独的空间，不必加进程锁的，但是如果你需要先实现进程数据共享，又害怕造成数据抢夺和脏数据的问题。就可以设置进程锁，与threading类似，在multiprocessing里也有同名的锁类RLock，Lock，Event，Condition和 Semaphore，连用法都是一样样的。

```python
from multiprocessing import Process, Array, RLock
import time

def func(i, lis, lc):
    lc.acquire()
    lis[0] = lis[0] - 1
    time.sleep(1)
    print('say hi', lis[0])
    lc.release()

if __name__ == '__main__':
    array = Array('i', 1)
    array[0] = 10
    lock = RLock()
    for i in range(10):
        p = Process(target=func, args=(i, array, lock))
        p.start()
```

执行结果：
```
say hi 9
say hi 8
say hi 7
say hi 6
say hi 5
say hi 4
say hi 3
say hi 2
say hi 1
say hi 0
```

### 进程池

Multiprocessing.Pool可以提供指定数量的进程供用户调用，当有新的请求提交到pool中时，如果池还没有满，那么就会创建一个新的进程用来执行该请求；但如果池中的进程数已经达到规定最大值，那么该请求就会等待，直到池中有进程结束，才会创建新的进程来执行它。在共享资源时，只能使用Multiprocessing.Manager类，而不能使用Queue或者Array。Pool类用于需要执行的目标很多，而手动限制进程数量又太繁琐时，如果目标少且不用控制进程数量则可以用Process类。

**构造方法：**

- Pool([processes[, initializer[, initargs[, maxtasksperchild[, context]]]]])
processes ：使用的工作进程的数量，如果processes是None那么使用 os.cpu_count()返回的数量。
- initializer： 如果initializer是None，那么每一个工作进程在开始的时候会调用initializer(*initargs)。
- maxtasksperchild：工作进程退出之前可以完成的任务数，完成后用一个新的工作进程来替代原进程，来让闲置的资源被释放。maxtasksperchild默认是None，意味着只要Pool存在工作进程就会一直存活。
- context: 用在制定工作进程启动时的上下文，一般使用 multiprocessing.Pool() 或者一个context对象的Pool()方法来创建一个池，两种方法都适当的设置了context。

**实例方法：**

- apply_async(func[, args[, kwds[, callback]]]) 它是非阻塞。
- apply(func[, args[, kwds]])是阻塞的
- close() 关闭pool，使其不在接受新的任务。
- terminate() 关闭pool，结束工作进程，不在处理未完成的任务。
- join() 主进程阻塞，等待子进程的退出， join方法要在close或terminate之后使用。

```python
import time
from multiprocessing import Pool

def func(args):
    time.sleep(1)
    print('正在执行进程%s' % args)

if __name__ == '__main__':
    p = Pool(5)
    for i in range(10):
        p.apply_async(func=func, args=(i, ))
    p.close()
    # time.sleep(2)
    # p.terminate()
    p.join()
```

执行结果：
```
正在执行进程0
正在执行进程1
正在执行进程2
正在执行进程3
正在执行进程4
正在执行进程5
正在执行进程6
正在执行进程7
正在执行进程8
正在执行进程9
```

**Pool+map函数**

说明：此写法缺点在于只能通过map向函数传递一个参数。

```python
from multiprocessing import Pool
def test(i):
    prin(i)
if __name__=="__main__":
    lists=[1,2,3]
    pool=Pool(processes=2) #定义最大的进程数
    pool.map(test,lists)   #p必须是一个可迭代变量。
    pool.close()
    pool.join()
```

`from multiprocessing.dummy import Pool as ThreadPool` 是多线程进程池，绑定一个cpu核心。`from multiprocessing import Pool`多进程，运行于多个cpu核心。multiprocessing 是多进程模块， 而multiprocessing.dummy是以相同API实现的多线程模块。没有绕过GIL情况下，多线程一定受GIL限制。

```python
from multiprocessing.dummy import Pool as tp
def fun(i):
    print(i+i+i+i)

list_i=[x for x in range(4)]

px = tp(processes=8)
# 开启8个线程池
px.map(fun, list_i)
px.close()
px.join()
```

执行结果：
```
0
4
8
12
```

**子进程返回值**

在实际使用多进程的时候，可能需要获取到子进程运行的返回值。如果只是用来存储，则可以将返回值保存到一个数据结构中；如果需要判断此返回值，从而决定是否继续执行所有子进程，则会相对比较复杂。另外在Multiprocessing中，可以利用Process与Pool创建子进程，这两种用法在获取子进程返回值上的写法上也不相同。这篇中，我们直接上代码，分析多进程中获取子进程返回值的不同用法，以及优缺点。
****

初级用法（Pool）

目的：存储子进程返回值

说明：如果只是单纯的存储子进程返回值，则可以使用Pool的apply_async异步进程池；当然也可以使用Process，用法与threading中的相同，这里只介绍前者。

实例：当进程池中所有子进程执行完毕后，输出每个子进程的返回值。

```python
from multiprocessing import Pool

def test(p):
    return p

if __name__=="__main__":
    pool = Pool(processes=10)
    result = []
    for i in range(50000):
       '''
       for循环执行流程：
       （1）添加子进程到pool，并将这个对象（子进程）添加到result这个列表中。（此时子进程并没有运行）
       （2）执行子进程（同时执行10个）
       '''
       result.append(pool.apply_async(test, args=(i,)))#维持执行的进程总数为10，当一个进程执行完后添加新进程.
    pool.close()
    pool.join()
    '''
    遍历result列表，取出子进程对象，访问get()方法，获取返回值。（此时所有子进程已执行完毕）
    '''
    for i in result:
        print(i.get())
```

****
高级用法（Pool）
目的：父进程实时获取子进程返回值，以此为标记结束所有进程。

说明：总共要执行50000个子进程（并发数量为10），当其中一个子进程返回True时，结束进程池。因为使用了apply_async为异步进程，因此在执行完for循环的添加子进程操作后（只是添加并没有执行完所有的子进程），可以直接执行while代码，实时判断子进程返回值是否有True，有的话结束所有进程。

优点：不必等到所有子进程结束再结束程序，只要得到想要的结果就可以提前结束，节省资源。

不足：当需要执行的子进程非常大时，不适用，因为for循环在添加子进程时，要花费很长的时间，虽然是异步，但是也需要等待for循环添加子进程操作结束才能执行while代码，因此会比较慢。

```python
from multiprocessing import Pool
import queue
import time

def test(p):
    time.sleep(0.001)
    if p == 10000:
        return True
    else:
        return False
if __name__=="__main__":
    pool = Pool(processes=10)
    q = queue.Queue()
    for i in range(50000):
        '''
        将子进程对象存入队列中。
        '''
        q.put(pool.apply_async(test, args=(i,)))#维持执行的进程总数为10，当一个进程执行完后添加新进程.
    '''
    因为这里使用的为pool.apply_async异步方法，因此子进程执行的过程中，父进程会执行while，获取返回值并校验。
    '''
    while 1:
        if q.get().get():
            pool.terminate() #结束进程池中的所有子进程。
            break
    pool.join()
```

****

多线程+多进程，添加执行子进程的过程中，不断获取返回值并校验，如果返回值为True则结果所有进程。

执行流程：利用多线程，创建一个执行pool_th函数线程，一个执行result_th函数线程，pool_th函数用来添加进程池，开启进程执行功能函数并将子进程对象存入队列，而result_th()函数用来不停地从队列中取子进程对象，调用get（）方法获取返回值。等发现其中存在子进程的返回值为True时，结束所有进程，最后结束线程。

优点：弥补了上一个例子不足，即使for循环的子进程数量很多，也能提高性能，因为for循环与判断子进程返回值同时进行。

```python
from multiprocessing import Pool
import queue
import threading
import time

def test(p):
    time.sleep(0.001)
    if p == 10000:
        return True
    else:
        return False
if __name__=="__main__":
    result = queue.Queue() #队列
    pool = Pool()
    def pool_th():
        for i in range(50000000): ##这里需要创建执行的子进程非常多
            try:
                result.put(pool.apply_async(test, args=(i,)))
            except:
                break
    def result_th():
        while 1:
            a = result.get().get() #获取子进程返回值
            if a:
                pool.terminate() #结束所有子进程
                break
    '''
    利用多线程，同时运行Pool函数创建执行子进程，以及运行获取子进程返回值函数。
    '''
    t1=threading.Thread(target=pool_th)
    t2=threading.Thread(target=result_th)
    t1.start()
    t2.start()
    t1.join()
    t2.join()
    pool.join()
```




