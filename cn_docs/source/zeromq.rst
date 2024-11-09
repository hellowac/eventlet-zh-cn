Zeromq
######

什么是 ØMQ?
============

**What is ØMQ?**

.. tab:: 中文

    “ØMQ 套接字就像是你拿一个普通的 TCP 套接字，注入一些从一个秘密的苏联原子研究项目中偷来的放射性同位素，用 1950 年代的宇宙射线轰击它，再把它交到一个吸毒的漫画作者手中，这个作者有着一个用紧身衣掩饰不住的对膨胀肌肉的癖好。”

    与传统套接字的主要区别  
    一般来说，传统套接字提供的是一个同步接口，用于连接导向的可靠字节流（SOCK_STREAM）或无连接的、不可靠的数据报文（SOCK_DGRAM）。相比之下，0MQ 套接字提供的是一个异步消息队列的抽象，具体的排队语义取决于所使用的套接字类型。传统套接字传输的是字节流或离散的数据报文，而 0MQ 套接字传输的是离散的消息。

    0MQ 套接字是异步的，这意味着物理连接的建立和拆除、重新连接以及有效的消息传输的时间安排对用户是透明的，由 0MQ 自行管理。进一步说，当对端不可用时，消息可以被排队，直到对端可以接收为止。

    传统套接字只允许严格的一对一（两个对等方）、多对一（多个客户端，一个服务器）或在某些情况下一对多（多播）关系。除 ZMQ::PAIR 外，0MQ 套接字可以通过 `connect()` 连接多个端点，同时使用 `bind()` 接受来自多个端点的传入连接，因此可以实现多对多的关系。

.. tab:: 英文

    "A ØMQ socket is what you get when you take a normal TCP socket, inject it with a mix of radioactive isotopes stolen
    from a secret Soviet atomic research project, bombard it with 1950-era cosmic rays, and put it into the hands of a drug-addled
    comic book author with a badly-disguised fetish for bulging muscles clad in spandex."

    Key differences to conventional sockets
    Generally speaking, conventional sockets present a synchronous interface to either connection-oriented reliable byte streams (SOCK_STREAM),
    or connection-less unreliable datagrams (SOCK_DGRAM). In comparison, 0MQ sockets present an abstraction of an asynchronous message queue,
    with the exact queueing semantics depending on the socket type in use. Where conventional sockets transfer streams of bytes or discrete datagrams,
    0MQ sockets transfer discrete messages.

    0MQ sockets being asynchronous means that the timings of the physical connection setup and teardown,
    reconnect and effective delivery are transparent to the user and organized by 0MQ itself.
    Further, messages may be queued in the event that a peer is unavailable to receive them.

    Conventional sockets allow only strict one-to-one (two peers), many-to-one (many clients, one server),
    or in some cases one-to-many (multicast) relationships. With the exception of ZMQ::PAIR,
    0MQ sockets may be connected to multiple endpoints using connect(),
    while simultaneously accepting incoming connections from multiple endpoints bound to the socket using bind(), thus allowing many-to-many relationships.

API 文档
=================

**API documentation**

.. tab:: 中文

    :mod:`eventlet.green.zmq` 模块提供了 ØMQ 支持。

.. tab:: 英文

    ØMQ support is provided in the :mod:`eventlet.green.zmq` module.
