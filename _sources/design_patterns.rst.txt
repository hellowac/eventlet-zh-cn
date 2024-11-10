.. _design-patterns:

设计模式
=================

**Design Patterns**

.. tab:: 中文

    Eventlet 的使用有很多基本模式。下面是一些展示其基本结构的示例。

.. tab:: 英文

    There are a bunch of basic patterns that Eventlet usage falls into.  Here are a few examples that show their basic structure.

客户端模式
--------------------

**Client Pattern**

.. tab:: 中文

    典型的客户端示例是一个网络爬虫。该用例接收一个 URL 列表，并希望获取它们的页面内容以便后续处理。以下是一个非常简单的示例::

        import eventlet
        from eventlet.green.urllib.request import urlopen

        urls = ["http://www.google.com/intl/en_ALL/images/logo.gif",
                "https://www.python.org/static/img/python-logo.png",
                "http://us.i1.yimg.com/us.yimg.com/i/ww/beta/y3.gif"]

        def fetch(url):
            return urlopen(url).read()

        pool = eventlet.GreenPool()
        for body in pool.imap(fetch, urls):
            print("got body", len(body))

    在 :ref:`web crawler example <web_crawler_example>` 中可以找到一个稍微复杂一点的版本。以下是此爬虫代码中一些有趣的行的解释。

    ``from eventlet.green... import urlopen`` 是导入 urllib 的协作让步版本的方式。除了使用绿色套接字进行通信外，其余部分与标准版本完全相同。这是 :ref:`import-green` 模式的一个示例。

    ``pool = eventlet.GreenPool()`` 构建了一个包含一千个绿色线程的 :class:`GreenPool <eventlet.greenpool.GreenPool>`。使用线程池是一种良好的实践，因为它为爬虫同时进行的任务数量提供了上限，这在输入数据剧烈变化时非常有用。

    ``for body in pool.imap(fetch, urls):`` 用于并行迭代调用 fetch 函数的结果。:meth:`imap <eventlet.greenpool.GreenPool.imap>` 以并行方式进行函数调用，并按执行顺序返回结果。

    客户端模式的关键方面是它收集每次函数调用的结果；而每次 fetch 是并发完成的，这本质上是一种隐形优化。同样需要注意的是，imap 是有内存限制的，当 URL 列表增长到成千上万时，它不会消耗数 GB 的内存（是的，我们在生产环境中确实遇到过这个问题！）。

.. tab:: 英文

    The canonical client-side example is a web crawler.  This use case is given a list of urls and wants to retrieve their bodies for later processing.  Here is a very simple example::

        import eventlet
        from eventlet.green.urllib.request import urlopen

        urls = ["http://www.google.com/intl/en_ALL/images/logo.gif",
            "https://www.python.org/static/img/python-logo.png",
            "http://us.i1.yimg.com/us.yimg.com/i/ww/beta/y3.gif"]

        def fetch(url):
            return urlopen(url).read()

        pool = eventlet.GreenPool()
        for body in pool.imap(fetch, urls):
            print("got body", len(body))

    There is a slightly more complex version of this in the :ref:`web crawler example <web_crawler_example>`.  Here's a tour of the interesting lines in this crawler.

    ``from eventlet.green... import urlopen`` is how you import a cooperatively-yielding version of urllib.  It is the same in all respects to the standard version, except that it uses green sockets for its communication.  This is an example of the :ref:`import-green` pattern.

    ``pool = eventlet.GreenPool()`` constructs a :class:`GreenPool <eventlet.greenpool.GreenPool>` of a thousand green threads.  Using a pool is good practice because it provides an upper limit on the amount of work that this crawler will be doing simultaneously, which comes in handy when the input data changes dramatically.

    ``for body in pool.imap(fetch, urls):`` iterates over the results of calling the fetch function in parallel.  :meth:`imap <eventlet.greenpool.GreenPool.imap>` makes the function calls in parallel, and the results are returned in the order that they were executed.

    The key aspect of the client pattern is that it involves collecting the results of each function call; the fact that each fetch is done concurrently is essentially an invisible optimization.  Note also that imap is memory-bounded and won't consume gigabytes of memory if the list of urls grows to the tens of thousands (yes, we had that problem in production once!).


服务器模式
--------------------

**Server Pattern**

.. tab:: 中文

    这是一个简单的服务器端示例，一个简单的回显服务器::

        import eventlet

        def handle(client):
            while True:
                c = client.recv(1)
                if not c: break
                client.sendall(c)

        server = eventlet.listen(('0.0.0.0', 6000))
        pool = eventlet.GreenPool(10000)
        while True:
            new_sock, address = server.accept()
            pool.spawn_n(handle, new_sock)

    文件 :ref:`echo server example <echo_server_example>` 包含该示例的一个更健壮且更复杂的版本。

    ``server = eventlet.listen(('0.0.0.0', 6000))`` 使用了一个便捷函数来创建监听套接字。

    ``pool = eventlet.GreenPool(10000)`` 创建了一个包含一万个绿色线程的线程池，可以处理一万个客户端连接。

    ``pool.spawn_n(handle, new_sock)`` 启动一个绿色线程来处理新的客户端。由于接受循环并不关心 ``handle`` 函数的返回值，因此使用了 :meth:`spawn_n <eventlet.greenpool.GreenPool.spawn_n>` 而不是 :meth:`spawn <eventlet.greenpool.GreenPool.spawn>`。

    服务器和客户端模式的区别主要在于服务器有一个不断调用 ``accept()`` 的 ``while`` 循环，并且完全将客户端套接字交给了 handle() 方法，而不是收集其返回结果。

.. tab:: 英文

    Here's a simple server-side example, a simple echo server::

        import eventlet

        def handle(client):
            while True:
                c = client.recv(1)
                if not c: break
                client.sendall(c)

        server = eventlet.listen(('0.0.0.0', 6000))
        pool = eventlet.GreenPool(10000)
        while True:
            new_sock, address = server.accept()
            pool.spawn_n(handle, new_sock)

    The file :ref:`echo server example <echo_server_example>` contains a somewhat more robust and complex version of this example.

    ``server = eventlet.listen(('0.0.0.0', 6000))`` uses a convenience function to create a listening socket.

    ``pool = eventlet.GreenPool(10000)`` creates a pool of green threads that could handle ten thousand clients.

    ``pool.spawn_n(handle, new_sock)`` launches a green thread to handle the new client.  The accept loop doesn't care about the return value of the ``handle`` function, so it uses :meth:`spawn_n <eventlet.greenpool.GreenPool.spawn_n>`, instead of :meth:`spawn <eventlet.greenpool.GreenPool.spawn>`.

    The difference between the server and the client patterns boils down to the fact that the server has a ``while`` loop calling ``accept()`` repeatedly, and that it hands off the client socket completely to the handle() method, rather than collecting the results.

调度模式
-------------------

**Dispatch Pattern**

.. tab:: 中文

    Linden Lab 常遇到的一个常见用例是“分派”设计模式。这是一种既是服务器又是其他服务客户端的模式。代理、聚合器、任务处理器等都是适用的术语。这也是 :class:`GreenPile <eventlet.greenpool.GreenPile>` 设计时所考虑的用例。

    这是一个稍微人为的示例：服务器接收包含 RSS 源 URL 列表的客户端 POST 请求。服务器并发获取所有源并返回标题列表给客户端。可以轻松将其改造成一个类似 Reader 的应用::

        import eventlet
        feedparser = eventlet.import_patched('feedparser')

        pool = eventlet.GreenPool()

        def fetch_title(url):
            d = feedparser.parse(url)
            return d.feed.get('title', '')

        def app(environ, start_response):
            pile = eventlet.GreenPile(pool)
            for url in environ['wsgi.input'].readlines():
                pile.spawn(fetch_title, url)
            titles = '\n'.join(pile)
            start_response('200 OK', [('Content-type', 'text/plain')])
            return [titles]

    此示例的完整版本位于 :ref:`feed_scraper_example`，包括启动特定端口上的 WSGI 服务器的代码。

    此示例使用一个全局（哎呀）:class:`GreenPool <eventlet.greenpool.GreenPool>` 来控制并发性。如果没有对外部请求数量的全局限制，客户端可能会导致服务器打开数万个到外部服务器的并发连接，进而导致 feedscraper 的 IP 被封禁，或引发其他意外或恶意的行为。线程池并不是完全的 DoS 保护，但至少是最低限度的防护。

    .. highlight:: python
        :linenothreshold: 1

    有趣的行在 app 函数中::

        pile = eventlet.GreenPile(pool)
        for url in environ['wsgi.input'].readlines():
            pile.spawn(fetch_title, url)
        titles = '\n'.join(pile)

    .. highlight:: python
        :linenothreshold: 1000

    请注意，在第 1 行中，Pile 是使用全局池作为参数构建的。这将 Pile 的并发性绑定到全局池。如果其他 feedscraper 客户端已经有 1000 个并发请求，则该请求将阻塞，直到某些请求完成。限制是有好处的！

    第 3 行只是一个 spawn 操作，但请注意，我们没有存储其返回值。这是因为返回值保存在 Pile 本身中。这在下一行中显而易见...

    第 4 行使用了 Pile 是一个迭代器的特性。迭代器中的每个元素都是 fetch_title 函数的一个返回值（字符串）。我们可以使用一个常见的 Python 表达式（:func:`join`）来按顺序连接这些返回值。

.. tab:: 英文

    One common use case that Linden Lab runs into all the time is a "dispatch" design pattern.  This is a server that is also a client of some other services.  Proxies, aggregators, job workers, and so on are all terms that apply here.  This is the use case that the :class:`GreenPile <eventlet.greenpool.GreenPile>` was designed for.

    Here's a somewhat contrived example: a server that receives POSTs from clients that contain a list of urls of RSS feeds.  The server fetches all the feeds concurrently and responds with a list of their titles to the client.  It's easy to imagine it doing something more complex than this, and this could be easily modified to become a Reader-style application::

        import eventlet
        feedparser = eventlet.import_patched('feedparser')

        pool = eventlet.GreenPool()

        def fetch_title(url):
            d = feedparser.parse(url)
            return d.feed.get('title', '')

        def app(environ, start_response):
            pile = eventlet.GreenPile(pool)
            for url in environ['wsgi.input'].readlines():
                pile.spawn(fetch_title, url)
            titles = '\n'.join(pile)
            start_response('200 OK', [('Content-type', 'text/plain')])
            return [titles]

    The full version of this example is in the :ref:`feed_scraper_example`, which includes code to start the WSGI server on a particular port.

    This example uses a global (gasp) :class:`GreenPool <eventlet.greenpool.GreenPool>` to control concurrency.  If we didn't have a global limit on the number of outgoing requests, then a client could cause the server to open tens of thousands of concurrent connections to external servers, thereby getting feedscraper's IP banned, or various other accidental-or-on-purpose bad behavior.  The pool isn't a complete DoS protection, but it's the bare minimum.

    .. highlight:: python
        :linenothreshold: 1

    The interesting lines are in the app function::

        pile = eventlet.GreenPile(pool)
        for url in environ['wsgi.input'].readlines():
            pile.spawn(fetch_title, url)
        titles = '\n'.join(pile)

    .. highlight:: python
        :linenothreshold: 1000

    Note that in line 1, the Pile is constructed using the global pool as its argument.  That ties the Pile's concurrency to the global's.  If there are already 1000 concurrent fetches from other clients of feedscraper, this one will block until some of those complete.  Limitations are good!

    Line 3 is just a spawn, but note that we don't store any return value from it.  This is because the return value is kept in the Pile itself.  This becomes evident in the next line...

    Line 4 is where we use the fact that the Pile is an iterator.  Each element in the iterator is one of the return values from the fetch_title function, which are strings.  We can use a normal Python idiom (:func:`join`) to concatenate these incrementally as they happen.
