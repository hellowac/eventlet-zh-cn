.. _glossary_guide:

术语
========

**Glossary**

.. tab:: 中文

    本词汇表简要描述了在 Eventlet 中通常使用的一些术语，尤其是在迁移上下文中的术语。  
    该词汇表的目标是确保每个人对所使用的术语有相同的理解。

    有关迁移的更多信息，请参见 :ref:`migration-guide`。

.. tab:: 英文

    This glossary provides a brief description of some of the terms used within
    Eventlet in general, and more specifically in the migration context.
    The goal of this glossary is to ensure that everybody has the same
    understanding of the used terms.

    For more information about anything the migration, see the
    :ref:`migration-guide`.

.. _glossary-concurrency:

并发
-----------

**Concurrency**

.. tab:: 中文

    **并发**是指两个或更多任务可以在重叠的时间**段**内开始、运行和完成。它不一定意味着它们会在**同一时刻**都在运行。例如，在单核机器上的**多任务处理**。

.. tab:: 英文

    **Concurrency** is when two or more tasks can start, run, and complete in
    overlapping time **periods**. It doesn't necessarily mean they'll ever both be
    running **at the same instant**. For example, _multitasking_ on a single-core
    machine.

.. _glossary-cooperative-multitasking:

协作式多任务
------------------------

**Cooperative Multitasking**

.. tab:: 中文

    每当一个 **线程** 开始休眠或等待网络 I/O 时，另一个线程就有机会获取 **全局解释器锁（GIL）** 并执行 Python 代码。这就是 **协作式多任务** 。

.. tab:: 英文

    Whenever a **thread** begins sleeping or awaiting network I/O, there is a
    chance for another thread to take the **GIL** and execute Python code.
    This is **cooperative multitasking**.

.. _glossary-coro:

一个协程
--------

**Coro**

.. tab:: 中文

    在 Python API 文档中，使用 **coro** 这个名称是一种常见的约定。它指的是一个协程；严格来说，是调用 `async def` 函数的结果，而不是函数本身。

.. tab:: 英文

    Using the name **coro** is a common convention in the Python API
    documentation. It refers to a coroutine; i.e., strictly speaking, the result
    of calling an async def function, and not the function itself.

.. _glossary-coroutine:

协程
---------

**Coroutine**

.. tab:: 中文

    **协程** 是允许执行被挂起和恢复的程序组件，具有广泛的应用。它们被描述为“可以暂停执行的函数”。

.. tab:: 英文

    **Coroutines** are programs components that allow execution to be suspended
    and resumed, generalizing. They have been described as "functions whose
    execution you can pause".

.. _glossary-future:

Future
------

.. tab:: 中文

    **Future** 代表某个活动的未来完成状态，并由事件循环管理。Future 是一个特殊的低级可等待对象，表示异步操作的最终结果。

.. tab:: 英文

    A **future** represents a future completion state of some activity and is
    managed by the loop. A Future is a special low-level awaitable object that
    represents an eventual result of an asynchronous operation.

.. _glossary-greenlet:

Greenlet
--------

.. tab:: 中文

    **Greenlet** 是一种轻量级的 **协程**，用于进程内的顺序并发编程（参见 **并发** ）。你通常可以将 greenlet 看作是协作式调度的 **线程**。其主要区别在于，由于它们是协作式调度的，你可以控制它们的执行时机，并且由于它们是 **协程**，许多 greenlet 可以存在于同一个原生 **线程** 中。

    Greenlet 是协作式的（参见 **协作式多任务**）和顺序执行的。这意味着当一个 greenlet 正在运行时，其他 greenlet 不能运行；程序员完全控制何时在 greenlet 之间切换执行。换句话说，使用 greenlet 时不应期待 **抢占式** 行为。

    Greenlet 也是一个 `库
    <https://greenlet.readthedocs.io/en/latest/>`_，提供 greenlet 机制。Eventlet 基于 greenlet 库。

.. tab:: 英文

    A **greenlet** is a lightweight **coroutine** for in-process sequential
    concurrent programming (see **concurrency**). You can usually think of
    greenlets as cooperatively scheduled **threads**. The major differences are
    that since they’re cooperatively scheduled, you are in control of when they
    execute, and since they are **coroutines**, many greenlets can exist in a
    single native **thread**.

    Greenlets are cooperative (see **cooperative multitasking**) and sequential.
    This means that when one greenlet is running, no other greenlet can be
    running; the programmer is fully in control of when execution switches between
    greenlets. In other words ones, when using greenlets, should not expect
    **preemptive** behavior.

    Greenlet is also the name of a `library
    <https://greenlet.readthedocs.io/en/latest/>`_ that provide the greenlet
    mechanism. Eventlet is based on the greenlet library.

.. _glossary-green-thread:

Green Thread
------------

**Green Thread**

.. tab:: 中文

    **Green thread** 是一种由运行时库或虚拟机（VM）调度的 **线程**，而不是由底层操作系统（OS）原生调度。Green thread 模拟多线程环境，而无需依赖任何原生操作系统功能，它们在用户空间而非内核空间中进行管理，使其能够在没有原生线程支持的环境中工作。

.. tab:: 英文

    A **green thread** is a **threads** that is scheduled by a runtime library
    or virtual machine (VM) instead of natively by the underlying operating system
    (OS). Green threads emulate multithreaded environments without relying on any
    native OS abilities, and they are managed in user space) instead of kernel
    space, enabling them to work in environments that do not have native thread
    support.

.. _glossary-gil:

全局解释器锁 (GIL)
-----------------------------

**Global Interpreter Lock (GIL)**

.. tab:: 中文

    **全局解释器锁（GIL）** 是 CPython 内部使用的一个锁，用于确保在 Python 虚拟机中一次只有一个 **线程** 在运行。通常，Python 只在字节码指令之间切换线程（参见 **抢占式多任务** 和 **协作式多任务** ）。

.. tab:: 英文

    A **global interpreter lock (GIL**) is a lock used internally to CPython to
    ensure that only one **thread** runs in the Python VM at a time. In general,
    Python offers to switch among threads only between bytecode instructions (see
    **preemptive multitasking** and **cooperative multitasking**). 

.. _glossary-parallelism:

并行
-----------

**Parallelism**

.. tab:: 中文

    **并行** 是指任务 _字面上_ 同时运行，例如在多核处理器上。它是当至少两个线程同时执行时出现的状态。

.. tab:: 英文

    **Parallelism** is when tasks _literally_ run at the same time, e.g., on a
    multicore processor. A condition that arises when at least two threads are
    executing simultaneously.

.. _glossary-preemptive:

抢占
---------------------

**Preemptive/Preemption**

.. tab:: 中文

    **抢占** 是暂时中断正在执行的 **任务**，并计划稍后恢复执行的行为。这种中断由外部调度程序执行，任务本身不提供任何协助或合作。

.. tab:: 英文

    **Preemption** is the act of temporarily interrupting an executing **task**,
    with the intention of resuming it at a later time. This interrupt is done by
    an external scheduler with no assistance or cooperation from the task.

.. _glossary-preemptive-multitasking:

抢占式多任务
-----------------------

**Preemptive multitasking**

.. tab:: 中文

    **抢占式多任务** 涉及使用中断机制，暂停当前正在执行的进程，并调用调度程序来确定下一个应该执行的进程。因此，所有进程在任何给定时间都会获得一定的 CPU 时间。

    CPython 也有 _抢占式多任务_ ：如果一个线程在 Python 2 中连续运行 1000 个字节码指令，或者在 Python 3 中运行 15 毫秒，它会放弃 GIL，允许另一个线程运行。

.. tab:: 英文

    **Preemptive multitasking** involves the use of an interrupt mechanism which
    suspends the currently executing process and invokes a scheduler to determine
    which process should execute next. Therefore, all processes will get some
    amount of CPU time at any given time.

    CPython also has _preemptive multitasking_: If a thread runs
    uninterrupted for 1000 bytecode instructions in Python 2, or runs 15
    milliseconds in Python 3, then it gives up the GIL and another thread may run.

.. _glossary-task:

任务
----

**Task**

.. tab:: 中文

    **任务** 是一个被调度并独立管理的 **协程**。任务是可等待对象，用于并发调度协程。

.. tab:: 英文

    A **task** is a scheduled and independently managed **coroutine**. Tasks are
    awaitable objects used to schedule coroutines concurrently.

.. _glossary-thread:

线程
------

**Thread**

.. tab:: 中文

    **线程** 是一种让程序将自身分成两个或多个同时（或伪同时）运行的任务的方式。线程和进程在不同操作系统中有所不同，但通常，线程是在进程内的，不同的线程共享同一进程中的资源，而在同一个多任务操作系统中的不同进程之间则不共享资源。

    Python 中线程何时切换？切换取决于上下文。线程可能会被中断（参见 **抢占式多任务**），或表现得像是合作式的（参见 **合作式多任务**）。

.. tab:: 英文

    **Threads** are a way for a program to divide (termed "split") itself into two
    or more simultaneously (or pseudo-simultaneously) running tasks. Threads and
    processes differ from one operating system to another but, in general, a
    thread is contained inside a process and different threads in the same process
    share same resources while different processes in the same multitasking
    operating system do not.

    When do threads switch in Python? The switch depends on the context. The
    threads may be interrupted (see **preemptive multitasking**) or behave
    cooperatively (see **cooperative multitasking**).
