.. _concurrency:

********************
并发执行
********************

这个章节描述的模块，支持并发执行代码, 根据所要执行的任务，选择恰当的工具
(CPU 限制还是 IO 限制 )，这是首选的开发风格
(事件驱动的协同多任务处理 与 抢先多任务处理 ). 这里是概览:


.. toctree::

   threading.rst
   multiprocessing.rst
   concurrent.rst
   concurrent.futures.rst
   subprocess.rst
   sched.rst
   queue.rst


下面是针对于上面一些服务提供支持的模块:

.. toctree::

   dummy_threading.rst
   _thread.rst
   _dummy_thread.rst
