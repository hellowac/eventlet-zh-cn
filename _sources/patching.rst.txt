绿化世界
==================

**Greening The World**

.. tab:: 中文

    编写像 Eventlet 这样的库所面临的挑战之一在于，内置的网络库本身并不支持我们所需的协作让步。我们必须在标准库模块的某些关键位置进行补丁处理，以便它们能够协作让步。过去我们曾考虑在导入 Eventlet 时自动执行此操作，但决定不采取这种方式，因为仅通过导入模块 B 就更改模块 A 的行为是不符合 Python 习惯的。

    因此，使用 Eventlet 的应用程序必须明确地为其自身“绿色化”环境，使用提供的一个或两个便捷方法。

.. tab:: 英文

    One of the challenges of writing a library like Eventlet is that the built-in networking libraries don't natively support the sort of cooperative yielding that we need.  What we must do instead is patch standard library modules in certain key places so that they do cooperatively yield.  We've in the past considered doing this automatically upon importing Eventlet, but have decided against that course of action because it is un-Pythonic to change the behavior of module A simply by importing module B.

    Therefore, the application using Eventlet must explicitly green the world for itself, using one or both of the convenient methods provided.

.. _import-green:

导入绿色
--------------

**Import Green**

.. tab:: 中文

    绿色化应用程序的第一种方式是从 ``eventlet.green`` 包中导入与网络相关的库。它包含与常见标准库接口相同的库，但已修改为能与绿色线程良好配合使用。使用这种方法是一种良好的工程实践，因为每个文件中的真正依赖关系都很明确::

        from eventlet.green import socket
        from eventlet.green import threading
        from eventlet.green import asyncore
    
    如果每个库都能以这种方式绿色化，那么效果最好。如果 ``eventlet.green`` 中缺少某个模块（例如非 Python 标准模块），则 :func:`~eventlet.patcher.import_patched` 函数可以帮助解决。它是内置导入语句的替代方案，在导入时绿色化任何模块。

    .. function:: eventlet.patcher.import_patched(module_name, *additional_modules, **kw_additional_modules)

        以绿色方式导入模块，使得模块使用像 socket 这样的网络库时会使用 Eventlet 的绿色版本。唯一必需的参数是要导入的模块名称::
        
            import eventlet
            httplib2 = eventlet.import_patched('httplib2')
            
        在幕后，它通过暂时将 sys.modules 中的“普通”版本的库替换为 eventlet.green 等效版本来工作。当待补丁模块的导入完成后，sys.modules 的状态会被恢复。因此，如果补丁模块包含 'import socket' 语句，import_patched 将使其引用 eventlet.green.socket。这个方法的一个弱点是它不适用于延迟绑定（即在运行时发生的导入）。幸运的是，延迟绑定的导入很少使用（因为它慢且违反了 `PEP-8 <http://www.python.org/dev/peps/pep-0008/>`_），所以在大多数情况下，import_patched 会正常工作。
        
        import_patched 的另一个特点是可以精确指定哪些模块被补丁处理。这样做可能会略微提高性能，因为只会导入需要的模块，而没有参数的 import_patched 会导入一堆模块以防需要它们。*additional_modules* 和 *kw_additional_modules* 参数都是名称/模块对的序列。可以使用其中的一个或两个::

            from eventlet.green import socket
            from eventlet.green import SocketServer        
            BaseHTTPServer = eventlet.import_patched('BaseHTTPServer',
                                    ('socket', socket),
                                    ('SocketServer', SocketServer))
            BaseHTTPServer = eventlet.import_patched('BaseHTTPServer',
                                    socket=socket, SocketServer=SocketServer)

.. tab:: 英文

    The first way of greening an application is to import networking-related libraries from the ``eventlet.green`` package.  It contains libraries that have the same interfaces as common standard ones, but they are modified to behave well with green threads.  Using this method is a good engineering practice, because the true dependencies are apparent in every file::

    from eventlet.green import socket
    from eventlet.green import threading
    from eventlet.green import asyncore
    
    This works best if every library can be imported green in this manner.  If ``eventlet.green`` lacks a module (for example, non-python-standard modules), then :func:`~eventlet.patcher.import_patched` function can come to the rescue.  It is a replacement for the builtin import statement that greens any module on import.

    .. function:: eventlet.patcher.import_patched(module_name, *additional_modules, **kw_additional_modules)
        :no-index:

        Imports a module in a greened manner, so that the module's use of networking libraries like socket will use Eventlet's green versions instead.  The only required argument is the name of the module to be imported::
        
            import eventlet
            httplib2 = eventlet.import_patched('httplib2')
            
        Under the hood, it works by temporarily swapping out the "normal" versions of the libraries in sys.modules for an eventlet.green equivalent.  When the import of the to-be-patched module completes, the state of sys.modules is restored.  Therefore, if the patched module contains the statement 'import socket', import_patched will have it reference eventlet.green.socket.  One weakness of this approach is that it doesn't work for late binding (i.e. imports that happen during runtime).  Late binding of imports is fortunately rarely done (it's slow and against `PEP-8 <http://www.python.org/dev/peps/pep-0008/>`_), so in most cases import_patched will work just fine.
        
        One other aspect of import_patched is the ability to specify exactly which modules are patched.  Doing so may provide a slight performance benefit since only the needed modules are imported, whereas import_patched with no arguments imports a bunch of modules in case they're needed.  The *additional_modules* and *kw_additional_modules* arguments are both sequences of name/module pairs.  Either or both can be used::
        
            from eventlet.green import socket
            from eventlet.green import SocketServer        
            BaseHTTPServer = eventlet.import_patched('BaseHTTPServer',
                                    ('socket', socket),
                                    ('SocketServer', SocketServer))
            BaseHTTPServer = eventlet.import_patched('BaseHTTPServer',
                                    socket=socket, SocketServer=SocketServer)

.. _monkey-patch:

Monkeypatching 标准库
----------------------------------------

**Monkeypatching the Standard Library**

.. tab:: 中文

    绿色化应用程序的另一种方式是直接对标准库进行猴子补丁。这种方法的缺点是看起来非常“魔法化”，但其优点是避免了延迟绑定的问题。

    .. function:: eventlet.patcher.monkey_patch(os=None, select=None, socket=None, thread=None, time=None, psycopg=None)

        此函数通过将关键系统模块的关键元素替换为绿色等效项来进行猴子补丁。如果未指定任何参数，则会对所有模块进行补丁处理::
        
            import eventlet
            eventlet.monkey_patch()

        关键字参数提供了一些控制，允许选择补丁处理哪些模块（如果这很重要）。大多数模块会补丁处理与其同名的单个模块（例如 time=True 表示补丁处理 time 模块 [time.sleep 被 eventlet.sleep 替换]）。这个规则的例外是 *socket*，它还会补丁处理 :mod:`ssl` 模块（如果存在）； *thread* 会补丁处理 :mod:`thread`、:mod:`threading` 和 :mod:`Queue`。
        
        下面是一个使用 monkey_patch 仅补丁几个模块的示例::
        
            import eventlet
            eventlet.monkey_patch(socket=True, select=True)
            
        尽可能在应用程序生命周期的早期调用 :func:`~eventlet.patcher.monkey_patch` 非常重要。尝试将它作为主模块中的第一行之一来执行。这样做的原因是，有时有一个类继承自需要绿色化的类——例如继承自 socket.socket 的类——而继承是在导入时进行的，因此猴子补丁应该在派生类定义之前进行。调用 monkey_patch 多次是安全的。

        psycopg 的猴子补丁依赖于 Daniele Varrazzo 的绿色 psycopg2 分支；有关更多信息，请参见 `公告 <https://lists.secondlife.com/pipermail/eventletdev/2010-April/000800.html>`_。

    .. function:: eventlet.patcher.is_monkey_patched(module)

        返回指定模块是否已被猴子补丁。 *module* 可以是模块本身或模块的名称。

        完全基于模块的名称，因此，如果您通过非标准方式导入模块（包括 :func:`~eventlet.patcher.import_patched`）， is_monkey_patched 可能无法正确判断该模块是否已被补丁处理。

.. tab:: 英文

    The other way of greening an application is simply to monkeypatch the standard
    library.  This has the disadvantage of appearing quite magical, but the advantage of avoiding the late-binding problem.

    .. function:: eventlet.patcher.monkey_patch(os=None, select=None, socket=None, thread=None, time=None, psycopg=None)

        This function monkeypatches the key system modules by replacing their key elements with green equivalents.  If no arguments are specified, everything is patched::
        
            import eventlet
            eventlet.monkey_patch()

        The keyword arguments afford some control over which modules are patched, in case that's important.  Most patch the single module of the same name (e.g. time=True means that the time module is patched [time.sleep is patched by eventlet.sleep]).  The exceptions to this rule are *socket*, which also patches the :mod:`ssl` module if present; and *thread*, which patches :mod:`thread`, :mod:`threading`, and :mod:`Queue`.
        
        Here's an example of using monkey_patch to patch only a few modules::
        
            import eventlet
            eventlet.monkey_patch(socket=True, select=True)
            
        It is important to call :func:`~eventlet.patcher.monkey_patch` as early in the lifetime of the application as possible.  Try to do it as one of the first lines in the main module.  The reason for this is that sometimes there is a class that inherits from a class that needs to be greened -- e.g. a class that inherits from socket.socket -- and inheritance is done at import time, so therefore the monkeypatching should happen before the derived class is defined.      It's safe to call monkey_patch multiple times.

        The psycopg monkeypatching relies on Daniele Varrazzo's green psycopg2 branch; see `the announcement <https://lists.secondlife.com/pipermail/eventletdev/2010-April/000800.html>`_ for more information.

    .. function:: eventlet.patcher.is_monkey_patched(module)

        Returns whether or not the specified module is currently monkeypatched. *module* can either be the module itself or the module's name.

        Based entirely off the name of the module, so if you import a module some other way than with the import keyword (including :func:`~eventlet.patcher.import_patched`), is_monkey_patched might not be correct about that particular module.
