:mod:`db_pool` -- DBAPI 2 数据库连接池connection pooling
========================================================

:mod:`db_pool` **-- DBAPI 2 database**

.. tab:: 中文

	`db_pool` 模块在管理数据库连接方面非常有用。它提供了三个主要优点：在数据库操作期间协同让出、对数据库主机的并发限制以及连接复用。`db_pool` 旨在与数据库无关，兼容任何符合 DB-API 2.0 的数据库模块。

	*目前已测试并用于 MySQLdb 和 psycopg2。*

	`ConnectionPool` 对象表示与特定数据库保持连接的连接池。构造函数的参数包括数据库软件特定的模块、主机名和身份验证所需的凭证。构造完成后，`ConnectionPool` 对象会决定何时与目标数据库建立或断开连接。

	>>> import MySQLdb
	>>> cp = ConnectionPool(MySQLdb, host='localhost', user='root', passwd='')


	一旦拥有该池对象，可通过调用 :meth:`~eventlet.db_pool.ConnectionPool.get` 来连接数据库：

	>>> conn = cp.get()

	此调用可能会创建一个新连接，或重用一个现有的打开连接，这取决于池中是否已有打开的连接。然后可以像平常一样使用连接对象。在完成操作后，必须将连接返回到池中：

	>>> conn = cp.get()
	>>> try:
	...     result = conn.cursor().execute('SELECT NOW()')
	... finally:
	...     cp.put(conn)

	在将连接对象返回池中之后，该对象将变得无效，如果调用其任何方法都会引发异常。

.. tab:: 英文

	The db_pool module is useful for managing database connections.  It provides three primary benefits: cooperative yielding during database operations, concurrency limiting to a database host, and connection reuse.  db_pool is intended to be database-agnostic, compatible with any DB-API 2.0 database module.

	*It has currently been tested and used with both MySQLdb and psycopg2.*

	A ConnectionPool object represents a pool of connections open to a particular database.  The arguments to the constructor include the database-software-specific module, the host name, and the credentials required for authentication.  After construction, the ConnectionPool object decides when to create and sever connections with the target database.

	>>> import MySQLdb
	>>> cp = ConnectionPool(MySQLdb, host='localhost', user='root', passwd='')

	Once you have this pool object, you connect to the database by calling :meth:`~eventlet.db_pool.ConnectionPool.get` on it:

	>>> conn = cp.get()

	This call may either create a new connection, or reuse an existing open connection, depending on whether it has one open already or not.  You can then use the connection object as normal.  When done, you must return the connection to the pool:

	>>> conn = cp.get()
	>>> try:
	...     result = conn.cursor().execute('SELECT NOW()')
	... finally:
	...     cp.put(conn)

	After you've returned a connection object to the pool, it becomes useless and will raise exceptions if any of its methods are called.

构造函数参数
----------------------

**Constructor Arguments**

.. tab:: 中文

	除了数据库凭证之外，`ConnectionPool` 的构造函数还接受一系列有用的关键字参数。

	* `min_size`, `max_size`：常规的池参数。 `max_size` 是最重要的构造参数——它决定了可以与目标数据库同时打开的连接数量。 `min_size` 用途不大。
	* `max_idle`：连接仅允许在池中保持未使用状态的时间是有限的。一个异步计时器会定期唤醒并关闭池中那些超过允许闲置时间的连接。没有此参数时，池中的连接数量会达到“峰值水位”，即当前打开连接数对应历史上的最高需求。这一参数仅对池中的连接起作用——如果取出一个连接，可以随意保留使用。如果该参数设置为 0，每个连接在返回池时都会被关闭。
	* `max_age`：连接的生命周期。其工作方式类似于 `max_idle`，但计时器从连接创建时开始，并在整个连接生命周期内进行跟踪。这意味着，如果在池外占用了一个连接，并执行了一个超过 `max_age` 的长时间操作，那么在将连接放回池中时，它将被关闭。与 `max_idle` 类似，`max_age` 不会关闭池外的连接。如果设置为 0，每个连接在返回池时都会被关闭。
	* `connect_timeout`：在 `connect()` 超时时等待的时间。如果数据库模块的 `connect()` 方法耗时过长，池的 `get()` 方法会抛出 `ConnectTimeout` 异常。

.. tab:: 英文

	In addition to the database credentials, there are a bunch of keyword constructor arguments to the ConnectionPool that are useful.

	* min_size, max_size : The normal Pool arguments.  max_size is the most important constructor argument -- it determines the number of concurrent connections can be open to the destination database.  min_size is not very useful.
	* max_idle : Connections are only allowed to remain unused in the pool for a limited amount of time.  An asynchronous timer periodically wakes up and closes any connections in the pool that have been idle for longer than they are supposed to be.  Without this parameter, the pool would tend to have a 'high-water mark', where the number of connections open at a given time corresponds to the peak historical demand.  This number only has effect on the connections in the pool itself -- if you take a connection out of the pool, you can hold on to it for as long as you want.  If this is set to 0, every connection is closed upon its return to the pool.
	* max_age : The lifespan of a connection.  This works much like max_idle, but the timer is measured from the connection's creation time, and is tracked throughout the connection's life.  This means that if you take a connection out of the pool and hold on to it for some lengthy operation that exceeds max_age, upon putting the connection back in to the pool, it will be closed.  Like max_idle, max_age will not close connections that are taken out of the pool, and, if set to 0, will cause every connection to be closed when put back in the pool.
	* connect_timeout : How long to wait before raising an exception on connect().  If the database module's connect() method takes too long, it raises a ConnectTimeout exception from the get() method on the pool.

DatabaseConnector
-----------------

**DatabaseConnector**

.. tab:: 中文

	如果你想轻松连接到多个数据库（谁不想呢），`DatabaseConnector` 就是为你准备的。它是一个池的池，包含了每个你连接的主机的 `ConnectionPool`。

	构造函数的参数如下：

	* `module`：数据库模块，例如 MySQLdb。它会直接传递给 `ConnectionPool`。
	* `credentials`：一个字典或类似字典的对象，映射主机名到连接参数字典。它用于 `ConnectionPool` 对象的构造函数。例如：

	>>> dc = DatabaseConnector(MySQLdb,
	...      {'db.internal.example.com': {'user': 'internal', 'passwd': 's33kr1t'},
	...       'localhost': {'user': 'root', 'passwd': ''}})

	如果凭证中包含一个名为 'default' 的主机，那么在尝试连接没有明确条目的主机时，会使用 'default' 的值。这在有一些共享参数的主机池中非常有用。

	* `conn_pool`：要使用的连接池类。默认为 `db_pool.ConnectionPool`。

	`DatabaseConnector` 构造函数的其余参数会传递给 `ConnectionPool`。

	*警告：`DatabaseConnector` 还有点不完整，它只适用于一部分使用场景。*

.. tab:: 英文

	If you want to connect to multiple databases easily (and who doesn't), the DatabaseConnector is for you.  It's a pool of pools, containing a ConnectionPool for every host you connect to.

	The constructor arguments are:

	* module : database module, e.g. MySQLdb.  This is simply passed through to the ConnectionPool.
	* credentials : A dictionary, or dictionary-alike, mapping hostname to connection-argument-dictionary.  This is used for the constructors of the ConnectionPool objects.  Example:

	>>> dc = DatabaseConnector(MySQLdb,
	...      {'db.internal.example.com': {'user': 'internal', 'passwd': 's33kr1t'},
	...       'localhost': {'user': 'root', 'passwd': ''}})

	If the credentials contain a host named 'default', then the value for 'default' is used whenever trying to connect to a host that has no explicit entry in the database.  This is useful if there is some pool of hosts that share arguments.

	* conn_pool : The connection pool class to use.  Defaults to db_pool.ConnectionPool.

	The rest of the arguments to the DatabaseConnector constructor are passed on to the ConnectionPool.

	*Caveat: The DatabaseConnector is a bit unfinished, it only suits a subset of use cases.*

.. automodule:: eventlet.db_pool
	:members:
	:undoc-members:
