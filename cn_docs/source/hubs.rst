.. _understanding_hubs:

理解 Eventlet Hub
===========================

**Understanding Eventlet Hubs**

.. tab:: 中文

    一个 hub 构成了 Eventlet 事件循环的基础，负责分发 I/O 事件并调度 greenthreads。正是 hub 的存在，将协程（虽然可以编程，但比较复杂）转变为 greenthreads（易于使用）。

    Eventlet 有多个 hub 实现，当你开始使用它时，它会尝试选择最适合你系统的 hub 实现。它支持的 hub 依次为：

    **asyncio**  
        | 基于 Asyncio 的 hub。可以在 Asyncio 事件循环中运行 Eventlet 代码。  
        | 使用这个 hub，Asyncio 和 Eventlet 可以在同一线程、同一进程中运行。  
        | 我们不建议为新项目使用 Eventlet。  
        | 我们鼓励现有的 Eventlet 项目从 Eventlet 迁移到 Asyncio。  
        | 这个 hub 允许逐步、平滑地迁移。  
        | 详情请参阅 :ref:`migration-guide`。  
        | 当前的兼容性状态请参见 :ref:`asyncio-compatibility`。  
    **epolls**  
        Linux 系统。这是 Linux 上最快的 hub。  
    **kqueue**  
        FreeBSD 和 Mac OSX 系统。支持 kqueue 的操作系统上最快的 hub。  
    **poll**  
        在支持该功能的平台上使用。  
    **selects**  
        最低共同分母，适用于所有平台。

    唯一一个非纯 Python 的 pyevent hub（使用 libevent）已被移除，因为它没有得到维护。欢迎您贡献快速的 hub 实现，使用 Cython、CFFI 或其他您选择的技术。

    如果选择的 hub 不适合应用程序，可以选择另一个 hub。您可以通过环境变量 :ref:`EVENTLET_HUB <env_vars>` 或 :func:`eventlet.hubs.use_hub` 来进行选择。

    .. function:: eventlet.hubs.use_hub(hub=None)

        使用此方法来控制 Eventlet 选择的 hub。传递所需的 hub 模块的名称。确保在应用程序开始进行任何 I/O 操作之前调用它！调用 `use_hub` 会完全替换旧的 hub，任何它所管理的文件描述符或定时器都将被遗忘。将调用放在主模块的前几行。::

            """ 这是主模块 """
            import eventlet.hubs
            eventlet.hubs.use_hub("eventlet.hubs.epolls")

        Hubs 作为线程本地类实例实现。 :func:`eventlet.hubs.use_hub` 只对当前线程起作用。当使用多个线程时，每个线程需要自己的 hub，在每个需要特定 hub 的线程函数开始时调用 :func:`eventlet.hubs.use_hub`。在实践中，可能不需要在每个线程中指定 hub；可以在主线程使用一个特殊的 hub，其他线程使用默认的 hub；这种混合 hub 配置通常是可行的。

        也可以使用第三方的 hub 模块来替代内置的某个 hub。只需将模块本身传递给 :func:`eventlet.hubs.use_hub`。编写这样一个 hub 的任务超出了本文档的范围，最好直接查看现有 hubs 的代码，了解它们是如何工作的。::

            import eventlet.hubs
            import mypackage.myhub
            eventlet.hubs.use_hub(mypackage.myhub)

        将 `None` 作为参数传递给 :func:`eventlet.hubs.use_hub` 会选择默认的 hub。

.. tab:: 英文

    A hub forms the basis of Eventlet's event loop, which dispatches I/O events and schedules greenthreads.  It is the existence of the hub that promotes coroutines (which can be tricky to program with) into greenthreads (which are easy).

    Eventlet has multiple hub implementations, and when you start using it, it tries to select the best hub implementation for your system.  The hubs that it supports are (in order of preference):

    **asyncio**
        | Asyncio based hub. Run Eventlet code in an Asyncio eventloop.
        | By using this hub, Asyncio and Eventlet can be run the same thread in the same process.
        | We discourage new Eventlet projects.
        | We encourage existing Eventlet projects to migrate from Eventlet to Asyncio.
        | This hub allow you incremental and smooth migration.
        | See the :ref:`migration-guide` for further details.
        | See the :ref:`asyncio-compatibility` for the current state of the art.
    **epolls**
        Linux. This is the fastest hub for Linux.
    **kqueue**
        FreeBSD and Mac OSX. Fastest hub for OS with kqueue.
    **poll**
        On platforms that support it.
    **selects**
        Lowest-common-denominator, available everywhere.

    The only non-pure Python, pyevent hub (using libevent) was removed because it was not maintained. You are warmly welcome to contribute fast hub implementation using Cython, CFFI or other technology of your choice.

    If the selected hub is not ideal for the application, another can be selected.  You can make the selection either with the environment variable :ref:`EVENTLET_HUB <env_vars>`, or with :func:`eventlet.hubs.use_hub`.

    .. function:: eventlet.hubs.use_hub(hub=None)

        Use this to control which hub Eventlet selects.  Call it with the name of the desired hub module.  Make sure to do this before the application starts doing any I/O!  Calling use_hub completely eliminates the old hub, and any file descriptors or timers that it had been managing will be forgotten.  Put the call as one of the first lines in the main module.::

            """ This is the main module """
            import eventlet.hubs
            eventlet.hubs.use_hub("eventlet.hubs.epolls")

        Hubs are implemented as thread-local class instances.  :func:`eventlet.hubs.use_hub` only operates on the current thread.  When using multiple threads that each need their own hub, call :func:`eventlet.hubs.use_hub` at the beginning of each thread function that needs a specific hub.  In practice, it may not be necessary to specify a hub in each thread; it works to use one special hub for the main thread, and let other threads use the default hub; this hybrid hub configuration will work fine.

        It is also possible to use a third-party hub module in place of one of the built-in ones.  Simply pass the module itself to :func:`eventlet.hubs.use_hub`.  The task of writing such a hub is a little beyond the scope of this document, it's probably a good idea to simply inspect the code of the existing hubs to see how they work.::

            import eventlet.hubs
            import mypackage.myhub
            eventlet.hubs.use_hub(mypackage.myhub)

        Supplying None as the argument to :func:`eventlet.hubs.use_hub` causes it to select the default hub.


Hub 的工作原理
-----------------

**How the Hubs Work**

.. tab:: 中文

    hub 有一个主 greenlet，MAINLOOP。当其中一个正在运行的协程需要进行 I/O 时，它会向 hub 注册一个监听器（这样 hub 就知道何时唤醒它），然后切换到 MAINLOOP（通过 ``get_hub().switch()``）。如果有其他协程已经准备好运行，MAINLOOP 会切换到它们，当它们完成或需要更多 I/O 时，它们会切换回 MAINLOOP。通过这种方式，MAINLOOP 确保每个协程在有工作要做时都会被调度。

    MAINLOOP 只有在第一次 I/O 操作发生时才会启动，并且它与 __main__ 运行的 greenlet 不同。这个惰性启动意味着代码可以开始使用 Eventlet，而无需进行大规模的重构。

.. tab:: 英文

    The hub has a main greenlet, MAINLOOP.  When one of the running coroutines needs
    to do some I/O, it registers a listener with the hub (so that the hub knows when to wake it up again), and then switches to MAINLOOP (via ``get_hub().switch()``).  If there are other coroutines that are ready to run, MAINLOOP switches to them, and when they complete or need to do more I/O, they switch back to the MAINLOOP.  In this manner, MAINLOOP ensures that every coroutine gets scheduled when it has some work to do.

    MAINLOOP is launched only when the first I/O operation happens, and it is not the same greenlet that __main__ is running in.  This lazy launching means that code can start using Eventlet without needing to be substantially restructured.

更多 Hub 相关功能
--------------------------

**More Hub-Related Functions**

.. autofunction:: eventlet.hubs.get_hub
.. autofunction:: eventlet.hubs.get_default_hub
.. autofunction:: eventlet.hubs.trampoline

