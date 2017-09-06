---
title: Effective Python - 学习笔记
date: 2017-09-05 13:20:54
tags: [Python基础]
---


## 1 Thinking In Pythonic

### 1.1 Knowing Bytes、Str、Unicode

<!--more-->

```pythons
def str_and_unicode():
    """
    In Python3: Str.encode() -> Bytes, Bytes.decode() -> Str
    In Python2: Unicode.encode() -> Str, Str.decode() -> Unicode
    """
    my_unicode = u'你好'
    my_str = my_unicode.encode('utf-8')
    print my_str
```

### 1.2 Using List Generator Instead Of Map & Filter

```python
def list_generator():
    a = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    sqr_list_gen = [x ** 2 for x in a]
    sqr_lambda = map(lambda x: x ** 2, a)
    sqr_list_gen_with_condition = [x ** 2 for x in a if x % 2 == 0]
    sqr_map_filter = map(lambda x: x ** 2, filter(lambda x: x % 2 == 0, a))
    print sqr_list_gen
    print sqr_lambda
    print sqr_list_gen_with_condition
    print sqr_map_filter
    print "============== Pretty Split Line =============="
    chile_ranks = {'tom': 1, 'helen': 2, 'harry_porter': 3}
    rank_dict = {rank: name for name, rank in chile_ranks.items()}  # Dict Generator
    print rank_dict
    print "============== Pretty Split Line =============="
    chile_len_set = {len(name) for name in rank_dict.values()}
    print chile_len_set
```


### 1.3 Don't Using List Generator More Than Twice With One Time


```python
def more_list_generator():
    matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
    # flat = [e for row in matrix for e in row]
    flat = [[x for x in row] for row in matrix]
    print flat
    sqr = [[e ** 2 for e in row] for row in matrix]
    print sqr

    my_lists = [
        [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
    ]
    # Write Multiple Lines While Using Multiple List Generators
    flat = [x for s1 in my_lists
            for s2 in s1
            for x in s2]
    print flat

    # Multiple Conditions While Creating List
    a = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    b = [x for x in a if x > 4 if x % 2 == 0]
    print b

    # Find The Elements Whose RowSum >= 10 And Element % 3 == 0
    filtered = [[x for x in row if x % 3 == 0] for row in matrix if sum(row) >= 10]
    # filtered = [e for row in matrix if sum(row) >= 10 for e in row if e % 3 == 0]
    print filtered
```

### 1.4 Generator Expression


```python
def generator_expression():
    l = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    l_it = (x for x in l)
    sqr_it = ((x, x ** 2) for x in l_it)
    for x in sqr_it:
        print x
```

### 1.5 Enumerate


```python
def enum_instead_of_range():
    from random import randint
    # Range Is Useful while Handle Integers
    random_bits = 0
    for i in range(64):
        if randint(0, 1):
            random_bits |= 1 << i
    print "============== Pretty Split Line =============="

    flavor_list = ['coco', 'coke', 'KungPaoKiTen', 'Banana']
    for flavor_index in range(len(flavor_list)):
        print '%d : %s is delicious' % (flavor_index + 1, flavor_list[flavor_index])

    print "============== Pretty Split Line =============="

    for i, flavor in enumerate(flavor_list, 1):
        print '%d : %s is delicious' % (i, flavor)

```

### 1.6 Zip


```python
def iterator_multiple_collections_by_zip():
    names = ['Niko', 'Tom', 'Helen']
    ages = [21, 15, 25]
    sizes = {'A': 35, 'B': 60, 'C': 22}
    for name, age, size, in zip(names, ages, sizes):
        print name, age, size

```

### 1.7 For Else


```python
def for_else():
    num_list = [x for x in range(100)]
    for n in num_list:
        print n
        if n == 20:
            break
    else:
        print 'for never break.'

```

### 1.8 Try Except Else Finally


```python
def try_except_else_finally():
    f = open('/Users/lixiwei-mac/Documents/IdeaProjects/PythonStudy/effective_python/test.txt', 'r')
    try:
        data = f.read()
        if len(data) < 100:
            raise Exception("Shit ! This fucking file is too short")
    except Exception as e:
        raise ValueError(e)
    else:
        print 'There is no exception happend.'
        print data
        return data
    finally:
        print 'closing file handle'
        f.close()

```

## 2 Function


### 2.1 Closure


```python
from itertools import islice


def sort_priority(values, group):
    def helper(x):
        if x in group:
            return 0, x
        return 1, x

    values.sort(key=helper)
    
def sort_priority_v2(values, group):
    found = [False]  # 位于此作用于的列表、元组、字典可以被内部函数搜索并修改

    def helper(x):
        # python3中可以使用nonlocal来在此处声明found，以表示查找并修改外层作用域的found
        if x in group:
            found[0] = True
        if found:
            return 0, x
        return 1, x

    values.sort(key=helper)
    return found
    
def do_sort_proority():
    nums = [2, 5, 6, 5, 4, 7, 45, 54, 8]
    group = {2, 4, 45, 8}
    found = sort_priority_v2(nums, group)
    print nums
    print found
```


### 2.2 Generator


```python
def get_first_letter(text):
    if text:
        yield 0
    for index, letter in enumerate(text):
        if letter == ' ':
            yield index + 1


def do_get_first_letter():
    # for index in get_first_letter('Niko Belic Is A Good Boy'):
    #     print index

    # print list(get_first_letter('Niko Belic Is A Good Boy'))

    index_iterator = get_first_letter('Niko Belic Is A Good Boy')
    print list(islice(index_iterator, 0, 3))


```


### 2.3 Iterate In Params


```python
def normalize(numbers):
    total = sum(numbers)  # return a new iterator
    result = []

    for value in numbers:  # return a new iterator
        percent = 100 * value / total
        result.append(percent)
    return result


class ReadVisits:
    def __init__(self, numbers):
        self.numbers = numbers

    def __iter__(self):
        for value in self.numbers:
            yield value


def do_nomalize():
    numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    visits = ReadVisits(numbers)
    print iter(visits) is iter(visits)
```


## 3 Property

### 3.1 Using property instead of get and set


```python

class Register(object):
    def __init__(self, name, age):
        self._name = name
        self.age = age
        self.logo = self._name + "," + str(self.age)

    @property
    def name(self):
        return self._name

    @name.setter
    def name(self, name):
        """You can do something more than just setting property in this method"""
        self._name = name  # Do not use self.name = name, it will raise max recursive exception
        self.logo = self._name + "," + str(self.age)


if __name__ == '__main__':
    r = Register('Niko', 18)
    r.name = 'Belic'
    print r.logo

```



## 4 Concurrency

### 4.1 用subprocess模块来管理子进程


```python

import subprocess
import threading
from Queue import Queue
from time import time, sleep

from concurrent.futures import ProcessPoolExecutor, ThreadPoolExecutor


def sub_process_test1():
    """
    使用Popen构造器来启动进程
    communicate方法读取紫禁城的输出信息，并等待其终止
    """
    proc = subprocess.Popen(['echo', 'Hello NikoBelic'], stdout=subprocess.PIPE)
    out, err = proc.communicate()
    print out.decode('utf-8')


def sub_process_test2():
    """ 使用poll查询子进程状态 """
    proc = subprocess.Popen(['sleep', '0.1'])
    while proc.poll() is None:
        print 'Working'
    print 'Exit status', proc.poll()


def run_sleep(period):
    proc = subprocess.Popen(['sleep', str(period)])
    return proc


def sub_process_test3():
    """ 测试并行的性能，10个单个耗时1秒钟的任务，串行需要10秒，并行只需要1秒"""
    start = time()
    procs = []
    for _ in range(10):
        proc = run_sleep(1)
        procs.append(proc)

    for proc in procs:
        proc.communicate()
    end = time()
    print 'Finished in %.3f seconds' % (end - start)

```
### 4.2 可以用现成来执行阻塞式IO，但不要用它做平行计算


```python

def factorize(number):
    for i in range(1, number + 1):
        if number % i == 0:
            yield i


def do_factorize():
    numbers = [12431241, 42352452, 4574543, 1231224]
    start = time()
    for number in numbers:
        list(factorize(number))
    end = time()
    print 'It tooke %.3f seconds' % (end - start)


def parallel_do_factorize():
    """ 改成并行模式发现时间还长了，因为是Python的GIL缘故，导致CPU不能真正的并行 """
    numbers = [12431241, 42352452, 4574543, 1231224]
    start = time()
    threads = []

    def run(number):
        list(factorize(number))

    for number in numbers:
        thread = threading.Thread(target=run, args=(number,))
        thread.start()
        threads.append(thread)
    for thread in threads:
        thread.join()
    end = time()
    print 'It tooke %.3f seconds' % (end - start)

```

### 4.3 在线程中使用Lock来防止数据竞争


```python

class Counter(object):
    def __init__(self):
        self.count = 0

    def increment(self, offset):
        """ A += B 会拆分三步进行计算，在多线程切换的情况下，会出现新值被旧值替换的问题 """
        self.count += offset


def worker(how_many, counter, split_num):
    for _ in range(how_many / split_num):
        counter.increment(1)


def run_threads(func, how_many, counter):
    threads = []
    split_num = 5
    for _ in range(split_num):
        args = (how_many, counter, split_num)
        thread = threading.Thread(target=func, args=args)
        thread.start()
        threads.append(thread)
    for thread in threads:
        thread.join()


def do_incr():
    how_many = 10 ** 5
    # counter = Counter()
    counter = LockCounter()
    run_threads(worker, how_many, counter)
    print 'should be %d , found %d' % (how_many, counter.count)
    # should be 100000 , found 67760


class LockCounter(object):
    def __init__(self):
        self.lock = threading.Lock()
        self.count = 0

    def increment(self, offset):
        with self.lock:
            self.count += offset
```

### 4.4 用Queue来协调各线程之间的工作


```python

from collections import deque, namedtuple


class MyQueue(object):
    """ 自定义队列模型 """

    def __init__(self):
        self.items = deque()
        self.lock = threading.Lock()

    def put(self, item):
        with self.lock:
            self.items.append(item)

    def get(self):
        with self.lock:
            return self.items.popleft()


class Worker(threading.Thread):
    """ 消费者线程 """

    def __init__(self, func, in_queue, out_queue):
        super(Worker, self).__init__()
        self.func = func
        self.in_queue = in_queue
        self.out_queue = out_queue
        self.polled_count = 0
        self.work_done = 0

    def run(self):
        while True:
            self.polled_count += 1
            try:
                self.in_queue.get()
            except IndexError:
                sleep(0.01)  # No work to do
            else:
                result = self.func()
                self.out_queue.put(result)
                self.work_done += 1


def download():
    print '正在执行 download 任务...'


def resize():
    print '正在执行 resize 任务 ...'


def upload():
    print '正在执行 upload 任务...'


def do_work_with_my_queue():
    """
    这种消费者生产者模型有4个缺点：
    1. 当消费者无法从队列中获取数据时，会导致CPU空转，浪费资源
    2. Worker的run方法会一直执行其循环，即使应该退出了也不会停止
    3. 为了判断所有任务是否已经处理完毕，需要一直循环检查
    4. 如果某个步骤停滞，将导致内存溢出、程序崩溃
    """
    download_queue = MyQueue()
    resize_queue = MyQueue()
    upload_queue = MyQueue()
    done_queue = MyQueue()

    threads = [
        Worker(download, download_queue, resize_queue),
        Worker(resize, resize_queue, upload_queue),
        Worker(upload, upload_queue, done_queue)
    ]

    for thread in threads:
        thread.start()

    download_num = 1000
    for _ in range(download_num):
        download_queue.put(object())

    while len(done_queue.items) < download_num:
        continue
    processed = len(done_queue.items)
    polled = sum(t.polled_count for t in threads)
    print 'Processed', processed, 'items after polling', polled, 'times'
    # Processed 1000 items after polling 3013 times


def test_queue():
    queue = Queue(1)

    def consumer():
        print 'Consumer waiting'
        queue.get()  # will blocked if queue is empty
        print 'Consumer done'

    def producer():
        print 'Producer putting'
        queue.put(object())  # will blocked if queue is full

        print 'Producer done'

    for _ in range(10):
        threading.Thread(target=producer).start()

    print 'Will consume the items'
    for _ in range(10):
        threading.Thread(target=consumer).start()

    queue.task_done()


class ClosableQueue(Queue):
    """ 使用Queue来解决以上问题 """
    SENTINEL = object()

    def close(self):
        self.put(self.SENTINEL)

    def __iter__(self):
        while True:
            item = self.get()

            try:
                if item is self.SENTINEL:
                    return
                yield item
            finally:
                self.task_done()


class ConsumerThread(threading.Thread):
    def __init__(self, queue):
        super(ConsumerThread, self).__init__()
        self.queue = queue

    def run(self):
        while True:
            self.queue.get()
            print threading.currentThread().getName() + " 消费了 "
            sleep(1)


class ProducerThread(threading.Thread):
    def __init__(self, queue):
        super(ProducerThread, self).__init__()
        self.queue = queue

    def run(self):
        while True:
            self.queue.put('1')
            print threading.currentThread().getName() + " 生产了 "
            sleep(1)


def do_consume_produce():
    queue = ClosableQueue()

    ProducerThread(queue).start()
    ConsumerThread(queue).start()
```

### 4.5 考虑用携程来并发地运行多个函数


```python

def my_coroutine():
    while True:
        received = yield
        print 'received', received


def do_test_coroutine():
    it = my_coroutine()
    next(it)
    it.send('First')
    it.send('Second')


def minimize():
    current = yield
    while True:
        value = yield current  # 实际上这里会拆分为两步： 返回current，等待下一个send参数
        current = min(value, current)


def do_test_minize():
    it = minimize()
    next(it)

    print it.send(10)
    print it.send(4)
    print it.send(22)
    print it.send(-1)


```

### 4.6 考虑用concurrent.futures来实现真正的平行计算


```python

def gcd(pair):
    a, b = pair
    low = min(a, b)
    for i in range(low, 0, -1):
        if a % i == 0 and b % i == 0:
            return i


def do_calc_gcd():
    numbers = [(4325435, 3453463), (756756734, 43263546), (4564565, 234523546), (46575467, 23421345)]
    # numbers = [(4325435, 3453463)]
    start = time()
    # results = list(map(gcd, numbers))  # 单线程 7s
    # pool = ThreadPoolExecutor(max_workers=4) # 线程池方法 12s
    pool = ProcessPoolExecutor(max_workers=4)  # 进程池方法 4s
    result = list(pool.map(gcd, numbers))
    end = time()
    print 'Took %.3f seconds' % (end - start)



```

