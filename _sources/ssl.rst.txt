SSL 与 Eventlet 的结合使用
==============================

**Using SSL With Eventlet**

.. tab:: 中文

    Eventlet 使得使用非阻塞 SSL 套接字变得非常容易。如果你使用的是 Python 3.7 或更高版本，系统已经配置好了，Eventlet 会包装内置的 ssl 模块。

    无论是哪种情况， ``green`` 模块都可以透明地处理 SSL 套接字，就像它们的标准模块一样。举个例子，:mod:`eventlet.green.urllib2` 可以用来以非阻塞的方式获取 https url，如下所示::

        from eventlet.green.urllib.request import urlopen
        from eventlet import spawn
        bodies = [spawn(urlopen, url)
            for url in ("https://secondlife.com","https://google.com")]
        for b in bodies:
            print(b.wait().read())

.. tab:: 英文

    Eventlet makes it easy to use non-blocking SSL sockets. If you're using Python 3.7 or later, you're all set, eventlet wraps the built-in ssl module.

    In either case, the ``green`` modules handle SSL sockets transparently, just like their standard counterparts.  As an example, :mod:`eventlet.green.urllib2` can be used to fetch https urls in as non-blocking a fashion as you please::

        from eventlet.green.urllib.request import urlopen
        from eventlet import spawn
        bodies = [spawn(urlopen, url)
            for url in ("https://secondlife.com","https://google.com")]
        for b in bodies:
            print(b.wait().read())


PyOpenSSL
----------

.. tab:: 中文

    :mod:`eventlet.green.OpenSSL` 与 pyOpenSSL_ `(docs) <http://pyopenssl.sourceforge.net/pyOpenSSL.html/>`_ 拥有完全相同的接口，并且适用于所有版本的 Python。这个模块比 :func:`socket.ssl` 更强大，根据你的需求，它可能在某些方面比 :mod:`ssl` 更有优势。

    为了测试，首先使用以下命令创建自签名证书 ::

        $ openssl genrsa 1024 > server.key
        $ openssl req -new -x509 -nodes -sha1 -days 365 -key server.key > server.cert

    将私钥和自签名证书保存在与 `server.py` 和 `client.py` 相同的目录中，方便起见。

    以下是一个服务器的示例（`server.py`）::

        from eventlet.green import socket
        from eventlet.green.OpenSSL import SSL

        # 不安全的上下文，仅供示例使用
        context = SSL.Context(SSL.SSLv23_METHOD)
        # 使用服务器私钥
        context.use_privatekey_file('server.key')
        # 使用自签名证书
        context.use_certificate_file('server.cert')

        # 创建底层绿色套接字并用 ssl 包裹它
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        connection = SSL.Connection(context, sock)

        # 配置为服务器
        connection.set_accept_state()
        connection.bind(('127.0.0.1', 8443))
        connection.listen(50)

        # 接受一个客户端连接，然后关闭
        client_conn, addr = connection.accept()
        print(client_conn.read(100))
        client_conn.shutdown()
        client_conn.close()
        connection.close()

    以下是一个客户端的示例（`client.py`）::

        import socket
        # 创建套接字
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        # 连接到服务器
        s.connect(('127.0.0.1', 8443))
        sslSocket = socket.ssl(s)
        print repr(sslSocket.server())
        print repr(sslSocket.issuer())
        sslSocket.write('Hello secure socket\n')
        # 关闭客户端
        s.close()

    运行示例::

    在第一个终端中

        $ python server.py

    在另一个终端中

        $ python client.py

.. tab:: 英文

    :mod:`eventlet.green.OpenSSL` has exactly the same interface as pyOpenSSL_ `(docs) <http://pyopenssl.sourceforge.net/pyOpenSSL.html/>`_, and works in all versions of Python.  This module is much more powerful than :func:`socket.ssl`, and may have some advantages over :mod:`ssl`, depending on your needs.

    For testing purpose first create self-signed certificate using following commands ::

        $ openssl genrsa 1024 > server.key
        $ openssl req -new -x509 -nodes -sha1 -days 365 -key server.key > server.cert

    Keep these Private key and Self-signed certificate in same directory as `server.py` and `client.py` for simplicity sake.

    Here's an example of a server (`server.py`) ::

        from eventlet.green import socket
        from eventlet.green.OpenSSL import SSL

        # insecure context, only for example purposes
        context = SSL.Context(SSL.SSLv23_METHOD)
        # Pass server's private key created
        context.use_privatekey_file('server.key')
        # Pass self-signed certificate created
        context.use_certificate_file('server.cert')

        # create underlying green socket and wrap it in ssl
        sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        connection = SSL.Connection(context, sock)

        # configure as server
        connection.set_accept_state()
        connection.bind(('127.0.0.1', 8443))
        connection.listen(50)

        # accept one client connection then close up shop
        client_conn, addr = connection.accept()
        print(client_conn.read(100))
        client_conn.shutdown()
        client_conn.close()
        connection.close()

    Here's an example of a client (`client.py`) ::

        import socket
        # Create socket
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        # Connect to server
        s.connect(('127.0.0.1', 8443))
        sslSocket = socket.ssl(s)
        print repr(sslSocket.server())
        print repr(sslSocket.issuer())
        sslSocket.write('Hello secure socket\n')
        # Close client
        s.close()

    Running example::

    In first terminal

        $ python server.py

    In another terminal

        $ python client.py

.. _pyOpenSSL: https://launchpad.net/pyopenssl
