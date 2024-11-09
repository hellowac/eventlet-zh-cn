线程
========

**Threads**

.. tab:: 中文

    Eventlet 是线程安全的，可以与普通的 Python 线程一起使用。其工作原理是协程被限制在其“父”Python 线程中。就像每个线程都包含自己的小世界，协程可以在自己内部进行切换，但不能在其他线程的协程之间切换。

    .. image:: /images/threading_illustration.png

    你只能使用“真实”的线程原语和管道进行跨线程通信。幸运的是，当你已经使用协程时，几乎没有理由再使用线程来实现并发。

    你大多数情况下使用线程的原因是为了包装一些不是“绿色”的操作，比如一个使用自己操作系统调用来执行套接字操作的 C 库。为了简化这些使用，提供了 :mod:`~eventlet.tpool` 模块。

.. tab:: 英文

    Eventlet is thread-safe and can be used in conjunction with normal Python threads.  The way this works is that coroutines are confined to their 'parent' Python thread.  It's like each thread contains its own little world of coroutines that can switch between themselves but not between coroutines in other threads.

    .. image:: /images/threading_illustration.png

    You can only communicate cross-thread using the "real" thread primitives and pipes.  Fortunately, there's little reason to use threads for concurrency when you're already using coroutines.

    The vast majority of the times you'll want to use threads are to wrap some operation that is not "green", such as a C library that uses its own OS calls to do socket operations.  The :mod:`~eventlet.tpool` module is provided to make these uses simpler.

Tpool - 简单线程池
---------------------------

**Tpool - Simple thread pool**

.. tab:: 中文

    使用 :mod:`~eventlet.tpool` 最简单的方式是通过 :func:`~eventlet.tpool.execute` 执行一个函数。该函数将在池中的一个随机线程中运行，而调用的协程则会阻塞，直到函数执行完成::

		>>> import thread
		>>> from eventlet import tpool
		>>> def my_func(starting_ident):
		...     print("running in new thread:", starting_ident != thread.get_ident())
		...
		>>> tpool.execute(my_func, thread.get_ident())
		running in new thread: True

    默认情况下，池中有 20 个线程，但你可以通过在导入 tpool 之前设置环境变量 ``EVENTLET_THREADPOOL_SIZE`` 来配置所需的池大小。

    .. automodule:: eventlet.tpool
        :members:

.. tab:: 英文

    The simplest thing to do with :mod:`~eventlet.tpool` is to :func:`~eventlet.tpool.execute` a function with it.  The function will be run in a random thread in the pool, while the calling coroutine blocks on its completion::

        >>> import thread
        >>> from eventlet import tpool
        >>> def my_func(starting_ident):
        ...     print("running in new thread:", starting_ident != thread.get_ident())
        ...
        >>> tpool.execute(my_func, thread.get_ident())
        running in new thread: True

    By default there are 20 threads in the pool, but you can configure this by setting the environment variable ``EVENTLET_THREADPOOL_SIZE`` to the desired pool size before importing tpool.

    .. automodule:: eventlet.tpool
        :members:
        :no-index:
