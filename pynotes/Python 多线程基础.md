# Python 多线程基础

- `threading.Thread`

- `start()`

- `join()`

- `threading.current_thread()`



```Python
import threading
import time


def task():
    for i in range(5):
        print("任务执行中", i)
        time.sleep(1)


t = threading.Thread(target=task)

t.start()
```

这里最关键的是：

```Plain Text
t = threading.Thread(target=task)
```

意思是：

> 创建一个线程对象，让这个线程以后去执行 `task()` 函数。
> 
> 

注意这里写的是：

```Plain Text
target=task
```

不是：

```Plain Text
target=task()
```

因为 `task` 是“把函数交给线程”，而 `task()` 是“现在立刻执行函数”。

## `start()` 是干什么的

这一句：

```Plain Text
t.start()
```

意思是：

> 启动线程。
> 
> 

启动之后，这个新线程就会开始执行：

```Plain Text
task()
```

所以你可以把它理解成：

```Plain Text
创建线程
    ↓
Thread(target=task)
    ↓
线程还没运行
    ↓
t.start()
    ↓
线程开始执行 task()
```

---

## 真正看看两个线程同时工作

我们修改一下：

```Python
import threading
import time


def task1():
    for i in range(5):
        print("任务1:", i)
        time.sleep(1)


def task2():
    for i in range(5):
        print("任务2:", i)
        time.sleep(1)


t1 = threading.Thread(target=task1)
t2 = threading.Thread(target=task2)

t1.start()
t2.start()
```

你可能会看到类似：

```Plain Text
任务1: 0
任务2: 0
任务1: 1
任务2: 1
任务1: 2
任务2: 2
...
```

这就和普通顺序程序不一样了。

如果不用线程：

```Plain Text
task1()task2()
```

执行过程是：

```Plain Text
task1 全部执行完
↓
task2 才开始
```

大约需要：

```Plain Text
5 秒 + 5 秒 = 10 秒
```

但多线程：

```Plain Text
t1.start()
t2.start()
```

两个任务的等待时间可以重叠。

因为每次：

```Plain Text
time.sleep(1)
```

当前线程都在等待。

于是另一个线程可以运行。

所以整个程序大概只需要：

```Plain Text
5 秒左右
```

这正好体现了我们上一节讲的：

> I/O 等待型任务非常适合并发。
> 
> 



## 主线程

你运行 Python 程序的时候，本身就已经有一个线程了：

```Plain Text
MainThread
```

叫做：

**主线程**。



```Python
import threading

print(threading.current_thread())
```

这里的输出应该是"MainThread"



我们也可以这么写

```Python
import threading
import time


def task():
    print("现在执行我的线程是：")
    print(threading.current_thread())


t = threading.Thread(target=task)

t.start()
```

输出是：

现在执行我的线程是：

\<Thread\(Thread\-1 \(task\), started 6154317824\)\>



当然我们可以给线程起名字：

```Python
t = threading.Thread(
    target=task,
    name="下载线程"
)
```



## join\(\)与start\(\)

```Python
import threading
import time


def task():
    time.sleep(3)
    print("线程执行完毕")


t = threading.Thread(target=task)

t.start()

print("程序结束")
```

你可能看到：

```Plain Text
程序结束
线程执行完毕
```

为什么？

因为：

```Plain Text
t.start()
```

启动线程之后，**主线程不会停下来等它**。

可以理解成：

```Plain Text
主线程：
创建 t
↓
启动 t
↓
继续执行 print("程序结束")


线程 t：
开始工作
↓
sleep 3 秒
↓
打印
```

所以两个线程分别往前走。

如果我们希望：

> 主线程必须等 `t` 执行结束以后才能继续。
> 
> 

就使用：

```Plain Text
t.join()
```

```Python
import threading
import time


def task():
    time.sleep(3)
    print("线程执行完毕")


t = threading.Thread(target=task)

t.start()

t.join()

print("程序结束")
```

这次输出顺序一定是：

```Plain Text
线程执行完毕
程序结束
```

因为：

```Plain Text
t.join()
```

相当于告诉主线程：

> 在这里等着，直到 `t` 执行结束。
> 
> 



你可以这样记：

```Plain Text
t.start()
```

意思：

> 你去干活。
> 
> 

而：

```Plain Text
t.join()
```

意思：

> 我在这里等你干完。
> 
> 

所以常见代码：

```Plain Text
t1.start()
t2.start()
t1.join()
t2.join()
```

意思是：

```Plain Text
启动线程1
启动线程2
↓
它们并发工作
↓
主线程等待线程1
主线程等待线程2
↓
全部结束
```

注意不要写成：

```Plain Text
t1.start()
t1.join()
t2.start()
t2.join()
```

虽然代码没有错，但是这样通常失去了并发意义。

因为变成：

```Plain Text
启动 t1
↓
等 t1 完成
↓
启动 t2
↓
等 t2 完成
```

又基本变回顺序执行了。





### Race Condition(竞态条件）和 Lock（锁）

##### 什么是race condition:

先看一个列子：
```python
import threading

count = 0


def add():
    global count

    for _ in range(100000):
        count += 1


t1 = threading.Thread(target=add)
t2 = threading.Thread(target=add)

t1.start()
t2.start()

t1.join()
t2.join()

print(count)
```



在这里，直觉告诉我们，最后的输出应该是:200000

但事实并非如此

多线程程序里，一个危险点是：

```
count += 1
```

看起来是一句话，实际上从逻辑上可以拆成：

```
temp = counttemp = temp + 1count = temp
```

也就是：

```
读取 count
↓
计算 count + 1
↓
把结果写回 count
```

问题就来了。

假设：

```
count = 10
```

线程 1 开始：

```
线程1读取 count
得到 10
```

然后操作系统突然切换到线程 2：

```
线程2读取 count
也得到 10

线程2计算：
10 + 1 = 11

线程2写回：
count = 11
```

然后再切回线程 1：

```
线程1之前读到的是 10

所以线程1计算：
10 + 1 = 11

线程1写回：
count = 11
```

两个线程明明各加了一次，本来应该：

```
10 → 11 → 12
```

结果却变成：

```
10 → 11
```

其中一次修改丢失了。

这就是典型的：

> **Race Condition，竞态条件。**

可以把 Race Condition 定义成：

> 多个线程并发访问和修改共享资源时，由于执行顺序的不确定性，导致程序结果不确定或错误。



那么怎么解决这个问题？

## 使用Lock

Python 提供：

```
threading.Lock()
```

例如：

```
lock = threading.Lock()
```

它相当于创建了一把锁。

然后修改共享资源的时候：

```
with lock:    ...
```

完整代码：

```python
import threading
import time

count = 0
lock = threading.Lock()


def add():
    global count

    for _ in range(1000):
        with lock:
            temp = count

            time.sleep(0.0001)

            temp += 1
            count = temp


t1 = threading.Thread(target=add)
t2 = threading.Thread(target=add)

t1.start()
t2.start()

t1.join()
t2.join()

print(count)
```



现在应该稳定输出：

```
2000
```

假设线程 1 先执行：

```
with lock:
```

它拿到了锁。

可以理解成：

```
线程1
↓
拿钥匙
↓
进入房间
↓
修改共享数据
```

此时线程 2 也执行：

```
with lock:
```

但发现：

```
锁已经在线程1手里
```

所以线程 2 只能等。

等线程 1 执行完：

```
with lock:    ...
```

线程 1 自动释放锁。

然后线程 2 才能进去。所以这样就避免了两个线程互相竞争的情况





# `acquire()` 和 `release()`

`with lock:` 本质上是更方便的写法。

你也可以写：

```
lock.acquire()count += 1lock.release()
```

意思分别是：

```
lock.acquire()
```

拿锁。

```
lock.release()
```

释放锁。

但是更推荐：

```
with lock:    count += 1
```

因为如果中间报错：

```
lock.acquire()count += 1# 这里报错lock.release()
```

那么：

```
lock.release()
```

可能根本执行不到。

锁一直没释放，其他线程就可能永远等着。

而：

```
with lock:
```

会帮助你正确管理锁。

所以实际写代码通常优先：

```
with lock:    ...
```

------

# 8. 什么是临界区 Critical Section？

这一段：

```
with lock:    count += 1
```

里面真正需要保护的代码叫：

> **Critical Section，临界区。**

也就是：

> 操作共享资源、不能让多个线程同时执行的那一小段代码。

例如：

```python
def task():
    do_something()

    with lock:
        shared_data += 1

    do_something_else()
```



只有：

```
shared_data += 1
```

真正需要加锁。

一般原则是：

> **只锁真正需要保护的共享资源操作。**

例如：

```python
def task():
    time.sleep(5)

    with lock:
        count += 1
```



这样：

```
sleep 期间大家都可以并发
```

只有最后修改 `count` 时才排队。
