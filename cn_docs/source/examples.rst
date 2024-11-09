示例
========

**Examples**

.. tab:: 中文

    这里有一些使用 Eventlet 的小示例程序。所有这些示例都可以在 Eventlet 源副本的``examples`` 目录中找到。

.. tab:: 英文

    Here are a bunch of small example programs that use Eventlet.  All of these examples can be found in the ``examples`` directory of a source copy of Eventlet.

.. _web_crawler_example:

Web 爬虫
------------

**Web Crawler**

``examples/webcrawler.py``

.. literalinclude:: ../../examples/webcrawler.py

.. _wsgi_server_example:

WSGI 服务器
------------

**WSGI Server**

``examples/wsgi.py``

.. literalinclude:: ../../examples/wsgi.py

.. _echo_server_example:

Echo 服务器
-----------

**Echo Server**

``examples/echoserver.py``

.. literalinclude:: ../../examples/echoserver.py

.. _socket_connect_example:

Socket Connect
--------------

**Socket Connect**

``examples/connect.py``

.. literalinclude:: ../../examples/connect.py

.. _chat_server_example:

多用户聊天服务器
-----------------------

**Multi-User Chat Server**

``examples/chat_server.py``

.. tab:: 中文

    这与回显服务器略有不同，因为它将消息广播给所有参与者，而不仅仅是发送者。

.. tab:: 英文

    This is a little different from the echo server, in that it broadcasts the 
    messages to all participants, not just the sender.
        
.. literalinclude:: ../../examples/chat_server.py

.. _feed_scraper_example:

Feed Scraper
-----------------------

**Feed Scraper**

``examples/feedscraper.py``

.. tab:: 中文

    此示例需要安装 `Feedparser <http://www.feedparser.org/>`_ 或将其放在 PYTHONPATH 上。

.. tab:: 英文

    This example requires `Feedparser <http://www.feedparser.org/>`_ to be installed or on the PYTHONPATH.

.. literalinclude:: ../../examples/feedscraper.py

.. _forwarder_example:

端口转发器
-----------------------

**Port Forwarder**

``examples/forwarder.py``

.. literalinclude:: ../../examples/forwarder.py

.. _recursive_crawler_example:

递归 Web 爬虫
-----------------------------------------

**Recursive Web Crawler**

``examples/recursive_crawler.py``

.. tab:: 中文

    这是一个递归网络爬虫的示例，它从种子 URL 获取链接页面。

.. tab:: 英文

    This is an example recursive web crawler that fetches linked pages from a seed url.

.. literalinclude:: ../../examples/recursive_crawler.py

.. _producer_consumer_example:

生产者消费者 Web 爬虫
-----------------------------------------

**Producer Consumer Web Crawler**

``examples/producer_consumer.py``

.. tab:: 中文

    这是生产者/消费者模式的示例实现，其功能与递归网络爬虫相同。

.. tab:: 英文

    This is an example implementation of the producer/consumer pattern as well as being identical in functionality to the recursive web crawler.

.. literalinclude:: ../../examples/producer_consumer.py

.. _websocket_example:

Websocket 服务器示例
--------------------------

**Websocket Server Example**

``examples/websocket.py``

.. tab:: 中文

    这实践了 websocket 服务器实现的一些功能。

.. tab:: 英文

    This exercises some of the features of the websocket server
    implementation.

.. literalinclude:: ../../examples/websocket.py

.. _websocket_chat_example:

Websocket 多用户聊天示例
-----------------------------------

**Websocket Multi-User Chat Example**

``examples/websocket_chat.py``

.. tab:: 中文

    这是 websocket 示例和多用户聊天示例的混合，展示了如何使用 websocket 执行与常规套接字相同的操作。

.. tab:: 英文

    This is a mashup of the websocket example and the multi-user chat example, showing how you can do the same sorts of things with websockets that you can do with regular sockets.

.. literalinclude:: ../../examples/websocket_chat.py
