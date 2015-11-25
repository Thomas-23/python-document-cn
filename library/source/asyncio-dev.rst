.. currentmodule:: asyncio

.. _asyncio-dev:

开发 asyncio
====================

异步编程不同于传统的 "有序" 编程.
这个页面列出了常见的陷阱并阐述了如何来避免这些陷阱。


.. _asyncio-debug-mode:

asyncio 的调试模式
---------------------

:mod:`asyncio` 模块的实现是为了性能。开发 asyncio ， 需要开启 debug 检查，这样可以更轻松开发异步代码。

设置应用程序开启所有的 debug 检查:

* 通过设置环境变量 :envvar:`PYTHONASYNCIODEBUG` 为 ``1`` 来开启全局的 asyncio debug 模式。

* 设置日志 :ref:`asyncio logger <asyncio-logger>` 的级别 :py:data:`logging.DEBUG`. 
  例如在启动时，调用 ``logging.basicConfig(level=logging.DEBUG)`` 。

* 配置 :mod:`warnings` 模块显示 :exc:`ResourceWarning` 警告信息。
  例如, 使用 Python 的 ``-Wdefault`` 命令行选项来显示警告信息。

debug 检查的例子:

* 日志 :ref:`coroutines 定义了但是没有 "yielded from" <asyncio-coroutine-not-scheduled>`

* 如果 :meth:`~BaseEventLoop.call_soon` 和 :meth:`~BaseEventLoop.call_at` 方法被错误的线程调用
  它们将会抛出异常。

* 日志 selector 执行时间

* 日志 回调方法执行耗时超过 100 毫秒. 这个
  :attr:`BaseEventLoop.slow_callback_duration` 属性表示 "慢" 回调函数的最小时间间隔以秒为单位。
  
* 当 transports 和 event loops 没有显式的关闭 :ref:`not closed explicitly <asyncio-close-transports>` 
  时， 会报出 :exc:`ResourceWarning` 警告信息。

.. seealso::

   :meth:`BaseEventLoop.set_debug` 方法和 :ref:`asyncio logger<asyncio-logger>` 。


清除
------------

Cancellation of tasks is not common in classic programming. In asynchronous
programming, not only it is something common, but you have to prepare your
code to handle it.

Futures and tasks can be cancelled explicitly with their :meth:`Future.cancel`
method. The :func:`wait_for` function cancels the waited task when the timeout
occurs. There are many other cases where a task can be cancelled indirectly.

Don't call :meth:`~Future.set_result` or :meth:`~Future.set_exception` method
of :class:`Future` if the future is cancelled: it would fail with an exception.
For example, write::

    if not fut.cancelled():
        fut.set_result('done')

Don't schedule directly a call to the :meth:`~Future.set_result` or the
:meth:`~Future.set_exception` method of a future with
:meth:`BaseEventLoop.call_soon`: the future can be cancelled before its method
is called.

If you wait for a future, you should check early if the future was cancelled to
avoid useless operations. Example::

    @coroutine
    def slow_operation(fut):
        if fut.cancelled():
            return
        # ... slow computation ...
        yield from fut
        # ...

The :func:`shield` function can also be used to ignore cancellation.


.. _asyncio-multithreading:

同步与多线程
------------------------------

An event loop runs in a thread and executes all callbacks and tasks in the same
thread. While a task is running in the event loop, no other task is running in
the same thread. But when the task uses ``yield from``, the task is suspended
and the event loop executes the next task.

To schedule a callback from a different thread, the
:meth:`BaseEventLoop.call_soon_threadsafe` method should be used. Example to
schedule a coroutine from a different thread::

    loop.call_soon_threadsafe(asyncio.ensure_future, coro_func())

Most asyncio objects are not thread safe. You should only worry if you access
objects outside the event loop. For example, to cancel a future, don't call
directly its :meth:`Future.cancel` method, but::

    loop.call_soon_threadsafe(fut.cancel)

To handle signals and to execute subprocesses, the event loop must be run in
the main thread.

The :meth:`BaseEventLoop.run_in_executor` method can be used with a thread pool
executor to execute a callback in different thread to not block the thread of
the event loop.

.. seealso::

   The :ref:`Synchronization primitives <asyncio-sync>` section describes ways
   to synchronize tasks.

   The :ref:`Subprocess and threads <asyncio-subprocess-threads>` section lists
   asyncio limitations to run subprocesses from different threads.




.. _asyncio-handle-blocking:

正确处理阻塞方法
-----------------------------------

Blocking functions should not be called directly. For example, if a function
blocks for 1 second, other tasks are delayed by 1 second which can have an
important impact on reactivity.

For networking and subprocesses, the :mod:`asyncio` module provides high-level
APIs like :ref:`protocols <asyncio-protocol>`.

An executor can be used to run a task in a different thread or even in a
different process, to not block the thread of the event loop. See the
:meth:`BaseEventLoop.run_in_executor` method.

.. seealso::

   The :ref:`Delayed calls <asyncio-delayed-calls>` section details how the
   event loop handles time.


.. _asyncio-logger:

日志
-------

The :mod:`asyncio` module logs information with the :mod:`logging` module in
the logger ``'asyncio'``.


.. _asyncio-coroutine-not-scheduled:

发现一直不执行的协程对象
----------------------------------------

When a coroutine function is called and its result is not passed to
:func:`ensure_future` or to the :meth:`BaseEventLoop.create_task` method,
the execution of the coroutine object will never be scheduled which is
probably a bug.  :ref:`Enable the debug mode of asyncio <asyncio-debug-mode>`
to :ref:`log a warning <asyncio-logger>` to detect it.

Example with the bug::

    import asyncio

    @asyncio.coroutine
    def test():
        print("never scheduled")

    test()

Output in debug mode::

    Coroutine test() at test.py:3 was never yielded from
    Coroutine object created at (most recent call last):
      File "test.py", line 7, in <module>
        test()

The fix is to call the :func:`ensure_future` function or the
:meth:`BaseEventLoop.create_task` method with the coroutine object.

.. seealso::

   :ref:`Pending task destroyed <asyncio-pending-task-destroyed>`.


发现一直不消失的异常
--------------------------------

Python usually calls :func:`sys.displayhook` on unhandled exceptions. If
:meth:`Future.set_exception` is called, but the exception is never consumed,
:func:`sys.displayhook` is not called. Instead, :ref:`a log is emitted
<asyncio-logger>` when the future is deleted by the garbage collector, with the
traceback where the exception was raised.

Example of unhandled exception::

    import asyncio

    @asyncio.coroutine
    def bug():
        raise Exception("not consumed")

    loop = asyncio.get_event_loop()
    asyncio.ensure_future(bug())
    loop.run_forever()
    loop.close()

Output::

    Task exception was never retrieved
    future: <Task finished coro=<coro() done, defined at asyncio/coroutines.py:139> exception=Exception('not consumed',)>
    Traceback (most recent call last):
      File "asyncio/tasks.py", line 237, in _step
        result = next(coro)
      File "asyncio/coroutines.py", line 141, in coro
        res = func(*args, **kw)
      File "test.py", line 5, in bug
        raise Exception("not consumed")
    Exception: not consumed

:ref:`Enable the debug mode of asyncio <asyncio-debug-mode>` to get the
traceback where the task was created. Output in debug mode::

    Task exception was never retrieved
    future: <Task finished coro=<bug() done, defined at test.py:3> exception=Exception('not consumed',) created at test.py:8>
    source_traceback: Object created at (most recent call last):
      File "test.py", line 8, in <module>
        asyncio.ensure_future(bug())
    Traceback (most recent call last):
      File "asyncio/tasks.py", line 237, in _step
        result = next(coro)
      File "asyncio/coroutines.py", line 79, in __next__
        return next(self.gen)
      File "asyncio/coroutines.py", line 141, in coro
        res = func(*args, **kw)
      File "test.py", line 5, in bug
        raise Exception("not consumed")
    Exception: not consumed

There are different options to fix this issue. The first option is to chain the
coroutine in another coroutine and use classic try/except::

    @asyncio.coroutine
    def handle_exception():
        try:
            yield from bug()
        except Exception:
            print("exception consumed")

    loop = asyncio.get_event_loop()
    asyncio.ensure_future(handle_exception())
    loop.run_forever()
    loop.close()

Another option is to use the :meth:`BaseEventLoop.run_until_complete`
function::

    task = asyncio.ensure_future(bug())
    try:
        loop.run_until_complete(task)
    except Exception:
        print("exception consumed")

.. seealso::

   The :meth:`Future.exception` method.


正确串接协程
--------------------------

When a coroutine function calls other coroutine functions and tasks, they
should be chained explicitly with ``yield from``. Otherwise, the execution is
not guaranteed to be sequential.

Example with different bugs using :func:`asyncio.sleep` to simulate slow
operations::

    import asyncio

    @asyncio.coroutine
    def create():
        yield from asyncio.sleep(3.0)
        print("(1) create file")

    @asyncio.coroutine
    def write():
        yield from asyncio.sleep(1.0)
        print("(2) write into file")

    @asyncio.coroutine
    def close():
        print("(3) close file")

    @asyncio.coroutine
    def test():
        asyncio.ensure_future(create())
        asyncio.ensure_future(write())
        asyncio.ensure_future(close())
        yield from asyncio.sleep(2.0)
        loop.stop()

    loop = asyncio.get_event_loop()
    asyncio.ensure_future(test())
    loop.run_forever()
    print("Pending tasks at exit: %s" % asyncio.Task.all_tasks(loop))
    loop.close()

Expected output::

    (1) create file
    (2) write into file
    (3) close file
    Pending tasks at exit: set()

Actual output::

    (3) close file
    (2) write into file
    Pending tasks at exit: {<Task pending create() at test.py:7 wait_for=<Future pending cb=[Task._wakeup()]>>}
    Task was destroyed but it is pending!
    task: <Task pending create() done at test.py:5 wait_for=<Future pending cb=[Task._wakeup()]>>

The loop stopped before the ``create()`` finished, ``close()`` has been called
before ``write()``, whereas coroutine functions were called in this order:
``create()``, ``write()``, ``close()``.

To fix the example, tasks must be marked with ``yield from``::

    @asyncio.coroutine
    def test():
        yield from asyncio.ensure_future(create())
        yield from asyncio.ensure_future(write())
        yield from asyncio.ensure_future(close())
        yield from asyncio.sleep(2.0)
        loop.stop()

Or without ``asyncio.ensure_future()``::

    @asyncio.coroutine
    def test():
        yield from create()
        yield from write()
        yield from close()
        yield from asyncio.sleep(2.0)
        loop.stop()


.. _asyncio-pending-task-destroyed:

挂起的任务被销毁了
----------------------

If a pending task is destroyed, the execution of its wrapped :ref:`coroutine
<coroutine>` did not complete. It is probably a bug and so a warning is logged.

Example of log::

    Task was destroyed but it is pending!
    task: <Task pending coro=<kill_me() done, defined at test.py:5> wait_for=<Future pending cb=[Task._wakeup()]>>

:ref:`Enable the debug mode of asyncio <asyncio-debug-mode>` to get the
traceback where the task was created. Example of log in debug mode::

    Task was destroyed but it is pending!
    source_traceback: Object created at (most recent call last):
      File "test.py", line 15, in <module>
        task = asyncio.ensure_future(coro, loop=loop)
    task: <Task pending coro=<kill_me() done, defined at test.py:5> wait_for=<Future pending cb=[Task._wakeup()] created at test.py:7> created at test.py:15>


.. seealso::

   :ref:`Detect coroutine objects never scheduled <asyncio-coroutine-not-scheduled>`.

.. _asyncio-close-transports:

关闭 transports 和事件循环
--------------------------------

When a transport is no more needed, call its ``close()`` method to release
resources. Event loops must also be closed explicitly.

If a transport or an event loop is not closed explicitly, a
:exc:`ResourceWarning` warning will be emitted in its destructor. By default,
:exc:`ResourceWarning` warnings are ignored. The :ref:`Debug mode of asyncio
<asyncio-debug-mode>` section explains how to display them.
