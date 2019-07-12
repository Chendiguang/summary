# Python开发的规范

函数定义等一些需要遵守的风格

* 函数定义

    ```bash
    # 函数名使用小写驼峰式，尽量用短命称来表明该函数的作用, 如果不行则

    # 这是python社区从3.6版本开始推荐的写法，主要是为了兼容静态语言的一些优点
    # 声明输入和输出类型并不会改变python编译器的行为,
    # 不过有助于与他人合作, 减少运行时的错误
    def greeting(name: str) -> str:
        return 'Hello ' + name

    # 在查阅各种开源库的源码发现, 都在逐步往这种风格迁移以防后期版本变动需要强制检查,
    # 例如tornado和各种内置库。这python之父Guido亲力推行的一个PEP

    # 其中可以使用mypy来进行检查
    $ pip3 install mypy
    $ mypy yourProgram

    # 强烈推荐一个内置库 typing, 里面内置了几乎所有你想要的数据类型

    from typing import Iterator

    def fib(n: int) -> Iterator[int]:
        a, b = 0, 1
        while a < n:
            yield a
            a, b = b, a + b
    ```

    关于mypy的更多用法请参考[mypy官方文档](https://mypy.readthedocs.io/en/latest/getting_started.html)

## 模块库

说明库的引入规则和使用的建议

---

* 库的引入

    ```bash
    # 遵循三层导入规则, 依次是内置库，第三方库，自己或团队定义的模块
    # 可以参考下面
    import os
    import sys

    import numpy as np  # thirdmodule

    import yourmodule
    ```

* 库的使用

    为了维护方便和生产安全尽量使用内置库;</br>
    并发库的使用:</br>
    1. 需要使用线程或进程并发编程时，尽量使用内置的[concurrent.futures](https://python3-cookbook.readthedocs.io/zh_CN/latest/c12/p08_perform_simple_parallel_programming.html)库

    2. 推荐使用[asyncio](https://docs.python.org/3/library/asyncio.html)和[iohttp](https://aiohttp.readthedocs.io/en/stable/)进行```网络编程```，因为这是最新引入的比较稳定协程库，</br> python之父亲自操刀，得到社区的鼎力支持，是python未来的主要方向，下面是一个简单的用法

        ```bash
        import asyncio
        import time

        async def say_after(delay, what):
            await asyncio.sleep(delay)
            print(what)

        async def main():
            print(f"started at {time.strftime('%X')}")

            await say_after(1, 'hello')
            await say_after(2, 'world')
        main()
        ```

    3. 在允许的情况下，不推荐使用threading，multiprocessing, 尽可能使用前面推荐使用的几个库

    装饰器使用</br>
    1. 在需要使用装饰器的情况，强烈推荐使用functools.wraps封装

        ```bash
        import time
        from functools import wraps

        def timethis(func):
            '''
            Decorator that reports the execution time.
            '''
            @wraps(func)
            def wrapper(*args, **kwargs):
                start = time.time()
                result = func(*args, **kwargs)
                end = time.time()
                print(func.__name__, end-start)
                return result
            return wrapper
        ```

    2. 更多用法和具体原因请参考[此文](https://python3-cookbook.readthedocs.io/zh_CN/latest/c09/p01_put_wrapper_around_function.html)

## 日志的使用

* 异常处理

    ```bash
        try:
            result = 5 / 0
        except Exception as e:
            logging.error('Error: %s', e)  # 1 
            logging.error('Error', exc_info=True) # 2
            logging.exception('Error')  # 3
            # 用法1不推荐, 推荐2或3, 这两种会展示详细的错误信息
    ```
