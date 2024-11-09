.. _maintenance_process:

维护流程
###################

**Maintenance Process**

.. tab:: 中文

    本节为 Eventlet 维护者提供指导和流程。它们主要致力于引导 Eventlet 的生命周期。

.. tab:: 英文

    This section provide guidances and process to eventlet
    maintainers. They are mostly dedicated to Eventlet' core maintainers lead the
    life cycle of eventlet.

发布
========

**Releases**

.. tab:: 中文

    这里我们描述了处理新版本时通常遵循的流程。

.. tab:: 英文

    Here we describe the process we usually follow to
    process a new release.

1. 创建 github 问题以跟踪发布
---------------------------------------------

**1. Create a github issue to track the release**

.. tab:: 中文

    第一步是 `打开一个新的 GitHub 问题 <open a new github issue>`_ ，以便提醒其他维护者我们打算发布新版本。他们可能希望，或者不希望，提交一个特定的补丁来解决某个特定问题。这个问题将允许他们提出他们的关切。

    以下是一些 `以前处理发布过程的 issue 示例 <previous examples of issues>`_，通常我们使用以下命名模式来命名这种类型的问题："[release] eventlet <下一个版本号>"。

    请将 `release` 标签添加到此问题中。这将有助于跟踪与发布相关的工作。

.. tab:: 英文

    The first step will be to `open a new github issue`_
    to warn other maintainers about our intention
    to produce a new release. They may want, or not,
    to land a specific patch to address a specific
    topic. This issue will allow them to raise their
    concerns.

    Here are some `previous examples of issues`_ specifically
    created to handle the release process. Usually we name this
    kind of issue with the following pattern "[release] eventlet <next-version-number>".

    Please add the `release` label to this issue. It would
    ease the tracking of works related to releases.

2. 准备变更日志
------------------------

**2. Prepare the changelog**

.. tab:: 中文

    现在，您需要通过更新位于 eventlet 项目根目录下的 `NEWS` 文件来更新变更日志。

    我们建议提供即将发布版本所包含变更的整体概览。这里的目标不是列出每个提交，而是总结此版本中的重大更改。

    一旦您的更改完成，提出一个拉取请求。

    请将 `changelog` 标签添加到这个拉取请求中。这将有助于跟踪与发布相关的工作。

    如果您愿意，您可以使用之前创建的问题来列出本版本中提交的每个变更。这里有一个示例 https://github.com/eventlet/eventlet/issues/897。

.. tab:: 英文

    You now have to update the changelog by updating
    the `NEWS` file available at the root of eventlet the project.

    We would recommand to give the big picture of the changes
    landed by the coming version. The goal here is not to list
    each commit, but rather, to give a summarize of the significant
    changes made during this versions.

    Once your changes are done, then propose a pull request.

    Please add the `changelog` label to this pull request. It would
    ease the tracking of works related to releases.

    If you want, you can use the issue previously created to list
    each commits landed in this new version. Here is an example https://github.com/eventlet/eventlet/issues/897.

3. 创建标签
-----------------

**3. Create the tag**

.. tab:: 中文

    翻译后的内容，保持格式和缩进：

    一旦变更日志补丁被合并，我们就可以生成相应的新标签，以下是我们用来执行此操作的命令：

    .. code-block:: sh 

        $ git fetch origin # 从远程仓库获取最新的更新
        $ git tag -s vX.Y.Z origin/master # 创建一个签名标签，其中 X.Y.Z 对应您想要生成的版本
        $ git push origin --tags

    不要犹豫在标签信息中提供变更列表。
    这里是一个示例 https://github.com/eventlet/eventlet/releases/tag/v0.34.3
    您可以简单地重用之前创建的变更日志。

    另外，GitHub 的 UI 也允许您创建标签。

.. tab:: 英文

    Once the changelog patch is merged, then we are now
    able to produce the new corresponding tag, here are the
    commands we use to do that:

    .. code-block:: sh 
    
        $ git fetch origin # get the latest updates from the remote repo
        $ git tag -s vX.Y.Z origin/master # create a signed tag where X.Y.Z correspond to the version you are eager to produce
        $ git push origin --tags
    
    Do not hesitate to provide the list of changes in the tags message.
    Here is an example https://github.com/eventlet/eventlet/releases/tag/v0.34.3
    You can simply reuse the changelog you made previously.

    Alternatively, the Github UI also allow you creating tags.

4. 最终检查
---------------

**4. Final checks**

.. tab:: 中文

    推送之前的操作将生成一个新的构建。这个构建将生成我们的发布版本，并将这个新版本推送到 Pypi。

    您应该确保这个新版本现在已经在 Pypi 上可用，链接：https://pypi.org/project/eventlet/#history。

    您的标签应该出现在这里：https://github.com/eventlet/eventlet/tags。

.. tab:: 英文

    Pushing the previous will produce a new build. This
    build will generate our release and will push this
    new version to Pypi.

    You should ensure that this new version is now
    well available on Pypi https://pypi.org/project/eventlet/#history.

    Your tag should be listed there https://github.com/eventlet/eventlet/tags.

5. 关闭问题
------------------

**5. Close the issue**

.. tab:: 中文

    如果之前的步骤成功，那么您现在可以更新之前创建的 Github 问题。

    我建议您发表评论，附上 pypi 链接和标签链接，如此处所示: https://github.com/eventlet/eventlet/issues/875#issuecomment-1887435752 。

    现在，您可以关闭这个 Github 问题。

.. tab:: 英文

    If the previous steps were successful, then you can
    now update the Github issue that you previously created.

    I'd recommend to put a comment with the pypi link and the tag link
    like there https://github.com/eventlet/eventlet/issues/875#issuecomment-1887435752.

    You can now close this Github issue.

.. _open a new github issue: https://github.com/eventlet/eventlet/issues/new
.. _previous examples of issues: https://github.com/eventlet/eventlet/issues?q=label%3Arelease+is%3Aclosed
