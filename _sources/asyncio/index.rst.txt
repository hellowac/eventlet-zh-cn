.. _asyncio-index:

Eventlet 中的 Asyncio
#########################

**Asyncio in Eventlet**

.. tab:: 中文

    

.. tab:: 英文

Asyncio 兼容性
=====================

**Asyncio Compatibility**

.. tab:: 中文

    最近引入了 Asyncio 和 Eventlet 之间的兼容性。

    您可能对这种兼容性的最新进展和潜在的限制感兴趣，因此请查看 :ref:`asyncio-compatibility`.

.. tab:: 英文

    Compatibility between Asyncio and Eventlet has been recently introduced.

    You may be interested by the state of the art of this compatibility and by
    the potential limitations, so please take a look at
    :ref:`asyncio-compatibility`.

Asyncio Hub 和函数
=======================

**Asyncio Hub & Functions**

.. tab:: 中文

    发现 :mod:`Asyncio Hub <eventlet.hubs.asyncio>`

    您可能还想看看 :mod:`Asyncio 兼容性函数 <eventlet.asyncio>`。

.. tab:: 英文

    Discover the :mod:`Asyncio Hub <eventlet.hubs.asyncio>`

    You may also want to take a look to the :mod:`Asyncio compatibility functions <eventlet.asyncio>`.

从 Eventlet 迁移到 Asyncio
==================================

**Migrating from Eventlet to Asyncio**


为什么要迁移？
--------------

**Why Migrating?**

.. tab:: 中文

    Eventlet 是一种过时且残破的技术。

    Eventlet 是在近 20 年前创建的（请参阅 Eventlet 的 :ref:`history`），
    当时 Python 尚未提供非阻塞功能。

    随着时间的推移，Python 现在提供 AsyncIO。

    在 Python 的发展过程中，Eventlet 的维护在 Python 的几个版本中停止，增加了 Eventlet 的 monkey patching 与 Python 的最新实现之间的差距。

    现在无法弥补这一差距。因此，我们决定以渐进的方式正式放弃 Eventlet 的维护。

    作为最后的努力，我们希望让 Eventlet 得到应有的休息。
    我们的目标是为您提供迁移 Eventlet 的指南，然后
    正确淘汰 Eventlet。

    有关促使这一努力的原因的更多详细信息，我们邀请
    读者查看与此计划放弃相关的讨论：

    https://review.opendev.org/c/openstack/governance/+/902585

.. tab:: 英文

    Eventlet is a broken and outdated technology.

    Eventlet was created almost 20 years ago (See the :ref:`history` of Eventlet),
    at a time where Python did not provided non-blocking features.

    Time passed and Python now provide AsyncIO.

    In parallel of the evolution of Python, the maintenance of Eventlet was
    discontinued during several versions of Python, increasing the gap between
    the monkey patching of Eventlet and the recent implementation of Python.

    This gap is now not recoverable. For this reason, we decided to officially
    abandon the maintenance of Eventlet in an incremental way.

    In a last effort, we want to lead Eventlet to a well deserved rest.
    Our goal is to provide you a guide to migrate off of Eventlet and then
    to properly retire Eventlet.

    For more details about the reasons who motivated this effort we invite the
    readers to show the discussions related to this scheduled abandon:

    https://review.opendev.org/c/openstack/governance/+/902585

入门
---------------

**Getting Started**

.. tab:: 中文

    想要同时使用 Asyncio 和 Eventlet 还是只想迁移出 Eventlet？

    遵循 :ref:`官方迁移指南 <migration-guide>`。

    我们鼓励读者首先查看 :ref:`glossary_guide`，以了解迁移过程中可能遇到的各种术语。

.. tab:: 英文

    Want to use Asyncio and Eventlet together or you simply want to migrate off of Eventlet?

    Follow the :ref:`official migration guide <migration-guide>`.

    We encourage readers to first look at the :ref:`glossary_guide` to learn about the various terms that may be encountered during the migration.

替代方案和提示
-------------------

**Alternatives & Tips**

.. tab:: 中文

    您想重构代码以替换 Eventlet 用法吗？请参阅建议的替代方案和提示：

    - :ref:`awaitlet_alternative`
    - :ref:`manage-your-deprecations`

.. tab:: 英文

    You want to refactor your code to replace Eventlet usages? See the proposed
    alternatives and tips:

    - :ref:`awaitlet_alternative`
    - :ref:`manage-your-deprecations`

迁移参考
=========


.. toctree::
   :maxdepth: 2

   compatibility
   migration
   guide/awaitlet
   guide/deprecation
   guide/glossary