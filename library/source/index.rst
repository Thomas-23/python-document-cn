.. _library-index:

###############################
  Python 标准库
###############################

在 :ref:`reference-index` 中已经准确的描述了Python语言的语法
而这本标准库参考手册则主要介绍Python发行版中的标准库，以及通常随着发行版发布的可选组件。

正如下面的目录中所表明的那样，Python标准库涉及的内容非常广泛，它提供了大量的工具。
标准库中包括一些用C编写的内置模块，提供一些必要的系统功能，
还包括一些用Python编写的模块，提供针对日常编程问题的一些标准化解决方案；
也包括一些针对Python程序可移植性设计的模块，它们将特定系统平台抽象为与平台无关的API接口。

windows平台下的Python安装程序一般包含完整的标准库以及一些额外组件。
而类Unix操作系统则一般将Python捆绑在包中，所以如有需要，使用系统的包管理工具获取一些或所有的可选组件。

当然除了标准库以外，你还可以在  `Python Package Index <https://pypi.python.org/pypi>`_ 中还可以获取成千上万的组件，
包括从单独的程序或模块到完整的应用开发框架。


.. toctree::
   :maxdepth: 2
   :numbered:

   intro.rst
   functions.rst
   constants.rst
   stdtypes.rst
   exceptions.rst

   text.rst
   binary.rst
   datatypes.rst
   numeric.rst
   functional.rst
   filesys.rst
   persistence.rst
   archiving.rst
   fileformats.rst
   crypto.rst
   allos.rst
   concurrency.rst
   ipc.rst
   netdata.rst
   markup.rst
   internet.rst
   mm.rst
   i18n.rst
   frameworks.rst
   tk.rst
   development.rst
   debug.rst
   distribution.rst
   python.rst
   custominterp.rst
   modules.rst
   language.rst
   misc.rst
   windows.rst
   unix.rst
   superseded.rst
   undoc.rst
