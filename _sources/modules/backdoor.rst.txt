:mod:`backdoor`——正在运行的进程中的 Python 交互式解释器
===============================================================================

:mod:`backdoor` **-- Python interactive interpreter within a running process**

.. tab:: 中文

    backdoor 模块便于检查长时间运行进程的状态。它提供了一个正常的 Python 交互式解释器，同时不会阻塞应用程序的正常运行。这在调试、性能调优或简单地观察系统实际行为时非常有用。

    在应用程序中，使用一个 greenthread 在监听套接字上运行 `backdoor_server` 进行启动::

      eventlet.spawn(backdoor.backdoor_server, eventlet.listen(('localhost', 3000)), locals())

    当这个服务运行时，可以通过 `telnet` 连接指定端口访问 backdoor。

    .. code-block:: sh

      $ telnet localhost 3000
      (python 版本，构建信息)
      输入 "help"、"copyright"、"credits" 或 "license" 以了解更多信息。
      >>> import myapp
      >>> dir(myapp)
      ['__all__', '__doc__', '__name__', 'myfunc']
      >>>

    backdoor 会在命令间隔中协作式地让出控制权，使应用程序的其余部分得以继续运行。因此，在持续服务请求的服务器上，您可以在解释器命令之间观察内部状态的变化。

.. tab:: 英文

    The backdoor module is convenient for inspecting the state of a long-running process.  It supplies the normal Python interactive interpreter in a way that does not block the normal operation of the application.  This can be useful for debugging, performance tuning, or simply learning about how things behave in situ.

    In the application, spawn a greenthread running backdoor_server on a listening socket::

        eventlet.spawn(backdoor.backdoor_server, eventlet.listen(('localhost', 3000)), locals())

    When this is running, the backdoor is accessible via telnet to the specified port.

    .. code-block:: sh

      $ telnet localhost 3000
      (python version, build info)
      Type "help", "copyright", "credits" or "license" for more information.
      >>> import myapp
      >>> dir(myapp)
      ['__all__', '__doc__', '__name__', 'myfunc']
      >>>

    The backdoor cooperatively yields to the rest of the application between commands, so on a running server continuously serving requests, you can observe the internal state changing between interpreter commands.

.. automodule:: eventlet.backdoor
	:members:
