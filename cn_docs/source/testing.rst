.. _testing-eventlet:

测试 Eventlet
================

**Testing Eventlet**

.. tab:: 中文

    Eventlet 使用 `Pytest <https://pytest/>`_ 进行测试。要运行测试，只需安装 pytest，然后在 eventlet 目录中执行：

    .. code-block:: sh

      $ pytest

    就完成了！

    许多测试会根据环境因素被跳过；例如，当您的操作系统不支持 kqueue 时，测试与 kqueue 相关的功能是没有意义的。这些测试在执行过程中会显示为 S，在测试运行后的总结中会告诉您跳过了多少个测试。

.. tab:: 英文

    Eventlet is tested using `Pytest <https://pytest/>`_.  To run tests, simply install pytest, and then, in the eventlet tree, do:

    .. code-block:: sh

      $ pytest

    That's it!

    Many tests are skipped based on environmental factors; for example, it makes no sense to test kqueue-specific functionality when your OS does not support it.  These are printed as S's during execution, and in the summary printed after the tests run it will tell you how many were skipped.

文档测试
--------

**Doctests**

.. tab:: 中文

  要运行许多 eventlet 模块中包含的 doctest，请使用以下命令：

  .. code-block:: sh

     $ pytest --doctest-modules eventlet/

  doctest 目前 `未通过 <https://github.com/eventlet/eventlet/issues/837>`__ 。

.. tab:: 英文

    To run the doctests included in many of the eventlet modules, use this command:

    .. code-block:: sh

      $ pytest --doctest-modules eventlet/

    The doctests currently `do not pass <https://github.com/eventlet/eventlet/issues/837>`_.


测试 Eventlet Hubs
---------------------

**Testing Eventlet Hubs**

.. tab:: 中文

    当您运行测试时，Eventlet 将使用当前平台最合适的 hub 来进行调度。在对 Eventlet 进行更改时，有时需要在默认 hub 以外的 hub 上测试这些更改。您可以通过 ``EVENTLET_HUB`` 环境变量来实现这一点。

    .. code-block:: sh

       $ EVENTLET_HUB=epolls pytest

    有关 hub 的完整列表，请参见 :ref:`understanding_hubs`。

.. tab:: 英文

    When you run the tests, Eventlet will use the most appropriate hub for the current platform to do its dispatch.  It's sometimes useful when making changes to Eventlet to test those changes on hubs other than the default.  You can do this with the ``EVENTLET_HUB`` environment variable.

    .. code-block:: sh

       $ EVENTLET_HUB=epolls pytest

    See :ref:`understanding_hubs` for the full list of hubs.


编写测试
-------------

**Writing Tests**

.. tab:: 中文

    以下是编写测试的一些注意事项，顺序无关紧要。

    编写模块 `foo` 的测试时，文件名约定为 `foo_test.py`。目前我们还没有针对更细粒度的测试的约定，但一个合理的命名方式可能是 `foo_class_test.py`。

    如果您编写的测试涉及客户端连接到一个已启动的服务器，最好不要使用硬编码的端口，因为这会使并行化测试变得更加困难。相反，应该将服务器绑定到端口 0，然后在连接客户端时查找其端口，如下所示::

      server_sock = eventlet.listener(('127.0.0.1', 0))
      client_sock = eventlet.connect(('localhost', server_sock.getsockname()[1]))

.. tab:: 英文

    What follows are some notes on writing tests, in no particular order.

    The filename convention when writing a test for module `foo` is to name the test `foo_test.py`.  We don't yet have a convention for tests that are of finer granularity, but a sensible one might be `foo_class_test.py`.

    If you are writing a test that involves a client connecting to a spawned server, it is best to not use a hardcoded port because that makes it harder to parallelize tests.  Instead bind the server to 0, and then look up its port when connecting the client, like this::

      server_sock = eventlet.listener(('127.0.0.1', 0))
      client_sock = eventlet.connect(('localhost', server_sock.getsockname()[1]))

覆盖测试
-------------

**Coverage**

.. tab:: 中文

    Coverage.py 是一个非常棒的工具，用于评估单元测试覆盖了多少代码。pytest 支持它，只要安装了 pytest-cov，就可以轻松生成 Eventlet 的覆盖率报告。方法如下：

    .. code-block:: sh

      pytest --cov=eventlet

    在运行完测试后，这将输出大量的模块名称和行号。由于某些原因，``--cover-inclusive`` 选项会导致一切出错，而不是实现其限制覆盖范围仅限于本地文件的目的，因此不要使用该选项。

    html 选项非常有用，因为它生成的 HTML 文件格式良好，比行号汤更易于阅读。以下命令生成注释，并将 HTML 文件输出到名为 "cover" 的目录：

    .. code-block:: sh

      coverage html -d cover --omit='tempmod,<console>,tests'

    （ ``tempmod`` 和 ``console`` 被省略，因为它们在单元测试完成后被丢弃，而 coverage.py 没有足够聪明来检测这一点。）

.. tab:: 英文

    Coverage.py is an awesome tool for evaluating how much code was exercised by unit tests.  pytest supports it pytest-cov is installed, so it's easy to generate coverage reports for eventlet.  Here's how:

    .. code-block:: sh

    pytest --cov=eventlet

    After running the tests to completion, this will emit a huge wodge of module names and line numbers.  For some reason, the ``--cover-inclusive`` option breaks everything rather than serving its purpose of limiting the coverage to the local files, so don't use that.

    The html option is quite useful because it generates nicely-formatted HTML files that are much easier to read than line-number soup.  Here's a command that generates the annotation, dumping the html files into a directory called "cover":

    .. code-block:: sh

      coverage html -d cover --omit='tempmod,<console>,tests'

    (``tempmod`` and ``console`` are omitted because they get thrown away at the completion of their unit tests and coverage.py isn't smart enough to detect this.)
