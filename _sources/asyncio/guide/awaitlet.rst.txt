.. _awaitlet_alternative:

Awaitlet 作为替代方案
==========================

**Awaitlet as an Alternative**

.. tab:: 中文

    有多年历史的应用程序可能已经经历了代码库的反复增长，因此，将这种现有代码库迁移到 AsyncIO 可能会很痛苦，甚至不现实。对于大多数这样的应用程序，迁移到 AsyncIO 可能意味着完全重写这些应用程序。

    `Awaitlet <https://awaitlet.sqlalchemy.org/en/latest/>`_ 是一个替代方案，它允许你在不面临迁移这些交付物所带来的一系列麻烦的情况下迁移现有代码库。

    Awaitlet 允许将现有的、使用线程和阻塞 API 编写的程序迁移到 asyncio，通过用 asyncio 兼容的方法替换前端和后端代码，但允许中间代码保持完全不变，无需在整个代码库中添加 ``async`` 或 ``await`` 关键字。它的主要用途是支持在 asyncio 和非-asyncio 运行时环境之间交叉兼容的代码。

    Awaitlet 是 `SQLAlchemy <https://www.sqlalchemy.org/>`_ 自身 asyncio 中介层的直接提取，且不依赖于 SQLAlchemy。该代码在多个环境中已广泛投入生产使用多年。

    .. warning::

        使用 Awaitlet 需要使用 :mod:`Asyncio Hub <eventlet.hubs.asyncio>`

        :ref:`understanding_hubs`

    以下是 Awaitlet 使用示例::

        import asyncio
        import awaitlet

        def asyncio_sleep():
            return awaitlet.awaitlet(asyncio.sleep(5, result='hello'))

        print(asyncio.run(awaitlet.async_def(asyncio_sleep)))

    我们邀请读者阅读 `Awaitlet 概述 <https://awaitlet.sqlalchemy.org/en/latest/synopsis.html>`_ 以更好地了解该库提供的机会。

.. tab:: 英文

    Applications with several years of existence may have seen their code base
    growing again and again, thus, migrating this kind of existing code
    base toward AsyncIO, would be painful or even unrealistic. For most of these
    applications, migrating to AsyncIO would may mean a complete rewriting of
    these applications.

    `Awaitlet <https://awaitlet.sqlalchemy.org/en/latest/>`_ is an alternative
    which allow you to migrate this kind of existing code base without getting
    the headaches associated to migrating such deliverables.

    Awaitlet allows existing programs written to use threads and blocking APIs to
    be ported to asyncio, by replacing frontend and backend code with asyncio
    compatible approaches, but allowing intermediary code to remain completely
    unchanged, with no addition of ``async`` or ``await`` keywords throughout the
    entire codebase needed. Its primary use is to support code that is
    cross-compatible with asyncio and non-asyncio runtime environments.

    Awaitlet is a direct extract of `SQLAlchemy <https://www.sqlalchemy.org/>`_’s
    own asyncio mediation layer, with no dependencies on SQLAlchemy. This code has
    been in widespread production use in thousands of environments for several
    years.

    .. warning::

        Using Awaitlet require to use the :mod:`Asyncio Hub
        <eventlet.hubs.asyncio>`

        :ref:`understanding_hubs`

    Here is an example of Awaitlet usage::

        import asyncio
        import awaitlet

        def asyncio_sleep():
            return awaitlet.awaitlet(asyncio.sleep(5, result='hello'))

        print(asyncio.run(awaitlet.async_def(asyncio_sleep)))

    We invite the reader to read the `Awaitlet synopsis
    <https://awaitlet.sqlalchemy.org/en/latest/synopsis.html>`_ to get a better
    overview of the opportunities offered by this library.
