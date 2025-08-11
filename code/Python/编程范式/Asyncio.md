asyncio是python内置的标准库，用于支持异步io。

异步操作是指，调用者不会立马得到返回，而是调用发出后立刻执行其他操作。被调用者完成操作后在通知调用者。这样的好处在于耗时的操作将不会阻塞当前线程。

异步操作有三个重要的概念：

- 协程（Coroutine）
- 事件循环（Event Loop）
- 任务（Task）

### 协程
我们可以通过async def关键字定义一个协程函数。

```python
import asyncio

async def say_hello():
    print("Hello")
    await asyncio.sleep(1)
    print("World")
```

协程函数是可以暂停和恢复执行的函数。它在执行时不会运行函数内部的代码，而是先返回一个协程对象。当协程对象获得控制权时才会执行内部代码，当执行完毕后会释放控制权给事件循环。协程函数通过await关键字暂停执行，等待异步操作完成。

### 事件循环
顾名思义，它是一个循环。事件循环循环做三件事情：。

- 检查是否有可以运行的协程对象。
- 如果有的话，将控制权转移给协程对象。
- 等待协程对象释放控制权。

### 任务
任务是对协程对象的封装。除了包括协程对象外，还包含了协程对象的状态信息，比如正在等待、正在执行、已完成等等... 主要用于让事件循环检查协程是否可以运行。

## 使用asyncio
使用asyncio只需要做三件事情:

- 定义协程函数
- 包装协程为任务
- 建立事件循环

```python
import asyncio
from time import sleep

# 定义协程函数
async def say_hello():
    print("Hello")
    await asyncio.sleep(1)
    print("World")
    
async def say_hi():
    print("Hi")
    await asyncio.sleep(1)
    print("async")


async def main():
    # 使用asyncio.create_task包装为tasks
    tasks = [asyncio.create_task(coro) for coro in [
        say_hello(),
        say_hi(),
    ]]
    
    await tasks
    
if __name__ = '__main__':
    # 使用asyncio.run创建事件循环
    asyncio.run(main())
```

注意：

- 同步函数不能直接调用异步函数。同步调用异步，需要使用asyncio.get_event_loop()获取当前事件循环，并使用asyncio.run_coroutine_threadsafe()方法将异步函数包装成一个可调用的线程安全的协程对象。然后，我们可以使用result()方法获取异步函数的返回值。但这样往往会丧失异步操作的优势。
- 异步函数可以调用同步函数，但要注意阻塞问题。
- 由于同步不能直接调用异步，一般异步应用的整个调用链都是异步的。