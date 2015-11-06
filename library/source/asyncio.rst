:mod:`asyncio` -- 异步 I/O, 事件循环, 协程任务
====================================================================

.. module:: asyncio
   :synopsis: 异步 I/O, 事件循环, 协程任务。

.. note::

   asyncio 是一个 :term:`临时性基础包<暂时性的包>` ，已经包含在标准库中。
   如果核心开发人员认为需要修改（决定包含还是移除这个模块）可能会向后不兼容。

.. versionadded:: 3.4

**源代码:**  :source:`Lib/asyncio/`

--------------

这个模块提供了使用协程编写并行代码的基础，还包括多路访问套接字和其他资源，运行网络客户端和服务端以及其他基本体。
下面是这个包内包含的更详细的列表:

* 一个包含各式各样系统实现的插件 :ref:`event loop <asyncio-event-loop>` ；

* :ref:`transport <asyncio-transport>` 和 :ref:`protocol <asyncio-protocol>` 的抽象
  （这俩类似于 `Twisted <https://twistedmatrix.com/trac/>`_  ）；

* 具体支持 TCP, UDP, SSL, subprocess pipes, 延迟调用, 以及其他（据系统而定）；

* 原 :class:`Future` 类演变成了一个 :mod:`concurrent.futures` 模块， 用于适配事件循环；

* 为了在编写协程代码时更加顺畅，协程任务基于 ``yield from`` (:PEP:`380`)之上了（译者注：原来2.*版本或者3.4以下版本的 Python 只支持 ``yield`` ）；

* 移除了 :class:`Future` 和 coroutines ；

* :ref:`synchronization primitives <asyncio-sync>` 在协程和单线程之间使用，取代这些原来 :mod:`threading` 模块中的；

* 很多时候你不得不使用一个类库来调用阻塞IO，这里提供了一个接口来清理线程池

异步编程比原来的 "有序"编程要复杂的多: 请参考 :ref:`Develop with asyncio <asyncio-dev>` 页面，它列出了
一些常见的陷阱以及解释了如何来避免这些陷阱。 在开发过程中 :ref:`Enable the debug mode <asyncio-debug-mode>` 以便
发现一些常见的问题。

目录表:

.. toctree::
   :maxdepth: 3

   asyncio-eventloop.rst
   asyncio-eventloops.rst
   asyncio-task.rst
   asyncio-protocol.rst
   asyncio-stream.rst
   asyncio-subprocess.rst
   asyncio-sync.rst
   asyncio-queue.rst
   asyncio-dev.rst

.. seealso::

   :PEP:`3156` 设计了 :mod:`asyncio` 模块。 :PEP:`3153` 说明了 transports 和 protocols 的原始意图。


