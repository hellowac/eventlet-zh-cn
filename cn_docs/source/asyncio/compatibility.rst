.. _asyncio-compatibility:

eventlet 中的 Asyncio 兼容性
#################################

**Asyncio compatibility in eventlet**

.. tab:: 中文

    应该能够做到：

    * 在同一个线程中运行 eventlet 和 asyncio。
    * 允许 asyncio 和 eventlet 进行交互：eventlet 代码可以使用基于 asyncio 的库，基于 asyncio 的代码可以获取来自 eventlet 的结果。

    如果这个可行，它将允许在项目内部和跨项目逐步从 eventlet 迁移到 asyncio：

    1. 在 OpenStack 库内部，代码可以是 asyncio 和 eventlet 代码的混合体。这意味着迁移不必在一次操作中完成，无论是在库内部还是在依赖它们的应用程序中。
    2. 即使一个 OpenStack 库完全迁移到 asyncio，它仍然可以被任何仍在运行 eventlet 的程序使用。

.. tab:: 英文

    It should be possible to:

    * Run eventlet and asyncio in the same thread.
    * Allow asyncio and eventlet to interact: eventlet code can use asyncio-based libraries, asyncio-based code can get results out of eventlet.

    If this works, it would allow migrating from eventlet to asyncio in a gradual manner both within and across projects:

    1. Within an OpenStack library, code could be a mixture of asyncio and eventlet code.
       This means migration doesn't have to be done in one stop, neither in libraries nor in the applications that depend on them.
    2. Even when an OpenStack library fully migrates to asyncio, it will still be usable by anything that is still running on eventlet.

现有技术
=========

**Prior art**

.. tab:: 中文

    * Gevent 与 eventlet 有类似的模型。
    它与 asyncio 之间有一个集成，遵循下面提出的模型: https://pypi.org/project/asyncio-gevent/
    * Twisted 可以在 asyncio 事件循环之上运行。
    另外，它包括用于将其 `Deferred` 对象（类似于 JavaScript Promise）映射到 Python 3 中较新版本引入的 async/await 模型的工具，反过来，它也添加了将 async/await 函数转换为 `Deferred` 的支持。
    在 eventlet 上下文中，`GreenThread` 需要类似的集成方式来与 Twisted 的 `Deferred` 兼容。

.. tab:: 英文

    * Gevent has a similar model to eventlet.
      There exists an integration between gevent and asyncio that follows model proposed below: https://pypi.org/project/asyncio-gevent/
    * Twisted can run on top of the asyncio event loop.
      Separately, it includes utilities for mapping its `Deferred` objects (similar to a JavaScript Promise) to the async/await model introduced in newer versions in Python 3, and in the opposite direction it added support for turning async/await functions into `Deferred`s.
      In an eventlet context, `GreenThread` would need a similar former of integration to Twisted's `Deferred`.

第 1 部分：实现 asyncio/eventlet 互操作性interoperability
======================================================

**Part 1: Implementing asyncio/eventlet **

.. tab:: 中文

    集成 eventlet 和 asyncio 涉及三个不同的部分

.. tab:: 英文

    There are three different parts involved in integrating eventlet and asyncio for purposes

1. 创建在 asyncio 上运行的 hub
------------------------------------

**1. Create a hub that runs on asyncio**

.. tab:: 中文

    与许多网络框架一样，eventlet 具有可插入事件循环，在这种情况下称为“集线器”。通常，集线器会包装系统 API，如 `select()` 和 `epoll()`，但也曾经有一个在 Twisted 上运行的集线器。
    创建在 asyncio 事件循环之上运行的集线器应该相当简单。

    完成此操作后，eventlet 和 asyncio 代码可以在同一进程和同一线程中运行，但它们之间仍然难以通信。
    后一个要求需要额外的工作，如下两项所述。

.. tab:: 英文

    Like many networking frameworks, eventlet has pluggable event loops, in this case called a "hub". Typically hubs wrap system APIs like `select()` and `epoll()`, but there also used to be a hub that ran on Twisted.
    Creating a hub that runs on top of the asyncio event loop should be fairly straightforward.

    Once this is done, eventlet and asyncio code can run in the same process and the same thread, but they would still have difficulties talking to each other.
    This latter requirement requires additional work, as covered by the next two items.

2. 从 eventlet 调用 `async def` 函数eventlet
----------------------------------------------

**2. Calling `async def` functions from **

.. tab:: 中文

    目标是允许类似这样的操作：

    .. code::

        import aiohttp
        from eventlet_asyncio import future_to_greenlet  # hypothetical API
        
        async def get_url_body(url):
            async with aiohttp.ClientSession() as session:
                async with session.get(url) as response:
                    return await response.text()
        
        def eventlet_code():
            green_thread = future_to_greenlet(get_url_body("https://example.com"))
            return green_thread.wait()

    代码大概类似于 `<https://github.com/gfmio/asyncio-gevent/blob/main/asyncio_gevent/future_to_greenlet.py>`__

.. tab:: 英文

    The goal is to allow something like this:

    .. code::

        import aiohttp
        from eventlet_asyncio import future_to_greenlet  # hypothetical API
        
        async def get_url_body(url):
            async with aiohttp.ClientSession() as session:
                async with session.get(url) as response:
                    return await response.text()
        
        def eventlet_code():
            green_thread = future_to_greenlet(get_url_body("https://example.com"))
            return green_thread.wait()

    The code would presumably be similar to `<https://github.com/gfmio/asyncio-gevent/blob/main/asyncio_gevent/future_to_greenlet.py>`__

3. 从 asyncio 调用 eventlet 代码
-------------------------------------

**3. Calling eventlet code from asyncio**

.. tab:: 中文

    目标是允许这样的事情：

    .. code::

        from urllib.request import urlopen
        from eventlet import spawn
        from eventlet_asyncio import greenlet_to_future  # hypothetical API
        
        def get_url_body(url):
            # Looks blocking, but actually isn't
            return urlopen(url).read()
        
        # This would likely be common pattern, so could be implemented as decorator...
        async def asyncio_code():
            greenlet = eventlet.spawn(get_url_body, "https://example.com")
            future = greenlet_to_future(greenlet)
            return await future

    该代码大概类似于 `<https://github.com/gfmio/asyncio-gev​​ent/blob/main/asyncio_gevent/future_to_greenlet.py>`__

.. tab:: 英文

    The goal is to allow something like this:

    .. code::

        from urllib.request import urlopen
        from eventlet import spawn
        from eventlet_asyncio import greenlet_to_future  # hypothetical API
        
        def get_url_body(url):
            # Looks blocking, but actually isn't
            return urlopen(url).read()
        
        # This would likely be common pattern, so could be implemented as decorator...
        async def asyncio_code():
            greenlet = eventlet.spawn(get_url_body, "https://example.com")
            future = greenlet_to_future(greenlet)
            return await future

    The code would presumably be similar to `<https://github.com/gfmio/asyncio-gev​​ent/blob/main/asyncio_gevent/future_to_greenlet.py>`__

4. 限制和潜在的意外行为
------------------------------------------------

**4. Limitations and potential unexpected behavior**

.. tab:: 中文

    ``concurrent.futures.thread`` 只是使用普通线程，而不是 Eventlet 的特殊线程。
    类似地， `asyncio.to_thread() <https://docs.python.org/3/library/asyncio-task.html#asyncio.to_thread>`_ 特别要求常规阻塞代码，它无法与 Eventlet 代码正确配合使用。

    Asyncio hub 不支持多个读取器。

.. tab:: 英文

    ``concurrent.futures.thread`` just uses normal threads, not Eventlet's special threads.
    Similarly, `asyncio.to_thread() <https://docs.python.org/3/library/asyncio-task.html#asyncio.to_thread>`_
    specifically requires regular blocking code, it won't work correctly with Eventlet code.

    Multiple readers are not supported by the Asyncio hub.

第 2 部分: 端口在技术层面上的工作原理level
==================================================

**Part 2: How a port would work on a technical**

.. tab:: 中文



.. tab:: 英文

移植库
=================

**Porting a library**

.. tab:: 中文

    1. 基于 eventlet 的 API 的使用将被 asyncio API 替代。
       例如，`urllib` 或 `requests` 可能会被 `aiohttp <https://docs.aiohttp.org/en/stable/>`_ 替代。
       上述的互操作性可以确保它继续与基于 eventlet 的 API 一起工作。

       `awesome-asyncio <https://github.com/timofurrer/awesome-asyncio>`_ GitHub 仓库提供了一个精选的 Python asyncio 框架、库、软件和资源列表。不要犹豫，去看看它。你可能会发现一些与 asyncio 兼容的候选库，允许你替换一些当前的底层库。
    2. 随着时间的推移，API 需要迁移为 `async` 函数，但在过渡阶段，仍然可以使用标准的 `def`，再次使用上述的互操作性层。
    3. 最终，所有“阻塞”API 被移除，此时所有内容可以切换为 `async def` 和 `await`，包括外部 API，库将不再依赖于 eventlet。

.. tab:: 英文

    1. Usage of eventlet-based APIs would be replaced with usage of asyncio APIs.
       For example, `urllib` or `requests` might be replaced with `aiohttp <https://docs.aiohttp.org/en/stable/>`_.
       The interoperability above can be used to make sure this continues to work with eventlet-based APIs.

       The `awesome-asyncio <https://github.com/timofurrer/awesome-asyncio>`_ github repository propose a curated list of awesome
       Python asyncio frameworks, libraries, software and resources. Do not hesitate to take a look at it. You may find
       candidates compatible with asyncio that can allow you to replace some of your actual underlying libraries.
   2. Over time, APIs would need be migrated to be `async` function, but in the intermediate time frame a standard `def` can still be used, again using the interoperability layer above.
   3. Eventually all "blocking" APIs have been removed, at which point everything can be switched to `async def` and `await`, including external API, and the library will no longer depend on eventlet.

移植应用程序
======================

**Porting an application**

.. tab:: 中文

    在启动 eventlet 之前，应用程序需要安装 asyncio hub。
    除此之外，迁移与库的迁移是一样的。

    一旦所有库都完全基于 asyncio，eventlet 的使用就可以被移除，改为运行 asyncio 循环。

.. tab:: 英文

    An application would need to install the asyncio hub before kicking off eventlet.
    Beyond that porting would be the same as a library.

    Once all libraries are purely asyncio-based, eventlet usage can be removed and an asyncio loop run instead.
