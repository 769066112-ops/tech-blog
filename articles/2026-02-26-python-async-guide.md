---
title: Python异步编程完全指南：从入门到实战（2026最新版）
tags: Python, 异步编程, asyncio, 并发, 教程
category: Python
description: 全面讲解Python异步编程，包括asyncio、async/await、协程、事件循环等核心概念，附带实战案例
---

# Python异步编程完全指南：从入门到实战（2026最新版）

## 前言

在现代Python开发中，异步编程已经成为处理I/O密集型任务的标配。无论是Web开发、爬虫、还是微服务，掌握异步编程都能让你的程序性能提升数倍。

本文将从零开始，带你彻底搞懂Python异步编程。

## 1. 为什么需要异步编程？

### 同步 vs 异步

```python
# 同步方式：依次执行，总耗时 = 所有任务之和
import time

def fetch_data(url):
    time.sleep(2)  # 模拟网络请求
    return f"Data from {url}"

start = time.time()
results = [fetch_data(f"https://api.example.com/{i}") for i in range(5)]
print(f"同步耗时: {time.time() - start:.1f}s")  # ~10s
```

```python
# 异步方式：并发执行，总耗时 ≈ 最慢的那个任务
import asyncio

async def fetch_data(url):
    await asyncio.sleep(2)  # 模拟网络请求
    return f"Data from {url}"

async def main():
    start = asyncio.get_event_loop().time()
    tasks = [fetch_data(f"https://api.example.com/{i}") for i in range(5)]
    results = await asyncio.gather(*tasks)
    elapsed = asyncio.get_event_loop().time() - start
    print(f"异步耗时: {elapsed:.1f}s")  # ~2s

asyncio.run(main())
```

**5倍性能提升**，这就是异步的威力。

## 2. 核心概念

### 2.1 协程（Coroutine）

协程是异步编程的基本单位，用 `async def` 定义：

```python
async def hello():
    print("Hello")
    await asyncio.sleep(1)
    print("World")
```

### 2.2 事件循环（Event Loop）

事件循环是异步程序的调度中心，负责管理和执行所有协程：

```python
# Python 3.10+ 推荐写法
asyncio.run(main())

# 或者手动管理
loop = asyncio.get_event_loop()
loop.run_until_complete(main())
```

### 2.3 Task 和 Future

```python
async def main():
    # 创建任务，立即开始执行
    task1 = asyncio.create_task(fetch_data("url1"))
    task2 = asyncio.create_task(fetch_data("url2"))
    
    # 等待所有任务完成
    result1 = await task1
    result2 = await task2
```

## 3. 实战：异步HTTP请求

```python
import aiohttp
import asyncio

async def fetch(session, url):
    async with session.get(url) as response:
        return await response.json()

async def main():
    urls = [
        "https://api.github.com/users/python",
        "https://api.github.com/users/django",
        "https://api.github.com/users/flask",
    ]
    
    async with aiohttp.ClientSession() as session:
        tasks = [fetch(session, url) for url in urls]
        results = await asyncio.gather(*tasks)
        
        for result in results:
            print(f"{result['login']}: {result['public_repos']} repos")

asyncio.run(main())
```

## 4. 实战：异步数据库操作

```python
import asyncpg

async def main():
    conn = await asyncpg.connect(
        host='localhost', database='mydb',
        user='user', password='pass'
    )
    
    # 批量插入
    records = [(f"user_{i}", f"user_{i}@example.com") for i in range(1000)]
    await conn.executemany(
        "INSERT INTO users(name, email) VALUES($1, $2)",
        records
    )
    
    # 并发查询
    async with conn.transaction():
        results = await conn.fetch("SELECT * FROM users LIMIT 10")
        for row in results:
            print(dict(row))
    
    await conn.close()

asyncio.run(main())
```

## 5. 异步编程最佳实践

### 5.1 错误处理

```python
async def safe_fetch(url):
    try:
        async with aiohttp.ClientSession() as session:
            async with session.get(url, timeout=aiohttp.ClientTimeout(total=10)) as resp:
                return await resp.json()
    except asyncio.TimeoutError:
        print(f"请求超时: {url}")
        return None
    except aiohttp.ClientError as e:
        print(f"请求失败: {e}")
        return None
```

### 5.2 限制并发数

```python
async def fetch_with_limit(urls, max_concurrent=10):
    semaphore = asyncio.Semaphore(max_concurrent)
    
    async def limited_fetch(url):
        async with semaphore:
            return await fetch(url)
    
    return await asyncio.gather(*[limited_fetch(url) for url in urls])
```

### 5.3 异步上下文管理器

```python
class AsyncDBPool:
    async def __aenter__(self):
        self.pool = await asyncpg.create_pool(dsn="postgresql://...")
        return self.pool
    
    async def __aexit__(self, *args):
        await self.pool.close()

async def main():
    async with AsyncDBPool() as pool:
        async with pool.acquire() as conn:
            result = await conn.fetch("SELECT 1")
```

## 6. 常见陷阱

### 陷阱1：忘记 await

```python
# ❌ 错误：没有 await，协程不会执行
async def main():
    fetch_data("url")  # 这只是创建了协程对象

# ✅ 正确
async def main():
    await fetch_data("url")
```

### 陷阱2：在异步函数中使用同步阻塞

```python
# ❌ 错误：time.sleep 会阻塞整个事件循环
async def bad():
    time.sleep(5)

# ✅ 正确：用 asyncio.sleep
async def good():
    await asyncio.sleep(5)

# ✅ 如果必须用同步代码，放到线程池
async def run_sync():
    loop = asyncio.get_event_loop()
    result = await loop.run_in_executor(None, blocking_function)
```

## 总结

| 概念 | 说明 |
|------|------|
| `async def` | 定义协程函数 |
| `await` | 等待协程完成 |
| `asyncio.gather()` | 并发执行多个协程 |
| `asyncio.create_task()` | 创建任务 |
| `asyncio.Semaphore` | 限制并发数 |
| `asyncio.run()` | 运行异步程序入口 |

掌握这些，你就能在实际项目中游刃有余地使用异步编程了。

---

> 如果这篇文章对你有帮助，请点赞收藏关注，后续会持续更新更多Python实战教程。
