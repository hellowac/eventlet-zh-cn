.. _env_vars:

环境变量
======================

**Environment Variables**

.. tab:: 中文

   Eventlet 的行为可以通过一些环境变量来控制。这些仅适用于高级用户。

   EVENTLET_HUB

      用于强制 Eventlet 使用指定的 hub，而不是最优的 hub。有关可接受的 hub 列表及其含义，请参见 :ref:`understanding_hubs` （请注意，选择不在列表上的 hub 会默默失败）。相当于在程序开始时调用 :meth:`eventlet.hubs.use_hub`。

   EVENTLET_THREADPOOL_SIZE

      :mod:`~eventlet.tpool` 中线程池的大小。之所以是环境变量，是因为 tpool 在第一次使用时会构建其池，因此对池大小的任何控制必须在此之前完成。

.. tab:: 英文

   Eventlet's behavior can be controlled by a few environment variables.
   These are only for the advanced user.

   EVENTLET_HUB 

      Used to force Eventlet to use the specified hub instead of the
      optimal one.  See :ref:`understanding_hubs` for the list of
      acceptable hubs and what they mean (note that picking a hub not on
      the list will silently fail).  Equivalent to calling
      :meth:`eventlet.hubs.use_hub` at the beginning of the program.

   EVENTLET_THREADPOOL_SIZE

      The size of the threadpool in :mod:`~eventlet.tpool`.  This is an
      environment variable because tpool constructs its pool on first
      use, so any control of the pool size needs to happen before then.
