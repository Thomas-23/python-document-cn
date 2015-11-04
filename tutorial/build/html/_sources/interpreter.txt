.. _tut-using:

****************************
使用 Python 解释器
****************************


.. _tut-invoking:

调用解释器
========================

在机器允许访问的情况下，Python 解释器通常被安装在 :file:`/usr/local/bin/python3.6` 目录下。
为了在shell中能够通过下面的命令启动解释器， 需要将 :file:`/usr/local/bin` 目录包含进 Unix shell 的
搜索路径里:

.. code-block:: text

   python3.6

[#]_ 由于 Python 解释器的安装路径是可选的，这也可能是其它路径，你可以联系安装 Python 的用户或
系统管理员确认（例如，:file:`/usr/local/python` 就是一个常见的选择）。

在 Windows 机器上，Python 通常安装在 :file:`C:\\Python36` 位置，当然你可以在运行安装向导时修改
此值。要想把此目录添加到你的 PATH 环境变量中，你可以在 DOS 窗口中输入以下命令::

   set path=%path%;C:\python36

通常你可以在主窗口输入一个文件结束符（Unix 系统是 :kbd:`Control-D`，Windows 系统是 :kbd:`Control-Z`）
让解释器以 0 状态码退出。如果那没有作用，你可以通过输入 ``quit()`` 命令退出解释器。

在支持读行的机器上， Python 解释器支持交互编辑， 历史记录和代码自动完成功能。通常在出现python窗口
的地方，输入 :kbd:`Control-P` 来快速检查是否支持命令行编辑。如果发出嘟嘟声，则说明你可以使用命令行编辑功能；
更多快捷键的介绍请参考 :ref:`tut-interacting`。如果没有任何声音，或者显示 ``^P`` 字符，则说明命令行
编辑功能不可用；你只能通过退格键从当前行删除已键入的字符并重新输入。

Python 解释器有些操作类似 Unix shell：当使用终端设备（tty）作为标准输入调用时，它交互的解释并执行命令；
当使用文件名参数或以文件作为标准输入调用时，它读取文件并将文件作为 *脚本* 执行。

第二种启动 Python 解释器的方法是 ``python -c command [arg] ...``，这种方法可以在 *命令行* 执行 Python 语句，
类似于 shell 中的 `-c`_ 选项。由于 Python 语句通常会包含空格或其他特殊 shell 字符，一般建议将 *命令* 用单引号包裹起来。

有一些 Python 模块也可以当作脚本使用。你可以使用 ``python -m module [arg] ...`` 命令调用它们，这类似在命令行
中键入完整的路径名执行 *模块* 源文件一样。

使用脚本文件时，经常会运行脚本然后进入交互模式。这也可以通过在脚本之前加上 `-i`_ 参数来实现。


.. _tut-argpassing:

参数传递
----------------

调用解释器时，脚本的名字和附加其后的参数转换为一个字符串列表，然后赋值给 ``sys`` 模块中的 ``argv`` 变量，
你可以通过执行 ``import sys`` 获得这个列表。列表的最少长度为1；当没有提供脚本和参数的时候， ``sys.argv[0]`` 是
一个空字符串。当给定了脚本的名字如 ``'-'`` （表示标准输入）时， ``sys.argv[0]`` 被设定为 ``'-'`` 。
使用 `-c`_ *指令* 时， ``sys.argv[0]`` 被设定为 ``'-c'`` 。使用 `-m`_ *模块* 参数时，``sys.argv[0]`` 被设定
为指定模块的全名。 `-c`_ *指令* 或者 `-m`_ *模块* 之后的参数不会被 Python 解释器的选项处理机制所截获，
而是留在 ``sys.argv`` 中，供脚本命令操作。


.. _tut-interactive:

交互模式
----------------

从 tty 读取命令时，我们称解释器工作于 *交互模式*。这种模式下它根据主提示符来执行，主提示符通常标识为三个大于号 (``>>>``) ；
继续的部分被称为 *从属提示符*，默认为三个点标识 (``...``) 。在第一行之前，解释器打印欢迎信息、版本号和授权提示::

   $ python3.6
   Python 3.6 (default, Sep 16 2015, 09:25:04)
   [GCC 4.8.2] on linux
   Type "help", "copyright", "credits" or "license" for more information.
   >>>

.. XXX update for new releases

输入多行结构时需要从属提示符，例如，下面这个 `if`_ 语句::

   >>> the_world_is_flat = 1
   >>> if the_world_is_flat:
   ...     print("Be careful not to fall off!")
   ...
   Be careful not to fall off!


关于交互模式更多的内容，请参见 :ref:`tut-interac`。


.. _tut-interp:

解释器及其环境
===================================

.. _tut-source-encoding:

源程序编码
--------------------

默认情况下，Python 源文件是 UTF-8 编码。在此编码下，全世界大多数语言的字符可以同时用在字符串、标识符
和注释中 --- 尽管 Python 标准库仅使用 ASCII 字符做为标识符，这只是任何可移植代码应该遵守的约定。
如果要正确的显示所有的字符，你的编辑器必须能识别出文件是 UTF-8 编码，并且它使用的字体能支持文件中所有的字符。

你也可以为源文件指定不同的字符编码。为此，在 ``#!`` 行后插入至少一行特殊的注释行来定义源文件的编码::

   # -*- coding: encoding -*-

通过此声明，源文件中所有的字符都会被当做用 *encoding* 指代的 UTF-8 编码对待。在 Python 库参考手册 `codecs`_ 一
节中你可以找到一张可用的编码列表。

例如，如果你的编辑器不支持 UTF-8 编码的文件，但支持像 Windows-1252 的其他一些编码，你可以定义::

   # -*- coding: cp-1252 -*-

这样就可以在源文件中使用 Windows-1252 字符集中的所有字符了。这个特殊的编码注释必须在文件中的 *第一或第二* 行定义。



.. rubric:: 脚注

.. [#] 在 Unix 系统上，Python 3.X 解释器默认未被安装成名为 ``python`` 的命令，所以它不会与
   同时安装在系统中的 Python 2.x 命令冲突。



.. _-c: https://docs.python.org/3/using/cmdline.html#cmdoption-c
.. _-i: https://docs.python.org/3/using/cmdline.html#cmdoption-i
.. _-m: https://docs.python.org/3/using/cmdline.html#cmdoption-m
.. _if: https://docs.python.org/3/reference/compound_stmts.html#if
.. _codecs: https://docs.python.org/3/library/codecs.html#module-codecs
