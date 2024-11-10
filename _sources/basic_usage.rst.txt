基本用法
=============

**Basic Usage**

.. tab:: 中文

    如果您是第一次使用 Eventlet，可以参考 :ref:`design-patterns` 文档中的示例，这些示例是很好的入门资源。

    Eventlet 是围绕绿色线程（即协程，我们可以互换使用这两个术语）概念构建的，用于执行与网络相关的工作。绿色线程与普通线程有以下两个主要区别：

    * 绿色线程非常轻量，几乎可以认为是免费的。您不需要像普通线程那样节约使用绿色线程。一般来说，每个网络连接至少会有一个绿色线程。
    * 绿色线程通过协作方式相互让步，而不是被抢占式调度。这种行为的主要优势在于共享数据结构不需要锁，因为只有显式调用让步时，其他绿色线程才能访问该数据结构。此外，还可以检查队列等原语，查看它们是否有待处理的数据。

.. tab:: 英文

    If it's your first time to Eventlet, you may find the illuminated examples in the :ref:`design-patterns` document to be a good starting point.

    Eventlet is built around the concept of green threads (i.e. coroutines, we use the terms interchangeably) that are launched to do network-related work.  Green threads differ from normal threads in two main ways:

    * Green threads are so cheap they are nearly free.  You do not have to conserve green threads like you would normal threads.  In general, there will be at least one green thread per network connection.
    * Green threads cooperatively yield to each other instead of preemptively being scheduled.  The major advantage from this behavior is that shared data structures don't need locks, because only if a yield is explicitly called can another green thread have access to the data structure.  It is also possible to inspect primitives such as queues to see if they have any pending data.

主要 API
---------------

**Primary API**

.. tab:: 中文

    Eventlet API 的设计目标是简洁和可读性。您应该能够阅读它的代码并理解其执行过程。我们偏向于使用较少的代码行，而不是过于巧妙的实现。 `就像 Python 本身 <http://www.python.org/dev/peps/pep-0020/>`_ ，在 Eventlet 中应该有一种且唯一明显的实现方式！

    尽管 Eventlet 包含许多模块，但大多数常用功能可以简单地通过 ``import eventlet`` 访问。以下是 ``eventlet`` 模块中可用功能的简要概述，并提供了每个功能的详细文档链接。

.. tab:: 英文

    The design goal for Eventlet's API is simplicity and readability.  You should be able to read its code and understand what it's doing.  Fewer lines of code are preferred over excessively clever implementations.  `Like Python itself <http://www.python.org/dev/peps/pep-0020/>`_, there should be one, and only one obvious way to do it in Eventlet!

    Though Eventlet has many modules, much of the most-used stuff is accessible simply by doing ``import eventlet``.  Here's a quick summary of the functionality available in the ``eventlet`` module, with links to more verbose documentation on each.

Greenthread 生成
+++++++++++++++++

**Greenthread Spawn**

.. tab:: 中文

    .. function:: eventlet.spawn(func, *args, **kw)
    
       启动一个绿色线程来调用 *func*。通过并行启动多个绿色线程，可以实现并行处理任务。``spawn`` 的返回值是一个 :class:`greenthread.GreenThread` 对象，可以用来获取 *func* 的返回值。更多细节请参见 :func:`spawn <eventlet.greenthread.spawn>`。
    
    .. function:: eventlet.spawn_n(func, *args, **kw)
    
       与 :func:`spawn` 相同，但无法得知函数的终止方式（即无返回值或异常）。这种方式执行速度更快。更多细节请参见 :func:`spawn_n <eventlet.greenthread.spawn_n>`。

    .. function:: eventlet.spawn_after(seconds, func, *args, **kw)
    
       在经过 *seconds* 秒后启动 *func*；这是 :func:`spawn` 的延迟版本。要中止启动并防止调用 *func*，可以对 :func:`spawn_after` 的返回值调用 :meth:`greenthread.GreenThread.cancel`。更多细节请参见 :func:`spawn_after <eventlet.greenthread.spawn_after>`。

.. tab:: 英文

    .. function:: eventlet.spawn(func, *args, **kw)
       :no-index:
    
       This launches a greenthread to call *func*.  Spawning off multiple greenthreads gets work done in parallel.  The return value from ``spawn`` is a :class:`greenthread.GreenThread` object, which can be used to retrieve the return value of *func*.  See :func:`spawn <eventlet.greenthread.spawn>` for more details.
    
    .. function:: eventlet.spawn_n(func, *args, **kw)
       :no-index:
    
       The same as :func:`spawn`, but it's not possible to know how the function terminated (i.e. no return value or exceptions).  This makes execution faster.  See :func:`spawn_n <eventlet.greenthread.spawn_n>` for more details.

    .. function:: eventlet.spawn_after(seconds, func, *args, **kw)
       :no-index:
    
       Spawns *func* after *seconds* have elapsed; a delayed version of :func:`spawn`.   To abort the spawn and prevent *func* from being called, call :meth:`eventlet.greenthread.GreenThread.cancel` on the return value of :func:`spawn_after`.  See :func:`spawn_after <eventlet.greenthread.spawn_after>` for more details.

Greenthread 控制
+++++++++++++++++

**Greenthread Control**

.. tab:: 中文

    .. function:: eventlet.sleep(seconds=0)

        暂停当前绿色线程，允许其他线程有机会执行。更多细节请参见 :func:`sleep <eventlet.greenthread.sleep>`。

    .. class:: eventlet.GreenPool

        池用于控制并发性。在应用程序中，通常希望仅消耗有限的内存，或者限制某段代码保持的连接数，以便为其他部分留出更多资源，或在面对不可预测的输入数据时保持一致性。GreenPool 提供了这种控制。有关如何使用这些的更多信息，请参见 :class:`GreenPool <eventlet.greenpool.GreenPool>`。

    .. class:: eventlet.GreenPile

        GreenPile 对象表示一组工作任务。本质上，GreenPile 是一个可以填充任务的迭代器，稍后可以读取其结果。更多细节请参见 :class:`GreenPile <eventlet.greenpool.GreenPile>`。

    .. class:: eventlet.Queue

        队列是用于在执行单元之间传递数据的基本构造。Eventlet 的 Queue 类用于在绿色线程之间通信，并提供了一些有用的功能来实现这一点。更多细节请参见 :class:`Queue <eventlet.queue.Queue>`。

    .. class:: eventlet.Timeout

        该类用于向任何操作添加超时功能。它会在 *timeout* 秒后在当前绿色线程中引发 *exception*。当 *exception* 省略或为 ``None`` 时，将引发 Timeout 实例本身。

        Timeout 对象是上下文管理器，因此可以在 with 语句中使用。更多细节请参见 :class:`Timeout <eventlet.timeout.Timeout>`。

.. tab:: 英文

    .. function:: eventlet.sleep(seconds=0)
        :no-index:

        Suspends the current greenthread and allows others a chance to process.  See :func:`sleep <eventlet.greenthread.sleep>` for more details.

    .. class:: eventlet.GreenPool
        :no-index:

        Pools control concurrency.  It's very common in applications to want to consume only a finite amount of memory, or to restrict the amount of connections that one part of the code holds open so as to leave more for the rest, or to behave consistently in the face of unpredictable input data.  GreenPools provide this control.  See :class:`GreenPool <eventlet.greenpool.GreenPool>` for more on how to use these.

    .. class:: eventlet.GreenPile
        :no-index:

        GreenPile objects represent chunks of work.  In essence a GreenPile is an iterator that can be stuffed with work, and the results read out later. See :class:`GreenPile <eventlet.greenpool.GreenPile>` for more details.
        
    .. class:: eventlet.Queue
        :no-index:

        Queues are a fundamental construct for communicating data between execution units.  Eventlet's Queue class is used to communicate between greenthreads, and provides a bunch of useful features for doing that.  See :class:`Queue <eventlet.queue.Queue>` for more details.
        
    .. class:: eventlet.Timeout
        :no-index:

        This class is a way to add timeouts to anything.  It raises *exception* in the current greenthread after *timeout* seconds.  When *exception* is omitted or ``None``, the Timeout instance itself is raised.
        
        Timeout objects are context managers, and so can be used in with statements.
        See :class:`Timeout <eventlet.timeout.Timeout>` for more details.

Patching 函数
+++++++++++++++++

**Patching Functions**

.. tab:: 中文

    .. function:: eventlet.import_patched(modulename, *additional_modules, **kw_additional_modules)

        以确保模块使用标准库模块的“绿色(green)”版本的方式导入模块，从而使所有操作都能够非阻塞地工作。唯一必需的参数是要导入的模块名称。更多信息请参见 :ref:`import-green`。

    .. function:: eventlet.monkey_patch(all=True, os=False, select=False, socket=False, thread=False, time=False)

        全局修补某些系统模块，使其对绿色线程友好。关键字参数允许控制哪些模块被修补。如果 *all* 为 True，则会忽略其他参数，修补所有模块。如果为 False，则其他关键字参数控制标准库特定子模块的修补。大多数参数只修补同名的单个模块（如 os、time、select）。例外情况有 socket，若存在，还会修补 ssl 模块；以及 thread，会修补 thread、threading 和 Queue 模块。多次调用 monkey_patch 是安全的。更多信息请参见 :ref:`monkey-patch`。

.. tab:: 英文
    
    .. function:: eventlet.import_patched(modulename, *additional_modules, **kw_additional_modules)
        :no-index:

        Imports a module in a way that ensures that the module uses "green" versions of the standard library modules, so that everything works nonblockingly.  The only required argument is the name of the module to be imported.  For more information see :ref:`import-green`.

    .. function:: eventlet.monkey_patch(all=True, os=False, select=False, socket=False, thread=False, time=False)
        :no-index:

        Globally patches certain system modules to be greenthread-friendly. The keyword arguments afford some control over which modules are patched. If *all* is True, then all modules are patched regardless of the other arguments. If it's False, then the rest of the keyword arguments control patching of specific subsections of the standard library.  Most patch the single module of the same name (os, time, select).  The exceptions are socket, which also patches the ssl module if present; and thread, which patches thread, threading, and Queue.  It's safe to call monkey_patch multiple times.  For more information see :ref:`monkey-patch`.

网络便利函数
+++++++++++++++++

**Network Convenience Functions**

.. tab:: 中文

    .. autofunction:: eventlet.connect

    .. autofunction:: eventlet.listen

    .. autofunction:: eventlet.wrap_ssl

    .. autofunction:: eventlet.serve

    .. autoclass:: eventlet.StopServe
        
    这些是 Eventlet 的基本原语；其他 Eventlet 模块中还有更多内容；请查看:doc:`modules` 。    

.. tab:: 英文

    .. autofunction:: eventlet.connect
        :no-index:

    .. autofunction:: eventlet.listen
        :no-index:

    .. autofunction:: eventlet.wrap_ssl
        :no-index:

    .. autofunction:: eventlet.serve
        :no-index:

    .. autoclass:: eventlet.StopServe
        :no-index:
        
    These are the basic primitives of Eventlet; there are a lot more out there in the other Eventlet modules; check out the :doc:`modules`.
