:mod:`pydoc` --- 文档生成和在线帮助系统
===============================================================

.. module:: pydoc
   :synopsis: 文档生成和在线帮助系统.
.. moduleauthor:: Ka-Ping Yee <ping@lfw.org>
.. sectionauthor:: Ka-Ping Yee <ping@lfw.org>


.. index::
   single: documentation; generation
   single: documentation; online
   single: help; online

**Source code:** :source:`Lib/pydoc.py`

--------------

:mod:`pydoc` 从 Python 模块中自动生成文档。这个文档可以以文本页的形式显示在控制台，提供给浏览器，或者
保存为 HTML 文件。

对于模块，类，函数和方法，显示行的文档是由对象的 docstring （如 :attr:`__doc__` 属性）衍化而来，
并且递归的完成它包含文档的成员。 如果没有 docstring, :mod:`pydoc` 会尝试从注释行块中获取描述，
只涉及以上源文件中定义的类， 函数， 方法，以及模块的最顶部（查看 :func:`inspect.getcomments` ). 

内置方法 :func:`help` 调用交互解释器中在线帮助系统, 它使用 :mod:`pydoc` 在控制台中生成文本文档. 
同样的文本文档可以通过在交互器之外的操作系统的命令提示符中运行 :program:`pydoc` 脚本查看。
例如， 运行::

   pydoc sys

在 shell 提示中将会显示 :mod:`sys` 模块的文档，显示格式类似于 Unix 的 :program:`man` 命令
生成的帮助页面。 :program:`pydoc` 命令的参数可以是它们的名字: 函数，模块，包,或者点式的类引用
,方法， 或者模块内的函数，包里面的模块。
如果 :program:`pydoc` 参数，看起来像一个路径(即它包含操作系统路径分隔符，例如 Unix的斜线),并且
指向一个已经存在Python 源代码文件， 那么将会生成这个源文件的文档。

.. note::

   为了找到对象和它们的文档， :mod:`pydoc` 导入模块来进行文档化。 因此任何在模块级别的代码将会在
   这种场合下执行。当一个文件以脚本的形式被调用而不是导入时，
   使用 ``if __name__ == '__main__':`` 防护那些仅执行的代码。

当打印输出到控制台时， :program:`pydoc` 会试着页面化输出结果，以便更容易阅读。
如果设置了 :envvar:`PAGER` 环境变量， :program:`pydoc` 的页面化程序将会使用这个值。 

在参数之前定义一个 ``-w`` 标识， 将会以 HTML 文档的形式写入当前目录下的一个文件中
而不是在控制台中显示文本。

在参数之前定义一个 ``-k`` 标识， 将会搜索所有使用模块关于 synopsis 的行， 来作为参数的关键字，它
类似于 Unix :program:`man` 命令。模块 synopsis 行是文档字符串的第一行。 

你也可以使用 :program:`pydoc` 在本机上启动一个 HTTP 服务， 然后通过 Web 浏览器来访问文档。
:program:`pydoc -p 1234` 将会在端口 1234 上启动一个 HTTP 服务， 在浏览器中以
 ``http://localhost:1234/`` 访问文档。如果以 ``0`` 作为端口后，那么将会选择任意一个没有使用的端口。

:program:`pydoc -b` 将会启动一个服务并且打开 web 浏览器到你模块索引页面，每个
浏览页在顶部都有一个导航条，它可以帮助你跳转到特殊的选项上，以关键字搜索所有模块 synopsis 行
以及跳转到模块索引，主题和关键字页面。

当 :program:`pydoc` 生成文档时， 它使用当前环境和路径来定位模块。 这样调用
:program:`pydoc spam` ， 文档会精确到模块的版本，如果你在解释器中，你可以输入
``import spam`` 来获得。

模块文件的核心模块假定为保存在
``https://docs.python.org/X.Y/library/`` ， 其中的 ``X`` 和 ``Y`` 
表示主要和次要 Python 解释器 版本号。 这可以在环境变量中改变 :envvar:`PYTHONDOCS` 
为一个不同的 URL 或者一个包含指南手册的本地目录来覆盖掉.

.. versionchanged:: 3.2
   Added the ``-b`` option.

.. versionchanged:: 3.3
   The ``-g`` command line option was removed.

.. versionchanged:: 3.4
   :mod:`pydoc` now uses :func:`inspect.signature` rather than
   :func:`inspect.getfullargspec` to extract signature information from
   callables.
