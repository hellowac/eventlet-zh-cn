.. _manage-your-deprecations:

管理您的弃用内容
========================

**Manage Your Deprecations**

.. tab:: 中文

    库或应用程序可能具有与 Eventlet 强烈相关的特定功能，比如 OpenStack `oslo.messaging
    <https://docs.openstack.org/oslo.messaging/latest/configuration/opts.html#oslo_messaging_rabbit.heartbeat_in_pthread>`_
    交付物中的 ``heartbeat_in_pthread`` 功能。

    从 Eventlet 迁移会使这些功能过时。由于这种功能暴露了配置端点，人们需要将其弃用，以便用户可以相应地更新其配置文件。然而，弃用过程可能需要几个月，甚至多个版本，才能期望看到这些功能被移除，从而阻碍了迁移。

    提议的解决方案是通过空的入口点模拟这些功能，这些入口点仅会引发弃用警告，告知用户他们需要更新配置文件。在 1 到 2 个新版本之后，这些空的模拟可以安全地移除，而不会影响任何人。

    换句话说，这些功能将保留在代码中，但它们不会执行任何操作。它们将是空功能，允许我们正确地进行迁移。

    以 ``heartbeat_in_pthread`` 功能为例，通过使用 Asyncio，我们不需要在单独的线程中运行心跳。这个功能，即 RabbitMQ 的心跳，将在一个协程中运行。协程将在主本地线程中运行。配置选项将继续可用，但它只会显示类似以下的弃用警告::

        __main__:1: DeprecationWarning: Using heartbeat_in_pthread is
        deprecated and will be removed in {SERIES}. Enabling that feature
        have no functional effects due to recent changes applied in the
        networking model used by oslo.messaging. Please plan an update of your
        configuration.

.. tab:: 英文

    Libraries or applications may have specific features who are strongly related
    to Eventlet, like the ``heartbeat_in_pthread`` feature in
    the Opentack `oslo.messaging
    <https://docs.openstack.org/oslo.messaging/latest/configuration/opts.html#oslo_messaging_rabbit.heartbeat_in_pthread>`_
    deliverable.

    Migrating off of Eventlet would make these features obsolete. As this kind of
    feature expose configuration endpoints people would have to deprecate them to
    allow your users to update their config files accordingly. However, the
    deprecation process would take several months or even numerous versions before
    hoping to see these features removed. Hence blocking the migration.

    The proposed solution is to mock these features with empty entrypoints
    who will only raise deprecation warnings to inform your users that they have
    to update their config files. After 1 or 2 new versions these empty mocks
    could be safely removed without impacting anybody.

    In other words, these feature will remain in the code, but they will do
    nothing. They will be empty feature allowing us to migrate properly.

    Example with the ``heartbeat_in_pthread`` feature, by using Asyncio
    we wouldn't have to run heartbeats in a separated threads. This feature,
    the RabbitMQ heartbeat, would be run in a coroutine. A coroutine who is
    ran in the main native thread. The config option will remain available but
    it will only show a deprecation warning like the following one::

        __main__:1: DeprecationWarning: Using heartbeat_in_pthread is
        deprecated and will be removed in {SERIES}. Enabling that feature
        have no functional effects due to recent changes applied in the
        networking model used by oslo.messaging. Please plan an update of your
        configuration.
