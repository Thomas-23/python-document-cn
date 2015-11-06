:mod:`_dummy_thread` --- 替换掉原来的 :mod:`_thread` 模块
==========================================================================

.. module:: _dummy_thread
   :synopsis: 替换掉原来的 _thread 模块.

**Source code:** :source:`Lib/_dummy_thread.py`

--------------

这个模块提供了一个与 :mod:`_thread` 模块重复了的接口。
这就表示在被导入的时候，同一个平台下面不提供 :mod:`_thread` 模块。

建议使用方式::

   try:
       import _thread
   except ImportError:
       import _dummy_thread as _thread

谨慎使用这个模块，当一个线程被创建的时候被阻塞了等待另一个线程被创建时候，容易出现死锁。这经常发生在阻塞 I/O。

