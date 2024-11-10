:mod:`websocket` -- Websocket 服务
=====================================

:mod:`websocket` **-- Websocket Server**

.. tab:: 中文

    这个模块提供了一种简单的方法来创建一个 `websocket
    <http://dev.w3.org/html5/websockets/>`_ 服务器。它通过在 :mod:`~eventlet.wsgi` 模块中进行一些调整，允许 WebSockets 与其他 WSGI 应用程序共存。

    要创建一个 websocket 服务器，只需使用 :class:`WebSocketWSGI` 装饰器装饰一个处理方法，并将其用作 WSGI 应用程序::

        from eventlet import wsgi, websocket
        import eventlet
        
        @websocket.WebSocketWSGI
        def hello_world(ws):
            ws.send("hello world")
        
        wsgi.server(eventlet.listen(('', 8090)), hello_world)

    .. note::

        请参阅 :func:`~eventlet.wsgi.server` 文档中的优雅终止警告。

    你可以在文件 ``examples/websocket.py`` 中找到这个代码的稍微复杂版本。

    从版本 0.9.13 开始，eventlet.websocket 支持 SSL WebSockets；只需使用 :ref:`SSL wsgi server <wsgi_ssl>`。

    .. note :: WebSocket 规范仍在开发中，响应规范变化时，本模块的工作方式可能需要进行更改。

.. tab:: 英文

    This module provides a simple way to create a `websocket
    <http://dev.w3.org/html5/websockets/>`_ server.  It works with a few
    tweaks in the :mod:`~eventlet.wsgi` module that allow websockets to
    coexist with other WSGI applications.

    To create a websocket server, simply decorate a handler method with
    :class:`WebSocketWSGI` and use it as a wsgi application::

        from eventlet import wsgi, websocket
        import eventlet
        
        @websocket.WebSocketWSGI
        def hello_world(ws):
            ws.send("hello world")
        
        wsgi.server(eventlet.listen(('', 8090)), hello_world)

    .. note::

        Please see graceful termination warning in :func:`~eventlet.wsgi.server`
        documentation


    You can find a slightly more elaborate version of this code in the file
    ``examples/websocket.py``.

    As of version 0.9.13, eventlet.websocket supports SSL websockets; all that's necessary is to use an :ref:`SSL wsgi server <wsgi_ssl>`.

    .. note :: The web socket spec is still under development, and it will be necessary to change the way that this module works in response to spec changes.


.. automodule:: eventlet.websocket
	:members:
