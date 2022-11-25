# Python利用singleflight解决缓存穿透问题

---

#### 通常获取缓存的方法

```python
data = cache.get(key)
if not data:
    data = get_data()
    cache.set(key, data)
return data
```

### 弊端

 在这种模式下，在高并发的场景下，很有可能出现缓存击穿的问题。

### 解决方案

1. **加锁**

    ```python 
    data = cache.get(key)
    if not data:
    	# 加锁
    	lock.lock()
        data = get_data()
        cache.set(key, data)
        # 释放锁
        lock.release()
    return data
    ```

这种加锁方案是不行的。这种方案下，虽热避免了并发地加载数据，保证数据库不会被打死，但依然没有解决重复操作的问题。而且这种方案下，大量请求串行化操作，会加大系统延迟。

2. **try-lock模式**

```python
data = cache.get(key)  #step1
if not data:
  locked = lock.try_lock()  #非阻塞
  if locked:
    data = load_data()   #step2
    cache.set(key,data)	 #step3
    lock.release()
  else:
    time.sleep(1)
    data = cache.get(key)
return data

```

这种方案，基本上已经能够满足我们的需求，只有获取到锁的线程会去加载数据，其他线程会休眠一段时间后，再次尝试从缓存中获取数据。但这里还是有两个细节需要注意，一是线程休眠是时间，太短的话可能新的数据还没加载到缓存，太长的话会影响性能。二是锁的粒度，锁的粒度如果太大，比如获取某条订单数据的时候，如果所有的订单都用同一把锁，还是会影响系统的性能，比较合理地方式应该是为每个订单创建一把锁。这里可以考虑使用redis实现分布式锁，用订单号作为分布式锁的key。

3. **singlefight**

    ```python
    from threading import Condition, Lock
    
    class Caller:
        def __init__(self):
            self._cond = Condition()
            self.val = None
            self.err = None
            self.done = False
    
        def result(self):
            with self._cond:
                while not self.done:
                    # 阻塞等待執行結果
                    self._cond.wait()
    
            if self.err:
                raise self.err
    
            return self.val
    
        def notify(self):
            with self._cond:
                self.done = True
                # 通知所有的阻塞線程，執行已經完成
                self._cond.notify_all()
    
    class SingleFlight:
        def __init__(self):
            self.map = {}
            self.lock = Lock()
    
        def do(self, key, fn, *args, **kwargs):
            # 獲取鎖
            self.lock.acquire()
            if key in self.map:
                # 已經存在對key的請求 ，則新線程不會重複處理key的請求釋放鎖,然後阻塞等待請求得到的結果
                caller = self.map[key]
                self.lock.release()
                return caller.result()
            caller = Caller()
            self.map[key] = caller
            self.lock.release()
    
            try:
                caller.val = fn(*args, **kwargs)
            except Exception as e:
                caller.err = e
            finally:
                # 通知該key下的阻塞線程，執行已經完成，可以獲取結果了
                caller.notify()
    
            self.lock.acquire()
            del self.map[key]
            self.lock.release()
    
            # 返回執行結果
            return caller.result()
    
    
    # 留作測試
    if __name__ == "__main__":
        import time
        import random
        from concurrent.futures import ThreadPoolExecutor, as_completed
    
        # 實例化線程
        exector = ThreadPoolExecutor(max_workers=20)
        single_flight = SingleFlight()
    
        def long_task(delay=1):
            print(f"this is long task")
            time.sleep(delay)
            return random.uniform(1, 100)
    
        def run_in_single_flight():
            return single_flight.do('long_task', long_task, delay=0.1)
    
        tasks = []
    
        for i in range(10):
            task = exector.submit(run_in_single_flight)
            tasks.append(task)
    
        for task in as_completed(tasks):
            data = task.result()
            print(f"data is {data}")
    
    ```

    