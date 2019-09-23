# Python性能调优

## 基础知识

* 类型占用内存

    这里使用sys.getsizeof()来查看基本类型对象占用的内存大小。特别的对比了python2和python3的区别

    ```python2```

    ```bash
    >>> import sys
    >>> sys.getsizeof(1)
    24
    >>> sys.getsizeof([])
    72
    >>> sys.getsizeof(())
    56
    >>> sys.getsizeof({})
    280
    >>> sys.getsizeof(True)
    24
    ```

    ```python3```

    ```bash
    >>> import sys
    >>> sys.getsizeof(1)
    28
    >>> sys.getsizeof([])
    64
    >>> sys.getsizeof([])
    64
    >>> sys.getsizeof({})
    240
    >>> sys.getsizeof(True)
    28
    ```

* 分析内存的工具

    主要有以下这几种：guppy、pysizer、pytracemalloc、objgraph
