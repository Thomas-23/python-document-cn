:mod:`__future__` --- 未来语句定义
==================================================

.. module:: __future__
   :synopsis: 未来语句定义

**Source code:** :source:`Lib/__future__.py`

--------------

:mod:`__future__` 是一个真实的模块, 它服务于下面三个目的:

* 为了避免与已经存在的工具冲突，它会分析导入的语句然后找到所要导入的模块。

* :ref:`future statements <future>` 确保在运行 2.1 稳定版本之前的版本时，会产生运行时异常 (
  在导入 :mod:`__future__` 时会失败， 因为在 2.1 之前的版本是没有这个模块的 )。

* 当有不兼容的变化被引入时，并且这些不兼容什么时候会做强制性要求时，会进行文档化。这是一种可执行的文档格式。
  通过导入 :mod:`__future__` 能够进行程序检查，并且测试导入的内容。

:file:`__future__.py` 中每条语句的形式如下::

   FeatureName = _Feature(OptionalRelease, MandatoryRelease,
                          CompilerFlag)


这里, 正常情况下, *OptionalRelease* 比 *MandatoryRelease* 小, 并且他俩都是5个元素的元组格式，像
下面的 :data:`sys.version_info`::

   (PY_MAJOR_VERSION, # the 2 in 2.1.0a3; an int
    PY_MINOR_VERSION, # the 1; an int
    PY_MICRO_VERSION, # the 0; an int
    PY_RELEASE_LEVEL, # "alpha", "beta", "candidate" or "final"; string
    PY_RELEASE_SERIAL # the 3; an int
   )

*OptionalRelease* 记录了第一次存在此功能的稳定版本。

这种情况下 *MandatoryRelease* 还不会出现,
*MandatoryRelease* 预示着这个功能将在某个版本中成为语言的一部分。

另外 *MandatoryRelease* 记录了这个功能是在什么时候成为这个语言的一部分的 ; 在某个版本或者之后的版本，模块无需再考虑使用未来语句
来使用这个功能，但你也可以继续那样导入。

*MandatoryRelease* 可以是 ``None``, 这表示一个计划好的功能被撤掉了

类 :class:`_Feature` 的实例有两个对应的方法，
:meth:`getOptionalRelease` 和 :meth:`getMandatoryRelease`.

*CompilerFlag* 是一个(bitfield)标志，为了能够使用动态编译代码功能，它作为第四个参数传递给内建方法
:func:`compile` 。这个标记存储在类 :class:`_Feature` 实例的 :attr:`compiler_flag` 属性中。

没有功能描述的将永远从 :mod:`__future__` 中删除。 由于是在 Python 2.1 中才引入的， 下面是语言中使用到的机制，
和与之对应的功能:

+------------------+-------------+--------------+---------------------------------------------+
| 功能             | 选择性存在  | 强制性存在   | 影响                                        |
+==================+=============+==============+=============================================+
| nested_scopes    | 2.1.0b1     | 2.2          | :pep:`227`:                                 |
|                  |             |              | *静态嵌套作用域*                            |
+------------------+-------------+--------------+---------------------------------------------+
| generators       | 2.2.0a1     | 2.3          | :pep:`255`:                                 |
|                  |             |              | *简单的生成器*                              |
+------------------+-------------+--------------+---------------------------------------------+
| division         | 2.2.0a2     | 3.0          | :pep:`238`:                                 |
|                  |             |              | *改变除法运算符*                            |
+------------------+-------------+--------------+---------------------------------------------+
| absolute_import  | 2.5.0a1     | 3.0          | :pep:`328`:                                 |
|                  |             |              | *导入相关: 多行和绝对/相对*                 |
+------------------+-------------+--------------+---------------------------------------------+
| with_statement   | 2.5.0a1     | 2.6          | :pep:`343`:                                 |
|                  |             |              | *"with" 语句*                               |
+------------------+-------------+--------------+---------------------------------------------+
| print_function   | 2.6.0a2     | 3.0          | :pep:`3105`:                                |
|                  |             |              | *将print作为一个函数*                       |
+------------------+-------------+--------------+---------------------------------------------+
| unicode_literals | 2.6.0a2     | 3.0          | :pep:`3112`:                                |
|                  |             |              | *Python 3的字节文字*                        |
+------------------+-------------+--------------+---------------------------------------------+
| generator_stop   | 3.5.0b1     | 3.7          | :pep:`479`:                                 |
|                  |             |              | *StopIteration 内部生成器处理*              |
+------------------+-------------+--------------+---------------------------------------------+


.. seealso::

   :ref:`future`
      编译器是如何处理未来导入的.
