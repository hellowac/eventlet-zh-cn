:mod:`eventlet.green.zmq` -- ØMQ support
========================================

.. tab:: 中文

    :mod:`pyzmq <zmq>` [1]_ 是一个用 Cython [3]_ 编写的与 C++ ØMQ [2]_ 库绑定的 Python 库。
    :mod:`eventlet.green.zmq` 是 `pyzmq` 的 greenthread 感知版本。

.. tab:: 英文

    :mod:`pyzmq <zmq>` [1]_ is a python binding to the C++ ØMQ [2]_ library written in Cython [3]_.
    :mod:`eventlet.green.zmq` is greenthread aware version of `pyzmq`.

.. automodule:: eventlet.green.zmq
    :show-inheritance:

.. currentmodule:: eventlet.green.zmq

.. autoclass:: Context
    :show-inheritance:

    .. automethod:: socket

.. autoclass:: Socket
    :show-inheritance:
    :inherited-members:

    .. automethod:: recv

    .. automethod:: send

.. module:: zmq


.. [1] http://github.com/zeromq/pyzmq
.. [2] http://www.zeromq.com
.. [3] http://www.cython.org
