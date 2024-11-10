:mod:`dagpool` -- 依赖驱动的 Greenthreads
================================================

:mod:`dagpool` **-- Dependency-Driven Greenthreads**

原理
*********

**Rationale**

.. tab:: 中文

    dagpool 模块提供了 :class:`DAGPool <eventlet.dagpool.DAGPool>` 类，用于解决一个 greenthread 生成的值可能被多个其他 greenthread 消费的情况——同时，一个消费的 greenthread 可能依赖于多个不同 greenthread 的输出。

    如果存在严格的多对一依赖关系的树结构——每个生产者 greenthread 仅向一个消费者提供结果，虽然一个消费者可能依赖多个生产者——可以通过递归地为每个消费者构造一个 :class:`GreenPool <eventlet.greenpool.GreenPool>` 的生产者 greenthread，并使用 :meth:`等待 <eventlet.greenpool.GreenPool.waitall>` 所有生产者完成。

    如果存在严格的一对多依赖关系的树结构——每个消费者 greenthread 仅依赖一个生产者，虽然一个生产者可能为多个消费者提供结果——可以通过让每个生产者在完成时启动一个消费者的 :class:`GreenPool <eventlet.greenpool.GreenPool>`。

    但是当存在多对多的依赖关系时，树结构不足以描述。这种结构被称为 `有向无环图 <https://en.wikipedia.org/wiki/Directed_acyclic_graph>`_ ，或 DAG。

    您可以考虑将 greenthread 按依赖顺序排序（ `拓扑排序 <https://en.wikipedia.org/wiki/Topological_sorting>`_ ），并将其启动在 GreenPool 中。但是 GreenPool 的并发性必须受到严格约束，以确保在其所有上游生产者完成之前，不会启动任何 greenthread——而适当的池大小依赖于数据。只有大小为 1 的池（将所有 greenthread 串行化）才能保证拓扑排序的正确结果。

    即使您将所有 greenthread 串行化，又如何将每个生产者的结果传递给所有的消费者，这些消费者可能会在非常不同的时间点启动？

    一种解决方案是为每个 greenthread 关联一个唯一键，并将其结果存储在公共 dict 中。然后，每个消费者 greenthread 可以通过键识别其直接上游生产者，并在该 dict 中找到其结果。

    这就是 DAGPool 的核心。

    DAGPool 实例拥有一个 dict，并将 greenthread 的结果存储在该 dict 中。您可以在 DAG 中 :meth:`启动 <eventlet.dagpool.DAGPool.spawn>` *所有* greenthread，并为每个 greenthread 指定其唯一的键——该键用于在完成后存储其结果——以及其直接依赖的上游生产者 greenthread 的键。

    键只需在 DAGPool 实例内唯一，无需使用 UUID。键可以是任何可以作为 dict 键的类型。使用字符串键可以更容易地推理 DAGPool 的行为，但并非必需。

    DAGPool 会将 (键, 值) 对的可迭代对象传递给每个 greenthread。每对中的键是该 greenthread 指定的一个上游生产者的键；值是该生产者 greenthread 返回的值。对按结果可用的顺序传递；消费 greenthread 会阻塞，直到下一个结果可以传递。

.. tab:: 英文

    The dagpool module provides the :class:`DAGPool <eventlet.dagpool.DAGPool>`
    class, which addresses situations in which the value produced by one
    greenthread might be consumed by several others -- while at the same time a
    consuming greenthread might depend on the output from several different
    greenthreads.

    If you have a tree with strict many-to-one dependencies -- each producer
    greenthread provides results to exactly one consumer, though a given consumer
    may depend on multiple producers -- that could be addressed by recursively
    constructing a :class:`GreenPool <eventlet.greenpool.GreenPool>` of producers
    for each consumer, then :meth:`waiting <eventlet.greenpool.GreenPool.waitall>`
    for all producers.

    If you have a tree with strict one-to-many dependencies -- each consumer
    greenthread depends on exactly one producer, though a given producer may
    provide results to multiple consumers -- that could be addressed by causing
    each producer to finish by launching a :class:`GreenPool
    <eventlet.greenpool.GreenPool>` of consumers.

    But when you have many-to-many dependencies, a tree doesn't suffice. This is
    known as a
    `Directed Acyclic Graph <https://en.wikipedia.org/wiki/Directed_acyclic_graph>`_,
    or DAG.

    You might consider sorting the greenthreads into dependency order
    (`topological sort <https://en.wikipedia.org/wiki/Topological_sorting>`_) and
    launching them in a GreenPool. But the concurrency of the GreenPool must be
    strictly constrained to ensure that no greenthread is launched before all its
    upstream producers have completed -- and the appropriate pool size is
    data-dependent. Only a pool of size 1 (serializing all the greenthreads)
    guarantees that a topological sort will produce correct results.

    Even if you do serialize all the greenthreads, how do you pass results from
    each producer to all its consumers, which might start at very different points
    in time?

    One answer is to associate each greenthread with a distinct key, and store its
    result in a common dict. Then each consumer greenthread can identify its
    direct upstream producers by their keys, and find their results in that dict.

    This is the essence of DAGPool.

    A DAGPool instance owns a dict, and stores greenthread results in that dict.
    You :meth:`spawn <eventlet.dagpool.DAGPool.spawn>` *all* greenthreads in the
    DAG, specifying for each its own key -- the key with which its result will be
    stored on completion -- plus the keys of the upstream producer greenthreads on
    whose results it directly depends.

    Keys need only be unique within the DAGPool instance; they need not be UUIDs.
    A key can be any type that can be used as a dict key. String keys make it
    easier to reason about a DAGPool's behavior, but are by no means required.

    The DAGPool passes to each greenthread an iterable of (key, value) pairs.
    The key in each pair is the key of one of the greenthread's specified upstream
    producers; the value is the value returned by that producer greenthread. Pairs
    are delivered in the order results become available; the consuming greenthread
    blocks until the next result can be delivered.

教程
********

**Tutorial**

示例
-------

**Example**

.. tab:: 中文

    考虑一些依赖于一组预编译库的编译语言程序。假设每个构建都需要其直接依赖的特定库构建作为输入。

    ::

        a  zlib
        | /  |
        |/   |
        b    c
        |   /|
        |  / |
        | /  |
        |/   |
        d    e

    在获取 b 和 c 的构建结果之前，无法进行 d 程序的构建。在获取 a 和 zlib 的构建结果之前，也无法进行 b 库的构建。但可以立即进行 a 和 zlib 的构建。

    因此，我们可以使用 DAGPool 实例来启动 greenthread 并运行如下函数：

    ::

        def builder(key, upstream):
            for libname, product in upstream:
                # ... 为 'key' 配置构建，使用 'libname' 的 'product'
            # 所有上游构建已完成
            # ... 运行 'key' 的构建
            return build_product_for_key

    :meth:`spawn <eventlet.dagpool.DAGPool.spawn>` 启动这些 greenthread：

    ::

        pool = DAGPool()
        # 传递给 spawn() 的上游生产者键可以来自任何可迭代对象，包括生成器
        pool.spawn("d", ("b", "c"), builder)
        pool.spawn("e", ["c"], builder)
        pool.spawn("b", ("a", "zlib"), builder)
        pool.spawn("c", ["zlib"], builder)
        pool.spawn("a", (), builder)

    与 :func:`eventlet.spawn() <eventlet.spawn>` 类似，如果需要为某些构建传递特定的构建标志，可以通过位置参数或关键字参数传递：

    ::

        def builder(key, upstream, cflags="", linkflags=""):
            ...

        pool.spawn("d", ("b", "c"), builder, "-o2")
        pool.spawn("e", ["c"], builder, linkflags="-pie")

    但是，如果对每个 builder() 调用的参数都是一致的（如原始示例中），可以构建依赖项的 dict 并调用 :meth:`spawn_many() <eventlet.dagpool.DAGPool.spawn_many>`：

    ::

        deps = dict(d=("b", "c"),
                    e=["c"],
                    b=("a", "zlib"),
                    c=["zlib"],
                    a=())
        pool.spawn_many(deps, builder)

    在 DAGPool 外部，可以通过多种方式获取 d 和 e 的结果（或任何构建 greenthread 的结果）。

    :meth:`pool.waitall() <eventlet.dagpool.DAGPool.waitall>` 等待所有启动的 greenthread 完成，并返回包含它们*所有*结果的 dict：

    ::

        final = pool.waitall()
        print("for d: {0}".format(final["d"]))
        print("for e: {0}".format(final["e"]))

    waitall() 是没有参数的 :meth:`wait() <eventlet.dagpool.DAGPool.wait>` 的别名：

    ::

        final = pool.wait()
        print("for d: {0}".format(final["d"]))
        print("for e: {0}".format(final["e"]))

    或者也可以只等待最终程序的结果：

    ::

        final = pool.wait(["d", "e"])

    返回的 dict 仅包含指定的键。可以将任何可迭代对象传递给 wait()，包括生成器。

    可以等待任何指定的 greenthread 集合；它们不需要是拓扑上的最后一个：

    ::

        # 一旦 a 和 zlib 都返回结果后立即返回，无论其他任务是否仍在运行
        leaves = pool.wait(["a", "zlib"])

    假设您只想等待*一个*最终程序的结果：

    ::

        final = pool.wait(["d"])
        dprog = final["d"]

    上面的 wait() 调用将在 greenthread d 返回结果后立即返回——无论 greenthread e 是否已完成。

    :meth:`__getitem()__ <eventlet.dagpool.DAGPool.__getitem__>` 是获取单个结果的简写：

    ::

        # 等待 greenthread d 返回其结果
        dprog = pool["d"]

    相反，:meth:`get() <eventlet.dagpool.DAGPool.get>` 会立即返回，不论结果是否已准备好：

    ::

        # 立即返回
        if pool.get("d") is None:
            ...

    当然，您的 greenthread 可能没有显式 return 语句，因此可能会隐式返回 None。您可能需要测试其他值。

    ::

        # 立即返回
        if pool.get("d", "notdone") == "notdone":
            ...

    假设您希望对每个最终程序执行某些处理（上传？），但又不希望等到它们都完成。无需轮询 get() 调用——使用 :meth:`wait_each() <eventlet.dagpool.DAGPool.wait_each>`：

    ::

        for key, result in pool.wait_each(["d", "e"]):
            # key 将是 d 或 e，按完成顺序
            # 处理结果...

    与 :meth:`wait() <eventlet.dagpool.DAGPool.wait>` 类似，如果省略 wait_each() 的参数，它将返回所有 greenthread 的结果：

    ::

        for key, result in pool.wait_each():
            # key 将是 a、zlib、b、c、d、e，按各自完成顺序
            # 处理其结果...

.. tab:: 英文

    Consider a couple of programs in some compiled language that depend on a set
    of precompiled libraries. Suppose every such build requires as input the
    specific set of library builds on which it directly depends.

    ::

        a  zlib
        | /  |
        |/   |
        b    c
        |   /|
        |  / |
        | /  |
        |/   |
        d    e

    We can't run the build for program d until we have the build results for both
    b and c. We can't run the build for library b until we have build results for
    a and zlib. We can, however, immediately run the builds for a and zlib.

    So we can use a DAGPool instance to spawn greenthreads running a function such
    as this:

    ::

        def builder(key, upstream):
            for libname, product in upstream:
                # ... configure build for 'key' to use 'product' for 'libname'
            # all upstream builds have completed
            # ... run build for 'key'
            return build_product_for_key

    :meth:`spawn <eventlet.dagpool.DAGPool.spawn>` all these greenthreads:

    ::

        pool = DAGPool()
        # the upstream producer keys passed to spawn() can be from any iterable,
        # including a generator
        pool.spawn("d", ("b", "c"), builder)
        pool.spawn("e", ["c"], builder)
        pool.spawn("b", ("a", "zlib"), builder)
        pool.spawn("c", ["zlib"], builder)
        pool.spawn("a", (), builder)

    As with :func:`eventlet.spawn() <eventlet.spawn>`, if you need to pass special
    build flags to some set of builds, these can be passed as either positional or
    keyword arguments:

    ::

        def builder(key, upstream, cflags="", linkflags=""):
            ...

        pool.spawn("d", ("b", "c"), builder, "-o2")
        pool.spawn("e", ["c"], builder, linkflags="-pie")

    However, if the arguments to each builder() call are uniform (as in the
    original example), you could alternatively build a dict of the dependencies
    and call :meth:`spawn_many() <eventlet.dagpool.DAGPool.spawn_many>`:

    ::

        deps = dict(d=("b", "c"),
                    e=["c"],
                    b=("a", "zlib"),
                    c=["zlib"],
                    a=())
        pool.spawn_many(deps, builder)

    From outside the DAGPool, you can obtain the results for d and e (or in fact
    for any of the build greenthreads) in any of several ways.

    :meth:`pool.waitall() <eventlet.dagpool.DAGPool.waitall>` waits until the last of the spawned
    greenthreads has completed, and returns a dict containing results for *all* of
    them:

    ::

        final = pool.waitall()
        print("for d: {0}".format(final["d"]))
        print("for e: {0}".format(final["e"]))

    waitall() is an alias for :meth:`wait() <eventlet.dagpool.DAGPool.wait>` with no arguments:

    ::

        final = pool.wait()
        print("for d: {0}".format(final["d"]))
        print("for e: {0}".format(final["e"]))

    Or you can specifically wait for only the final programs:

    ::

        final = pool.wait(["d", "e"])

    The returned dict will contain only the specified keys. The keys may be passed
    into wait() from any iterable, including a generator.

    You can wait for any specified set of greenthreads; they need not be
    topologically last:

    ::

        # returns as soon as both a and zlib have returned results, regardless of
        # what else is still running
        leaves = pool.wait(["a", "zlib"])

    Suppose you want to wait specifically for just *one* of the final programs:

    ::

        final = pool.wait(["d"])
        dprog = final["d"]

    The above wait() call will return as soon as greenthread d returns a result --
    regardless of whether greenthread e has finished.

    :meth:`__getitem()__ <eventlet.dagpool.DAGPool.__getitem__>` is shorthand for
    obtaining a single result:

    ::

        # waits until greenthread d returns its result
        dprog = pool["d"]

    In contrast, :meth:`get() <eventlet.dagpool.DAGPool.get>` returns immediately,
    whether or not a result is ready:

    ::

        # returns immediately
        if pool.get("d") is None:
            ...

    Of course, your greenthread might not include an explicit return statement and
    hence might implicitly return None. You might have to test some other value.

    ::

        # returns immediately
        if pool.get("d", "notdone") == "notdone":
            ...

    Suppose you want to process each of the final programs in some way (upload
    it?), but you don't want to have to wait until they've both finished. You
    don't have to poll get() calls -- use :meth:`wait_each()
    <eventlet.dagpool.DAGPool.wait_each>`:

    ::

        for key, result in pool.wait_each(["d", "e"]):
            # key will be d or e, in completion order
            # process result...

    As with :meth:`wait() <eventlet.dagpool.DAGPool.wait>`, if you omit the
    argument to wait_each(), it delivers results for all the greenthreads of which
    it's aware:

    ::

        for key, result in pool.wait_each():
            # key will be a, zlib, b, c, d, e, in whatever order each completes
            # process its result...

自省
-------------

**Introspection**

.. tab:: 中文

    假设您已设置了一个 :class:`DAGPool <eventlet.dagpool.DAGPool>`，其依赖关系如上所示。然而，当您调用 :meth:`waitall() <eventlet.dagpool.DAGPool.waitall>` 时，却发现它并未返回！DAGPool 实例卡住了！

    您可以将 waitall() 改为 :meth:`wait_each() <eventlet.dagpool.DAGPool.wait_each>`，并在每个键可用时打印它：

    ::

        for key, result in pool.wait_each():
            print("got result for {0}".format(key))
            # ... 处理结果 ...

    一旦构建完成，这将输出：

    ::

        got result for a

    然后停止。嗯，有点奇怪！

    您可以检查 :meth:`running <eventlet.dagpool.DAGPool.running>` greenthread 的数量：

    ::

        >>> print(pool.running())
        4

    以及 :meth:`waiting <eventlet.dagpool.DAGPool.waiting>` greenthread 的数量：

    ::

        >>> print(pool.waiting())
        4

    通常，更有用的是询问*哪些* greenthread 仍在 :meth:`运行中 <eventlet.dagpool.DAGPool.running_keys>`：

    ::

        >>> print(pool.running_keys())
        ('c', 'b', 'e', 'd')

    在这种情况下，我们已知 a 已经完成。

    可以查看所有可用结果：

    ::

        >>> print(pool.keys())
        ('a',)
        >>> print(pool.items())
        (('a', result_from_a),)

    :meth:`keys() <eventlet.dagpool.DAGPool.keys>` 和 :meth:`items() <eventlet.dagpool.DAGPool.items>` 方法只会返回结果已实际可用的键和条目，反映了底层字典的内容。

    但是，到底是什么阻碍了工作流？我们究竟在 :meth:`等待 <eventlet.dagpool.DAGPool.waiting_for>` 什么？

    ::

        >>> print(pool.waiting_for("d"))
        set(['c', 'b'])

    (waiting_for() 的可选参数是一个*单独的*键。)

    还不够清楚...

    ::

        >>> print(pool.waiting_for("b"))
        set(['zlib'])
        >>> print(pool.waiting_for("zlib"))
        KeyError: 'zlib'

    啊哈！我们在最初配置 DAGPool 时竟然忘记包含 zlib 构建了！

    （在非交互式使用中，不传递 waiting_for() 的参数会更有帮助。这样可以返回一个字典，指示每个 greenthread 键正在等待哪些其他键。）

    ::

        from pprint import pprint
        pprint(pool.waiting_for())

        {'b': set(['zlib']), 'c': set(['zlib']), 'd': set(['b', 'c']), 'e': set(['c'])}

    在这种情况下，合理的解决方法是启动 zlib greenthread：

    ::

        pool.spawn("zlib", (), builder)

    即使这是对该 DAGPool 实例的最后一次方法调用，它也会解除所有其他 DAGPool greenthread 的阻塞。

.. tab:: 英文

    Let's say you have set up a :class:`DAGPool <eventlet.dagpool.DAGPool>` with
    the dependencies shown above. To your consternation, your :meth:`waitall()
    <eventlet.dagpool.DAGPool.waitall>` call does not return! The DAGPool instance
    is stuck!

    You could change waitall() to :meth:`wait_each()
    <eventlet.dagpool.DAGPool.wait_each>`, and print each key as it becomes
    available:

    ::

        for key, result in pool.wait_each():
            print("got result for {0}".format(key))
            # ... process ...

    Once the build for a has completed, this produces:

    ::

        got result for a

    and then stops. Hmm!

    You can check the number of :meth:`running <eventlet.dagpool.DAGPool.running>`
    greenthreads:

    ::

        >>> print(pool.running())
        4

    and the number of :meth:`waiting <eventlet.dagpool.DAGPool.waiting>`
    greenthreads:

    ::

        >>> print(pool.waiting())
        4

    It's often more informative to ask *which* greenthreads are :meth:`still
    running <eventlet.dagpool.DAGPool.running_keys>`:

    ::

        >>> print(pool.running_keys())
        ('c', 'b', 'e', 'd')

    but in this case, we already know a has completed.

    We can ask for all available results:

    ::

        >>> print(pool.keys())
        ('a',)
        >>> print(pool.items())
        (('a', result_from_a),)

    The :meth:`keys() <eventlet.dagpool.DAGPool.keys>` and :meth:`items()
    <eventlet.dagpool.DAGPool.items>` methods only return keys and items for
    which results are actually available, reflecting the underlying dict.

    But what's blocking the works? What are we :meth:`waiting for
    <eventlet.dagpool.DAGPool.waiting_for>`?

    ::

        >>> print(pool.waiting_for("d"))
        set(['c', 'b'])

    (waiting_for()'s optional argument is a *single* key.)

    That doesn't help much yet...

    ::

        >>> print(pool.waiting_for("b"))
        set(['zlib'])
        >>> print(pool.waiting_for("zlib"))
        KeyError: 'zlib'

    Aha! We forgot to even include the zlib build when we were originally
    configuring this DAGPool!

    (For non-interactive use, it would be more informative to omit waiting_for()'s
    argument. This usage returns a dict indicating, for each greenthread key,
    which other keys it's waiting for.)

    ::

        from pprint import pprint
        pprint(pool.waiting_for())

        {'b': set(['zlib']), 'c': set(['zlib']), 'd': set(['b', 'c']), 'e': set(['c'])}

    In this case, a reasonable fix would be to spawn the zlib greenthread:

    ::

        pool.spawn("zlib", (), builder)

    Even if this is the last method call on this DAGPool instance, it should
    unblock all the rest of the DAGPool greenthreads.

发布
-------

**Posting**

.. tab:: 中文

    如果我们已经拥有了 zlib 的构建结果，那么可以使用 :meth:`post() <eventlet.dagpool.DAGPool.post>` 该结果，而不是重新构建该库：

    ::

        pool.post("zlib", result_from_zlib)

    这同样可以解锁 DAGPool 其他的 greenthread。

.. tab:: 英文

    If we happen to have zlib build results in hand already, though, we could
    instead :meth:`post() <eventlet.dagpool.DAGPool.post>` that result instead of rebuilding the library:

    ::

        pool.post("zlib", result_from_zlib)

    This, too, should unblock the rest of the DAGPool greenthreads.

预加载
----------

**Preloading**

.. tab:: 中文

    如果重建过程耗时较长，记录部分结果可能会很有帮助，这样在中断的情况下，您可以从上次中断处继续，而不必重新构建先前的所有内容。

    您可以迭代地将这些先前结果 :meth:`post() <eventlet.dagpool.DAGPool.post>` 到一个新的 DAGPool 实例中；或者可以使用已有的字典来 :meth:`预加载 <eventlet.dagpool.DAGPool.__init__>` :class:`DAGPool <eventlet.dagpool.DAGPool>`：

    ::

        pool = DAGPool(dict(a=result_from_a, zlib=result_from_zlib))

    任何依赖于 a 或 zlib 的 DAGPool greenthread 都可以立即消费这些结果。

    还可以通过 (key, result) 对的可迭代对象构造 DAGPool。

.. tab:: 英文

    If rebuilding takes nontrivial realtime, it might be useful to record partial
    results, so that in case of interruption you can restart from where you left
    off rather than having to rebuild everything prior to that point.

    You could iteratively :meth:`post() <eventlet.dagpool.DAGPool.post>` those
    prior results into a new DAGPool instance; alternatively you can
    :meth:`preload <eventlet.dagpool.DAGPool.__init__>` the :class:`DAGPool
    <eventlet.dagpool.DAGPool>` from an existing dict:

    ::

        pool = DAGPool(dict(a=result_from_a, zlib=result_from_zlib))

    Any DAGPool greenthreads that depend on either a or zlib can immediately
    consume those results.

    It also works to construct DAGPool with an iterable of (key, result) pairs.

异常传播
---------------------

**Exception Propagation**

.. tab:: 中文

    但是如果我们启动了一个失败的 zlib 构建会怎样？假设 zlib greenthread 以异常终止？在这种情况下，b、c、d 或 e 都无法继续进行！而且我们不希望永远等待它们。

    ::

        dprog = pool["d"]
        eventlet.dagpool.PropagateError: PropagateError(d): PropagateError: PropagateError(c): PropagateError: PropagateError(zlib): OriginalError

    DAGPool 提供了一个 :class:`PropagateError <eventlet.dagpool.PropagateError>` 异常专门用于封装此类失败。如果某个 DAGPool greenthread 以 Exception 子类终止，DAGPool 会将该异常包装在一个 PropagateError 实例中，其 *key* 属性是失败的 greenthread 的键，而 *exc* 属性是终止它的异常。这个 PropagateError 被存储为该 greenthread 的结果。

    尝试使用存储有 PropagateError 结果的 greenthread 的结果时，会引发该 PropagateError。

    ::

        pool["zlib"]
        eventlet.dagpool.PropagateError: PropagateError(zlib): OriginalError

    因此，当 greenthread c 尝试使用 zlib 的结果时，会引发 zlib 的 PropagateError。除非 greenthread c 的构建函数处理该 PropagateError 异常，否则该 greenthread 将自行终止。该 PropagateError 将被再次包装到另一个 PropagateError 中，其 *key* 属性是 c， *exc* 属性是 zlib 的 PropagateError。

    同样，当 greenthread d 尝试使用 c 的结果时，会引发 c 的 PropagateError。这反过来又被包装在一个 PropagateError 中，其 *key* 是 d，*exc* 是 c 的 PropagateError。

    当尝试获取 d 的结果时，如上所示，将引发 d 的 PropagateError。

    您可以通过代码追踪失败路径，以确定最初的失败（如果需要的话）：

    ::

        orig_err = err
        key = "unknown"
        while isinstance(orig_err, PropagateError):
            key = orig_err.key
            orig_err = orig_err.exc

.. tab:: 英文

    But what if we spawn a zlib build that fails? Suppose the zlib greenthread
    terminates with an exception? In that case none of b, c, d or e can proceed!
    Nor do we want to wait forever for them.

    ::

        dprog = pool["d"]
        eventlet.dagpool.PropagateError: PropagateError(d): PropagateError: PropagateError(c): PropagateError: PropagateError(zlib): OriginalError

    DAGPool provides a :class:`PropagateError <eventlet.dagpool.PropagateError>`
    exception specifically to wrap such failures. If a DAGPool greenthread
    terminates with an Exception subclass, the DAGPool wraps that exception in a
    PropagateError instance whose *key* attribute is the key of the failing
    greenthread and whose *exc* attribute is the exception that terminated it.
    This PropagateError is stored as the result from that greenthread.

    Attempting to consume the result from a greenthread for which a PropagateError
    was stored raises that PropagateError.

    ::

        pool["zlib"]
        eventlet.dagpool.PropagateError: PropagateError(zlib): OriginalError

    Thus, when greenthread c attempts to consume the result from zlib, the
    PropagateError for zlib is raised. Unless the builder function for greenthread
    c handles that PropagateError exception, that greenthread will itself
    terminate. That PropagateError will be wrapped in another PropagateError whose
    *key* attribute is c and whose *exc* attribute is the PropagateError for zlib.

    Similarly, when greenthread d attempts to consume the result from c, the
    PropagateError for c is raised. This in turn is wrapped in a PropagateError
    whose *key* is d and whose *exc* is the PropagateError for c.

    When someone attempts to consume the result from d, as shown above, the
    PropagateError for d is raised.

    You can programmatically chase the failure path to determine the original
    failure if desired:

    ::

        orig_err = err
        key = "unknown"
        while isinstance(orig_err, PropagateError):
            key = orig_err.key
            orig_err = orig_err.exc

扫描成功/异常的情况
---------------------------------

**Scanning for Success / Exceptions**

.. tab:: 中文

    异常传播意味着我们既不会执行无用的构建，也不会等待那些永远不会到达的结果。

    然而，这确实使得获取 *确实* 成功的构建的 *部分* 结果变得困难。

    为此，您可以调用 :meth:`wait_each_success() <eventlet.dagpool.DAGPool.wait_each_success>`：

    ::

        for key, result in pool.wait_each_success():
            print("{0} succeeded".format(key))
            # ... 处理结果 ...

        a succeeded

    另一个问题是，虽然在示例中有五个不同的 greenthread 失败了，但我们只看到了一条失败链。您可以使用 :meth:`wait_each_exception() <eventlet.dagpool.DAGPool.wait_each_exception>` 列举所有坏消息：

    ::

        for key, err in pool.wait_each_exception():
            print("{0} failed with {1}".format(key, err.exc.__class__.__name__))

        c failed with PropagateError
        b failed with PropagateError
        e failed with PropagateError
        d failed with PropagateError
        zlib failed with OriginalError

    wait_each_exception() 会将每个 PropagateError 包装器作为结果返回，而不是作为异常引发。

    注意我们打印 :code:`err.exc.__class__.__name__`，因为 :code:`err.__class__.__name__` 始终为 PropagateError。

    wait_each_success() 和 wait_each_exception() 都可以接受一个键的可迭代对象作为要报告的范围：

    ::

        for key, result in pool.wait_each_success(["d", "e"]):
            print("{0} succeeded".format(key))

        (无输出)

        for key, err in pool.wait_each_exception(["d", "e"]):
            print("{0} failed with {1}".format(key, err.exc.__class__.__name__))

        e failed with PropagateError
        d failed with PropagateError

    由于在指定键的 greenthreads （或所有键）终止之前我们无法确定如何分类每一个，因此 wait_each_success() 和 wait_each_exception() 必须等待直到所有指定键（或所有键）的 greenthreads 已经以某种方式终止。

.. tab:: 英文

    Exception propagation means that we neither perform useless builds nor wait for
    results that will never arrive.

    However, it does make it difficult to obtain *partial* results for builds that
    *did* succeed.

    For that you can call :meth:`wait_each_success()
    <eventlet.dagpool.DAGPool.wait_each_success>`:

    ::

        for key, result in pool.wait_each_success():
            print("{0} succeeded".format(key))
            # ... process result ...

        a succeeded

    Another problem is that although five different greenthreads failed in the
    example, we only see one chain of failures. You can enumerate the bad news
    with :meth:`wait_each_exception() <eventlet.dagpool.DAGPool.wait_each_exception>`:

    ::

        for key, err in pool.wait_each_exception():
            print("{0} failed with {1}".format(key, err.exc.__class__.__name__))

        c failed with PropagateError
        b failed with PropagateError
        e failed with PropagateError
        d failed with PropagateError
        zlib failed with OriginalError

    wait_each_exception() yields each PropagateError wrapper as if it were the
    result, rather than raising it as an exception.

    Notice that we print :code:`err.exc.__class__.__name__` because
    :code:`err.__class__.__name__` is always PropagateError.

    Both wait_each_success() and wait_each_exception() can accept an iterable of
    keys to report:

    ::

        for key, result in pool.wait_each_success(["d", "e"]):
            print("{0} succeeded".format(key))

        (no output)

        for key, err in pool.wait_each_exception(["d", "e"]):
            print("{0} failed with {1}".format(key, err.exc.__class__.__name__))

        e failed with PropagateError
        d failed with PropagateError

    Both wait_each_success() and wait_each_exception() must wait until the
    greenthreads for all specified keys (or all keys) have terminated, one way or
    the other, because of course we can't know until then how to categorize each.

模块内容
===============

**Module Contents**

.. automodule:: eventlet.dagpool
	:members:
