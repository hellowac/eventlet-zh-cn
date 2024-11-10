Eventlet 文档
######################

**Eventlet Documentation**

警告
=======

**Warning**

.. tab:: 中文

    **现在强烈不建议在新项目中使用 eventlet！请阅读以下内容。**

    Eventlet 创建于将近 18 年前，当时 CPython 标准库中缺乏异步功能。随着时间的推移，eventlet 和 CPython 各自发展，但近年来 eventlet 的维护活动减少，导致 eventlet 与 CPython 实现之间的差距不断扩大。

    这个差距现在已经过大，可能会在您的应用程序中引发意外的副作用和错误。

    Eventlet 现在遵循新的维护策略。 **只提供稳定性和错误修复的维护**。 **不再接受新功能**，除非与 asyncio 迁移相关。 **不建议在新项目中使用**。 **我们的目标是规划 eventlet 的退役**，并为您提供迁移出 eventlet 的方法。

    如果您正在寻找用于管理异步网络编程的库，并且尚未使用 eventlet，那么我们建议您使用 `asyncio`_，这是 CPython 标准库的官方异步库。

    如果您已经在使用 eventlet，我们希望能够为某些用例提供迁移到 asyncio 的途径；请参见 :ref:`migration-guide`。只有与迁移解决方案相关的新功能会被接受。

    如果您对维护目标或迁移有疑问，欢迎 `开启新的 issue <open a new issue>`_ ，我们会很乐意回答您的问题。

.. tab:: 英文

    **New usages of eventlet are now heavily discouraged! Please read the
    following.**

    Eventlet was created almost 18 years ago, at a time where async
    features were absent from the CPython stdlib. With time eventlet evolved and
    CPython too, but since several years the maintenance activity of eventlet
    decreased leading to a growing gap between eventlet and the CPython
    implementation.

    This gap is now too high and can lead you to unexpected side effects and bugs
    in your applications.

    Eventlet now follows a new maintenance policy. **Only maintenance for
    stability and bug fixing** will be provided. **No new features will be
    accepted**, except those related to the asyncio migration. **Usages in new
    projects are discouraged**. **Our goal is to plan the retirement of eventlet**
    and to give you ways to move away from eventlet.

    If you are looking for a library to manage async network programming,
    and if you do not yet use eventlet, then, we encourage you to use `asyncio`_,
    which is the official async library of the CPython stdlib.

    If you already use eventlet, we hope to enable migration to asyncio for some use
    cases; see :ref:`migration-guide`. Only new features related to the migration
    solution will be accepted.

    If you have questions concerning maintenance goals or concerning
    the migration do not hesitate to `open a new issue`_, we will be happy to
    answer them.

.. _asyncio: https://docs.python.org/3/library/asyncio.html
.. _open a new issue: https://github.com/eventlet/eventlet/issues/new

安装
============

**Installation**

.. tab:: 中文

    获取 Eventlet 的最简单方法是使用 pip::

      pip install -U eventlet

    要安装最新的开发版本一次::

      pip install -U https://github.com/eventlet/eventlet/archive/master.zip

.. tab:: 英文

    The easiest way to get Eventlet is to use pip::

      pip install -U eventlet

    To install latest development version once::

      pip install -U https://github.com/eventlet/eventlet/archive/master.zip

使用
=====

**Usage**

.. tab:: 中文

    代码会说话！这是一个简单的网络爬虫，可以同时抓取一堆 URL:

    .. code-block:: python

        urls = [
            "http://www.google.com/intl/en_ALL/images/logo.gif",
            "http://python.org/images/python-logo.gif",
            "http://us.i1.yimg.com/us.yimg.com/i/ww/beta/y3.gif",
        ]

        import eventlet
        from eventlet.green.urllib.request import urlopen

        def fetch(url):
            return urlopen(url).read()

        pool = eventlet.GreenPool()
        for body in pool.imap(fetch, urls):
            print("got body", len(body))

.. tab:: 英文

    Code talks!  This is a simple web crawler that fetches a bunch of urls concurrently:

    .. code-block:: python

        urls = [
            "http://www.google.com/intl/en_ALL/images/logo.gif",
            "http://python.org/images/python-logo.gif",
            "http://us.i1.yimg.com/us.yimg.com/i/ww/beta/y3.gif",
        ]

        import eventlet
        from eventlet.green.urllib.request import urlopen

        def fetch(url):
            return urlopen(url).read()

        pool = eventlet.GreenPool()
        for body in pool.imap(fetch, urls):
            print("got body", len(body))

支持的 Python 版本
=========================

**Supported Python Versions**

.. tab:: 中文

    目前支持 CPython 3.7+。

.. tab:: 英文

    Currently supporting CPython 3.7+.


概念和参考
=====================

**Concepts & References**

.. toctree::
   :maxdepth: 2

   asyncio/index
   basic_usage
   design_patterns
   patching
   examples
   ssl
   threading
   zeromq
   hubs
   environment
   modules

想要贡献？
===================

**Want to contribute?**

.. toctree::
   :maxdepth: 2

   contribute
   testing
   maintenance

许可证
=======

**License**

.. tab:: 中文

    Eventlet 是根据开源 `MIT 许可证<http://www.opensource.org/licenses/mit-license.php>`_ 的条款提供的。

.. tab:: 英文

    Eventlet is made available under the terms of the open source `MIT license <http://www.opensource.org/licenses/mit-license.php>`_

变更日志
=========

**Changelog**

.. tab:: 中文

    有关 Eventlet 发布版本的更多详细信息，请查看 `changelog`_.

.. tab:: 英文

    For further details about released versions of Eventlet please take a
    look at the `changelog`_.

作者和历史
=================

**Authors & History**

.. tab:: 中文

    如果您有疑问，或者您可能发现了错误，并且想要联系作者或维护者，请查看 :ref:`authors`。

    您想了解有关 Eventlet 的历史的更多信息，请查看 :ref:`history`。

.. tab:: 英文

    You have questions or you may have find a bug and you want to contact authors
    or maintainers, then please take a look at :ref:`authors`.

    You want to learn more about the history of Eventlet, then, please take a
    look at :ref:`history`.

索引和表格
==================

**Indices and tables**

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
* `changelog`_


.. _changelog: https://github.com/eventlet/eventlet/blob/master/NEWS
