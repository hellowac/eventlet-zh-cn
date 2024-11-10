:mod:`timeout` -- 通用超时
========================================

:mod:`timeout` **-- Universal Timeouts**

.. tab:: 中文

    .. class:: eventlet.timeout.Timeout

        在当前 greenthread 中，在 *timeout* 秒后抛出 *exception*::

            timeout = Timeout(seconds, exception)
            try:
                ... # 执行代码受限于超时时间
            finally:
                timeout.cancel()

        当 *exception* 被省略或为 ``None`` 时，抛出的是 :class:`Timeout` 实例本身：

            >>> Timeout(0.1)
            >>> eventlet.sleep(0.2)
            Traceback (most recent call last):
            ...
            Timeout: 0.1 seconds

        你可以使用 ``with`` 语句来增加便利性::

            with Timeout(seconds, exception) as timeout:
                pass # ... 代码块 ...

        这相当于第一个例子中的 try/finally 语句块。

        使用 ``with`` 语句时还有一个附加功能：如果 *exception* 为 ``False``，超时仍然会被触发，但 ``with`` 语句会抑制它，因此外部代码块不会看到该异常::

            data = None
            with Timeout(5, False):
                data = mysock.makefile().readline()
            if data is None:
                ... # 5 秒内没有读取到一行
            else:
                ... # 5 秒内读取到了一行

        作为一个非常特殊的情况，如果 *seconds* 为 None，计时器不会被调度，只有在你计划直接抛出它时，它才有用。

        有两个关于 Timeout 的注意事项需要了解：

        * 如果 try/finally 或 with 块中的代码块没有合作性地让出控制权，超时将无法触发。在 Eventlet 中，这通常不会成为问题，但需要注意的是，你不能使用此类超时来处理仅依赖 CPU 的操作。
        * 如果代码块捕获了异常且没有重新抛出 :class:`BaseException`（例如使用 ``except:``），它将捕获 Timeout 异常，并可能无法按预期中止。

        在捕获超时时，请记住，你捕获的可能不是你设置的超时；如果你计划屏蔽一个超时，请始终检查它是否是你设置的同一个实例::

            timeout = Timeout(1)
            try:
                ...
            except Timeout as t:
                if t is not timeout:
                    raise # 不是我设置的超时

        .. automethod:: cancel
        .. autoattribute:: pending


    .. function:: eventlet.timeout.with_timeout(seconds, function, *args, **kwds)

        为某个（可协作的）函数调用加上超时；如果该函数未在超时前返回，将取消该函数并返回一个标志值。

        :param seconds: 超时发生前的秒数
        :type seconds: int 或 float
        :param func: 要执行的可调用对象；它必须协作性地让出控制权，否则超时无法触发
        :param \*args: 传递给 *func* 的位置参数
        :param \*\*kwds: 传递给 *func* 的关键字参数
        :param timeout_value: 超时发生时返回的值（默认抛出 :class:`~eventlet.timeout.Timeout`）

        :rtype: 如果 *func* 在 *seconds* 内返回，则返回 *func* 的返回值；如果超时且提供了 `timeout_value`，则返回 `timeout_value`，否则抛出 :class:`Timeout`。

        :exception Timeout: 如果 *func* 超时且未提供 ``timeout_value``。
        :exception: 由 *func* 引发的任何异常

        示例::

            data = with_timeout(30, urllib2.open, 'http://www.google.com/', timeout_value="")

        这里 *data* 要么是 ``get()`` 调用的结果，要么是如果返回太慢则是空字符串。如果 ``get()`` 调用引发任何异常，则会传递给调用者。

.. tab:: 英文

    .. class:: eventlet.timeout.Timeout
        :no-index:

        Raises *exception* in the current greenthread after *timeout* seconds::

            timeout = Timeout(seconds, exception)
            try:
                ... # execution here is limited by timeout
            finally:
                timeout.cancel()

        When *exception* is omitted or is ``None``, the :class:`Timeout` instance
        itself is raised:

            >>> Timeout(0.1)
            >>> eventlet.sleep(0.2)
            Traceback (most recent call last):
            ...
            Timeout: 0.1 seconds

        You can use the  ``with`` statement for additional convenience::

            with Timeout(seconds, exception) as timeout:
                pass # ... code block ...

        This is equivalent to the try/finally block in the first example.

        There is an additional feature when using the ``with`` statement: if
        *exception* is ``False``, the timeout is still raised, but the with
        statement suppresses it, so the code outside the with-block won't see it::

            data = None
            with Timeout(5, False):
                data = mysock.makefile().readline()
            if data is None:
                ... # 5 seconds passed without reading a line
            else:
                ... # a line was read within 5 seconds

        As a very special case, if *seconds* is None, the timer is not scheduled,
        and is only useful if you're planning to raise it directly.

        There are two Timeout caveats to be aware of:

        * If the code block in the try/finally or with-block never cooperatively yields, the timeout cannot be raised.  In Eventlet, this should rarely be a problem, but be aware that you cannot time out CPU-only operations with this class.
        * If the code block catches and doesn't re-raise :class:`BaseException`  (for example, with ``except:``), then it will catch the Timeout exception, and might not abort as intended.

        When catching timeouts, keep in mind that the one you catch may not be the
        one you set; if you plan on silencing a timeout, always check that it's the
        same instance that you set::

            timeout = Timeout(1)
            try:
                ...
            except Timeout as t:
                if t is not timeout:
                    raise # not my timeout

        .. automethod:: cancel
            :no-index:

        .. autoattribute:: pending
            :no-index:

    .. function:: eventlet.timeout.with_timeout(seconds, function, *args, **kwds)
        :no-index:

        Wrap a call to some (yielding) function with a timeout; if the called
        function fails to return before the timeout, cancel it and return a flag
        value.

        :param seconds: seconds before timeout occurs
        :type seconds: int or float
        :param func: the callable to execute with a timeout; it must cooperatively yield, or else the timeout will not be able to trigger
        :param \*args: positional arguments to pass to *func*
        :param \*\*kwds: keyword arguments to pass to *func*
        :param timeout_value: value to return if timeout occurs (by default raises
        :class:`~eventlet.timeout.Timeout`)

        :rtype: Value returned by *func* if *func* returns before *seconds*, else
        *timeout_value* if provided, else raises :class:`Timeout`.

        :exception Timeout: if *func* times out and no ``timeout_value`` has
        been provided.
        :exception: Any exception raised by *func*

        Example::

            data = with_timeout(30, urllib2.open, 'http://www.google.com/', timeout_value="")

        Here *data* is either the result of the ``get()`` call, or the empty string
        if it took too long to return.  Any exception raised by the ``get()`` call
        is passed through to the caller.
