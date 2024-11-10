.. _migration-guide:

迁移出 Eventlet
=========================

**Migrating off of Eventlet**

.. tab:: 中文

    Eventlet 有两个主要的使用场景：

    1. 作为一个必需的网络框架，类似于使用 ``asyncio``、 ``trio`` 或者旧的框架如 ``Twisted`` 和 ``tornado``。

    2. 作为一个可选的、可插拔的后端，允许透明地将阻塞 API 替换为事件循环，而无需更改任何代码。
    这就是 Celery 和 Gunicorn 使用 eventlet 的方式。

    假装看起来像一个阻塞 API，但实际上在底层使用事件循环，需要精确地模拟一个不断变化和不断扩展的 API 足迹，这对于一个志愿者驱动的开源项目来说是根本无法持续的。
    这就是为什么 Eventlet 不鼓励新用户使用的原因。

    **本文件的大部分内容将集中在第一个使用场景：将 Eventlet 作为唯一的网络框架。**
    对于这个使用场景，我们建议迁移到 Python 的 ``asyncio``，并且我们正在提供一些基础设施来使这一过程变得更加容易，并允许进行 *渐进式* 迁移。

    对于第二个使用场景，我们认为这是一个根本无法持续的做法，并鼓励上游框架找到不同的解决方案。

.. tab:: 英文

    There are two main use cases for Eventlet:

    1. As a required networking framework, much like one would use ``asyncio``,
    ``trio``, or older frameworks like ``Twisted`` and ``tornado``.

    2. As an optional, pluggable backend that allows swapping out blocking APIs
    for an event loop, transparently, without changing any code.
    This is how Celery and Gunicorn use eventlet.

    Pretending to look like a blocking API while actually using an event loop
    underneath requires exact emulation of an ever-changing and ever-increasing
    API footprint, which is fundamentally unsustainable for a volunteer-driven
    open source project.
    This is why Eventlet is discouraging new users.

    **Most of this document will focus on the first use case: Eventlet as the sole
    networking framework.**
    For this use case, we recommend migrating to Python's ``asyncio``, and we are
    providing infrastructure that will make this much easier, and allow for
    *gradual* migration.

    For the second use case, we believe this is a fundamentally unsustainable
    approach and encourage the upstream frameworks to come up with different
    solutions.

步骤 1. 切换到 ``asyncio`` Hub
-------------------------------------

**Step 1. Switch to the ``asyncio`` Hub**

.. tab:: 中文

    Eventlet 有不同的可插拔网络事件循环。
    通过将事件循环切换为使用 ``asyncio``，你可以在同一个线程和同一个进程中运行 ``asyncio`` 和 Eventlet 代码。

    为此，在启动 Eventlet 程序之前，将 ``EVENTLET_HUB`` 环境变量设置为 ``asyncio``。
    例如，如果你通过 shell 脚本启动程序，可以执行 ``export EVENTLET_HUB=asyncio``。

    或者，你可以显式地在启动时指定 ``asyncio`` hub，在猴子补丁或者任何其他设置工作之前进行设置::

        import eventlet.hubs
        eventlet.hubs.use_hub("eventlet.hubs.asyncio")

.. tab:: 英文

    Eventlet has different pluggable networking event loops.
    By switching the event loop to use ``asyncio``, you enable running ``asyncio``
    and Eventlet code in the same thread in the same process.

    To do so, set the ``EVENTLET_HUB`` environment variable to ``asyncio`` before
    starting your Eventlet program.
    For example, if you start your program with a shell script, you can do
    ``export EVENTLET_HUB=asyncio``.

    Alternatively, you can explicitly specify the ``asyncio`` hub at startup,
    before monkey patching or any other setup work::

        import eventlet.hubs
        eventlet.hubs.use_hub("eventlet.hubs.asyncio")

步骤 2. 将代码迁移到 ``asyncio``
-----------------------------------

**Step 2. Migrate code to ``asyncio``**

.. tab:: 中文

    现在你已经在 ``asyncio`` 上运行 Eventlet 了，你可以使用一些新的 API 来从 Eventlet 代码调用 ``asyncio``，反之亦然。

    要从 Eventlet 代码调用 ``asyncio`` 代码，你可以将协程（或任何可以 ``await`` 的东西）包装成一个 Eventlet 的 ``GreenThread``。
    例如，如果你想从 Eventlet 发起一个 HTTP 请求，你可以使用基于 ``asyncio`` 的 ``aiohttp`` 库::

        import aiohttp
        from eventlet.asyncio import spawn_for_awaitable

        async def request():
            async with aiohttp.ClientSession() as session:
                url = "https://example.com"
                async with session.get(url) as response:
                    html = await response.text()
                    return html


        # 这将创建一个协程；通常你会 ``await`` 它：
        coro = request()

        # 你可以用 Eventlet GreenThread 包装这个协程，类似于
        # ``eventlet.spawn()``：
        gthread = spawn_for_awaitable(request())

        # 然后获取它的结果，即 https://example.com 的内容：
        result = gthread.wait()

    在另一个方向，任何 ``eventlet.greenthread.GreenThread`` 都可以在 ``async`` 函数中被 ``await``。
    换句话说，``async`` 函数可以调用 Eventlet 代码::

        def blocking_eventlet_api():
            eventlet.sleep(1)
            # 做一些其他伪阻塞的工作
            # ...
            return 12

        async def my_async_func():
            gthread = eventlet.spawn(blocking_eventlet_api)
            # 在正常的 Eventlet 代码中我们会调用 gthread.wait()，但是由于这是一个
            # async 函数，我们需要改为 await：
            result = await gthread
            # 结果现在是 12
            # ...

    ``asyncio.Future`` 的取消和 ``eventlet.GreenThread`` 的终止应当在两者之间传播。

    通过这两个 API（以后会有更多），你可以逐步将应用程序或库的部分迁移到 ``asyncio``。
    例如，像 ``urlopen()`` 或 ``requests.get()`` 这样的阻塞 API 调用可以替换为对 ``aiohttp`` 的调用。

    根据你的 Eventlet 使用情况，在迁移过程中，你可能需要废弃与 Eventlet 相关的 CLI 选项，建议读者查看： :ref:`manage-your-deprecations`。

    `awesome-asyncio <https://github.com/timofurrer/awesome-asyncio>`_ github 仓库提供了一个精心挑选的 Python asyncio 框架、库、软件和资源的列表。不要犹豫，去看看它。你可能会找到一些与 asyncio 兼容的候选库，帮助你替换一些实际的底层库。

.. tab:: 英文

    Now that you're running Eventlet on top of ``asyncio``, you can use some new
    APIs to call from Eventlet code into ``asyncio``, and vice-versa.

    To call ``asyncio`` code from Eventlet code, you can wrap a coroutine (or
    anything you can ``await``) into an Eventlet ``GreenThread``.
    For example, if you want to make a HTTP request from Eventlet, you can use
    the ``asyncio``-based ``aiohttp`` library::

        import aiohttp
        from eventlet.asyncio import spawn_for_awaitable

        async def request():
            async with aiohttp.ClientSession() as session:
                url = "https://example.com"
                async with session.get(url) as response:
                    html = await response.text()
                    return html


        # This makes a coroutine; typically you'd ``await`` it:
        coro = request()

        # You can wrap this coroutine with an Eventlet GreenThread, similar to
        # ``evenlet.spawn()``:
        gthread = spawn_for_awaitable(request())

        # And then get its result, the body of https://example.com:
        result = gthread.wait()

    In the other direction, any ``eventlet.greenthread.GreenThread`` can be
    ``await``-ed in ``async`` functions.
    In other words ``async`` functions can call into Eventlet code::

        def blocking_eventlet_api():
            eventlet.sleep(1)
            # do some other pseudo-blocking work
            # ...
            return 12

        async def my_async_func():
            gthread = eventlet.spawn(blocking_eventlet_api)
            # In normal Eventlet code we'd call gthread.wait(), but since this is an
            # async function we'll want to await instead:
            result = await gthread
            # result is now 12
            # ...

    Cancellation of ``asyncio.Future`` and killing of ``eventlet.GreenThread``
    should propagate between the two.

    Using these two APIs, with more to come, you can gradually migrate portions of
    your application or library to ``asyncio``.
    Calls to blocking APIs like ``urlopen()`` or ``requests.get()`` can get
    replaced with calls to ``aiohttp``, for example.

    Depending on your Eventlet usage, during your migration, you may have to
    deprecate CLI options that are related to Eventlet, we invite the reader
    to take a look to :ref:`manage-your-deprecations`.

    The `awesome-asyncio <https://github.com/timofurrer/awesome-asyncio>`_ github
    repository propose a curated list of awesome Python asyncio frameworks,
    libraries, software and resources. Do not hesitate to take a look at it.
    You may find candidates compatible with asyncio that can allow you to replace
    some of your actual underlying libraries.

步骤 3. 完全放弃 Eventlet
--------------------------------

**Step 3. Drop Eventlet altogether**

.. tab:: 中文

    最终，您将完全不再依赖 Eventlet：您的所有代码都将基于 ``asyncio``。
    此时，您可以删除 Eventlet 并切换到直接运行 ``asyncio`` 循环。

.. tab:: 英文

    Eventually you won't be relying on Eventlet at all: all your code will be
    ``asyncio``-based.
    At this point you can drop Eventlet and switch to running the ``asyncio``
    loop directly.

已知限制和正在进行的工作
--------------------------------------

**Known limitations and work in progress**

.. tab:: 中文

    一般来说，``async`` 函数和 Eventlet 绿色线程是两个独立的世界，它们恰好可以互相调用。

    在 ``async`` 函数中：

    * Eventlet 的线程局部变量可能无法正常工作。
    * ``eventlet.greenthread.getcurrent()`` 不会返回你预期的结果。
    * 如果直接使用，``eventlet`` 的锁和队列将无法正常工作。
    * Eventlet 不支持多个读取器，因此使用 ``eventlet.debug.hub_prevent_multiple_readers`` 也不会有效。

    在 Eventlet 绿色线程中：

    * 如果直接使用，``asyncio`` 的锁将无法正常工作。

    我们预计随着对迁移和集成的进一步了解，我们会逐步增加更多的迁移和集成 API，包括常见的习惯用法和迁移需求。
    你可以在 `GitHub 问题 <https://github.com/eventlet/eventlet/issues/868>`_ 中跟踪进展，如果遇到问题，也可以提交新的问题。

.. tab:: 英文

    In general, ``async`` functions and Eventlet green threads are two separate
    universes that just happen to be able to call each other.

    In ``async`` functions:

    * Eventlet thread locals probably won't work correctly.
    * ``evenlet.greenthread.getcurrent()`` won't give the result you expect.
    * ``eventlet`` locks and queues won't work if used directly.
    * Eventlet multiple readers are not supported, and so using
    ``eventtlet.debug.hub_prevent_multiple_readers`` neither.

    In Eventlet greenlets:

    * ``asyncio`` locks won't work if used directly.

    We expect to add more migration and integration APIs over time as we learn
    more about what works, common idioms, and requirements for migration.
    You can track progress in the
    `GitHub issue <https://github.com/eventlet/eventlet/issues/868>`_, and file
    new issues if you have problems.

替代方案
------------

**Alternatives**

.. tab:: 中文

    如果你真的想继续使用 Eventlet 的伪阻塞方法，可以使用 `gevent <https://www.gevent.org/>`_。
    但请记住，使 Eventlet 维护在长期内不可持续的相同技术问题也适用于 Gevent。

.. tab:: 英文

    If you really want to continue with Eventlet's pretend-to-be-blocking
    approach, you can use `gevent <https://www.gevent.org/>`_.
    But keep in mind that the same technical issues that make Eventlet maintenance
    unsustainable over the long term also apply to Gevent.
