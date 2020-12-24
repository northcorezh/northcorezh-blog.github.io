# Queue实战案例



## 基础参数



Queue.qsize()

​	返回队列的大致大小。注意，qsize() > 0 不保证后续的 get() 不被阻塞，qsize() < maxsize 也不保证 put() 不被阻塞。



Queue.empty()

​	如果队列为空，返回 `True` ，否则返回 `False` 。如果 empty() 返回 `True` ，不保证后续调用的 put() 不被阻塞。类似的，如果 empty() 返回 `False` ，也不保证后续调用的 get() 不被阻塞。



Queue.put(item, block=True, timeout=None)

​	将 *item* 放入队列。如果可选参数 *block* 是 true 并且 *timeout* 是 `None` (默认)，则在必要时阻塞至有空闲插槽可用。如果 *timeout* 是个正数，将最多阻塞 *timeout* 秒，如果在这段时间没有可用的空闲插槽，将引发 [`Full`](https://docs.python.org/zh-cn/3/library/queue.html#queue.Full) 异常。反之 (*block* 是 false)，如果空闲插槽立即可用，则把 *item* 放入队列，否则引发 [`Full`](https://docs.python.org/zh-cn/3/library/queue.html#queue.Full) 异常 ( 在这种情况下，*timeout* 将被忽略)



Queue.get()

​	从队列中移除并返回一个项目。如果可选参数 *block* 是 true 并且 *timeout* 是 `None` (默认值)，则在必要时阻塞至项目可得到。如果 *timeout* 是个正数，将最多阻塞 *timeout* 秒，如果在这段时间内项目不能得到，将引发 [`Empty`](https://docs.python.org/zh-cn/3/library/queue.html#queue.Empty) 异常。反之 (*block* 是 false) , 如果一个项目立即可得到，则返回一个项目，否则引发 [`Empty`](https://docs.python.org/zh-cn/3/library/queue.html#queue.Empty) 异常 (这种情况下，*timeout* 将被忽略)。

​	POSIX系统3.0之前，以及所有版本的Windows系统中，如果 *block* 是 true 并且 *timeout* 是 `None` ， 这个操作将进入基础锁的不间断等待。这意味着，没有异常能发生，尤其是 SIGINT 将不会触发 [`KeyboardInterrupt`](https://docs.python.org/zh-cn/3/library/exceptions.html#KeyboardInterrupt) 异常。



Queue.join()

​	阻塞至队列中所有的元素都被接收和处理完毕。当条目添加到队列的时候，未完成任务的计数就会增加。每当消费者线程调用 [`task_done()`](https://docs.python.org/zh-cn/3/library/queue.html#queue.Queue.task_done) 表示这个条目已经	被回收，该条目所有工作已经完成，未完成计数	就会减少。当未完成计数降到零的时候， [`join()`](https://docs.python.org/zh-cn/3/library/queue.html#queue.Queue.join) 阻塞被解除。



### 案例一

```python
import threading
import time
from queue import Queue

# 队列数为1
_tasklist = Queue(1)
def queue_worker():
    while True:
        if _tasklist.empty():
            print('---- empty> ')
        # 获取队列数据
        data = _tasklist.get()
        if data['status'] != 0:
            print(data)

# 使用线程去处理队列任务
t1 = threading.Thread(target=queue_worker, daemon=True)
t1.start()  
q.put({'status': 0})
q.put({'status': 1, 'aaa': 'bbbb'})

time.sleep(5)
print('-------master ')
```









参考: https://docs.python.org/zh-cn/3/library/queue.html#queue.Queue