:mod:`wsgi` -- WSGI 服务器
===========================

:mod:`wsgi` **-- WSGI server**

.. tab:: 中文

    wsgi 模块提供了一种简单易用的方法来启动一个基于事件驱动的 `WSGI <http://wsgi.org/wsgi/>`_ 服务器。这可以作为应用程序中的嵌入式 Web 服务器，或者作为更完整功能的 Web 服务器包的基础。一个这样的包是 `Spawning <http://pypi.python.org/pypi/Spawning/>`_。

    要启动一个 wsgi 服务器，只需创建一个套接字并调用 :func:`eventlet.wsgi.server` 进行启动::

        from eventlet import wsgi
        import eventlet

        def hello_world(env, start_response):
            start_response('200 OK', [('Content-Type', 'text/plain')])
            return ['Hello, World!\r\n']

        wsgi.server(eventlet.listen(('', 8090)), hello_world)

    你可以在文件 ``examples/wsgi.py`` 中找到这个代码的稍微复杂版本。

.. tab:: 英文

    The wsgi module provides a simple and easy way to start an event-driven
    `WSGI <http://wsgi.org/wsgi/>`_ server.  This can serve as an embedded
    web server in an application, or as the basis for a more full-featured web
    server package.  One such package is `Spawning <http://pypi.python.org/pypi/Spawning/>`_.

    To launch a wsgi server, simply create a socket and call :func:`eventlet.wsgi.server` with it::

        from eventlet import wsgi
        import eventlet

        def hello_world(env, start_response):
            start_response('200 OK', [('Content-Type', 'text/plain')])
            return ['Hello, World!\r\n']

        wsgi.server(eventlet.listen(('', 8090)), hello_world)


    You can find a slightly more elaborate version of this code in the file
    ``examples/wsgi.py``.

.. automodule:: eventlet.wsgi
	:members:

.. _wsgi_ssl:

SSL
---

.. tab:: 中文

    创建一个安全的服务器只比基本示例稍微复杂一些。所需的仅仅是将一个 SSL 包装的套接字传递给 :func:`~eventlet.wsgi.server` 方法::

        wsgi.server(eventlet.wrap_ssl(eventlet.listen(('', 8090)),
                                    certfile='cert.crt',
                                    keyfile='private.key',
                                    server_side=True),
                    hello_world)

    应用程序可以通过 ``env['wsgi.url_scheme']`` 环境变量的值来检测它们是否处于安全服务器内。

.. tab:: 英文

    Creating a secure server is only slightly more involved than the base example.  All that's needed is to pass an SSL-wrapped socket to the :func:`~eventlet.wsgi.server` method::

        wsgi.server(eventlet.wrap_ssl(eventlet.listen(('', 8090)),
                                    certfile='cert.crt',
                                    keyfile='private.key',
                                    server_side=True),
                    hello_world)

    Applications can detect whether they are inside a secure server by the value of the ``env['wsgi.url_scheme']`` environment variable.


支持 Post Hooks 的非标准扩展
--------------------------------------------

**Non-Standard Extension to Support Post Hooks**

.. tab:: 中文

    Eventlet 的 WSGI 服务器支持 WSGI 规范的非标准扩展，其中 :samp:`env['eventlet.posthooks']` 包含一个 `post hooks` 数组，这些钩子会在完全发送响应后被调用。每个 post hook 是一个 :samp:`(func, args, kwargs)` 元组，`func` 会在 WSGI 环境字典之后依次接收 `args` 和 `kwargs`。

    例如::

        from eventlet import wsgi
        import eventlet

        def hook(env, arg1, arg2, kwarg3=None, kwarg4=None):
            print('Hook called: %s %s %s %s %s' % (env, arg1, arg2, kwarg3, kwarg4))

        def hello_world(env, start_response):
            env['eventlet.posthooks'].append(
                (hook, ('arg1', 'arg2'), {'kwarg3': 3, 'kwarg4': 4}))
            start_response('200 OK', [('Content-Type', 'text/plain')])
            return ['Hello, World!\r\n']

        wsgi.server(eventlet.listen(('', 8090)), hello_world)

    上述代码将在处理每个请求时打印 WSGI 环境以及其他传递的函数参数。

    Post hooks 在需要在响应完全发送给客户端后执行代码时非常有用（或者当客户端提前断开连接时）。一个例子是更准确地记录带宽使用情况，因为客户端断开连接时使用的带宽比实际的 Content-Length 要少。

.. tab:: 英文

    Eventlet's WSGI server supports a non-standard extension to the WSGI
    specification where :samp:`env['eventlet.posthooks']` contains an array of
    `post hooks` that will be called after fully sending a response. Each post hook
    is a tuple of :samp:`(func, args, kwargs)` and the `func` will be called with
    the WSGI environment dictionary, followed by the `args` and then the `kwargs`
    in the post hook.

    For example::

        from eventlet import wsgi
        import eventlet

        def hook(env, arg1, arg2, kwarg3=None, kwarg4=None):
            print('Hook called: %s %s %s %s %s' % (env, arg1, arg2, kwarg3, kwarg4))

        def hello_world(env, start_response):
            env['eventlet.posthooks'].append(
                (hook, ('arg1', 'arg2'), {'kwarg3': 3, 'kwarg4': 4}))
            start_response('200 OK', [('Content-Type', 'text/plain')])
            return ['Hello, World!\r\n']

        wsgi.server(eventlet.listen(('', 8090)), hello_world)

    The above code will print the WSGI environment and the other passed function
    arguments for every request processed.

    Post hooks are useful when code needs to be executed after a response has been
    fully sent to the client (or when the client disconnects early). One example is
    for more accurate logging of bandwidth used, as client disconnects use less
    bandwidth than the actual Content-Length.


“100 Continue”响应标头
-------------------------------

**"100 Continue" Response Headers**

.. tab:: 中文

    Eventlet 的 WSGI 服务器支持在 HTTP "100 Continue" 临时响应中发送（可选）头部。这在 WSGI 服务器希望将 PUT 请求作为单一的 HTTP 请求/响应对完成时非常有用，同时还希望在同一个 HTTP 事务中与客户端进行通信。一个例子是，HTTP 服务器可能希望通过头部向客户端传递关于其能够接受的数据负载特征的提示。例如，HTTP 服务器可能会在随附的 "100 Continue" 响应中通过头部传递一个提示，表示它是否能够接受加密的数据负载，从而使客户端能够在开始发送数据之前做出加密与未加密的选择。

    这对于 WSGI 服务器来说是有效的，因为 WSGI 规范要求 HTTP 的 expect/continue 机制（PEP333）。

    要定义 "100 Continue" 响应头，可以调用 :func:`set_hundred_continue_response_header` 在 :samp:`env['wsgi.input']` 上，如下所示的示例::

        from eventlet import wsgi
        import eventlet

        def wsgi_app(env, start_response):
            # 定义 "100 Continue" 响应头
            env['wsgi.input'].set_hundred_continue_response_headers(
                [('Hundred-Continue-Header-1', 'H1'),
                ('Hundred-Continue-Header-k', 'Hk')])
            # 以下的 read() 会导致向客户端发送 "100 Continue" 响应。
            # 头部 'Hundred-Continue-Header-1' 和 'Hundred-Continue-Header-K' 会随着响应
            # 在 "HTTP/1.1 100 Continue\r\n" 状态行之后发送
            text = env['wsgi.input'].read()
            start_response('200 OK', [('Content-Length', str(len(text)))])
            return [text]

    您可以在文件 ``tests/wsgi_test.py`` 中找到一个更详细的示例，函数 :func:`test_024a_expect_100_continue_with_headers`。

    根据 HTTP RFC 7231（http://tools.ietf.org/html/rfc7231#section-6.2），客户端需要能够处理一个或多个 100 continue 响应。一个典型的使用案例可能是一个用户协议，其中服务器可能希望使用 100-continue 响应来通知客户端它正在处理请求，客户端不应超时。

    为了支持多个 100-continue 响应，eventlet wsgi 模块导出了 API :func:`send_hundred_continue_response`。

    对于分块和非分块的 HTTP 场景，示例使用案例包含在 wsgi 测试用例 ``tests/wsgi_test.py`` 中，函数 :func:`test_024b_expect_100_continue_with_headers_multiple_chunked` 和 :func:`test_024c_expect_100_continue_with_headers_multiple_nonchunked`。

.. tab:: 英文

    Eventlet's WSGI server supports sending (optional) headers with HTTP "100 Continue"
    provisional responses.  This is useful in such cases where a WSGI server expects
    to complete a PUT request as a single HTTP request/response pair, and also wants to
    communicate back to client as part of the same HTTP transaction.  An example is
    where the HTTP server wants to pass hints back to the client about characteristics
    of data payload it can accept.  As an example, an HTTP server may pass a hint in a
    header the accompanying "100 Continue" response to the client indicating it can or
    cannot accept encrypted data payloads, and thus client can make the encrypted vs
    unencrypted decision before starting to send the data).  

    This works well for WSGI servers as the WSGI specification mandates HTTP
    expect/continue mechanism (PEP333).

    To define the "100 Continue" response headers, one may call
    :func:`set_hundred_continue_response_header` on :samp:`env['wsgi.input']`
    as shown in the following example::

        from eventlet import wsgi
        import eventlet

        def wsgi_app(env, start_response):
            # Define "100 Continue" response headers
            env['wsgi.input'].set_hundred_continue_response_headers(
                [('Hundred-Continue-Header-1', 'H1'),
                ('Hundred-Continue-Header-k', 'Hk')])
            # The following read() causes "100 Continue" response to
            # the client.  Headers 'Hundred-Continue-Header-1' and 
            # 'Hundred-Continue-Header-K' are sent with the response
            # following the "HTTP/1.1 100 Continue\r\n" status line
            text = env['wsgi.input'].read()
            start_response('200 OK', [('Content-Length', str(len(text)))])
            return [text]

    You can find a more elaborate example in the file:
    ``tests/wsgi_test.py``, :func:`test_024a_expect_100_continue_with_headers`.


    Per HTTP RFC 7231 (http://tools.ietf.org/html/rfc7231#section-6.2) a client is
    required to be able to process one or more 100 continue responses.  A sample
    use case might be a user protocol where the server may want to use a 100-continue
    response to indicate to a client that it is working on a request and the 
    client should not timeout.

    To support multiple 100-continue responses, evenlet wsgi module exports
    the API :func:`send_hundred_continue_response`.

    Sample use cases for chunked and non-chunked HTTP scenarios are included
    in the wsgi test case ``tests/wsgi_test.py``,
    :func:`test_024b_expect_100_continue_with_headers_multiple_chunked` and
    :func:`test_024c_expect_100_continue_with_headers_multiple_nonchunked`.

