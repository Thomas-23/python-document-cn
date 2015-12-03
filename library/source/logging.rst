:mod:`logging` --- Python 日志工具
==============================================

.. module:: logging
   :synopsis: Flexible event logging system for applications.


.. moduleauthor:: Vinay Sajip <vinay_sajip@red-dove.com>
.. sectionauthor:: Vinay Sajip <vinay_sajip@red-dove.com>


.. index:: pair: Errors; logging

.. sidebar:: 重要

   这个页面包含 API 参考信息. 对于入门教程和更高级的讨论， 请看

   * :ref:`Basic Tutorial <logging-basic-tutorial>`
   * :ref:`Advanced Tutorial <logging-advanced-tutorial>`
   * :ref:`Logging Cookbook <logging-cookbook>`


**Source code:** :source:`Lib/logging/__init__.py`

--------------

这个模块定义函数和类，实现了一个用于应用程序和库的灵活的事件日志系统.

标准库模块体提供的 日志 API 的关键作用是所有的 Python 模块都能加入到日志中来, 
因此你的应用程序日志能够包含你自己的信息，这些信息集合了来自第三方模块的信息.

这个模块提供很多功能和灵活性。如果你不熟悉使用日志，最好的办法是通过入门教程
掌握它（查看右边的链接）.

模块定义的基本类，包括它们的方法，列出如下.

* Loggers 暴露出接口以供应用程序代码直接使用.
* Handlers 发送由 loggers 创建的日志记录到达正确的目的地.
* Filters 提供了一个出色条理的工具由于决定具体哪些日志记录被输出.
* Formatters 指定最终输出的日志记录格式.


.. _logger:

日志对象
--------------

Loggers 有以下属性和方法。注意 Loggers 从不直接实例化， 通常通过模块级别的方法
``logging.getLogger(name)`` 来实例化.  多次调用同一个名字的 :func:`getLogger` 
将会返回同一个日志对象。

``name`` 是潜在的一个间隔分割有层次的值，例如
``foo.bar.baz`` (虽然它也可以是仅仅很清晰的,例如 ``foo`` ).
Loggers 是在一个层次列表上逐层下降的，这些日志对象视为更高层的子日志对象.  
例如，给定一个 logger 的名字为 ``foo``,
loggers 的名字为 ``foo.bar``, ``foo.bar.baz``, 和 ``foo.bam`` 它们都是
``foo`` 的后代.  logger 名次层次关系类似于 Python 包的层次关系，
如果你组织你的 loggers 在每个模块中使用推荐的构造方式 ``logging.getLogger(__name__)`` ，
它们会完全相同, 那是因为在一个模块中 ``__name__`` 就是 Python 包命名空间中模块名字。


.. class:: Logger

.. attribute:: Logger.propagate

   如果这个取值为真, 对于这个日志对象的事件记录将会传递给更高级别的（父集别的）
   日志对象的处理器, 增加任何处理器都会附加到这个日志对象. 
   消息直接传递到父集的日志处理器 - 无论父集日志对象的级别还是过滤器的问题都考虑在内.

   如果取值为假，日志消息不会传递给父集的日志对象的处理器.

   构造器设置这个属性为 ``True``.

   .. note:: 如果你附加了一个处理器到一个日志对象和它有一个或多个父集，
      它可能会发送同一个记录多次。通常来说， 你不应该附加一个处理器到超过一个
      日志对象  - 如果你仅仅附件它到合适的日志对象，即它是在日志的最高层次上, 
      那么它将会通过全部的子孙日志对象观察所有的事件记录， 以供它们传播设置将其余的
      设置为 ``True``.  一个常用的场景是附件处理器给根日志对象，然后让其传播，以留意剩下的.

.. method:: Logger.setLevel(lvl)

   对 logger 的 *lvl* 设置一个阈值。 记录的消息严重程度低于 *lvl* 会被忽略。
   如果一个 logger 被创建，级别设置为
   :const:`NOTSET` (当 logger 是根 logger时，将导致所有的消息都被处理, 如果 logger 是一个父类但不是
   根，那么会委托给父类). 注意，根 logger 是以级别 :const:`WARNING` 创建的.

   术语 '委托给父类' 意味着如果一个 logger 有一个 NOTSET 级别， 它会贯穿整个父集 loggers 直到找到一个
   祖先类不是 NOTSET 为止，否则将使用 root.

   如果一个祖先类的级别不是 NOTSET, 那么从这个祖先类搜索开始地方，祖先类的级别将视为有效的 logger 级别, 
   然后使用它来决定一个日志事件的处理。

   如果到达了根，还是只有 NOTSET, 那么所有的消息将会被处理.
   否则，根的级别将会作为有效级别.

   请查看级别列表 :ref:`levels` .

   .. versionchanged:: 3.2
      *lvl* 参数现在可以接收一个字符串描述的级别，例如 'INFO' 可以作为整数常数  :const:`INFO` 的替代
      注意，注意，不管怎样， 级别的内部存储还是整数，并且例如 :meth:`getEffectiveLevel` 和
      :meth:`isEnabledFor` 这样的方法，将会返回或期望传递的是整数.


.. method:: Logger.isEnabledFor(lvl)

   指示是否一个消息的严重程度 *lvl* 将会被这个 logger 处理。
   这个方法通过 ``logging.disable(lvl)`` 检查的首先是模块级别的日志级别
   然后通过 :meth:`getEffectiveLevel` 来决定这个模块的有效日志级别。


.. method:: Logger.getEffectiveLevel()

   指示这个 logger 的有效级别。如果已经通过 :meth:`setLevel` 设置了一个非
   :const:`NOTSET` 的级别, 它将被返回。否则贯穿整个日志层次直到根日志找到一个
   非 :const:`NOTSET` 值进行返回。返回的值是一个整数，通常为一个 
   :const:`logging.DEBUG` , :const:`logging.INFO` , 等


.. method:: Logger.getChild(suffix)

   返回当前日志的子孙集日志, 它们通过 suffix 决定。
   像这样, ``logging.getLogger('abc').getChild('def.ghi')`` 将会返回与
   ``logging.getLogger('abc.def.ghi')`` 相同的日志. 当父类日志的名字使用的是
   ``__name__`` 而不是一个单纯的字符串时，这是一个便捷的方法而且很有用。
    
   .. versionadded:: 3.2


.. method:: Logger.debug(msg, *args, **kwargs)

   在这个 logger 上记录消息的级别为 :const:`DEBUG` . 这个 *msg* 是一个消息格式化字符串
   参数 *args* 会使用字符串格式化操作符合并成 *msg* 。（注意这表示你可以在格式化字符串中
   使用关键字，可以结合一个单独的字典参数.)

   在 *kwargs* 有三个关键字参数用于检查:
   *exc_info*, *stack_info*, 和 *extra*.

   如果 *exc_info* 没有赋值为假，它会触发异常信息添加到日志消息中. 
   可以是一个异常元组（在格式化中由
   :func:`sys.exc_info` 返回)或者提供一个异常实例，供它使用，否则
   :func:`sys.exc_info` 被调用时获得异常信息。

   第二个可选参数是 *stack_info*, 默认为
   ``False``. 如果为真，堆栈信息将会加到日志信息中，包含实际的日志调用。注意它不同
   于通过指定的  *exc_info* 的堆栈信息: 这个格式化器是一个堆栈框架，
   在当前线程中是从栈的底部往上进行日志调用, 而后一个是关于堆栈框架的，当搜索异常处理器时
   它已经解开，后面跟着一个异常.

   你可以独立于  *exc_info* 指定 *stack_info* ,例如仅仅演示怎样在你的代码中获得一个点
   即使没有异常抛出。 堆栈框架会打印出下面的一个头部行，如下::

       Stack (most recent call last):

   这是模仿 ``Traceback (most recent call last):`` 它用于显示异常框架.

   第三个关键字参数时候 *extra* ,它用于使用用户定义的属性，传递一个字典生成
   LogRecord 的 __dict__ 来创建日志事件. 这些自定义的属性能够在后续使用，例如, 
   它们能够被纳入日志消息中，例子::

      FORMAT = '%(asctime)-15s %(clientip)s %(user)-8s %(message)s'
      logging.basicConfig(format=FORMAT)
      d = {'clientip': '192.168.0.1', 'user': 'fbloggs'}
      logger = logging.getLogger('tcpserver')
      logger.warning('Protocol problem: %s', 'connection reset', extra=d)

   能够输出像下面这样的信息  ::

      2006-02-08 22:20:02,165 192.168.0.1 fbloggs  Protocol problem: connection reset

   字典中的键传递到 *extra* 中，不能与日志系统产生冲突。 (关于日志系统使用的键，
   详情请看 :class:`Formatter` 文档.)

   如果你选择在日志消息中使用这些属性，你应该多加练习。在上面的例子中，
   比如 :class:`Formatter` 已经设置了一个格式化字符串，期望 'clientip' ， 'user' 
   属性在 LogRecord 属性当中. 如果它们缺失，消息将不会记录，因为会出现一个格式化
   字符串异常。因此在这种情况下，你必需每次都给字典 *extra* 传递这些键.

   这样做很烦人，这个功能的目的是用于特殊的情况下的，例如多线程严重级别，同一段代码运行
   在多个上下文中，并且与之相关的条件也依赖与这个上下文(例如，上面例子中的远程客户端 ip 
   地址和认证的用户名字)，这种情况下，它就可以像指定 :class:`Formatter` 一样
   使用特别的 :class:`Handler` 。

   .. versionadded:: 3.2
      增加 *stack_info* 参数.

   .. versionchanged:: 3.5
      参数 *exc_info* 现在可以接受异常实例.


.. method:: Logger.info(msg, *args, **kwargs)

   在这个 logger 上面记录消息的级别为 :const:`INFO` . 这些参数的解释同 :meth:`debug`.


.. method:: Logger.warning(msg, *args, **kwargs)

   在这个 logger 上面记录消息的级别为 :const:`WARNING` . 这些参数的解释同 :meth:`debug`.

   .. note:: 有一个过时的方法 ``warn`` , 它是与 ``warning`` 功能相当的. 由于 ``warn`` 
      已经弃用, 请不要使用它 - 使用 ``warning`` 代替.

.. method:: Logger.error(msg, *args, **kwargs)

   在这个 logger 上面记录消息的级别为 :const:`ERROR` . 这些参数的解释同 :meth:`debug`.


.. method:: Logger.critical(msg, *args, **kwargs)

   在这个 logger 上面记录消息的级别为 :const:`CRITICAL` . 这些参数的解释同 :meth:`debug`.


.. method:: Logger.log(lvl, msg, *args, **kwargs)

   在这个 logger 上面使用一个整数作为消息级别. 其他参数的解释同 :meth:`debug`.


.. method:: Logger.exception(msg, *args, **kwargs)

   在这个 logger 上面记录消息的级别为 :const:`ERROR` . 这些参数的解释同 :meth:`debug`. 
   异常信息将会被加到日志信息中。这个方法应该只能被一个异常处理器调用.


.. method:: Logger.addFilter(filt)

   给这个 logger 增加一个指定的过滤器 *filt* .


.. method:: Logger.removeFilter(filt)

   在这个 logger 上 移除一个指定的过滤器 *filt* .


.. method:: Logger.filter(record)

   在记录上使用这个 logger 的过滤器，并且如果这个记录被处理了返回一个真值. 
   过滤器是依次进行的，直到有一个返回假值。 如果它们都没有返回假值，这个记录会传递给
   处理器处理， 如果出现了返回假值，后面就没有别的操作了。


.. method:: Logger.addHandler(hdlr)

   在这个 logger 上面增加一个指定的处理器 *hdlr* .


.. method:: Logger.removeHandler(hdlr)

   移除这个 logger 上指定的处理器 *hdlr* 。


.. method:: Logger.findCaller(stack_info=False)

   找到调用者源文件的文件名和行号。返回文件名，行号
   函数名字和堆栈信息，是一个4个元素的元组。除非设置 *stack_info* 
   为 *True* 否则堆栈信息返回为  *None* .


.. method:: Logger.handle(record)

   通过传递一条记录到所有与之关联的 logger 和 它的父集来进行处理。（直到发现 一个
   *propagate* 假值).这个方法用于从一个套接字接收到 unpickled 记录， 它们同样在本地创建
    :meth:`~Logger.filter` 应用于日志级别过滤。


.. method:: Logger.makeRecord(name, lvl, fn, lno, msg, args, exc_info, func=None, extra=None, sinfo=None)

   这是一个工厂方法，可以在子类中重写，来创建一个指定的 :class:`LogRecord` 实例.

.. method:: Logger.hasHandlers()

   检查这个  logger 有配置和好的处理器。它会去找自身和 logger 层次父类中处理器.
   如果发现一个处理器返回 ``True`` , 否则返回 ``False``. 如果  'propagate'  属性设置为
   假，这个方法会停止搜索 logger 上面的层次。 - 那么这将是最后一个检查是否存在处理器的 Logger.

   .. versionadded:: 3.2


.. _levels:

日志级别
--------------

日志级别的数字值见下表. 如果你想定义自己的级别，将对它们会有兴趣,
并且需要指定一个与预先设置好的级别一个不同的值。如果你定了一个相同的数字值，它
会重写预先设置好的值；这样预先设置好的值就丢失了.

+--------------+---------------+
| Level        | Numeric value |
+==============+===============+
| ``CRITICAL`` | 50            |
+--------------+---------------+
| ``ERROR``    | 40            |
+--------------+---------------+
| ``WARNING``  | 30            |
+--------------+---------------+
| ``INFO``     | 20            |
+--------------+---------------+
| ``DEBUG``    | 10            |
+--------------+---------------+
| ``NOTSET``   | 0             |
+--------------+---------------+


.. _handler:

处理器对象
---------------

处理器对象有下面一些属性和方法。注意 :class:`Handler` 从来不直接进行实例化; 
这个类将作为子类的一个基础类存在。不论如何，在你子类的 :meth:`__init__` 
方法中需要调用 :meth:`Handler.__init__`.


.. method:: Handler.__init__(level=NOTSET)

   通过设置一个级别初始化一个 :class:`Handler` 实例, 设置一系列的过滤到空列表中并且
   使用 :meth:`createLock` 创建
   一个锁来序列化访问一个 I/O 机制.


.. method:: Handler.createLock()

   初始化一个线程锁，它可以序列号访问底层的 I/O 功能，由于底层的 I/O 并非线程安全的.


.. method:: Handler.acquire()

   获取由 :meth:`createLock` 创建的线程锁.


.. method:: Handler.release()

   释放由 :meth:`acquire` 获取到的线程锁.


.. method:: Handler.setLevel(lvl)

   Sets the threshold for this handler to *lvl*. Logging messages which are less
   severe than *lvl* will be ignored. When a handler is created, the level is set
   to :const:`NOTSET` (which causes all messages to be processed).

   See :ref:`levels` for a list of levels.

   .. versionchanged:: 3.2
      The *lvl* parameter now accepts a string representation of the
      level such as 'INFO' as an alternative to the integer constants
      such as :const:`INFO`.


.. method:: Handler.setFormatter(form)

   Sets the :class:`Formatter` for this handler to *form*.


.. method:: Handler.addFilter(filt)

   Adds the specified filter *filt* to this handler.


.. method:: Handler.removeFilter(filt)

   Removes the specified filter *filt* from this handler.


.. method:: Handler.filter(record)

   Applies this handler's filters to the record and returns a true value if the
   record is to be processed. The filters are consulted in turn, until one of
   them returns a false value. If none of them return a false value, the record
   will be emitted. If one returns a false value, the handler will not emit the
   record.


.. method:: Handler.flush()

   Ensure all logging output has been flushed. This version does nothing and is
   intended to be implemented by subclasses.


.. method:: Handler.close()

   Tidy up any resources used by the handler. This version does no output but
   removes the handler from an internal list of handlers which is closed when
   :func:`shutdown` is called. Subclasses should ensure that this gets called
   from overridden :meth:`close` methods.


.. method:: Handler.handle(record)

   Conditionally emits the specified logging record, depending on filters which may
   have been added to the handler. Wraps the actual emission of the record with
   acquisition/release of the I/O thread lock.


.. method:: Handler.handleError(record)

   This method should be called from handlers when an exception is encountered
   during an :meth:`emit` call. If the module-level attribute
   ``raiseExceptions`` is ``False``, exceptions get silently ignored. This is
   what is mostly wanted for a logging system - most users will not care about
   errors in the logging system, they are more interested in application
   errors. You could, however, replace this with a custom handler if you wish.
   The specified record is the one which was being processed when the exception
   occurred. (The default value of ``raiseExceptions`` is ``True``, as that is
   more useful during development).


.. method:: Handler.format(record)

   Do formatting for a record - if a formatter is set, use it. Otherwise, use the
   default formatter for the module.


.. method:: Handler.emit(record)

   Do whatever it takes to actually log the specified logging record. This version
   is intended to be implemented by subclasses and so raises a
   :exc:`NotImplementedError`.

For a list of handlers included as standard, see :mod:`logging.handlers`.

.. _formatter-objects:

格式化对象
-----------------

.. currentmodule:: logging

:class:`Formatter` 有下面一些属性和方法. 它们负责转换 :class:`LogRecord` 
为一个普通的字符串，这个字符串可能适合人理解或者其他外部系统. 
:class:`Formatter` 基类允许指定一个格式化字符. 如果没有提供这个格式化字符串
默认使用 ``'%(message)s'`` , 在日志调用时它仅仅消息信息. 如果想包含其他额外选项
作为输出格式例如时间戳，请往下读。

一个 Formatter 可以以一个格式化字符串初始化，它应用为与 :class:`LogRecord` 
的属性 - 就像上面提到的默认值一样，对用户消息的影响因素为 :class:`LogRecord`'
消息属性来预格式化.  这个格式化字符串包含标准的 Python %-形式的映射关键字
. 对于格式化字符串的详细信息请看章节 :ref:`old-string-formatting` 。

有用的映射关键字由 :class:`LogRecord` 给出，在章节
:ref:`logrecord-attributes` 上。


.. class:: Formatter(fmt=None, datefmt=None, style='%')

   Returns a new instance of the :class:`Formatter` class.  The instance is
   initialized with a format string for the message as a whole, as well as a
   format string for the date/time portion of a message.  If no *fmt* is
   specified, ``'%(message)s'`` is used.  If no *datefmt* is specified, the
   ISO8601 date format is used.

   The *style* parameter can be one of '%', '{' or '$' and determines how
   the format string will be merged with its data: using one of %-formatting,
   :meth:`str.format` or :class:`string.Template`. See :ref:`formatting-styles`
   for more information on using {- and $-formatting for log messages.

   .. versionchanged:: 3.2
      The *style* parameter was added.


   .. method:: format(record)

      The record's attribute dictionary is used as the operand to a string
      formatting operation. Returns the resulting string. Before formatting the
      dictionary, a couple of preparatory steps are carried out. The *message*
      attribute of the record is computed using *msg* % *args*. If the
      formatting string contains ``'(asctime)'``, :meth:`formatTime` is called
      to format the event time. If there is exception information, it is
      formatted using :meth:`formatException` and appended to the message. Note
      that the formatted exception information is cached in attribute
      *exc_text*. This is useful because the exception information can be
      pickled and sent across the wire, but you should be careful if you have
      more than one :class:`Formatter` subclass which customizes the formatting
      of exception information. In this case, you will have to clear the cached
      value after a formatter has done its formatting, so that the next
      formatter to handle the event doesn't use the cached value but
      recalculates it afresh.

      If stack information is available, it's appended after the exception
      information, using :meth:`formatStack` to transform it if necessary.


   .. method:: formatTime(record, datefmt=None)

      This method should be called from :meth:`format` by a formatter which
      wants to make use of a formatted time. This method can be overridden in
      formatters to provide for any specific requirement, but the basic behavior
      is as follows: if *datefmt* (a string) is specified, it is used with
      :func:`time.strftime` to format the creation time of the
      record. Otherwise, the ISO8601 format is used.  The resulting string is
      returned.

      This function uses a user-configurable function to convert the creation
      time to a tuple. By default, :func:`time.localtime` is used; to change
      this for a particular formatter instance, set the ``converter`` attribute
      to a function with the same signature as :func:`time.localtime` or
      :func:`time.gmtime`. To change it for all formatters, for example if you
      want all logging times to be shown in GMT, set the ``converter``
      attribute in the ``Formatter`` class.

      .. versionchanged:: 3.3
         Previously, the default ISO 8601 format was hard-coded as in this
         example: ``2010-09-06 22:38:15,292`` where the part before the comma is
         handled by a strptime format string (``'%Y-%m-%d %H:%M:%S'``), and the
         part after the comma is a millisecond value. Because strptime does not
         have a format placeholder for milliseconds, the millisecond value is
         appended using another format string, ``'%s,%03d'`` – and both of these
         format strings have been hardcoded into this method. With the change,
         these strings are defined as class-level attributes which can be
         overridden at the instance level when desired. The names of the
         attributes are ``default_time_format`` (for the strptime format string)
         and ``default_msec_format`` (for appending the millisecond value).

   .. method:: formatException(exc_info)

      Formats the specified exception information (a standard exception tuple as
      returned by :func:`sys.exc_info`) as a string. This default implementation
      just uses :func:`traceback.print_exception`. The resulting string is
      returned.

   .. method:: formatStack(stack_info)

      Formats the specified stack information (a string as returned by
      :func:`traceback.print_stack`, but with the last newline removed) as a
      string. This default implementation just returns the input value.

.. _filter:

过滤器对象
--------------

``Filters`` 可以被 ``Handlers`` 和 ``Loggers`` 使用，对于复杂的过滤，可以考虑使用级别. 
filter 基类仅允许在确定某个日志层次点下面的事件. 例如一个过滤器实例化为 'A.B' 
将允许 'A.B', 'A.B.C' ,'A.B.C.D', 'A.B.D' 等的日志记录的事件, 但不允许 
'A.BB', 'B.A.B' 等. 如果初始化为空字符串，所有的事件都会通过.


.. class:: Filter(name='')

   Returns an instance of the :class:`Filter` class. If *name* is specified, it
   names a logger which, together with its children, will have its events allowed
   through the filter. If *name* is the empty string, allows every event.


   .. method:: filter(record)

      Is the specified record to be logged? Returns zero for no, nonzero for
      yes. If deemed appropriate, the record may be modified in-place by this
      method.

Note that filters attached to handlers are consulted before an event is
emitted by the handler, whereas filters attached to loggers are consulted
whenever an event is logged (using :meth:`debug`, :meth:`info`,
etc.), before sending an event to handlers. This means that events which have
been generated by descendant loggers will not be filtered by a logger's filter
setting, unless the filter has also been applied to those descendant loggers.

You don't actually need to subclass ``Filter``: you can pass any instance
which has a ``filter`` method with the same semantics.

.. versionchanged:: 3.2
   You don't need to create specialized ``Filter`` classes, or use other
   classes with a ``filter`` method: you can use a function (or other
   callable) as a filter. The filtering logic will check to see if the filter
   object has a ``filter`` attribute: if it does, it's assumed to be a
   ``Filter`` and its :meth:`~Filter.filter` method is called. Otherwise, it's
   assumed to be a callable and called with the record as the single
   parameter. The returned value should conform to that returned by
   :meth:`~Filter.filter`.

Although filters are used primarily to filter records based on more
sophisticated criteria than levels, they get to see every record which is
processed by the handler or logger they're attached to: this can be useful if
you want to do things like counting how many records were processed by a
particular logger or handler, or adding, changing or removing attributes in
the LogRecord being processed. Obviously changing the LogRecord needs to be
done with some care, but it does allow the injection of contextual information
into logs (see :ref:`filters-contextual`).

.. _log-record:

日志记录对象
-----------------

在每一次有东西要记录时， :class:`LogRecord` 实例由 :class:`Logger` 自动创建，
也可以通过 :func:`makeLogRecord` 手动创建（例如， 从一个 pickled 事件中接收电报）.


.. class:: LogRecord(name, level, pathname, lineno, msg, args, exc_info, func=None, sinfo=None)

   Contains all the information pertinent to the event being logged.

   The primary information is passed in :attr:`msg` and :attr:`args`, which
   are combined using ``msg % args`` to create the :attr:`message` field of the
   record.

   :param name:  The name of the logger used to log the event represented by
                 this LogRecord. Note that this name will always have this
                 value, even though it may be emitted by a handler attached to
                 a different (ancestor) logger.
   :param level: The numeric level of the logging event (one of DEBUG, INFO etc.)
                 Note that this is converted to *two* attributes of the LogRecord:
                 ``levelno`` for the numeric value and ``levelname`` for the
                 corresponding level name.
   :param pathname: The full pathname of the source file where the logging call
                    was made.
   :param lineno: The line number in the source file where the logging call was
                  made.
   :param msg: The event description message, possibly a format string with
               placeholders for variable data.
   :param args: Variable data to merge into the *msg* argument to obtain the
                event description.
   :param exc_info: An exception tuple with the current exception information,
                    or *None* if no exception information is available.
   :param func: The name of the function or method from which the logging call
                was invoked.
   :param sinfo: A text string representing stack information from the base of
                 the stack in the current thread, up to the logging call.

   .. method:: getMessage()

      Returns the message for this :class:`LogRecord` instance after merging any
      user-supplied arguments with the message. If the user-supplied message
      argument to the logging call is not a string, :func:`str` is called on it to
      convert it to a string. This allows use of user-defined classes as
      messages, whose ``__str__`` method can return the actual format string to
      be used.

   .. versionchanged:: 3.2
      The creation of a ``LogRecord`` has been made more configurable by
      providing a factory which is used to create the record. The factory can be
      set using :func:`getLogRecordFactory` and :func:`setLogRecordFactory`
      (see this for the factory's signature).

   This functionality can be used to inject your own values into a
   LogRecord at creation time. You can use the following pattern::

      old_factory = logging.getLogRecordFactory()

      def record_factory(*args, **kwargs):
          record = old_factory(*args, **kwargs)
          record.custom_attribute = 0xdecafbad
          return record

      logging.setLogRecordFactory(record_factory)

   With this pattern, multiple factories could be chained, and as long
   as they don't overwrite each other's attributes or unintentionally
   overwrite the standard attributes listed above, there should be no
   surprises.


.. _logrecord-attributes:

日志记录属性
--------------------

The LogRecord has a number of attributes, most of which are derived from the
parameters to the constructor. (Note that the names do not always correspond
exactly between the LogRecord constructor parameters and the LogRecord
attributes.) These attributes can be used to merge data from the record into
the format string. The following table lists (in alphabetical order) the
attribute names, their meanings and the corresponding placeholder in a %-style
format string.

If you are using {}-formatting (:func:`str.format`), you can use
``{attrname}`` as the placeholder in the format string. If you are using
$-formatting (:class:`string.Template`), use the form ``${attrname}``. In
both cases, of course, replace ``attrname`` with the actual attribute name
you want to use.

In the case of {}-formatting, you can specify formatting flags by placing them
after the attribute name, separated from it with a colon. For example: a
placeholder of ``{msecs:03d}`` would format a millisecond value of ``4`` as
``004``. Refer to the :meth:`str.format` documentation for full details on
the options available to you.

+----------------+-------------------------+-----------------------------------------------+
| Attribute name | Format                  | Description                                   |
+================+=========================+===============================================+
| args           | You shouldn't need to   | The tuple of arguments merged into ``msg`` to |
|                | format this yourself.   | produce ``message``, or a dict whose values   |
|                |                         | are used for the merge (when there is only one|
|                |                         | argument, and it is a dictionary).            |
+----------------+-------------------------+-----------------------------------------------+
| asctime        | ``%(asctime)s``         | Human-readable time when the                  |
|                |                         | :class:`LogRecord` was created.  By default   |
|                |                         | this is of the form '2003-07-08 16:49:45,896' |
|                |                         | (the numbers after the comma are millisecond  |
|                |                         | portion of the time).                         |
+----------------+-------------------------+-----------------------------------------------+
| created        | ``%(created)f``         | Time when the :class:`LogRecord` was created  |
|                |                         | (as returned by :func:`time.time`).           |
+----------------+-------------------------+-----------------------------------------------+
| exc_info       | You shouldn't need to   | Exception tuple (à la ``sys.exc_info``) or,   |
|                | format this yourself.   | if no exception has occurred, *None*.         |
+----------------+-------------------------+-----------------------------------------------+
| filename       | ``%(filename)s``        | Filename portion of ``pathname``.             |
+----------------+-------------------------+-----------------------------------------------+
| funcName       | ``%(funcName)s``        | Name of function containing the logging call. |
+----------------+-------------------------+-----------------------------------------------+
| levelname      | ``%(levelname)s``       | Text logging level for the message            |
|                |                         | (``'DEBUG'``, ``'INFO'``, ``'WARNING'``,      |
|                |                         | ``'ERROR'``, ``'CRITICAL'``).                 |
+----------------+-------------------------+-----------------------------------------------+
| levelno        | ``%(levelno)s``         | Numeric logging level for the message         |
|                |                         | (:const:`DEBUG`, :const:`INFO`,               |
|                |                         | :const:`WARNING`, :const:`ERROR`,             |
|                |                         | :const:`CRITICAL`).                           |
+----------------+-------------------------+-----------------------------------------------+
| lineno         | ``%(lineno)d``          | Source line number where the logging call was |
|                |                         | issued (if available).                        |
+----------------+-------------------------+-----------------------------------------------+
| module         | ``%(module)s``          | Module (name portion of ``filename``).        |
+----------------+-------------------------+-----------------------------------------------+
| msecs          | ``%(msecs)d``           | Millisecond portion of the time when the      |
|                |                         | :class:`LogRecord` was created.               |
+----------------+-------------------------+-----------------------------------------------+
| message        | ``%(message)s``         | The logged message, computed as ``msg %       |
|                |                         | args``. This is set when                      |
|                |                         | :meth:`Formatter.format` is invoked.          |
+----------------+-------------------------+-----------------------------------------------+
| msg            | You shouldn't need to   | The format string passed in the original      |
|                | format this yourself.   | logging call. Merged with ``args`` to         |
|                |                         | produce ``message``, or an arbitrary object   |
|                |                         | (see :ref:`arbitrary-object-messages`).       |
+----------------+-------------------------+-----------------------------------------------+
| name           | ``%(name)s``            | Name of the logger used to log the call.      |
+----------------+-------------------------+-----------------------------------------------+
| pathname       | ``%(pathname)s``        | Full pathname of the source file where the    |
|                |                         | logging call was issued (if available).       |
+----------------+-------------------------+-----------------------------------------------+
| process        | ``%(process)d``         | Process ID (if available).                    |
+----------------+-------------------------+-----------------------------------------------+
| processName    | ``%(processName)s``     | Process name (if available).                  |
+----------------+-------------------------+-----------------------------------------------+
| relativeCreated| ``%(relativeCreated)d`` | Time in milliseconds when the LogRecord was   |
|                |                         | created, relative to the time the logging     |
|                |                         | module was loaded.                            |
+----------------+-------------------------+-----------------------------------------------+
| stack_info     | You shouldn't need to   | Stack frame information (where available)     |
|                | format this yourself.   | from the bottom of the stack in the current   |
|                |                         | thread, up to and including the stack frame   |
|                |                         | of the logging call which resulted in the     |
|                |                         | creation of this record.                      |
+----------------+-------------------------+-----------------------------------------------+
| thread         | ``%(thread)d``          | Thread ID (if available).                     |
+----------------+-------------------------+-----------------------------------------------+
| threadName     | ``%(threadName)s``      | Thread name (if available).                   |
+----------------+-------------------------+-----------------------------------------------+

.. versionchanged:: 3.1
   *processName* was added.


.. _logger-adapter:

日志适配器对象
---------------------

:class:`LoggerAdapter` instances are used to conveniently pass contextual
information into logging calls. For a usage example, see the section on
:ref:`adding contextual information to your logging output <context-info>`.

.. class:: LoggerAdapter(logger, extra)

   Returns an instance of :class:`LoggerAdapter` initialized with an
   underlying :class:`Logger` instance and a dict-like object.

   .. method:: process(msg, kwargs)

      Modifies the message and/or keyword arguments passed to a logging call in
      order to insert contextual information. This implementation takes the object
      passed as *extra* to the constructor and adds it to *kwargs* using key
      'extra'. The return value is a (*msg*, *kwargs*) tuple which has the
      (possibly modified) versions of the arguments passed in.

In addition to the above, :class:`LoggerAdapter` supports the following
methods of :class:`Logger`: :meth:`~Logger.debug`, :meth:`~Logger.info`,
:meth:`~Logger.warning`, :meth:`~Logger.error`, :meth:`~Logger.exception`,
:meth:`~Logger.critical`, :meth:`~Logger.log`, :meth:`~Logger.isEnabledFor`,
:meth:`~Logger.getEffectiveLevel`, :meth:`~Logger.setLevel` and
:meth:`~Logger.hasHandlers`. These methods have the same signatures as their
counterparts in :class:`Logger`, so you can use the two types of instances
interchangeably.

.. versionchanged:: 3.2
   The :meth:`~Logger.isEnabledFor`, :meth:`~Logger.getEffectiveLevel`,
   :meth:`~Logger.setLevel` and :meth:`~Logger.hasHandlers` methods were added
   to :class:`LoggerAdapter`.  These methods delegate to the underlying logger.


线程安全
-------------

The logging module is intended to be thread-safe without any special work
needing to be done by its clients. It achieves this though using threading
locks; there is one lock to serialize access to the module's shared data, and
each handler also creates a lock to serialize access to its underlying I/O.

If you are implementing asynchronous signal handlers using the :mod:`signal`
module, you may not be able to use logging from within such handlers. This is
because lock implementations in the :mod:`threading` module are not always
re-entrant, and so cannot be invoked from such signal handlers.


模块级别的方法
----------------------

除了上面描述的类，还有一些模块级别的方法.


.. function:: getLogger(name=None)

   返回一个指定名字的 logger , 如果 name ``None``, 返回层次中的根 logger. 
   如果指定， 这个名字是一个点号分割的层次名字，例如
   *'a'*, *'a.b'* 或者 *'a.b.c.d'* , 选择这些名字完全取决于使用日志功能的开发人员.

   所有调用这个指定了名字的方法将返回同一个 logger 实例.这就意味着，
   日志实例无需在应用程序不同的部分之间来回传递。


.. function:: getLoggerClass()

   Return either the standard :class:`Logger` class, or the last class passed to
   :func:`setLoggerClass`. This function may be called from within a new class
   definition, to ensure that installing a customized :class:`Logger` class will
   not undo customizations already applied by other code. For example::

      class MyLogger(logging.getLoggerClass()):
          # ... override behaviour here


.. function:: getLogRecordFactory()

   Return a callable which is used to create a :class:`LogRecord`.

   .. versionadded:: 3.2
      This function has been provided, along with :func:`setLogRecordFactory`,
      to allow developers more control over how the :class:`LogRecord`
      representing a logging event is constructed.

   See :func:`setLogRecordFactory` for more information about the how the
   factory is called.

.. function:: debug(msg, *args, **kwargs)

   Logs a message with level :const:`DEBUG` on the root logger. The *msg* is the
   message format string, and the *args* are the arguments which are merged into
   *msg* using the string formatting operator. (Note that this means that you can
   use keywords in the format string, together with a single dictionary argument.)

   There are three keyword arguments in *kwargs* which are inspected: *exc_info*
   which, if it does not evaluate as false, causes exception information to be
   added to the logging message. If an exception tuple (in the format returned by
   :func:`sys.exc_info`) is provided, it is used; otherwise, :func:`sys.exc_info`
   is called to get the exception information.

   The second optional keyword argument is *stack_info*, which defaults to
   ``False``. If true, stack information is added to the logging
   message, including the actual logging call. Note that this is not the same
   stack information as that displayed through specifying *exc_info*: The
   former is stack frames from the bottom of the stack up to the logging call
   in the current thread, whereas the latter is information about stack frames
   which have been unwound, following an exception, while searching for
   exception handlers.

   You can specify *stack_info* independently of *exc_info*, e.g. to just show
   how you got to a certain point in your code, even when no exceptions were
   raised. The stack frames are printed following a header line which says::

       Stack (most recent call last):

   This mimics the ``Traceback (most recent call last):`` which is used when
   displaying exception frames.

   The third optional keyword argument is *extra* which can be used to pass a
   dictionary which is used to populate the __dict__ of the LogRecord created for
   the logging event with user-defined attributes. These custom attributes can then
   be used as you like. For example, they could be incorporated into logged
   messages. For example::

      FORMAT = '%(asctime)-15s %(clientip)s %(user)-8s %(message)s'
      logging.basicConfig(format=FORMAT)
      d = {'clientip': '192.168.0.1', 'user': 'fbloggs'}
      logging.warning('Protocol problem: %s', 'connection reset', extra=d)

   would print something like::

      2006-02-08 22:20:02,165 192.168.0.1 fbloggs  Protocol problem: connection reset

   The keys in the dictionary passed in *extra* should not clash with the keys used
   by the logging system. (See the :class:`Formatter` documentation for more
   information on which keys are used by the logging system.)

   If you choose to use these attributes in logged messages, you need to exercise
   some care. In the above example, for instance, the :class:`Formatter` has been
   set up with a format string which expects 'clientip' and 'user' in the attribute
   dictionary of the LogRecord. If these are missing, the message will not be
   logged because a string formatting exception will occur. So in this case, you
   always need to pass the *extra* dictionary with these keys.

   While this might be annoying, this feature is intended for use in specialized
   circumstances, such as multi-threaded servers where the same code executes in
   many contexts, and interesting conditions which arise are dependent on this
   context (such as remote client IP address and authenticated user name, in the
   above example). In such circumstances, it is likely that specialized
   :class:`Formatter`\ s would be used with particular :class:`Handler`\ s.

   .. versionadded:: 3.2
      The *stack_info* parameter was added.

.. function:: info(msg, *args, **kwargs)

   Logs a message with level :const:`INFO` on the root logger. The arguments are
   interpreted as for :func:`debug`.


.. function:: warning(msg, *args, **kwargs)

   Logs a message with level :const:`WARNING` on the root logger. The arguments
   are interpreted as for :func:`debug`.

   .. note:: There is an obsolete function ``warn`` which is functionally
      identical to ``warning``. As ``warn`` is deprecated, please do not use
      it - use ``warning`` instead.


.. function:: error(msg, *args, **kwargs)

   Logs a message with level :const:`ERROR` on the root logger. The arguments are
   interpreted as for :func:`debug`.


.. function:: critical(msg, *args, **kwargs)

   Logs a message with level :const:`CRITICAL` on the root logger. The arguments
   are interpreted as for :func:`debug`.


.. function:: exception(msg, *args, **kwargs)

   Logs a message with level :const:`ERROR` on the root logger. The arguments are
   interpreted as for :func:`debug`. Exception info is added to the logging
   message. This function should only be called from an exception handler.

.. function:: log(level, msg, *args, **kwargs)

   Logs a message with level *level* on the root logger. The other arguments are
   interpreted as for :func:`debug`.

   .. note:: The above module-level convenience functions, which delegate to the
      root logger, call :func:`basicConfig` to ensure that at least one handler
      is available. Because of this, they should *not* be used in threads,
      in versions of Python earlier than 2.7.1 and 3.2, unless at least one
      handler has been added to the root logger *before* the threads are
      started. In earlier versions of Python, due to a thread safety shortcoming
      in :func:`basicConfig`, this can (under rare circumstances) lead to
      handlers being added multiple times to the root logger, which can in turn
      lead to multiple messages for the same event.

.. function:: disable(lvl)

   Provides an overriding level *lvl* for all loggers which takes precedence over
   the logger's own level. When the need arises to temporarily throttle logging
   output down across the whole application, this function can be useful. Its
   effect is to disable all logging calls of severity *lvl* and below, so that
   if you call it with a value of INFO, then all INFO and DEBUG events would be
   discarded, whereas those of severity WARNING and above would be processed
   according to the logger's effective level. If
   ``logging.disable(logging.NOTSET)`` is called, it effectively removes this
   overriding level, so that logging output again depends on the effective
   levels of individual loggers.


.. function:: addLevelName(lvl, levelName)

   Associates level *lvl* with text *levelName* in an internal dictionary, which is
   used to map numeric levels to a textual representation, for example when a
   :class:`Formatter` formats a message. This function can also be used to define
   your own levels. The only constraints are that all levels used must be
   registered using this function, levels should be positive integers and they
   should increase in increasing order of severity.

   .. note:: If you are thinking of defining your own levels, please see the
      section on :ref:`custom-levels`.

.. function:: getLevelName(lvl)

   Returns the textual representation of logging level *lvl*. If the level is one
   of the predefined levels :const:`CRITICAL`, :const:`ERROR`, :const:`WARNING`,
   :const:`INFO` or :const:`DEBUG` then you get the corresponding string. If you
   have associated levels with names using :func:`addLevelName` then the name you
   have associated with *lvl* is returned. If a numeric value corresponding to one
   of the defined levels is passed in, the corresponding string representation is
   returned. Otherwise, the string 'Level %s' % lvl is returned.

   .. note:: Levels are internally integers (as they need to be compared in the
      logging logic). This function is used to convert between an integer level
      and the level name displayed in the formatted log output by means of the
      ``%(levelname)s`` format specifier (see :ref:`logrecord-attributes`).

   .. versionchanged:: 3.4
      In Python versions earlier than 3.4, this function could also be passed a
      text level, and would return the corresponding numeric value of the level.
      This undocumented behaviour was considered a mistake, and was removed in
      Python 3.4, but reinstated in 3.4.2 due to retain backward compatibility.

.. function:: makeLogRecord(attrdict)

   Creates and returns a new :class:`LogRecord` instance whose attributes are
   defined by *attrdict*. This function is useful for taking a pickled
   :class:`LogRecord` attribute dictionary, sent over a socket, and reconstituting
   it as a :class:`LogRecord` instance at the receiving end.


.. function:: basicConfig(**kwargs)

   Does basic configuration for the logging system by creating a
   :class:`StreamHandler` with a default :class:`Formatter` and adding it to the
   root logger. The functions :func:`debug`, :func:`info`, :func:`warning`,
   :func:`error` and :func:`critical` will call :func:`basicConfig` automatically
   if no handlers are defined for the root logger.

   This function does nothing if the root logger already has handlers
   configured for it.

   .. note:: This function should be called from the main thread
      before other threads are started. In versions of Python prior to
      2.7.1 and 3.2, if this function is called from multiple threads,
      it is possible (in rare circumstances) that a handler will be added
      to the root logger more than once, leading to unexpected results
      such as messages being duplicated in the log.

   The following keyword arguments are supported.

   .. tabularcolumns:: |l|L|

   +--------------+---------------------------------------------+
   | Format       | Description                                 |
   +==============+=============================================+
   | ``filename`` | Specifies that a FileHandler be created,    |
   |              | using the specified filename, rather than a |
   |              | StreamHandler.                              |
   +--------------+---------------------------------------------+
   | ``filemode`` | Specifies the mode to open the file, if     |
   |              | filename is specified (if filemode is       |
   |              | unspecified, it defaults to 'a').           |
   +--------------+---------------------------------------------+
   | ``format``   | Use the specified format string for the     |
   |              | handler.                                    |
   +--------------+---------------------------------------------+
   | ``datefmt``  | Use the specified date/time format.         |
   +--------------+---------------------------------------------+
   | ``style``    | If ``format`` is specified, use this style  |
   |              | for the format string. One of '%', '{' or   |
   |              | '$' for %-formatting, :meth:`str.format` or |
   |              | :class:`string.Template` respectively, and  |
   |              | defaulting to '%' if not specified.         |
   +--------------+---------------------------------------------+
   | ``level``    | Set the root logger level to the specified  |
   |              | level.                                      |
   +--------------+---------------------------------------------+
   | ``stream``   | Use the specified stream to initialize the  |
   |              | StreamHandler. Note that this argument is   |
   |              | incompatible with 'filename' - if both are  |
   |              | present, a ``ValueError`` is raised.        |
   +--------------+---------------------------------------------+
   | ``handlers`` | If specified, this should be an iterable of |
   |              | already created handlers to add to the root |
   |              | logger. Any handlers which don't already    |
   |              | have a formatter set will be assigned the   |
   |              | default formatter created in this function. |
   |              | Note that this argument is incompatible     |
   |              | with 'filename' or 'stream' - if both are   |
   |              | present, a ``ValueError`` is raised.        |
   +--------------+---------------------------------------------+

   .. versionchanged:: 3.2
      The ``style`` argument was added.

   .. versionchanged:: 3.3
      The ``handlers`` argument was added. Additional checks were added to
      catch situations where incompatible arguments are specified (e.g.
      ``handlers`` together with ``stream`` or ``filename``, or ``stream``
      together with ``filename``).


.. function:: shutdown()

   Informs the logging system to perform an orderly shutdown by flushing and
   closing all handlers. This should be called at application exit and no
   further use of the logging system should be made after this call.


.. function:: setLoggerClass(klass)

   Tells the logging system to use the class *klass* when instantiating a logger.
   The class should define :meth:`__init__` such that only a name argument is
   required, and the :meth:`__init__` should call :meth:`Logger.__init__`. This
   function is typically called before any loggers are instantiated by applications
   which need to use custom logger behavior.


.. function:: setLogRecordFactory(factory)

   Set a callable which is used to create a :class:`LogRecord`.

   :param factory: The factory callable to be used to instantiate a log record.

   .. versionadded:: 3.2
      This function has been provided, along with :func:`getLogRecordFactory`, to
      allow developers more control over how the :class:`LogRecord` representing
      a logging event is constructed.

   The factory has the following signature:

   ``factory(name, level, fn, lno, msg, args, exc_info, func=None, sinfo=None, **kwargs)``

      :name: The logger name.
      :level: The logging level (numeric).
      :fn: The full pathname of the file where the logging call was made.
      :lno: The line number in the file where the logging call was made.
      :msg: The logging message.
      :args: The arguments for the logging message.
      :exc_info: An exception tuple, or None.
      :func: The name of the function or method which invoked the logging
             call.
      :sinfo: A stack traceback such as is provided by
              :func:`traceback.print_stack`, showing the call hierarchy.
      :kwargs: Additional keyword arguments.


模块级别的属性
-----------------------

.. attribute:: lastResort

   A "handler of last resort" is available through this attribute. This
   is a :class:`StreamHandler` writing to ``sys.stderr`` with a level of
   ``WARNING``, and is used to handle logging events in the absence of any
   logging configuration. The end result is to just print the message to
   ``sys.stderr``. This replaces the earlier error message saying that
   "no handlers could be found for logger XYZ". If you need the earlier
   behaviour for some reason, ``lastResort`` can be set to ``None``.

   .. versionadded:: 3.2

集合警告模块
------------------------------------

The :func:`captureWarnings` function can be used to integrate :mod:`logging`
with the :mod:`warnings` module.

.. function:: captureWarnings(capture)

   This function is used to turn the capture of warnings by logging on and
   off.

   If *capture* is ``True``, warnings issued by the :mod:`warnings` module will
   be redirected to the logging system. Specifically, a warning will be
   formatted using :func:`warnings.formatwarning` and the resulting string
   logged to a logger named ``'py.warnings'`` with a severity of :const:`WARNING`.

   If *capture* is ``False``, the redirection of warnings to the logging system
   will stop, and warnings will be redirected to their original destinations
   (i.e. those in effect before ``captureWarnings(True)`` was called).


.. seealso::

   Module :mod:`logging.config`
      Configuration API for the logging module.

   Module :mod:`logging.handlers`
      Useful handlers included with the logging module.

   :pep:`282` - A Logging System
      The proposal which described this feature for inclusion in the Python standard
      library.

   `Original Python logging package <http://www.red-dove.com/python_logging.html>`_
      This is the original source for the :mod:`logging` package.  The version of the
      package available from this site is suitable for use with Python 1.5.2, 2.1.x
      and 2.2.x, which do not include the :mod:`logging` package in the standard
      library.

