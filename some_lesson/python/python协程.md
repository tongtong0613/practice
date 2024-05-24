## Asyncio
[原文链接](http://www.langzi.fun/Asyncio%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B.html)


在python3.5之前，都是使用生成器的一些技巧完成协程任务，他们的调度方式依然是 `事件循环+协程模式`。这样设计结构和维护虽然相对于回调函数简单一些，但是代码还是有一些混乱，并且又当作生成器又当作协程，都是还是一些技巧性的东西，为了将语义变得更加明确，于是在python3.5使用了async和await(功能与yield from类似)关键词正式定义原生协程，asyncio是python解决异步io编程的一个完整框架。

它具有如下定义：

1. 包含各种特定系统实现的模块化事件循环
2. 传输与协议抽象
3. 对TCP,UDP,SSL,子进程，延时调用以及其他的具体支持
4. 模仿futures模块适用于事件循环使用到Future类
5. 基于yield from的协议和任务，可以使用顺序执行的方式编写并发代码
6. 必须使用一个将产生阻塞IO的调用时，有接口可以把这个事件转移到线程池
7. 模仿threading模块中的同步语法，可以用在单线程内实现协程同步

协程编程离不开的三大要点：

1. 事件循环
2. 回调(驱动生成器)
3. epoll/select(IO多路复用)
   
Asyncio是一个异步编程的框架，可以解决异步编程，协程调度问题，线程问题，是整个异步IO的解决方案。

### 事件循环

**简单案例(访问一个网站)**
```python
async def get_url_title(url):
# 使用关键词async定义一个协程
    print('开始访问网站:{}'.format(url))
    await asyncio.sleep(2)
    # 这一步至关重要
    # asyncio.sleep(2) 功能:异步非阻塞等待2s，作用是模拟访问网站消耗的时间
    # await 的作用类似 yield，即这个时候把线程资源控制权交出去,监听这个描述符直到这个任务完成
    # await 后面只能接三种类型
    '''
    1. 协程:Python 协程属于 可等待 对象，因此可以在其他协程中被等待:
    2. 任务:任务 被用来设置日程以便 并发 执行协程。(当一个协程通过 asyncio.create_task() 等函数被打包为一个 任务，该协程将自动排入日程准备立即运行)
    3. Future 对象:Future 是一种特殊的 低层级 可等待对象，表示一个异步操作的 最终结果。(当一个 Future 对象 被等待，这意味着协程将保持等待直到该 Future 对象在其他地方操作完毕。)

    如果await time.sleep(2) 是会报错的
    '''
    print('网站访问成功')

if __name__ == '__main__':
    start_time = time.time()
    loop = asyncio.get_event_loop()
    # 一行代码创造事件循环
    loop.run_until_complete(get_url_title('www.baidu.com'))
    # 这是一个阻塞的方法,可以理解成多线程中的join方法
    # 直到get_url_title('http://www.test.com')完成后，才会继续执行下面的代码
    end_time = time.time()
    print('消耗时间:{}'.format(end_time-start_time))
```

执行结果：
```
开始访问网站www.baidu.com
网站访问成功
消耗时间2.00264835357666
```

**简单案例(访问多个网站)**

协程的优势是多任务协作，单任务访问网站没法发挥出他的功能，一次性访问多个网站或者一次性等待多个IO响应时间才能发挥它的优势。

```python
# -*- coding:utf-8 -*-
import asyncio
import time

async def get_url_title(url):
    print('开始访问网站:{}'.format(url))
    await asyncio.sleep(2)
    print('网站访问成功')

if __name__ == '__main__':
    start_time = time.time()
    loop = asyncio.get_event_loop()
    # 创造一个事件循环
    tasks = [get_url_title('http://www.test.com')for i in range(10)]
    # 这个列表代表总任务量，即执行10次get_url_title()函数
    loop.run_until_complete(asyncio.wait(tasks))
    # asyncio.wait后面接上非空可迭代对象,一般来说是功能函数列表
    # 功能是一次性提交多个任务，等待完成
    # loop.run_until_complete(asyncio.gather(*tasks))
    # 和上面代码功能一致，但是gather更加高级，如果是列表就需要加上*
    # 这里会等到全部的任务执行完后才会执行后面的代码
    end_time = time.time()
    print('消耗时间:{}'.format(end_time-start_time))
```

执行结果：
```
开始访问网站www.baidu.com
开始访问网站www.baidu.com
开始访问网站www.baidu.com
开始访问网站www.baidu.com
开始访问网站www.baidu.com
开始访问网站www.baidu.com
开始访问网站www.baidu.com
开始访问网站www.baidu.com
开始访问网站www.baidu.com
开始访问网站www.baidu.com
网站访问成功
网站访问成功
网站访问成功
网站访问成功
网站访问成功
网站访问成功
网站访问成功
网站访问成功
网站访问成功
网站访问成功
消耗时间2.0006277561187744
```

gather与wait的区别：

- gather更擅长于将函数聚合在一起
- wait更擅长筛选运行状况
  
即gather更加高级，他可以将任务分组，也可以取消任务

```python
import asyncio

async def get_url_title(url):
    print('开始访问网站:{}'.format(url))
    await asyncio.sleep(2)
    print('网站访问成功')
    return 'success'

if __name__ == '__main__':
    loop = asyncio.get_event_loop()
    # 使用wait方法
    # tasks = [get_url_title('http://www.test.com')for i in range(10)]
    # loop.run_until_complete(asyncio.wait(tasks))

    # 使用gather方法实现分组导入(方法1)
    # group1 = [get_url_title('http://www.test.com')for i in range(3)]
    # group2 = [get_url_title('http://www.baidu.com')for i in range(5)]
    # loop.run_until_complete(asyncio.gather(*group1,*group2))
    # 这种方法会把两个全部一次性导入

    # 使用gather方法实现分组导入(方法2)
    group1 = [get_url_title('http://www.baidu.com')for i in range(3)]
    group2 = [get_url_title('http://www.hupu.com')for i in range(5)]
    group1 = asyncio.gather(*group1)
    group2 = asyncio.gather(*group2)
    #group2.cancel() 取消group2任务
    loop.run_until_complete(asyncio.gather(group1,group2))
    # 这种方法会先把group1导入，然后导入group2
```

执行结果：
```
开始访问网站http://www.baidu.com
开始访问网站http://www.baidu.com
开始访问网站http://www.baidu.com
开始访问网站http://www.hupu.com
开始访问网站http://www.hupu.com
开始访问网站http://www.hupu.com
开始访问网站http://www.hupu.com
开始访问网站http://www.hupu.com
网站访问成功
网站访问成功
网站访问成功
网站访问成功
网站访问成功
网站访问成功
网站访问成功
网站访问成功
```

另外一种使用gather获取返回结果:

```python
import asyncio

async def get_url_title(url):
    print('开始访问网站:{}'.format(url))
    await asyncio.sleep(2)
    print('网站访问成功')
    return 'success'

if __name__ == '__main__':
    loop = asyncio.get_event_loop()
    # 使用gather方法传递任务获取结果
    group1 = asyncio.ensure_future(get_url_title('http://www.baidu.com'))
    loop.run_until_complete(asyncio.gather(group1))
    # 如果不是列表就不需要加*
    # 这里只能使用gather，而不能用wait
    # 否则会报错TypeError: expect a list of futures, not Task
    print(group1.result())

```

返回结果：

```
开始访问网站:http://www.baidu.com
网站访问成功
success
```

**简单案例(获取返回值)**

```python
# -*- coding:utf-8 -*-
import asyncio
import time

async def get_url_title(url):
    print('开始访问网站:{}'.format(url))
    await asyncio.sleep(2)
    print('网站访问成功')
    return 'success'

if __name__ == '__main__':
    start_time = time.time()
    loop = asyncio.get_event_loop()
    # 创建一个事件循环

    get_future = loop.create_task(get_url_title('http://www.test.com'))
    #get_future = asyncio.ensure_future(get_url_title('http://www.test.com'))
    # 这两行代码功能用法一模一样

    loop.run_until_complete(get_future)
    print('获取结果:{}'.format(get_future.result()))
    # 获取结果

    end_time = time.time()
    print('消耗时间:{}'.format(end_time-start_time))
```

返回结果：

```
开始访问网站:http://www.test.com
网站访问成功
获取结果:success
消耗时间:2.0019724369049072
```

如果是多个网址传入，访问多个网址的返回值呢？只需要把前面的知识点汇总一起即可使用：

```python
if __name__ == '__main__':
    start_time = time.time()
    loop = asyncio.get_event_loop()
    # 创建一个事件循环

    tasks = [loop.create_task(get_url_title('http://www.test.com')) for i in range(10)]
    # 把所有要返回的函数加载到一个列表

    loop.run_until_complete(asyncio.wait(tasks))
    # 这里和上面用法一样

    print('获取结果:{}'.format([x.result() for x in tasks]))
    # 因为结果都在一个列表，在列表中取值即可

    end_time = time.time()
    print('消耗时间:{}'.format(end_time-start_time))

```
返回结果：

```
开始访问网站:http://www.test.com
开始访问网站:http://www.test.com
开始访问网站:http://www.test.com
开始访问网站:http://www.test.com
开始访问网站:http://www.test.com
开始访问网站:http://www.test.com
开始访问网站:http://www.test.com
开始访问网站:http://www.test.com
开始访问网站:http://www.test.com
开始访问网站:http://www.test.com
网站访问成功
网站访问成功
网站访问成功
网站访问成功
网站访问成功
网站访问成功
网站访问成功
网站访问成功
网站访问成功
网站访问成功
获取结果:['success', 'success', 'success', 'success', 'success', 'success', 'success', 'success', 'success', 'success']
消耗时间:2.0016491413116455
```

**简单案例(回调函数)**

上面的例子是一个协程函数，当这个协程函数的await xxx执行完毕后，想要执行另一个函数后，然后再返回这个协程函数的返回结果该这么做：

```PYTHON
# -*- coding:utf-8 -*-
import asyncio
from functools import partial
# partial的功能是 固定函数参数，返回一个新的函数。你可以这么理解：
'''
from functools import partial
    def go(x,y):
        return x+y
    g = partial(go,y=2)
    print(g(1))
返回结果：3

    g = partial(go,x=5,y=2)
    print(g())
返回结果：7

'''
async def get_url_title(url):
    print('开始访问网站:{}'.format(url))
    await asyncio.sleep(2)
    print('网站访问成功')
    # 当这个协程函数快要结束返回值的时候，会调用下面的call_back函数
    # 等待call_back函数执行完毕后，才返回这个协程函数的值
    return 'success'

def call_back(future,url):
    # 注意这里必须要传递future参数，因为这里的future即代表下面的get_future对象
    print('检测网址:{}状态正常'.format(url))

if __name__ == '__main__':
    loop = asyncio.get_event_loop()
    # 创建一个事件循环

    get_future = loop.create_task(get_url_title('http://www.test.com'))
    # 将一个任务注册到loop事件循环中

    get_future.add_done_callback(partial(call_back,url = 'http://www.test.com'))
    # 这里是设置，当上面的任务完成要返回结果的时候，执行call_back函数
    # 注意call_back函数不能加上()，也就意味着你只能依靠partial方法进行传递参数

    loop.run_until_complete(get_future)
    # 等待任务完成
    print('获取结果:{}'.format(get_future.result()))
    # 获取结果
```

返回结果：

```
开始访问网站:http://www.test.com
网站访问成功
检测网址:http://www.test.com状态正常
获取结果:success
```

**梳理**
1. 协程函数必须要使用关键词async定义
2. 如果遇到了要等待的对象，必须要使用await
3. 使用await后面的任务，必须是可等待对象(三种主要类型: 协程, 任务 和 Future.)
4. 运行前，必须要创建一个事件循环(loop = asyncio.get_event_loop(),一行代码即可)
5. 然后把任务加载到该事件循环中即可
6. 如果需要获取协程函数的返回值，需要使用loop.create_task()或asyncio.ensure_future()函数，在最后使用.result()获取返回结果。
7. 如果想要把多个任务注册到loop中，需要使用一个列表包含他们，调用的时候使用asyncio.wait(list)

### 取消协程任务
存在多个任务协程，想使用ctrl c退出协程，使用例子讲解：

```python
import asyncio
async def get_time_sleep(t):
    print('开始运行，等待:{}s'.format(t))
    await asyncio.sleep(t)
    print('运行结束')

if __name__ == '__main__':
    loop = asyncio.get_event_loop()
    # 创建一个事件循环
    task_1 = get_time_sleep(1)
    task_2 = get_time_sleep(2)
    task_3 = get_time_sleep(3)

    tasks = [task_1,task_2,task_3]
    # 三个协程任务加载到一个列表

    try:
        loop.run_until_complete(asyncio.wait(tasks))
    except KeyboardInterrupt:
        # 当检测到键盘输入 ctrl c的时候
        all_tasks = asyncio.Task.all_tasks()
        # 获取注册到loop下的所有task
        for task in all_tasks:
            print('开始取消协程{}'.format(task))
            task.cancel()
            # 取消该协程,如果取消成功则返回True
        loop.stop()
        # 停止循环
        loop.run_forever()
        # loop事件循环一直运行
        # 这两步必须要做
    finally:
        loop.close()
        # 关闭事件循环
```

执行结果：
```
开始运行，等待:3s
开始运行，等待:1s
开始运行，等待:2s
运行结束
运行结束
开始取消协程:<Task pending coro=<get_time_sleep() running at test.py:7> wait_for=<Future pending cb=[<TaskWakeupMethWrapper object at 0x0000022C61281C18>()]> cb=[_wait.<lo
cals>._on_completion() at C:\Users\root\AppData\Local\Programs\Python\Python36\lib\asyncio\tasks.py:380]>
开始取消协程:<Task finished coro=<get_time_sleep() done, defined at test.py:5> result=None>
开始取消协程:<Task finished coro=<get_time_sleep() done, defined at test.py:5> result=None>
开始取消协程:<Task pending coro=<wait() running at C:\Users\root\AppData\Local\Programs\Python\Python36\lib\asyncio\tasks.py:313> wait_for=<Future pending cb=[<TaskWakeupM
ethWrapper object at 0x0000022C61281BB8>()]>>
```

run_forever 会一直运行，直到 stop 被调用，但是你不能像下面这样调 stop

```python
loop.run_forever()
loop.stop()
```

run_forever 不返回，stop 永远也不会被调用。所以，只能在协程中调 stop：

```python
async def do_some_work(loop, x):
    print('Waiting ' + str(x))
    await asyncio.sleep(x)
    print('Done')
    loop.stop()
```

这样并非没有问题，假如有多个协程在 loop 里运行：

```python
asyncio.ensure_future(do_some_work(loop, 1))
asyncio.ensure_future(do_some_work(loop, 3))

loop.run_forever()
```

第二个协程没结束，loop 就停止了——被先结束的那个协程给停掉的。
要解决这个问题，可以用 gather 把多个协程合并成一个 future，并添加回调，然后在回调里再去停止 loop。

```python
import asyncio
import time
import functools

async def do_some_work(x):
    print('Waiting ' + str(x))
    await asyncio.sleep(x)
    print('Done')

def callback(future, loop):
    print(future)
    loop.stop()

if __name__ == '__main__':
    start_time = time.time()
    loop = asyncio.get_event_loop()
    task = asyncio.gather(do_some_work(1), do_some_work(3))
    task.add_done_callback(functools.partial(callback, loop=loop))
    loop.run_forever()

```

执行结果：

```
Waiting 3
Waiting 1
Done
Done
<_GatheringFuture finished result=[None, None]>
```

其实这基本上就是 run_until_complete 的实现了，run_until_complete 在内部也是调用 run_forever。

关于loop.close()，简单来说，loop 只要不关闭，就还可以再运行。

```python
loop.run_until_complete(do_some_work(loop, 1))
loop.run_until_complete(do_some_work(loop, 3))
loop.close()
```

但是如果关闭了，就不能再运行了：

```python
loop.run_until_complete(do_some_work(loop, 1))
loop.close()
loop.run_until_complete(do_some_work(loop, 3))  # 此处异常
```

**梳理**
1. 通过gather()启动的协程任务，是可以直接取消的，并且还能获取取消是否成功
2. 可以通过 asyncio.Task.all_tasks()获取所有的协程任务
3. 如果使用run_forever()的话会一直运行，只能通过loop.stop()停止


### 协程嵌套

```python
import asyncio
async def sum_tion(x,y):
    print('开始执行传入参数相加:{} + {}'.format(x,y))
    await asyncio.sleep(1)
    # 模拟等待1S
    return (x+y)

async def print_sum(x,y):
    result = await sum_tion(x,y)
    print(result)

if __name__ == '__main__':
    loop = asyncio.get_event_loop()

    loop.run_until_complete(print_sum(1000,2000))

    loop.close()
```

返回结果：

```
开始执行传入参数相加:1000 + 2000
3000
```

执行流程：

1. run_until_complete运行，会注册task（协程：print_sum）并开启事件循环
2. print_sum协程中嵌套了子协程，此时print_sum协程暂停（类似委托生成器），转到子协程（协程：sum_tion）中运行代码，期间子协程需sleep1秒钟，直接将结果反馈到event loop中，即将控制权转回调用方，而中间的print_sum暂停不操作
3. 1秒后，调用方将控制权给到子协程（调用方与子协程直接通信），子协程执行接下来的代码，直到再遇到wait（此实例没有）
4. 最后执行到return语句，子协程向上级协程（print_sum抛出异常：StopIteration），同时将return返回的值返回给上级协程（print_sum中的result接收值），print_sum继续执行暂时时后续的代码，直到遇到return语句
5. 向 event loop 抛出StopIteration异常，此时协程任务都已经执行完毕，事件循环执行完成（event loop ：the loop is stopped），close事件循环。
   

如果想要获取协程嵌套函数返回的值，就必须使用回调：

```python
import asyncio
async def sum_tion(x,y)->int:
    print('开始执行传入参数相加:{} + {}'.format(x,y))
    await asyncio.sleep(1)
    # 模拟等待1S
    return (x+y)

async def print_sum(x,y):
    result = await sum_tion(x,y)
    return result

def callback(future):
    return future.result()

if __name__ == '__main__':
    loop = asyncio.get_event_loop()

    future = loop.create_task(print_sum(100,200))
    # 如果想要获取嵌套协程返回的值，就必须使用回调

    future.add_done_callback(callback)
    loop.run_until_complete(future)

    print(future.result())

    loop.close()
```

返回结果：

```
开始执行传入参数相加:100 + 200
300
```

### 定时启动任务
asyncio提供定时启动协程任务，通过call_soon,call_later,call_at实现，他们的区别如下：

**call_soon**
call_soon是立即执行

```python
def callback(sleep_times):
    print("预计消耗时间 {} s".format(sleep_times))
def stoploop(loop):
    print('时间消耗完毕')
    loop.stop()


if __name__ == "__main__":
    start_time = time.time()
    loop = asyncio.get_event_loop()
    # 创建一个事件循环
    loop.call_soon(callback,5)
    # 立即启动callback函数
    loop.call_soon(stoploop,loop)
    # 上面执行完毕后，立即启动执行stoploop函数
    loop.run_forever()
    #要用这个run_forever运行，因为没有传入协程
    print('总共耗时:{}'.format(time.time()-start_time))
```

返回结果：

```
预计消耗时间 5 s
时间消耗完毕
总共耗时:0.0010013580322265625
```

**call_later**
call_later是设置一定时间启动执行

```python
def callback(sleep_times):
    print("预计消耗时间 {} s".format(sleep_times))
def stoploop(loop):
    print('时间消耗完毕')
    loop.stop()


if __name__ == "__main__":
    start_time = time.time()
    loop = asyncio.get_event_loop()


    loop.call_later(1,callback,1.0)
    # 等待1秒后执行callback函数，传入参数是1.0
    loop.call_later(5,stoploop,loop)
    # 等待5秒后执行stoploop函数，传入参数是loop

    loop.run_forever()
    print('总共耗时:{}'.format(time.time()-start_time))
```

返回结果：

```
预计消耗时间 1.0 s
时间消耗完毕
总共耗时:5.002613544464111
```

**call_at**
call_at类似与call_later，但是他指定的时间不再是传统意义上的时间，而是loop的内部时钟时间，效果和call_later一样， call_at内部其实调用了call_later

```python
import time
import asyncio

def callback(loop):
    print("传入loop.time()时间为: {} s".format(loop.time()))
def stoploop(loop):
    print('时间消耗完毕')
    loop.stop()


if __name__ == "__main__":
    start_time = time.time()
    loop = asyncio.get_event_loop()

    now = loop.time()
    # loop内部的时钟时间
    loop.call_at(now+1,callback,loop)
    # 等待loop内部时钟时间加上1s后，执行callba函数，传入参数为loop
    loop.call_at(now+3,callback,loop)
    # 等待loop内部时钟时间加上3s后，执行callba函数，传入参数为loop
    loop.call_at(now+5,stoploop,loop)
    # 等待loop内部时钟时间加上1s后，执行stoploop函数，传入参数为loop
```

返回结果:

```
传入loop.time()时间为: 3989.39 s
传入loop.time()时间为: 3991.39 s
时间消耗完毕
总共耗时:5.002060174942017
```

call_soon_threadsafe 线程安全的call_soon
call_soon_threadsafe用法和call_soon一致。但在涉及多线程时， 会使用它.

**梳理**
- call_soon直接启动
- call_later自己定时启动
- call_at根据loop.time()内部的时间，设置等待时间启动
- call_soon_threadsafe和call_soon方法一致，是保证线程安全的
- 他们都是比较底层的，在正常使用时很少用到。


### 结合线程池

Asyncio是异步IO编程的解决方案，异步IO是包括多线程，多进程，和协程的。所以asyncio是可以完成多线程多进程和协程的，在开头说到，协程是单线程的，如果遇到阻塞的话，会阻塞所有的代码任务，所以是不能加入阻塞IO的，但是比如requests库是阻塞的，socket如果不设置setblocking(false)的话，也是阻塞的，这个时候可以放到一个线程中去做也是可以解决的，即在协程中集成阻塞IO，就加入多线程一起解决问题。

**用requests完成异步编程(使用线程池)**

```python
from concurrent.futures import ThreadPoolExecutor
import requests
import asyncio
import time
import re

def get_url_title(url):
    # 功能是获取网址的标题
    r = requests.get(url)
    try:
        title = re.search('<title>(.*?)</title>',r.content.decode(),re.S|re.I).group(1)
    except Exception as e:
        title = e
    print(title)

if __name__ == '__main__':
    start_time = time.time()

    loop = asyncio.get_event_loop()
    # 创建一个事件循环
    p = ThreadPoolExecutor(5)
    # 创建一个线程池，开启5个线程
    tasks = [loop.run_in_executor(p,get_url_title,'http://www.langzi.fun')for i in range(10)]
    # 这一步很重要，使用loop.run_in_executor()函数:内部接受的是阻塞的线程池，执行的函数，传入的参数
    # 即对网站访问10次，使用线程池访问
    loop.run_until_complete(asyncio.wait(tasks))
    # 等待所有的任务完成
    print(time.time()-start_time)
```

返回结果：
```
 Langzi - Never Setter 永不将就 - 致力于Python开发网络安全工具,分享Python底层与进阶知识，漏洞扫描器开发与爬虫开发 
 Langzi - Never Setter 永不将就 - 致力于Python开发网络安全工具,分享Python底层与进阶知识，漏洞扫描器开发与爬虫开发 
 Langzi - Never Setter 永不将就 - 致力于Python开发网络安全工具,分享Python底层与进阶知识，漏洞扫描器开发与爬虫开发 
 Langzi - Never Setter 永不将就 - 致力于Python开发网络安全工具,分享Python底层与进阶知识，漏洞扫描器开发与爬虫开发 
 Langzi - Never Setter 永不将就 - 致力于Python开发网络安全工具,分享Python底层与进阶知识，漏洞扫描器开发与爬虫开发 
 Langzi - Never Setter 永不将就 - 致力于Python开发网络安全工具,分享Python底层与进阶知识，漏洞扫描器开发与爬虫开发 
 Langzi - Never Setter 永不将就 - 致力于Python开发网络安全工具,分享Python底层与进阶知识，漏洞扫描器开发与爬虫开发 
 Langzi - Never Setter 永不将就 - 致力于Python开发网络安全工具,分享Python底层与进阶知识，漏洞扫描器开发与爬虫开发 
 Langzi - Never Setter 永不将就 - 致力于Python开发网络安全工具,分享Python底层与进阶知识，漏洞扫描器开发与爬虫开发 
 Langzi - Never Setter 永不将就 - 致力于Python开发网络安全工具,分享Python底层与进阶知识，漏洞扫描器开发与爬虫开发 
5.589502334594727
```

访问10次消耗时间为5.5s，尝试将 p = ThreadPoolExecutor(10)，线程数量设置成10个线程，消耗时间为4.6s，改用从进程池p = ProcessPoolExecutor(10)，也是一样可以运行的，不过10个进程消耗时间也是5.5s，并且消耗更多的CPU资源。

**用socket完成异步编程(使用线程池)**

```python
import asyncio
from concurrent.futures import ThreadPoolExecutor
import socket
from urllib.parse import urlparse
import time
import re


def get_url(url):
    # 通过socket请求html
    url = urlparse(url)
    host = url.netloc
    path = url.path
    if path == "":
        path = '/'

    # 建立socket连接
    client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    client.connect((host, 80))
    client.send(
        "GET {} HTTP/1.1\r\nHost:{}\r\nConnection:close\r\n\r\n".format(path, host).encode('utf8'))
    data = b""
    while True:
        d = client.recv(1024)
        if d:
            data += d
        else:
            break
    data = data.decode('utf8')
    html_data = data.split('\r\n\r\n')[1]
    # 把请求头信息去掉， 只要网页内容
    title = re.search('<title>(.*?)</title>',html_data,re.S|re.I).group(1)
    print(title)
    client.close()


if __name__ == '__main__':
    start_time = time.time()
    loop = asyncio.get_event_loop()
    p = ThreadPoolExecutor(3)  # 线程池 放3个线程
    tasks = [loop.run_in_executor(p,get_url,'http://www.langzi.fun') for i in range(10)]
    loop.run_until_complete(asyncio.wait(tasks))
    print('last time:{}'.format(time.time() - start_time))
```

返回结果：

```
 Langzi - Never Setter 永不将就 - 致力于Python开发网络安全工具,分享Python底层与进阶知识，漏洞扫描器开发与爬虫开发 
 Langzi - Never Setter 永不将就 - 致力于Python开发网络安全工具,分享Python底层与进阶知识，漏洞扫描器开发与爬虫开发 
 Langzi - Never Setter 永不将就 - 致力于Python开发网络安全工具,分享Python底层与进阶知识，漏洞扫描器开发与爬虫开发 
 Langzi - Never Setter 永不将就 - 致力于Python开发网络安全工具,分享Python底层与进阶知识，漏洞扫描器开发与爬虫开发 
 Langzi - Never Setter 永不将就 - 致力于Python开发网络安全工具,分享Python底层与进阶知识，漏洞扫描器开发与爬虫开发 
 Langzi - Never Setter 永不将就 - 致力于Python开发网络安全工具,分享Python底层与进阶知识，漏洞扫描器开发与爬虫开发 
 Langzi - Never Setter 永不将就 - 致力于Python开发网络安全工具,分享Python底层与进阶知识，漏洞扫描器开发与爬虫开发 
 Langzi - Never Setter 永不将就 - 致力于Python开发网络安全工具,分享Python底层与进阶知识，漏洞扫描器开发与爬虫开发 
 Langzi - Never Setter 永不将就 - 致力于Python开发网络安全工具,分享Python底层与进阶知识，漏洞扫描器开发与爬虫开发 
 Langzi - Never Setter 永不将就 - 致力于Python开发网络安全工具,分享Python底层与进阶知识，漏洞扫描器开发与爬虫开发 
last time:5.132313966751099
```

**使用socket完成http请求(未使用线程池)**

```python
import asyncio
from urllib.parse import urlparse
import time


async def get_url(url):
    # 通过socket请求html
    url = urlparse(url)
    host = url.netloc
    path = url.path
    if path == "":
        path = '/'

    # 建立socket连接
    reader, writer = await asyncio.open_connection(host, 80)  # 协程 与服务端建立连接
    writer.write(
        "GET {} HTTP/1.1\r\nHost:{}\r\nConnection:close\r\n\r\n".format(path, host).encode('utf8'))
    all_lines = []
    async for raw_line in reader:  # __aiter__ __anext__魔法方法
        line = raw_line.decode('utf8')
        all_lines.append(line)
    html = '\n'.join(all_lines)
    return html


async def main():
    tasks = []
    tasks = [asyncio.ensure_future(get_url('http://www.langzi.fun')) for i in range(10)]
    for task in asyncio.as_completed(tasks):  # 完成一个 print一个
        result = await task
        print(result)

if __name__ == '__main__':
    start_time = time.time()
    loop = asyncio.get_event_loop()
    loop.run_until_complete(main())
    print('last time:{}'.format(time.time() - start_time))
```

**梳理**

- 协程中遇到必须要使用阻塞任务的时候，可以把阻塞代码放到线程池中运行
- 线程池中的代码放到loop.run_in_executor()里面，并且所有任务保存到列表
- 最后通过loop.run_until_complate(asyncio.wait(任务列表))中运行
asyncio能通过socket实现与服务端建立连接

### 与多进程的结合

既然异步协程和多进程对网络请求都有提升，那么为什么不把二者结合起来呢？在最新的 PyCon 2018 上，来自 Facebook 的 John Reese 介绍了 asyncio 和 multiprocessing 各自的特点，并开发了一个新的库，叫做 aiomultiprocess

这个库的安装方式是：

    pip3 install aiomultiprocess
需要 Python 3.6 及更高版本才可使用。

使用这个库，我们可以将上面的例子改写如下：

```python
import asyncio
import aiohttp
import time
from aiomultiprocess import Pool

start = time.time()

async def get(url):
    session = aiohttp.ClientSession()
    response = await session.get(url)
    result = await response.text()
    await session.close()
    return result

async def request():
    url = 'https://www.baidu.com'
    urls = [url for _ in range(30)]
    async with Pool() as pool:
        result = await pool.map(get, urls)
        return result

if __name__ == '__main__':

    coroutine = request()
    task = asyncio.ensure_future(coroutine)
    loop = asyncio.get_event_loop()
    loop.run_until_complete(task)
    print(task.result())

    end = time.time()
    print('Cost time:', end - start)
```


### 同步与通信

和多线程多进程任务一样，协程也可以实现和需要进行同步与通信。

**简单例子(顺序启动多任务)**

协程是单线程的，他的执行依赖于事件循环中最后的loop.run_until_complate()

```python
import asyncio

num = 0

async def add():
    global num
    for i in range(10):
        await asyncio.sleep(0.1)
        num += i
async def desc():
    global num
    for i in range(10):
        await asyncio.sleep(0.2)
        num -= i

if __name__ == '__main__':
    loop = asyncio.get_event_loop()
    tasks = [add(),desc()]
    loop.run_until_complete(asyncio.wait(tasks))
    # 这里执行顺序是先执行add函数，然后执行desc函数
    # 所以最后的结果是0
    loop.close()
    print(num)
```

返回结果：

```
0
```

这里使用一个共有变量，协程下不需要加锁。

**简单例子(Lock(锁))**

```python
import asyncio
import functools

def unlock(lock):
    lock.release()
    print('线程锁已释放')


async def test(name, lock):
    print('{}等待线程释放'.format(name))
    # ---------------------------------
    # with await lock:
    #     print(f'{name} 线程锁上锁')
    # ---------------------------------
    # 上面这两行代码等同于：
    # ---------------------------------
    # await lock.acquire()
    # print(f'{name} 线程锁上锁')
    # lock.release()
    # ---------------------------------
    # 等同于：
    # ---------------------------------
    # async with lock:
    #    print(f'{name} 线程锁上锁')
    # ---------------------------------
    await lock.acquire()
    print(f'{name}线程锁上锁')
    lock.release()
    print(f'{name}线程锁释放')


async def main(loop):
    lock = asyncio.Lock()
    await lock.acquire()
    print('线程锁已上锁')
    loop.call_later(0.5, functools.partial(unlock, lock))
    # call_later() 表达推迟一段时间的回调, 第一个参数是以秒为单位的延迟, 第二个参数是回调函数
    await asyncio.wait([test('任务1', lock), test('任务2', lock)])


if __name__ == "__main__":
    loop = asyncio.get_event_loop()
    loop.run_until_complete(main(loop))
    loop.close()
```

运行结果:

```
线程锁已上锁
任务1等待线程释放
任务2等待线程释放
线程锁已释放
任务1线程锁上锁
任务1线程锁释放
任务2线程锁上锁
任务2线程锁释放
```

**简单例子(Semaphore(信号量))**

可以使用 Semaphore(信号量) 来控制并发访问的数量:

```python
import asyncio
from aiohttp import ClientSession

async def fetch(sem, url):
    async with sem:
        async with ClientSession() as session:
            async with session.get(url) as response:
                status = response.status
                res = await response.text()
                print('{}:{}'.format(response.url, status))
                return res

def callback(future):
    for result in future.result():
        print(result)

if __name__ == '__main__':
    loop = asyncio.get_event_loop()
    url = 'https://www.baidu.com'
    sem = asyncio.Semaphore(2)
    task = [asyncio.ensure_future(fetch(sem, url)) for _ in range(4)]
    gather_task = asyncio.gather(*task)
    gather_task.add_done_callback(callback)
    loop.run_until_complete(asyncio.gather(gather_task))

```

运行结果：
```
https://www.baidu.com:200
https://www.baidu.com:200
https://www.baidu.com:200
https://www.baidu.com:200
<html>
<head>
	<script>
		location.replace(location.href.replace("https://","http://"));
	</script>
</head>
<body>
	<noscript><meta http-equiv="refresh" content="0;url=http://www.baidu.com/"></noscript>
</body>
</html>
<html>
<head>
	<script>
		location.replace(location.href.replace("https://","http://"));
	</script>
</head>
<body>
	<noscript><meta http-equiv="refresh" content="0;url=http://www.baidu.com/"></noscript>
</body>
</html>
<html>
<head>
	<script>
		location.replace(location.href.replace("https://","http://"));
	</script>
</head>
<body>
	<noscript><meta http-equiv="refresh" content="0;url=http://www.baidu.com/"></noscript>
</body>
</html>
<html>
<head>
	<script>
		location.replace(location.href.replace("https://","http://"));
	</script>
</head>
<body>
	<noscript><meta http-equiv="refresh" content="0;url=http://www.baidu.com/"></noscript>
</body>
</html>
```

**简单例子(Condition(条件))**

```python
import asyncio


async def consumer(cond, name, second):
    await asyncio.sleep(second)
    async with cond:
        print('{}: 正在等待'.format(name))
        await cond.wait()
        print('{}: 得到响应'.format(name))

async def prodecer(cond):
    print(f'producer准备sleep 2秒')
    await asyncio.sleep(2)
    for n in range(1, 3):
        async with cond:
            print('生产者 {} 号'.format(n))
            cond.notify(n=n)
        await asyncio.sleep(0.1)

async def producer2(cond):
    print(f'producer2准备sleep 2秒')
    await asyncio.sleep(2)
    with await cond:
        print('释放信号量，通知所有消费者')
        cond.notify_all()


async def main(loop):
    condition = asyncio.Condition()
    task = loop.create_task(prodecer(condition))
    consumers = [consumer(condition, name, index) for index, name in enumerate(('c1', 'c2'))]
    await asyncio.wait(consumers)
    task.cancel()
    print('---分割线---')
    task = loop.create_task(producer2(condition))
    consumers = [consumer(condition, name, index) for index, name in enumerate(('c1', 'c2'))]
    await asyncio.wait(consumers)
    task.cancel()


if __name__ == '__main__':
    loop = asyncio.get_event_loop()
    loop.run_until_complete(main(loop))
    loop.close()
```

运行结果：
```
producer准备sleep 2秒
c1: 正在等待
c2: 正在等待
生产者 1 号
c1: 得到响应
生产者 2 号
c2: 得到响应
---分割线---
producer2准备sleep 2秒
c1: 正在等待
c2: 正在等待
释放信号量，通知所有消费者
c1: 得到响应
c2: 得到响应
```

多线程Condition示例改为协程：

```python
import asyncio

num = 0

async def run(con, name, action):
    global num
    await con.acquire()
    print('开始执行%s' % name)
    while True:
        await asyncio.sleep(1)
        if action == 'add':
            num += 1
        elif action == 'reduce':
            num -= 1
        else:
            exit(1)
        print('当前num为：%s' % num)
        if num >= 5 or num <= 0:
            print('暂停执行%s' % name)
            con.notify()
            await con.wait()
            print('开始执行%s' % name)

async def main(loop):
    condition = asyncio.Condition()
    task = [run(condition, '任务A', 'reduce'), run(condition, '任务B', 'add')]
    await asyncio.wait(task)


if __name__ == '__main__':
    loop = asyncio.get_event_loop()
    loop.run_until_complete(main(loop))
    loop.close()
```

**简单例子(Event(事件))**

与 Lock(锁) 不同的是, 事件被触发的时候, 两个消费者不用获取锁, 就要尽快地执行下去了

```python
import asyncio
import functools


def set_event(event):
    print('开始设置事件')
    event.set()


async def test(name, event):
    print('{} 的事件未设置'.format(name))
    await event.wait()
    print('{} 的事件已设置'.format(name))


async def main(loop):
    event = asyncio.Event()
    # 声明事件
    print('事件是否设置: {}'.format(event.is_set()))
    loop.call_later(0.1, functools.partial(set_event, event))
    # 在0.1s后执行set_event()函数，对事件进行设置
    await asyncio.wait([test('e1', event), test('e2', event)])
    print('最终事件状态: {}'.format(event.is_set()))


if __name__ == '__main__':
    loop = asyncio.get_event_loop()
    loop.run_until_complete(main(loop))
    loop.close()
```

返回结果：

```
事件是否设置: False
e1 的事件未设置
e2 的事件未设置
开始设置事件
e1 的事件已设置
e2 的事件已设置
最终事件状态: True
```

多线程Event示例改为协程：
```python
import asyncio
import functools


# 定义一个事件的对象
async def lighter(event):
    green_time = 5
    # 绿灯时间
    red_time = 5
    # 红灯时间
    event.set()
    # 初始设为绿灯
    while True:
        print("\33[32;0m 绿灯亮...\033[0m")
        await asyncio.sleep(green_time)
        event.clear()
        print("\33[31;0m 红灯亮...\033[0m")
        await asyncio.sleep(red_time)
        event.set()

async def run(name, event):
    while True:
        if event.is_set():
        # 判断当前是否"放行"状态
            print("一辆[%s] 呼啸开过..." % name)
            await asyncio.sleep(1)
        else:
            print("一辆[%s]开来，看到红灯，无奈的停下了..." % name)
            await event.wait()
            print("[%s] 看到绿灯亮了，瞬间飞起....." % name)

async def main(loop):
    event = asyncio.Event()
    tasks = [run(car, event) for car in ['奔驰', '宝马', '奥迪']] + [lighter(event)]
    await asyncio.wait(tasks)

if __name__ == '__main__':
    loop = asyncio.get_event_loop()
    loop.run_until_complete(main(loop))
    loop.close()
```

**简单例子(协程间通信)**

协程是单线程，因此使用list、dict就可以实现通信，而不会有线程安全问题，当然可以使用asyncio.Queue

```python
from asyncio import Queue
queue = Queue(maxsize=3)   
# queue的put和get需要用await
```

举个例子：

```python
import asyncio
from asyncio import Queue
import random
import string
q = Queue(maxsize=100)

async def add():
    while 1:
        await q.put(random.choice(string.ascii_letters))

async def desc():
    while 1:
        res = await q.get()
        print(res)
        await asyncio.sleep(1)

if __name__ == '__main__':
    loop = asyncio.get_event_loop()
    loop.run_until_complete(asyncio.wait([add(),desc()]))
    loop.run_forever()

```

返回结果:

```
D
b
S
x
...
```

