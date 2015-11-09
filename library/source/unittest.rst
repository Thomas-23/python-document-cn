:mod:`unittest` --- Unit testing framework
==========================================

.. module:: unittest
   :synopsis: Unit testing framework for Python.
.. moduleauthor:: Steve Purcell <stephen_purcell@yahoo.com>
.. sectionauthor:: Steve Purcell <stephen_purcell@yahoo.com>
.. sectionauthor:: Fred L. Drake, Jr. <fdrake@acm.org>
.. sectionauthor:: Raymond Hettinger <python@rcn.com>

(如果你已经对测试相关的基础概念非常熟悉了，那么你可以跳过 :ref:`the list of assert methods <assert-methods>`.)

单元测试框架 :mod:`unittest` 最初的灵感来自于 JUnit ， Junit 是一个在其他语言中，很受欢迎的单元测试框架。
它支持包括，测试自动化，所有的测试可以共享setup 和 shutdown 中的代码，可以按组合聚集测试，以及独立于测试报告框架。

为了实现这些功能, :mod:`unittest` 以面向对象的方式，提供了一些重要的概念:

test fixture
   一个 :dfn:`test fixture` 描述了执行一个或多个测试所要做的准备工作，以及与之相关的清理工作。
   这包括，例如创建一个零时文件或者代理数据库，目录或者启动一个服务进程。

test case
   一个 :dfn:`test case` 是一个独立的单元测试.  通过一系列的输入来检查一个指定的返回.
   :mod:`unittest` 提供了一个基类, :class:`TestCase` , 可以用它来创建新的测试用例。

test suite
   一个 :dfn:`test suite` 是测试用例，测试集或者两者都有的测试集合。它用来聚集测试一块执行。

test runner
   一个 :dfn:`test runner` 是一个用来安排执行测试的组件并且为用户提供运行结果。运行者可以使用图形接口。


.. seealso::

   Module :mod:`doctest`
      另一个挺受欢迎的支持测试的模块。

   `使用模式进行简单的 Smalltalk 测试 <https://web.archive.org/web/20150315073817/http://www.xprogramming.com/testfram.htm>`_
      Kent Beck 共享的关于 :mod:`unittest` 测试框架使用模式的原稿.

   `Nose <https://nose.readthedocs.org/en/latest/>`_ and `py.test <http://pytest.org>`_
      第三方的单元测试框架，它使用了轻量级的语法来编写测试。
      例如, ``assert func(10) == 42``.

   `Python 测试工具分类 <https://wiki.python.org/moin/PythonTestingToolsTaxonomy>`_
      关于 Python 的扩展的测试工具列表， 包括功能测试框架和仿对象库。

   `Python 测试邮件列表 <http://lists.idyll.org/listinfo/testing-in-python>`_
      一个讨论 Python 测试和测试工具的有趣的组织。

   在 Python 发布的源代码中有一个用来发现和执行测试 GUI 脚本 :file:`Tools/unittestgui/unittestgui.py`
   它的目地是使单元测试新手更容易上手。 对于生产环境，推荐使用持续集成系统来驱动，例如
   `Buildbot <http://buildbot.net/>`_, `Jenkins <http://jenkins-ci.org/>`_
   或者  `Hudson <http://hudson-ci.org/>`_.


.. _unittest-minimal-example:

基础的例子
-------------

:mod:`unittest` 模块提供丰富的工具集来组织和运行测试。这一章节将演示一小部分工具，但足以满足大多数用户的需要了。

这里是一个测试三个字符串方法的简短脚本::

  import unittest

  class TestStringMethods(unittest.TestCase):

    def test_upper(self):
        self.assertEqual('foo'.upper(), 'FOO')

    def test_isupper(self):
        self.assertTrue('FOO'.isupper())
        self.assertFalse('Foo'.isupper())

    def test_split(self):
        s = 'hello world'
        self.assertEqual(s.split(), ['hello', 'world'])
        # check that s.split fails when the separator is not a string
        with self.assertRaises(TypeError):
            s.split(2)

  if __name__ == '__main__':
      unittest.main()


测试用例是通过 :class:`unittest.TestCase` 的子类来创建的.  这三个独立的测试都定义成方法，并且他们名字都是以
``test`` 开头。  这种命名约定告诉测试执行者哪些方法是表示要测试的。

这里测试的关键在于调用 :meth:`~TestCase.assertEqual` 来检查期望的结果；
:meth:`~TestCase.assertTrue` 或者 :meth:`~TestCase.assertFalse`
来验证条件; 调用 :meth:`~TestCase.assertRaises` 来验证一个指定的异常是否会被抛出。
使用这些方法来代替 :keyword:`assert` 语句可以使得测试运行组件积累测试结果然后生成报告。

方法 :meth:`~TestCase.setUp` 和 :meth:`~TestCase.tearDown` 允许你定义一些指令，
这些指令在每个测试开始前和运行结束时都会被执行。关于这些方法， :ref:`organizing-tests` 章节包含了更详细的信息。

最后一块显示了一个运行测试的简单方法。 :func:`unittest.main` 给测试脚本提供了一个命令行接口。
当从命令行运行时， 脚本会生成类似如下的结果::

   ...
   ----------------------------------------------------------------------
   Ran 3 tests in 0.000s

   OK

如果给你的脚步提供一个 ``-v`` 的选项， 它会指示 :func:`unittest.main` 开启更多更高级别的结果信息。
并生成如下结果::

   test_isupper (__main__.TestStringMethods) ... ok
   test_split (__main__.TestStringMethods) ... ok
   test_upper (__main__.TestStringMethods) ... ok

   ----------------------------------------------------------------------
   Ran 3 tests in 0.001s

   OK

在上面这些例子中，使用的是 :mod:`unittest` 最常用的功能，它们足以满足我们日常生活中测试的需要。
接下来的文档中将追本溯源的来解释所有的功能。


.. _unittest-command-line-interface:

命令行接口
----------------------

单元测试模块能够在模块，类，甚至独立的方法中使用命令行来运行测试::

   python -m unittest test_module1 test_module2
   python -m unittest test_module.TestClass
   python -m unittest test_module.TestClass.test_method

你可以提供一个以任意模块名字，和完全符合要求的类或者方法的名字组成的列表。

测试模块也可以指定文件路径::

   python -m unittest tests/test_something.py

这允许你完全使用一个 shell 文件名字来指定测试模块。这个指定的文件必需依然是可以导入的模块。
文件路径将通过移除后缀名 '.py' ，以及将路径分隔符转换为 '. '的方式，将其变为一个模块名字。
如果你想在不到导入模块的情况下执行一个测试文件，那么你应该使用直接执行文件的方式。

如果你想在运行测试的时候，获取更多的信息 (更多高级别的信息)，使用 -v 参数。::

   python -m unittest -v test_module

如果执行时没有跟参数，那么从 :ref:`unittest-test-discovery` 开始::

   python -m unittest

如果想要列出所有的命令行选项::

   python -m unittest -h

.. versionchanged:: 3.2
   在早起的版本中，它只能运行单个的测试方法，不能是模块或者类。


命令行参数选项
~~~~~~~~~~~~~~~~~~~~

:program:`unittest` 支持下面这些命令行参数:

.. program:: unittest

.. cmdoption:: -b, --buffer

   在运行测试的时候，标准的输出和错误流会加入到缓冲区。当传过来的测试消失后输出结果。
   在测试失败或者有错误时，它会加入到失败消息中，输出正常的结果。

.. cmdoption:: -c, --catch

   :kbd:`Control-C` 会在测试运行中，等待当前的测试结束，然后报告到目前为止所有的测试结果。第二个 :kbd:`Control-C`
   会抛出正常情况的 :exc:`KeyboardInterrupt` 异常.

   可以查看 `Signal Handling`_ 函数， 这个函数提供了上面的功能.

.. cmdoption:: -f, --failfast

   在遇到错误或者失败时，停止运行测试。

.. cmdoption:: --locals

   在追朔信息中显示本地变量。

.. versionadded:: 3.2
   加入 ``-b``, ``-c`` 和 ``-f`` 命令行选项。

.. versionadded:: 3.5
   加入 ``--locals`` 命令行选项

在运行一个项目或者子项目所有测试时，命令行可以用来发现测试。


.. _unittest-test-discovery:

发现测试
--------------

.. versionadded:: 3.2

Unittest 支持发现简单的测试。 为了能够支持发现测试，所有的测试文件必需是能够从项目的最上层目录导入的
:ref:`modules <tut-modules>` 或者 :ref:`packages <tut-packages>` 。
（也就是说他们的名字必需是合法的 :ref:`identifiers <identifiers>` ）

发现测试是在 :meth:`TestLoader.discover` 中实现的， 但是它同样能够在命令行中使用。
基本的命令行使用如下::

   cd project_directory
   python -m unittest discover

.. note::

   作为一个快捷方式, ``python -m unittest`` 和 ``python -m unittest discover`` 是等价的.
   如果你想给测试发现模块传递参数， ``discover`` 的子命令必需显示的给出。

``discover`` 的子命令有下面一些选项:

.. program:: unittest discover

.. cmdoption:: -v, --verbose

   更详细的输出

.. cmdoption:: -s, --start-directory directory

   从哪个目录开始查找 (默认为 ``.`` )

.. cmdoption:: -p, --pattern pattern

   使用模式来匹配文件 (默认为 ``test*.py`` )

.. cmdoption:: -t, --top-level-directory directory

   项目的最上层目录 (默认为的开始目录)

:option:`-s`, :option:`-p`, and :option:`-t` 选项可以按照这种顺序选择性的提供。
下面这两个命令行是相等的::

   python -m unittest discover -s project_directory -p "*_test.py"
   python -m unittest discover project_directory "*_test.py"

类似于路径你也可以以一个包的名字为起始目录，例如 ``myproject.subpackage.test`` . 你提供的包
的名字将会被导入进来，并且它所在系统的目录位置会作为起始目录。

.. caution::

    测试查找模块通过导入来加载测试的。一旦从你指定的起始目录找到了所有的测试文件，它会把这些文件路径转换为包名导入进来。
    例如 :file:`foo/bar/baz.py` 文件将会以 ``foo.bar.baz`` 导入进来。

    如果你有一个全局的安装包，并尝试在另一个不同的备份包中来查找测试，会出现一个错误地方导入的问题 (import *could* ) .
    如果出现这种问题，测试查找模块会警告你并且退出。

    但如果你是以包名而不是目录路径的方式提供起始目录的话，那么查找模块会假设导入的位置即是想要的其实位置，
    因此你不会收到像上面那样的警告。

通过  `load_tests protocol`_ ，测试模块和包能够定制测试加载和查找。

.. versionchanged:: 3.4
   测试查找支持 :term:`namespace packages <namespace package>`.


.. _organizing-tests:

组织测试代码
--------------------

单元测试中基本的构建代码块是 :dfn:`test cases` --- 一个必需设置和检查正确与否的单一场景。在 :mod:`unittest`
中，测试用例由 :class:`unittest.TestCase` 实例所描述。如果想创建自己的测试用例，你必需编写 :class:`TestCase`
的子类或者使用 :class:`FunctionTestCase` 。

:class:`TestCase` 实例中的测试代码必需完整自包含的， 这样的话它们可以独立或者跟其他测试用例以任意组合的数量来运行。

简单的 :class:`TestCase` 子类只简单的实现了一个测试方法(如以 ``test`` 开头的方法) 来执行指定的测试代码::

   import unittest

   class DefaultWidgetSizeTestCase(unittest.TestCase):
       def test_default_widget_size(self):
           widget = Widget('The widget')
           self.assertEqual(widget.size(), (50, 50))

注意为了测试一些东西，我们使用了 :class:`TestCase` 基类提供的 :meth:`assert\*` 方法。
如果一个测试失败了，将抛出一个异常，然后 :mod:`unittest` 会标记这个测试为 :dfn:`failure` 。
其他任何的异常都被视为 :dfn:`errors` 。

可以同时有很多个测试，并且它们的 set-up 可以重复做一些事情。幸运的是，我们可以实现一个叫做 :meth:`~TestCase.setUp` 方法，
然后提取 set-up 的代码放入其中，我们的测试框架在每一个测试运行时，会自动调用这个方法::

   import unittest

   class SimpleWidgetTestCase(unittest.TestCase):
       def setUp(self):
           self.widget = Widget('The widget')

       def test_default_widget_size(self):
           self.assertEqual(self.widget.size(), (50,50),
                            'incorrect default size')

       def test_widget_resize(self):
           self.widget.resize(100,150)
           self.assertEqual(self.widget.size(), (100,150),
                            'wrong size after resize')

.. note::
   测试的执行顺序由测试方法的名字的排序来决定，关于排序与内置的字符串排序方法有关。

如果在测试运行过程中， :meth:`~TestCase.setUp` 方法抛出了一个异常。那么框架会认为测试遇到了错误，测试方法将不会被执行。

同样的, 我们可以在测试方法运行结束后，提供一个 :meth:`~TestCase.tearDown` 方法用于扫尾的工作::

   import unittest

   class SimpleWidgetTestCase(unittest.TestCase):
       def setUp(self):
           self.widget = Widget('The widget')

       def tearDown(self):
           self.widget.dispose()

只要 :meth:`~TestCase.setUp` 运行成功，无论测试方法是否成功或者失败， :meth:`~TestCase.tearDown` 方法都会被执行。

像这样的测试代码工作环境我们叫他 :dfn:`fixture`.

根据测试的功能，测试用例实例被组合在一起。 :mod:`unittest` 为这种情况提供了一种机制: :dfn:`test suite`,
它由 :mod:`unittest` 的 :class:`TestSuite` 类表示。大多数情况下，调用 :func:`unittest.main` 方法
会做正确的事情，并且为你收集所有模块的测试，然后执行这些测试。

有时, 您可能想定制测试集的构建。
你可以这样做::

   def suite():
       suite = unittest.TestSuite()
       suite.addTest(WidgetTestCase('test_default_size'))
       suite.addTest(WidgetTestCase('test_resize'))
       return suite

你可以放置这些测试用例和测试集的定义到同一个模块，就像他们测试的代码那样(例如 :file:`widget.py`),
但是将这些测试代码放置在不同的模块有很多优势，例如 :file:`test_widget.py`:

* 这个测试模块可以在命令行中独立的运行。

* 测试代码可以很容易的从发布代码中区分开来。

* 对于改变测试代码来满足代码要求缺乏诱惑力，这缺乏一个好的理由。

* 测试代码并不会像它们要测的代码那样频繁的更新。

* 测试代码可以很容易被重构。

* 在测试 C 编写的模块时，无论如何必需在一个单独的模块中， 为什么我们不保持一致了?

* 如果测试策略发生了改变， 我们不需要去改变源代码。


.. _legacy-unit-tests:

重用老的测试代码
----------------------

一些用户会发现它们有一些已存在的测试代码运行在 :mod:`unittest` 上， 但是这些老测试方法并没有全部转换为 :class:`TestCase` 的子类

基于这个原因, :mod:`unittest` 提供了一个 :class:`FunctionTestCase` 类.
它是 :class:`TestCase` 的子类，用来包裹已经存在的测试方法。
它同样可以提供 Set-up 和 tear-down 方法.

给出下面的测试方法::

   def testSomething():
       something = makeSomething()
       assert something.name is not None
       # ...

它可以创建一个像下面那样的等价的测试用例实例， 并提供可选的
set-up 和 tear-down 方法::

   testcase = unittest.FunctionTestCase(testSomething,
                                        setUp=makeSomethingDB,
                                        tearDown=deleteSomethingDB)

.. note::

   尽管 :class:`FunctionTestCase` 能够基于 :mod:`unittest` 基础系统上快速转换已经存在的测试方法。但这种做法并不推荐
   花一些时间创建一个合适的 :class:`TestCase` 子类， 将会使得未来重构测试非常的轻松。

有些情况下，存在的测试可能是为 :mod:`doctest` 模块编写的, 因此 :mod:`doctest` 提供了一个 :class:`DocTestSuite`类， 它能够
从已经存在的，基于 :mod:`doctest` 基础上的测试，自动创建一个 :class:`unittest.TestSuite` 实例。


.. _unittest-skipping:

跳过测试和期望失败
------------------------------------

.. versionadded:: 3.1

Unittest 支持跳过单独的测试方法和整个测试类。 另外它还支持标记一个测试为 "期望失败"， 这种测试即使失败了，但是在 :class:`TestResult`
看来它不是失败的。

跳过测试与使用 :func:`skip` :term:`decorator` 或者与其加了条件的变体有关。

基础的跳过像下面这样::

   class MyTestCase(unittest.TestCase):

       @unittest.skip("demonstrating skipping")
       def test_nothing(self):
           self.fail("shouldn't happen")

       @unittest.skipIf(mylib.__version__ < (1, 3),
                        "not supported in this library version")
       def test_format(self):
           # Tests that work for only a certain version of the library.
           pass

       @unittest.skipUnless(sys.platform.startswith("win"), "requires Windows")
       def test_windows_support(self):
           # windows specific testing code
           pass

在使用详细输出模式时，下面是上面例子的运行结果::

   test_format (__main__.MyTestCase) ... skipped 'not supported in this library version'
   test_nothing (__main__.MyTestCase) ... skipped 'demonstrating skipping'
   test_windows_support (__main__.MyTestCase) ... skipped 'requires Windows'

   ----------------------------------------------------------------------
   Ran 3 tests in 0.005s

   OK (skipped=3)

类也能像方法那样被跳过::

   @unittest.skip("showing class skipping")
   class MySkippedTestCase(unittest.TestCase):
       def test_not_run(self):
           pass

:meth:`TestCase.setUp` 也同样能够跳过测试. 当有资源无法使用时，使用了set up，这种方法很有用。

期望失败使用 :func:`expectedFailure` 修饰器. ::

   class ExpectedFailureTestCase(unittest.TestCase):
       @unittest.expectedFailure
       def test_fail(self):
           self.assertEqual(1, 0, "broken")

很容易包裹一个你自己的跳过修饰器，当你想跳过某个测试的时候，你可以通过在测试上调用 :func:`skip` 来创建一个修饰器。
除非传递的对象中包含一个特定属性的测试， 否则下面这个修饰器会跳过测试::

   def skipUnlessHasattr(obj, attr):
       if hasattr(obj, attr):
           return lambda func: func
       return unittest.skip("{!r} doesn't have {!r}".format(obj, attr))

下面的修饰器实现了跳过测试和期望失败:

.. decorator:: skip(reason)

   无条件的跳过修饰的测试。 参数 *reason* 应该用来描述这个测试被跳过的原因。

.. decorator:: skipIf(condition, reason)

   如果条件为真的话，跳过修饰的测试。

.. decorator:: skipUnless(condition, reason)

   除非 *condition* 为真，否则跳过修饰的测试。

.. decorator:: expectedFailure

   标记这个测试的期望结果是失败的。如果在运行中测试失败，这个测试将不会以失败对待。

.. exception:: SkipTest(reason)

   在跳过测试的时候，这个异常会被抛出。

   通常你可以使用 :meth:`TestCase.skipTest` 或者其中一个跳过修饰器来代替直接抛出这个异常。

跳过的测试不会执行 :meth:`~TestCase.setUp` 或 :meth:`~TestCase.tearDown` 。
跳过的类不会执行 :meth:`~TestCase.setUpClass` 或 :meth:`~TestCase.tearDownClass` 。
跳过的模块不会执行 :func:`setUpModule` 或 :func:`tearDownModule` 。


.. _subtests:

使用 subtests 区分测试迭代次数
---------------------------------------------

.. versionadded:: 3.4

当你的很多测试，只有一些细微的区别又或是实例化一些参数的时候， unittest 允许你在测试方法体的内部，使用
:meth:`~TestCase.subTest` 上下文管理方法区分它们。

例如下面这个测试::

   class NumbersTest(unittest.TestCase):

       def test_even(self):
           """
           Test that numbers between 0 and 5 are all even.
           """
           for i in range(0, 6):
               with self.subTest(i=i):
                   self.assertEqual(i % 2, 0)

将会生成下面的输出::

   ======================================================================
   FAIL: test_even (__main__.NumbersTest) (i=1)
   ----------------------------------------------------------------------
   Traceback (most recent call last):
     File "subtests.py", line 32, in test_even
       self.assertEqual(i % 2, 0)
   AssertionError: 1 != 0

   ======================================================================
   FAIL: test_even (__main__.NumbersTest) (i=3)
   ----------------------------------------------------------------------
   Traceback (most recent call last):
     File "subtests.py", line 32, in test_even
       self.assertEqual(i % 2, 0)
   AssertionError: 1 != 0

   ======================================================================
   FAIL: test_even (__main__.NumbersTest) (i=5)
   ----------------------------------------------------------------------
   Traceback (most recent call last):
     File "subtests.py", line 32, in test_even
       self.assertEqual(i % 2, 0)
   AssertionError: 1 != 0

如果不是使用 subtest, 在第一次失败之后会停止执行，这种情况下错误不容易诊断出来，因为
这里 ``i`` 的值并没有显示出来::

   ======================================================================
   FAIL: test_even (__main__.NumbersTest)
   ----------------------------------------------------------------------
   Traceback (most recent call last):
     File "subtests.py", line 32, in test_even
       self.assertEqual(i % 2, 0)
   AssertionError: 1 != 0


.. _unittest-contents:

类和方法
---------------------

这一章节将深入表述 :mod:`unittest` 的API.


.. _testcase-objects:

Test cases
~~~~~~~~~~

.. class:: TestCase(methodName='runTest')

   在 :mod:`unittest` 领域，类 :class:`TestCase` 的实例描述了单元测试逻辑。
   这个类将作为基类使用，具体的测试通过具体的子类实现。这个类实现了测试运行组件 （test runner） 所要求的接口，
   并由它来驱动测试，方法中的测试代码能够用来检查以及报告不同种类的失败原因。

   每一个 :class:`TestCase` 的实例将用到一个基本的方法: 就是 *methodName*。
   你在多数使用 :class:`TestCase` 的时候, 你不会去修改这个 *methodName* ，也不会重复实现
   这个默认的 ``runTest()`` 方法。

   .. versionchanged:: 3.2
      :class:`TestCase` 在不提供 *methodName* 的情况下，可以被成功实例化。
      这样做使得在交互解释器中使用个 :class:`TestCase` 更加简单。

   :class:`TestCase` 实例提供了三组方法: 第一组是用来跑测试的， 第二组是用来给测试实现者检查条件和输出错误报告的
   第三组是一些探查方法，用来给测试自身收集信息的。

   第一组中的方法 (运行测试) :

   .. method:: setUp()

      调用这个方法来准备测试夹具. 它是在测试方法调用之前直接被调用的;
      它不像 :exc:`AssertionError` 或者 :exc:`SkipTest` ， 这个方法抛出
      任何异常将被当作错误处理而不是测试失败。默认实现为不做任何事。


   .. method:: tearDown()

      在测试方法调用完并记录结果后，直接调用这个方法。即使测试方法抛出异常，这个方法
      也会被调用， 因此在子类中实现它时，要格外小心检查它的内部状态。除 :exc:`AssertionError`
      和 :exc:`SkipTest` 之外的任何其他异常，这个方法都会认为是错误而不是测试失败。
      这个方法仅在 :meth:`setUp` 方法被成功执行的情况下，被调用，它不关心测试方法的输出结果。
      默认实现为不做任何事。


   .. method:: setUpClass()

      在一个独立的测试类执行之前，会调用这个方法。``setUpClass`` 被调用的时候，类将是唯一的一个参数，并且
      这个方法必需被 :func:`classmethod` 修饰::

        @classmethod
        def setUpClass(cls):
            ...

      详情请看 `Class and Module Fixtures`_ 。

      .. versionadded:: 3.2


   .. method:: tearDownClass()

      在一个单独的类中所有测试运行完之后，会调用这个方法。
      类将作为 ``tearDownClass`` 方法的唯一参数， 并且它必需被 :meth:`classmethod` 修饰::

        @classmethod
        def tearDownClass(cls):
            ...

      详情请看 `Class and Module Fixtures`_ 。

      .. versionadded:: 3.2


   .. method:: run(result=None)

      运行测试, 将收集的结果以 *result* 参数传递给 :class:`TestResult` 对象。
      如果 *result* 被忽略了或者为 ``None``, 将创建并使用一个临时的 result 对象
      (通过调用 :meth:`defaultTestResult` 方法实现). result 对象将返回给 :meth:`run`
      的调用者。

      同样的效果很难通过简单调用 :class:`TestCase` 实例实现。

      .. versionchanged:: 3.3
         前面版本的 ``run`` 将不返回结果. 也不会调用实例。

   .. method:: skipTest(reason)

      在测试方法中或者 :meth:`setUp` 中调用这个方法，将跳过当前的测试。
      详情请查看 :ref:`unittest-skipping` 。

      .. versionadded:: 3.1


   .. method:: subTest(msg=None, **params)

      返回一个用来执行封闭代码块的上下文管理器，它被当作为一个 subtest 。
      参数 *msg* 和 *params* 是可选的, 当一个 subtest 失败时，里面的任何的值都会显示出来。
      这样允许你更清晰的识别它们。

      一个测试用例可以包含任意个 subtest 描述, 并且它们可以任意的嵌套。

      详情请查看 :ref:`subtests` 。

      .. versionadded:: 3.4


   .. method:: debug()

      在测试的时候不收集测试结果。 这就允许测试抛出异常，然后传送给调用者，并且可以用来支持在调试器下运行测试

   .. _assert-methods:

   类 :class:`TestCase` 提供一系列的方法来检查测试方法和报告失败，例如:

   +-----------------------------------------+-----------------------------+---------------+
   | Method                                  | Checks that                 | New in        |
   +=========================================+=============================+===============+
   | :meth:`assertEqual(a, b)                | ``a == b``                  |               |
   | <TestCase.assertEqual>`                 |                             |               |
   +-----------------------------------------+-----------------------------+---------------+
   | :meth:`assertNotEqual(a, b)             | ``a != b``                  |               |
   | <TestCase.assertNotEqual>`              |                             |               |
   +-----------------------------------------+-----------------------------+---------------+
   | :meth:`assertTrue(x)                    | ``bool(x) is True``         |               |
   | <TestCase.assertTrue>`                  |                             |               |
   +-----------------------------------------+-----------------------------+---------------+
   | :meth:`assertFalse(x)                   | ``bool(x) is False``        |               |
   | <TestCase.assertFalse>`                 |                             |               |
   +-----------------------------------------+-----------------------------+---------------+
   | :meth:`assertIs(a, b)                   | ``a is b``                  | 3.1           |
   | <TestCase.assertIs>`                    |                             |               |
   +-----------------------------------------+-----------------------------+---------------+
   | :meth:`assertIsNot(a, b)                | ``a is not b``              | 3.1           |
   | <TestCase.assertIsNot>`                 |                             |               |
   +-----------------------------------------+-----------------------------+---------------+
   | :meth:`assertIsNone(x)                  | ``x is None``               | 3.1           |
   | <TestCase.assertIsNone>`                |                             |               |
   +-----------------------------------------+-----------------------------+---------------+
   | :meth:`assertIsNotNone(x)               | ``x is not None``           | 3.1           |
   | <TestCase.assertIsNotNone>`             |                             |               |
   +-----------------------------------------+-----------------------------+---------------+
   | :meth:`assertIn(a, b)                   | ``a in b``                  | 3.1           |
   | <TestCase.assertIn>`                    |                             |               |
   +-----------------------------------------+-----------------------------+---------------+
   | :meth:`assertNotIn(a, b)                | ``a not in b``              | 3.1           |
   | <TestCase.assertNotIn>`                 |                             |               |
   +-----------------------------------------+-----------------------------+---------------+
   | :meth:`assertIsInstance(a, b)           | ``isinstance(a, b)``        | 3.2           |
   | <TestCase.assertIsInstance>`            |                             |               |
   +-----------------------------------------+-----------------------------+---------------+
   | :meth:`assertNotIsInstance(a, b)        | ``not isinstance(a, b)``    | 3.2           |
   | <TestCase.assertNotIsInstance>`         |                             |               |
   +-----------------------------------------+-----------------------------+---------------+

   All the assert methods accept a *msg* argument that, if specified, is used
   as the error message on failure (see also :data:`longMessage`).
   Note that the *msg* keyword argument can be passed to :meth:`assertRaises`,
   :meth:`assertRaisesRegex`, :meth:`assertWarns`, :meth:`assertWarnsRegex`
   only when they are used as a context manager.

   .. method:: assertEqual(first, second, msg=None)

      Test that *first* and *second* are equal.  If the values do not
      compare equal, the test will fail.

      In addition, if *first* and *second* are the exact same type and one of
      list, tuple, dict, set, frozenset or str or any type that a subclass
      registers with :meth:`addTypeEqualityFunc` the type-specific equality
      function will be called in order to generate a more useful default
      error message (see also the :ref:`list of type-specific methods
      <type-specific-methods>`).

      .. versionchanged:: 3.1
         Added the automatic calling of type-specific equality function.

      .. versionchanged:: 3.2
         :meth:`assertMultiLineEqual` added as the default type equality
         function for comparing strings.


   .. method:: assertNotEqual(first, second, msg=None)

      Test that *first* and *second* are not equal.  If the values do
      compare equal, the test will fail.

   .. method:: assertTrue(expr, msg=None)
               assertFalse(expr, msg=None)

      Test that *expr* is true (or false).

      Note that this is equivalent to ``bool(expr) is True`` and not to ``expr
      is True`` (use ``assertIs(expr, True)`` for the latter).  This method
      should also be avoided when more specific methods are available (e.g.
      ``assertEqual(a, b)`` instead of ``assertTrue(a == b)``), because they
      provide a better error message in case of failure.


   .. method:: assertIs(first, second, msg=None)
               assertIsNot(first, second, msg=None)

      Test that *first* and *second* evaluate (or don't evaluate) to the
      same object.

      .. versionadded:: 3.1


   .. method:: assertIsNone(expr, msg=None)
               assertIsNotNone(expr, msg=None)

      Test that *expr* is (or is not) None.

      .. versionadded:: 3.1


   .. method:: assertIn(first, second, msg=None)
               assertNotIn(first, second, msg=None)

      Test that *first* is (or is not) in *second*.

      .. versionadded:: 3.1


   .. method:: assertIsInstance(obj, cls, msg=None)
               assertNotIsInstance(obj, cls, msg=None)

      Test that *obj* is (or is not) an instance of *cls* (which can be a
      class or a tuple of classes, as supported by :func:`isinstance`).
      To check for the exact type, use :func:`assertIs(type(obj), cls) <assertIs>`.

      .. versionadded:: 3.2



   It is also possible to check the production of exceptions, warnings and
   log messages using the following methods:

   +---------------------------------------------------------+--------------------------------------+------------+
   | Method                                                  | Checks that                          | New in     |
   +=========================================================+======================================+============+
   | :meth:`assertRaises(exc, fun, *args, **kwds)            | ``fun(*args, **kwds)`` raises *exc*  |            |
   | <TestCase.assertRaises>`                                |                                      |            |
   +---------------------------------------------------------+--------------------------------------+------------+
   | :meth:`assertRaisesRegex(exc, r, fun, *args, **kwds)    | ``fun(*args, **kwds)`` raises *exc*  | 3.1        |
   | <TestCase.assertRaisesRegex>`                           | and the message matches regex *r*    |            |
   +---------------------------------------------------------+--------------------------------------+------------+
   | :meth:`assertWarns(warn, fun, *args, **kwds)            | ``fun(*args, **kwds)`` raises *warn* | 3.2        |
   | <TestCase.assertWarns>`                                 |                                      |            |
   +---------------------------------------------------------+--------------------------------------+------------+
   | :meth:`assertWarnsRegex(warn, r, fun, *args, **kwds)    | ``fun(*args, **kwds)`` raises *warn* | 3.2        |
   | <TestCase.assertWarnsRegex>`                            | and the message matches regex *r*    |            |
   +---------------------------------------------------------+--------------------------------------+------------+
   | :meth:`assertLogs(logger, level)                        | The ``with`` block logs on *logger*  | 3.4        |
   | <TestCase.assertLogs>`                                  | with minimum *level*                 |            |
   +---------------------------------------------------------+--------------------------------------+------------+

   .. method:: assertRaises(exception, callable, *args, **kwds)
               assertRaises(exception, msg=None)

      Test that an exception is raised when *callable* is called with any
      positional or keyword arguments that are also passed to
      :meth:`assertRaises`.  The test passes if *exception* is raised, is an
      error if another exception is raised, or fails if no exception is raised.
      To catch any of a group of exceptions, a tuple containing the exception
      classes may be passed as *exception*.

      If only the *exception* and possibly the *msg* arguments are given,
      return a context manager so that the code under test can be written
      inline rather than as a function::

         with self.assertRaises(SomeException):
             do_something()

      When used as a context manager, :meth:`assertRaises` accepts the
      additional keyword argument *msg*.

      The context manager will store the caught exception object in its
      :attr:`exception` attribute.  This can be useful if the intention
      is to perform additional checks on the exception raised::

         with self.assertRaises(SomeException) as cm:
             do_something()

         the_exception = cm.exception
         self.assertEqual(the_exception.error_code, 3)

      .. versionchanged:: 3.1
         Added the ability to use :meth:`assertRaises` as a context manager.

      .. versionchanged:: 3.2
         Added the :attr:`exception` attribute.

      .. versionchanged:: 3.3
         Added the *msg* keyword argument when used as a context manager.


   .. method:: assertRaisesRegex(exception, regex, callable, *args, **kwds)
               assertRaisesRegex(exception, regex, msg=None)

      Like :meth:`assertRaises` but also tests that *regex* matches
      on the string representation of the raised exception.  *regex* may be
      a regular expression object or a string containing a regular expression
      suitable for use by :func:`re.search`.  Examples::

         self.assertRaisesRegex(ValueError, "invalid literal for.*XYZ'$",
                                int, 'XYZ')

      or::

         with self.assertRaisesRegex(ValueError, 'literal'):
            int('XYZ')

      .. versionadded:: 3.1
         under the name ``assertRaisesRegexp``.

      .. versionchanged:: 3.2
         Renamed to :meth:`assertRaisesRegex`.

      .. versionchanged:: 3.3
         Added the *msg* keyword argument when used as a context manager.


   .. method:: assertWarns(warning, callable, *args, **kwds)
               assertWarns(warning, msg=None)

      Test that a warning is triggered when *callable* is called with any
      positional or keyword arguments that are also passed to
      :meth:`assertWarns`.  The test passes if *warning* is triggered and
      fails if it isn't.  Any exception is an error.
      To catch any of a group of warnings, a tuple containing the warning
      classes may be passed as *warnings*.

      If only the *warning* and possibly the *msg* arguments are given,
      return a context manager so that the code under test can be written
      inline rather than as a function::

         with self.assertWarns(SomeWarning):
             do_something()

      When used as a context manager, :meth:`assertWarns` accepts the
      additional keyword argument *msg*.

      The context manager will store the caught warning object in its
      :attr:`warning` attribute, and the source line which triggered the
      warnings in the :attr:`filename` and :attr:`lineno` attributes.
      This can be useful if the intention is to perform additional checks
      on the warning caught::

         with self.assertWarns(SomeWarning) as cm:
             do_something()

         self.assertIn('myfile.py', cm.filename)
         self.assertEqual(320, cm.lineno)

      This method works regardless of the warning filters in place when it
      is called.

      .. versionadded:: 3.2

      .. versionchanged:: 3.3
         Added the *msg* keyword argument when used as a context manager.


   .. method:: assertWarnsRegex(warning, regex, callable, *args, **kwds)
               assertWarnsRegex(warning, regex, msg=None)

      Like :meth:`assertWarns` but also tests that *regex* matches on the
      message of the triggered warning.  *regex* may be a regular expression
      object or a string containing a regular expression suitable for use
      by :func:`re.search`.  Example::

         self.assertWarnsRegex(DeprecationWarning,
                               r'legacy_function\(\) is deprecated',
                               legacy_function, 'XYZ')

      or::

         with self.assertWarnsRegex(RuntimeWarning, 'unsafe frobnicating'):
             frobnicate('/etc/passwd')

      .. versionadded:: 3.2

      .. versionchanged:: 3.3
         Added the *msg* keyword argument when used as a context manager.

   .. method:: assertLogs(logger=None, level=None)

      A context manager to test that at least one message is logged on
      the *logger* or one of its children, with at least the given
      *level*.

      If given, *logger* should be a :class:`logging.Logger` object or a
      :class:`str` giving the name of a logger.  The default is the root
      logger, which will catch all messages.

      If given, *level* should be either a numeric logging level or
      its string equivalent (for example either ``"ERROR"`` or
      :attr:`logging.ERROR`).  The default is :attr:`logging.INFO`.

      The test passes if at least one message emitted inside the ``with``
      block matches the *logger* and *level* conditions, otherwise it fails.

      The object returned by the context manager is a recording helper
      which keeps tracks of the matching log messages.  It has two
      attributes:

      .. attribute:: records

         A list of :class:`logging.LogRecord` objects of the matching
         log messages.

      .. attribute:: output

         A list of :class:`str` objects with the formatted output of
         matching messages.

      Example::

         with self.assertLogs('foo', level='INFO') as cm:
            logging.getLogger('foo').info('first message')
            logging.getLogger('foo.bar').error('second message')
         self.assertEqual(cm.output, ['INFO:foo:first message',
                                      'ERROR:foo.bar:second message'])

      .. versionadded:: 3.4


   There are also other methods used to perform more specific checks, such as:

   +---------------------------------------+--------------------------------+--------------+
   | Method                                | Checks that                    | New in       |
   +=======================================+================================+==============+
   | :meth:`assertAlmostEqual(a, b)        | ``round(a-b, 7) == 0``         |              |
   | <TestCase.assertAlmostEqual>`         |                                |              |
   +---------------------------------------+--------------------------------+--------------+
   | :meth:`assertNotAlmostEqual(a, b)     | ``round(a-b, 7) != 0``         |              |
   | <TestCase.assertNotAlmostEqual>`      |                                |              |
   +---------------------------------------+--------------------------------+--------------+
   | :meth:`assertGreater(a, b)            | ``a > b``                      | 3.1          |
   | <TestCase.assertGreater>`             |                                |              |
   +---------------------------------------+--------------------------------+--------------+
   | :meth:`assertGreaterEqual(a, b)       | ``a >= b``                     | 3.1          |
   | <TestCase.assertGreaterEqual>`        |                                |              |
   +---------------------------------------+--------------------------------+--------------+
   | :meth:`assertLess(a, b)               | ``a < b``                      | 3.1          |
   | <TestCase.assertLess>`                |                                |              |
   +---------------------------------------+--------------------------------+--------------+
   | :meth:`assertLessEqual(a, b)          | ``a <= b``                     | 3.1          |
   | <TestCase.assertLessEqual>`           |                                |              |
   +---------------------------------------+--------------------------------+--------------+
   | :meth:`assertRegex(s, r)              | ``r.search(s)``                | 3.1          |
   | <TestCase.assertRegex>`               |                                |              |
   +---------------------------------------+--------------------------------+--------------+
   | :meth:`assertNotRegex(s, r)           | ``not r.search(s)``            | 3.2          |
   | <TestCase.assertNotRegex>`            |                                |              |
   +---------------------------------------+--------------------------------+--------------+
   | :meth:`assertCountEqual(a, b)         | *a* and *b* have the same      | 3.2          |
   | <TestCase.assertCountEqual>`          | elements in the same number,   |              |
   |                                       | regardless of their order      |              |
   +---------------------------------------+--------------------------------+--------------+


   .. method:: assertAlmostEqual(first, second, places=7, msg=None, delta=None)
               assertNotAlmostEqual(first, second, places=7, msg=None, delta=None)

      Test that *first* and *second* are approximately (or not approximately)
      equal by computing the difference, rounding to the given number of
      decimal *places* (default 7), and comparing to zero.  Note that these
      methods round the values to the given number of *decimal places* (i.e.
      like the :func:`round` function) and not *significant digits*.

      If *delta* is supplied instead of *places* then the difference
      between *first* and *second* must be less or equal to (or greater than) *delta*.

      Supplying both *delta* and *places* raises a ``TypeError``.

      .. versionchanged:: 3.2
         :meth:`assertAlmostEqual` automatically considers almost equal objects
         that compare equal.  :meth:`assertNotAlmostEqual` automatically fails
         if the objects compare equal.  Added the *delta* keyword argument.


   .. method:: assertGreater(first, second, msg=None)
               assertGreaterEqual(first, second, msg=None)
               assertLess(first, second, msg=None)
               assertLessEqual(first, second, msg=None)

      Test that *first* is respectively >, >=, < or <= than *second* depending
      on the method name.  If not, the test will fail::

         >>> self.assertGreaterEqual(3, 4)
         AssertionError: "3" unexpectedly not greater than or equal to "4"

      .. versionadded:: 3.1


   .. method:: assertRegex(text, regex, msg=None)
               assertNotRegex(text, regex, msg=None)

      Test that a *regex* search matches (or does not match) *text*.  In case
      of failure, the error message will include the pattern and the *text* (or
      the pattern and the part of *text* that unexpectedly matched).  *regex*
      may be a regular expression object or a string containing a regular
      expression suitable for use by :func:`re.search`.

      .. versionadded:: 3.1
         under the name ``assertRegexpMatches``.
      .. versionchanged:: 3.2
         The method ``assertRegexpMatches()`` has been renamed to
         :meth:`.assertRegex`.
      .. versionadded:: 3.2
         :meth:`.assertNotRegex`.


   .. method:: assertCountEqual(first, second, msg=None)

      Test that sequence *first* contains the same elements as *second*,
      regardless of their order. When they don't, an error message listing the
      differences between the sequences will be generated.

      Duplicate elements are *not* ignored when comparing *first* and
      *second*. It verifies whether each element has the same count in both
      sequences. Equivalent to:
      ``assertEqual(Counter(list(first)), Counter(list(second)))``
      but works with sequences of unhashable objects as well.

      .. versionadded:: 3.2


   .. _type-specific-methods:

   The :meth:`assertEqual` method dispatches the equality check for objects of
   the same type to different type-specific methods.  These methods are already
   implemented for most of the built-in types, but it's also possible to
   register new methods using :meth:`addTypeEqualityFunc`:

   .. method:: addTypeEqualityFunc(typeobj, function)

      Registers a type-specific method called by :meth:`assertEqual` to check
      if two objects of exactly the same *typeobj* (not subclasses) compare
      equal.  *function* must take two positional arguments and a third msg=None
      keyword argument just as :meth:`assertEqual` does.  It must raise
      :data:`self.failureException(msg) <failureException>` when inequality
      between the first two parameters is detected -- possibly providing useful
      information and explaining the inequalities in details in the error
      message.

      .. versionadded:: 3.1

   The list of type-specific methods automatically used by
   :meth:`~TestCase.assertEqual` are summarized in the following table.  Note
   that it's usually not necessary to invoke these methods directly.

   +-----------------------------------------+-----------------------------+--------------+
   | Method                                  | Used to compare             | New in       |
   +=========================================+=============================+==============+
   | :meth:`assertMultiLineEqual(a, b)       | strings                     | 3.1          |
   | <TestCase.assertMultiLineEqual>`        |                             |              |
   +-----------------------------------------+-----------------------------+--------------+
   | :meth:`assertSequenceEqual(a, b)        | sequences                   | 3.1          |
   | <TestCase.assertSequenceEqual>`         |                             |              |
   +-----------------------------------------+-----------------------------+--------------+
   | :meth:`assertListEqual(a, b)            | lists                       | 3.1          |
   | <TestCase.assertListEqual>`             |                             |              |
   +-----------------------------------------+-----------------------------+--------------+
   | :meth:`assertTupleEqual(a, b)           | tuples                      | 3.1          |
   | <TestCase.assertTupleEqual>`            |                             |              |
   +-----------------------------------------+-----------------------------+--------------+
   | :meth:`assertSetEqual(a, b)             | sets or frozensets          | 3.1          |
   | <TestCase.assertSetEqual>`              |                             |              |
   +-----------------------------------------+-----------------------------+--------------+
   | :meth:`assertDictEqual(a, b)            | dicts                       | 3.1          |
   | <TestCase.assertDictEqual>`             |                             |              |
   +-----------------------------------------+-----------------------------+--------------+



   .. method:: assertMultiLineEqual(first, second, msg=None)

      Test that the multiline string *first* is equal to the string *second*.
      When not equal a diff of the two strings highlighting the differences
      will be included in the error message. This method is used by default
      when comparing strings with :meth:`assertEqual`.

      .. versionadded:: 3.1


   .. method:: assertSequenceEqual(first, second, msg=None, seq_type=None)

      Tests that two sequences are equal.  If a *seq_type* is supplied, both
      *first* and *second* must be instances of *seq_type* or a failure will
      be raised.  If the sequences are different an error message is
      constructed that shows the difference between the two.

      This method is not called directly by :meth:`assertEqual`, but
      it's used to implement :meth:`assertListEqual` and
      :meth:`assertTupleEqual`.

      .. versionadded:: 3.1


   .. method:: assertListEqual(first, second, msg=None)
               assertTupleEqual(first, second, msg=None)

      Tests that two lists or tuples are equal.  If not, an error message is
      constructed that shows only the differences between the two.  An error
      is also raised if either of the parameters are of the wrong type.
      These methods are used by default when comparing lists or tuples with
      :meth:`assertEqual`.

      .. versionadded:: 3.1


   .. method:: assertSetEqual(first, second, msg=None)

      Tests that two sets are equal.  If not, an error message is constructed
      that lists the differences between the sets.  This method is used by
      default when comparing sets or frozensets with :meth:`assertEqual`.

      Fails if either of *first* or *second* does not have a :meth:`set.difference`
      method.

      .. versionadded:: 3.1


   .. method:: assertDictEqual(first, second, msg=None)

      Test that two dictionaries are equal.  If not, an error message is
      constructed that shows the differences in the dictionaries. This
      method will be used by default to compare dictionaries in
      calls to :meth:`assertEqual`.

      .. versionadded:: 3.1



   .. _other-methods-and-attrs:

   Finally the :class:`TestCase` provides the following methods and attributes:


   .. method:: fail(msg=None)

      Signals a test failure unconditionally, with *msg* or ``None`` for
      the error message.


   .. attribute:: failureException

      This class attribute gives the exception raised by the test method.  If a
      test framework needs to use a specialized exception, possibly to carry
      additional information, it must subclass this exception in order to "play
      fair" with the framework.  The initial value of this attribute is
      :exc:`AssertionError`.


   .. attribute:: longMessage

      If set to ``True`` then any explicit failure message you pass in to the
      :ref:`assert methods <assert-methods>` will be appended to the end of the
      normal failure message.  The normal messages contain useful information
      about the objects involved, for example the message from assertEqual
      shows you the repr of the two unequal objects. Setting this attribute
      to ``True`` allows you to have a custom error message in addition to the
      normal one.

      This attribute defaults to ``True``. If set to False then a custom message
      passed to an assert method will silence the normal message.

      The class setting can be overridden in individual tests by assigning an
      instance attribute to ``True`` or ``False`` before calling the assert methods.

      .. versionadded:: 3.1


   .. attribute:: maxDiff

      This attribute controls the maximum length of diffs output by assert
      methods that report diffs on failure. It defaults to 80*8 characters.
      Assert methods affected by this attribute are
      :meth:`assertSequenceEqual` (including all the sequence comparison
      methods that delegate to it), :meth:`assertDictEqual` and
      :meth:`assertMultiLineEqual`.

      Setting ``maxDiff`` to None means that there is no maximum length of
      diffs.

      .. versionadded:: 3.2


   Testing frameworks can use the following methods to collect information on
   the test:


   .. method:: countTestCases()

      Return the number of tests represented by this test object.  For
      :class:`TestCase` instances, this will always be ``1``.


   .. method:: defaultTestResult()

      Return an instance of the test result class that should be used for this
      test case class (if no other result instance is provided to the
      :meth:`run` method).

      For :class:`TestCase` instances, this will always be an instance of
      :class:`TestResult`; subclasses of :class:`TestCase` should override this
      as necessary.


   .. method:: id()

      Return a string identifying the specific test case.  This is usually the
      full name of the test method, including the module and class name.


   .. method:: shortDescription()

      Returns a description of the test, or ``None`` if no description
      has been provided.  The default implementation of this method
      returns the first line of the test method's docstring, if available,
      or ``None``.

      .. versionchanged:: 3.1
         In 3.1 this was changed to add the test name to the short description
         even in the presence of a docstring.  This caused compatibility issues
         with unittest extensions and adding the test name was moved to the
         :class:`TextTestResult` in Python 3.2.


   .. method:: addCleanup(function, *args, **kwargs)

      Add a function to be called after :meth:`tearDown` to cleanup resources
      used during the test. Functions will be called in reverse order to the
      order they are added (LIFO). They are called with any arguments and
      keyword arguments passed into :meth:`addCleanup` when they are
      added.

      If :meth:`setUp` fails, meaning that :meth:`tearDown` is not called,
      then any cleanup functions added will still be called.

      .. versionadded:: 3.1


   .. method:: doCleanups()

      This method is called unconditionally after :meth:`tearDown`, or
      after :meth:`setUp` if :meth:`setUp` raises an exception.

      It is responsible for calling all the cleanup functions added by
      :meth:`addCleanup`. If you need cleanup functions to be called
      *prior* to :meth:`tearDown` then you can call :meth:`doCleanups`
      yourself.

      :meth:`doCleanups` pops methods off the stack of cleanup
      functions one at a time, so it can be called at any time.

      .. versionadded:: 3.1


.. class:: FunctionTestCase(testFunc, setUp=None, tearDown=None, description=None)

   This class implements the portion of the :class:`TestCase` interface which
   allows the test runner to drive the test, but does not provide the methods
   which test code can use to check and report errors.  This is used to create
   test cases using legacy test code, allowing it to be integrated into a
   :mod:`unittest`-based test framework.


.. _deprecated-aliases:

Deprecated aliases
##################

For historical reasons, some of the :class:`TestCase` methods had one or more
aliases that are now deprecated.  The following table lists the correct names
along with their deprecated aliases:

   ==============================  ====================== ======================
    Method Name                     Deprecated alias       Deprecated alias
   ==============================  ====================== ======================
    :meth:`.assertEqual`            failUnlessEqual        assertEquals
    :meth:`.assertNotEqual`         failIfEqual            assertNotEquals
    :meth:`.assertTrue`             failUnless             assert\_
    :meth:`.assertFalse`            failIf
    :meth:`.assertRaises`           failUnlessRaises
    :meth:`.assertAlmostEqual`      failUnlessAlmostEqual  assertAlmostEquals
    :meth:`.assertNotAlmostEqual`   failIfAlmostEqual      assertNotAlmostEquals
    :meth:`.assertRegex`                                   assertRegexpMatches
    :meth:`.assertRaisesRegex`                             assertRaisesRegexp
   ==============================  ====================== ======================

   .. deprecated:: 3.1
         the fail* aliases listed in the second column.
   .. deprecated:: 3.2
         the assert* aliases listed in the third column.
   .. deprecated:: 3.2
         ``assertRegexpMatches`` and ``assertRaisesRegexp`` have been renamed to
         :meth:`.assertRegex` and :meth:`.assertRaisesRegex`


.. _testsuite-objects:

Grouping tests
~~~~~~~~~~~~~~

.. class:: TestSuite(tests=())

   This class represents an aggregation of individual tests cases and test suites.
   The class presents the interface needed by the test runner to allow it to be run
   as any other test case.  Running a :class:`TestSuite` instance is the same as
   iterating over the suite, running each test individually.

   If *tests* is given, it must be an iterable of individual test cases or other
   test suites that will be used to build the suite initially. Additional methods
   are provided to add test cases and suites to the collection later on.

   :class:`TestSuite` objects behave much like :class:`TestCase` objects, except
   they do not actually implement a test.  Instead, they are used to aggregate
   tests into groups of tests that should be run together. Some additional
   methods are available to add tests to :class:`TestSuite` instances:


   .. method:: TestSuite.addTest(test)

      Add a :class:`TestCase` or :class:`TestSuite` to the suite.


   .. method:: TestSuite.addTests(tests)

      Add all the tests from an iterable of :class:`TestCase` and :class:`TestSuite`
      instances to this test suite.

      This is equivalent to iterating over *tests*, calling :meth:`addTest` for
      each element.

   :class:`TestSuite` shares the following methods with :class:`TestCase`:


   .. method:: run(result)

      Run the tests associated with this suite, collecting the result into the
      test result object passed as *result*.  Note that unlike
      :meth:`TestCase.run`, :meth:`TestSuite.run` requires the result object to
      be passed in.


   .. method:: debug()

      Run the tests associated with this suite without collecting the
      result. This allows exceptions raised by the test to be propagated to the
      caller and can be used to support running tests under a debugger.


   .. method:: countTestCases()

      Return the number of tests represented by this test object, including all
      individual tests and sub-suites.


   .. method:: __iter__()

      Tests grouped by a :class:`TestSuite` are always accessed by iteration.
      Subclasses can lazily provide tests by overriding :meth:`__iter__`. Note
      that this method may be called several times on a single suite (for
      example when counting tests or comparing for equality) so the tests
      returned by repeated iterations before :meth:`TestSuite.run` must be the
      same for each call iteration. After :meth:`TestSuite.run`, callers should
      not rely on the tests returned by this method unless the caller uses a
      subclass that overrides :meth:`TestSuite._removeTestAtIndex` to preserve
      test references.

      .. versionchanged:: 3.2
         In earlier versions the :class:`TestSuite` accessed tests directly rather
         than through iteration, so overriding :meth:`__iter__` wasn't sufficient
         for providing tests.

      .. versionchanged:: 3.4
         In earlier versions the :class:`TestSuite` held references to each
         :class:`TestCase` after :meth:`TestSuite.run`. Subclasses can restore
         that behavior by overriding :meth:`TestSuite._removeTestAtIndex`.

   In the typical usage of a :class:`TestSuite` object, the :meth:`run` method
   is invoked by a :class:`TestRunner` rather than by the end-user test harness.


Loading and running tests
~~~~~~~~~~~~~~~~~~~~~~~~~

.. class:: TestLoader()

   The :class:`TestLoader` class is used to create test suites from classes and
   modules.  Normally, there is no need to create an instance of this class; the
   :mod:`unittest` module provides an instance that can be shared as
   :data:`unittest.defaultTestLoader`.  Using a subclass or instance, however,
   allows customization of some configurable properties.

   :class:`TestLoader` objects have the following attributes:


   .. attribute:: errors

      A list of the non-fatal errors encountered while loading tests. Not reset
      by the loader at any point. Fatal errors are signalled by the relevant
      a method raising an exception to the caller. Non-fatal errors are also
      indicated by a synthetic test that will raise the original error when
      run.

      .. versionadded:: 3.5


   :class:`TestLoader` objects have the following methods:


   .. method:: loadTestsFromTestCase(testCaseClass)

      Return a suite of all tests cases contained in the :class:`TestCase`\ -derived
      :class:`testCaseClass`.

      A test case instance is created for each method named by
      :meth:`getTestCaseNames`. By default these are the method names
      beginning with ``test``. If :meth:`getTestCaseNames` returns no
      methods, but the :meth:`runTest` method is implemented, a single test
      case is created for that method instead.


   .. method:: loadTestsFromModule(module, pattern=None)

      Return a suite of all tests cases contained in the given module. This
      method searches *module* for classes derived from :class:`TestCase` and
      creates an instance of the class for each test method defined for the
      class.

      .. note::

         While using a hierarchy of :class:`TestCase`\ -derived classes can be
         convenient in sharing fixtures and helper functions, defining test
         methods on base classes that are not intended to be instantiated
         directly does not play well with this method.  Doing so, however, can
         be useful when the fixtures are different and defined in subclasses.

      If a module provides a ``load_tests`` function it will be called to
      load the tests. This allows modules to customize test loading.
      This is the `load_tests protocol`_.  The *pattern* argument is passed as
      the third argument to ``load_tests``.

      .. versionchanged:: 3.2
         Support for ``load_tests`` added.

      .. versionchanged:: 3.5
         The undocumented and unofficial *use_load_tests* default argument is
         deprecated and ignored, although it is still accepted for backward
         compatibility.  The method also now accepts a keyword-only argument
         *pattern* which is passed to ``load_tests`` as the third argument.


   .. method:: loadTestsFromName(name, module=None)

      Return a suite of all tests cases given a string specifier.

      The specifier *name* is a "dotted name" that may resolve either to a
      module, a test case class, a test method within a test case class, a
      :class:`TestSuite` instance, or a callable object which returns a
      :class:`TestCase` or :class:`TestSuite` instance.  These checks are
      applied in the order listed here; that is, a method on a possible test
      case class will be picked up as "a test method within a test case class",
      rather than "a callable object".

      For example, if you have a module :mod:`SampleTests` containing a
      :class:`TestCase`\ -derived class :class:`SampleTestCase` with three test
      methods (:meth:`test_one`, :meth:`test_two`, and :meth:`test_three`), the
      specifier ``'SampleTests.SampleTestCase'`` would cause this method to
      return a suite which will run all three test methods. Using the specifier
      ``'SampleTests.SampleTestCase.test_two'`` would cause it to return a test
      suite which will run only the :meth:`test_two` test method. The specifier
      can refer to modules and packages which have not been imported; they will
      be imported as a side-effect.

      The method optionally resolves *name* relative to the given *module*.

   .. versionchanged:: 3.5
      If an :exc:`ImportError` or :exc:`AttributeError` occurs while traversing
      *name* then a synthetic test that raises that error when run will be
      returned. These errors are included in the errors accumulated by
      self.errors.


   .. method:: loadTestsFromNames(names, module=None)

      Similar to :meth:`loadTestsFromName`, but takes a sequence of names rather
      than a single name.  The return value is a test suite which supports all
      the tests defined for each name.


   .. method:: getTestCaseNames(testCaseClass)

      Return a sorted sequence of method names found within *testCaseClass*;
      this should be a subclass of :class:`TestCase`.


   .. method:: discover(start_dir, pattern='test*.py', top_level_dir=None)

      Find all the test modules by recursing into subdirectories from the
      specified start directory, and return a TestSuite object containing them.
      Only test files that match *pattern* will be loaded. (Using shell style
      pattern matching.) Only module names that are importable (i.e. are valid
      Python identifiers) will be loaded.

      All test modules must be importable from the top level of the project. If
      the start directory is not the top level directory then the top level
      directory must be specified separately.

      If importing a module fails, for example due to a syntax error, then
      this will be recorded as a single error and discovery will continue.  If
      the import failure is due to :exc:`SkipTest` being raised, it will be
      recorded as a skip instead of an error.

      If a package (a directory containing a file named :file:`__init__.py`) is
      found, the package will be checked for a ``load_tests`` function. If this
      exists then it will be called
      ``package.load_tests(loader, tests, pattern)``. Test discovery takes care
      to ensure that a package is only checked for tests once during an
      invocation, even if the load_tests function itself calls
      ``loader.discover``.

      If ``load_tests`` exists then discovery does *not* recurse into the
      package, ``load_tests`` is responsible for loading all tests in the
      package.

      The pattern is deliberately not stored as a loader attribute so that
      packages can continue discovery themselves. *top_level_dir* is stored so
      ``load_tests`` does not need to pass this argument in to
      ``loader.discover()``.

      *start_dir* can be a dotted module name as well as a directory.

      .. versionadded:: 3.2

      .. versionchanged:: 3.4
         Modules that raise :exc:`SkipTest` on import are recorded as skips,
           not errors.
         Discovery works for :term:`namespace packages <namespace package>`.
         Paths are sorted before being imported so that execution order is
           the same even if the underlying file system's ordering is not
           dependent on file name.

      .. versionchanged:: 3.5
         Found packages are now checked for ``load_tests`` regardless of
         whether their path matches *pattern*, because it is impossible for
         a package name to match the default pattern.


   The following attributes of a :class:`TestLoader` can be configured either by
   subclassing or assignment on an instance:


   .. attribute:: testMethodPrefix

      String giving the prefix of method names which will be interpreted as test
      methods.  The default value is ``'test'``.

      This affects :meth:`getTestCaseNames` and all the :meth:`loadTestsFrom\*`
      methods.


   .. attribute:: sortTestMethodsUsing

      Function to be used to compare method names when sorting them in
      :meth:`getTestCaseNames` and all the :meth:`loadTestsFrom\*` methods.


   .. attribute:: suiteClass

      Callable object that constructs a test suite from a list of tests. No
      methods on the resulting object are needed.  The default value is the
      :class:`TestSuite` class.

      This affects all the :meth:`loadTestsFrom\*` methods.


.. class:: TestResult

   This class is used to compile information about which tests have succeeded
   and which have failed.

   A :class:`TestResult` object stores the results of a set of tests.  The
   :class:`TestCase` and :class:`TestSuite` classes ensure that results are
   properly recorded; test authors do not need to worry about recording the
   outcome of tests.

   Testing frameworks built on top of :mod:`unittest` may want access to the
   :class:`TestResult` object generated by running a set of tests for reporting
   purposes; a :class:`TestResult` instance is returned by the
   :meth:`TestRunner.run` method for this purpose.

   :class:`TestResult` instances have the following attributes that will be of
   interest when inspecting the results of running a set of tests:


   .. attribute:: errors

      A list containing 2-tuples of :class:`TestCase` instances and strings
      holding formatted tracebacks. Each tuple represents a test which raised an
      unexpected exception.

   .. attribute:: failures

      A list containing 2-tuples of :class:`TestCase` instances and strings
      holding formatted tracebacks. Each tuple represents a test where a failure
      was explicitly signalled using the :meth:`TestCase.assert\*` methods.

   .. attribute:: skipped

      A list containing 2-tuples of :class:`TestCase` instances and strings
      holding the reason for skipping the test.

      .. versionadded:: 3.1

   .. attribute:: expectedFailures

      A list containing 2-tuples of :class:`TestCase` instances and strings
      holding formatted tracebacks.  Each tuple represents an expected failure
      of the test case.

   .. attribute:: unexpectedSuccesses

      A list containing :class:`TestCase` instances that were marked as expected
      failures, but succeeded.

   .. attribute:: shouldStop

      Set to ``True`` when the execution of tests should stop by :meth:`stop`.

   .. attribute:: testsRun

      The total number of tests run so far.

   .. attribute:: buffer

      If set to true, ``sys.stdout`` and ``sys.stderr`` will be buffered in between
      :meth:`startTest` and :meth:`stopTest` being called. Collected output will
      only be echoed onto the real ``sys.stdout`` and ``sys.stderr`` if the test
      fails or errors. Any output is also attached to the failure / error message.

      .. versionadded:: 3.2

   .. attribute:: failfast

      If set to true :meth:`stop` will be called on the first failure or error,
      halting the test run.

      .. versionadded:: 3.2

   .. attribute:: tb_locals

      If set to true then local variables will be shown in tracebacks.

      .. versionadded:: 3.5

   .. method:: wasSuccessful()

      Return ``True`` if all tests run so far have passed, otherwise returns
      ``False``.

      .. versionchanged:: 3.4
         Returns ``False`` if there were any :attr:`unexpectedSuccesses`
         from tests marked with the :func:`expectedFailure` decorator.

   .. method:: stop()

      This method can be called to signal that the set of tests being run should
      be aborted by setting the :attr:`shouldStop` attribute to ``True``.
      :class:`TestRunner` objects should respect this flag and return without
      running any additional tests.

      For example, this feature is used by the :class:`TextTestRunner` class to
      stop the test framework when the user signals an interrupt from the
      keyboard.  Interactive tools which provide :class:`TestRunner`
      implementations can use this in a similar manner.

   The following methods of the :class:`TestResult` class are used to maintain
   the internal data structures, and may be extended in subclasses to support
   additional reporting requirements.  This is particularly useful in building
   tools which support interactive reporting while tests are being run.


   .. method:: startTest(test)

      Called when the test case *test* is about to be run.

   .. method:: stopTest(test)

      Called after the test case *test* has been executed, regardless of the
      outcome.

   .. method:: startTestRun()

      Called once before any tests are executed.

      .. versionadded:: 3.1


   .. method:: stopTestRun()

      Called once after all tests are executed.

      .. versionadded:: 3.1


   .. method:: addError(test, err)

      Called when the test case *test* raises an unexpected exception. *err* is a
      tuple of the form returned by :func:`sys.exc_info`: ``(type, value,
      traceback)``.

      The default implementation appends a tuple ``(test, formatted_err)`` to
      the instance's :attr:`errors` attribute, where *formatted_err* is a
      formatted traceback derived from *err*.


   .. method:: addFailure(test, err)

      Called when the test case *test* signals a failure. *err* is a tuple of
      the form returned by :func:`sys.exc_info`: ``(type, value, traceback)``.

      The default implementation appends a tuple ``(test, formatted_err)`` to
      the instance's :attr:`failures` attribute, where *formatted_err* is a
      formatted traceback derived from *err*.


   .. method:: addSuccess(test)

      Called when the test case *test* succeeds.

      The default implementation does nothing.


   .. method:: addSkip(test, reason)

      Called when the test case *test* is skipped.  *reason* is the reason the
      test gave for skipping.

      The default implementation appends a tuple ``(test, reason)`` to the
      instance's :attr:`skipped` attribute.


   .. method:: addExpectedFailure(test, err)

      Called when the test case *test* fails, but was marked with the
      :func:`expectedFailure` decorator.

      The default implementation appends a tuple ``(test, formatted_err)`` to
      the instance's :attr:`expectedFailures` attribute, where *formatted_err*
      is a formatted traceback derived from *err*.


   .. method:: addUnexpectedSuccess(test)

      Called when the test case *test* was marked with the
      :func:`expectedFailure` decorator, but succeeded.

      The default implementation appends the test to the instance's
      :attr:`unexpectedSuccesses` attribute.


   .. method:: addSubTest(test, subtest, outcome)

      Called when a subtest finishes.  *test* is the test case
      corresponding to the test method.  *subtest* is a custom
      :class:`TestCase` instance describing the subtest.

      If *outcome* is :const:`None`, the subtest succeeded.  Otherwise,
      it failed with an exception where *outcome* is a tuple of the form
      returned by :func:`sys.exc_info`: ``(type, value, traceback)``.

      The default implementation does nothing when the outcome is a
      success, and records subtest failures as normal failures.

      .. versionadded:: 3.4


.. class:: TextTestResult(stream, descriptions, verbosity)

   A concrete implementation of :class:`TestResult` used by the
   :class:`TextTestRunner`.

   .. versionadded:: 3.2
      This class was previously named ``_TextTestResult``. The old name still
      exists as an alias but is deprecated.


.. data:: defaultTestLoader

   Instance of the :class:`TestLoader` class intended to be shared.  If no
   customization of the :class:`TestLoader` is needed, this instance can be used
   instead of repeatedly creating new instances.


.. class:: TextTestRunner(stream=None, descriptions=True, verbosity=1, failfast=False, \
                          buffer=False, resultclass=None, warnings=None, *, tb_locals=False)

   A basic test runner implementation that outputs results to a stream. If *stream*
   is ``None``, the default, :data:`sys.stderr` is used as the output stream. This class
   has a few configurable parameters, but is essentially very simple.  Graphical
   applications which run test suites should provide alternate implementations. Such
   implementations should accept ``**kwargs`` as the interface to construct runners
   changes when features are added to unittest.

   By default this runner shows :exc:`DeprecationWarning`,
   :exc:`PendingDeprecationWarning`, :exc:`ResourceWarning` and
   :exc:`ImportWarning` even if they are :ref:`ignored by default
   <warning-ignored>`. Deprecation warnings caused by :ref:`deprecated unittest
   methods <deprecated-aliases>` are also special-cased and, when the warning
   filters are ``'default'`` or ``'always'``, they will appear only once
   per-module, in order to avoid too many warning messages.  This behavior can
   be overridden using the :option:`-Wd` or :option:`-Wa` options and leaving
   *warnings* to ``None``.

   .. versionchanged:: 3.2
      Added the ``warnings`` argument.

   .. versionchanged:: 3.2
      The default stream is set to :data:`sys.stderr` at instantiation time rather
      than import time.

   .. versionchanged:: 3.5
      Added the tb_locals parameter.

   .. method:: _makeResult()

      This method returns the instance of ``TestResult`` used by :meth:`run`.
      It is not intended to be called directly, but can be overridden in
      subclasses to provide a custom ``TestResult``.

      ``_makeResult()`` instantiates the class or callable passed in the
      ``TextTestRunner`` constructor as the ``resultclass`` argument. It
      defaults to :class:`TextTestResult` if no ``resultclass`` is provided.
      The result class is instantiated with the following arguments::

        stream, descriptions, verbosity

   .. method:: run(test)

      This method is the main public interface to the `TextTestRunner`. This
      method takes a :class:`TestSuite` or :class:`TestCase` instance. A
      :class:`TestResult` is created by calling
      :func:`_makeResult` and the test(s) are run and the
      results printed to stdout.


.. function:: main(module='__main__', defaultTest=None, argv=None, testRunner=None, \
                   testLoader=unittest.defaultTestLoader, exit=True, verbosity=1, \
                   failfast=None, catchbreak=None, buffer=None, warnings=None)

   A command-line program that loads a set of tests from *module* and runs them;
   this is primarily for making test modules conveniently executable.
   The simplest use for this function is to include the following line at the
   end of a test script::

      if __name__ == '__main__':
          unittest.main()

   You can run tests with more detailed information by passing in the verbosity
   argument::

      if __name__ == '__main__':
          unittest.main(verbosity=2)

   The *defaultTest* argument is either the name of a single test or an
   iterable of test names to run if no test names are specified via *argv*.  If
   not specified or ``None`` and no test names are provided via *argv*, all
   tests found in *module* are run.

   The *argv* argument can be a list of options passed to the program, with the
   first element being the program name.  If not specified or ``None``,
   the values of :data:`sys.argv` are used.

   The *testRunner* argument can either be a test runner class or an already
   created instance of it. By default ``main`` calls :func:`sys.exit` with
   an exit code indicating success or failure of the tests run.

   The *testLoader* argument has to be a :class:`TestLoader` instance,
   and defaults to :data:`defaultTestLoader`.

   ``main`` supports being used from the interactive interpreter by passing in the
   argument ``exit=False``. This displays the result on standard output without
   calling :func:`sys.exit`::

      >>> from unittest import main
      >>> main(module='test_module', exit=False)

   The *failfast*, *catchbreak* and *buffer* parameters have the same
   effect as the same-name `command-line options`_.

   The *warning* argument specifies the :ref:`warning filter <warning-filter>`
   that should be used while running the tests.  If it's not specified, it will
   remain ``None`` if a :option:`-W` option is passed to :program:`python`,
   otherwise it will be set to ``'default'``.

   Calling ``main`` actually returns an instance of the ``TestProgram`` class.
   This stores the result of the tests run as the ``result`` attribute.

   .. versionchanged:: 3.1
      The *exit* parameter was added.

   .. versionchanged:: 3.2
      The *verbosity*, *failfast*, *catchbreak*, *buffer*
      and *warnings* parameters were added.

   .. versionchanged:: 3.4
      The *defaultTest* parameter was changed to also accept an iterable of
      test names.


load_tests Protocol
###################

.. versionadded:: 3.2

Modules or packages can customize how tests are loaded from them during normal
test runs or test discovery by implementing a function called ``load_tests``.

If a test module defines ``load_tests`` it will be called by
:meth:`TestLoader.loadTestsFromModule` with the following arguments::

    load_tests(loader, standard_tests, pattern)

where *pattern* is passed straight through from ``loadTestsFromModule``.  It
defaults to ``None``.

It should return a :class:`TestSuite`.

*loader* is the instance of :class:`TestLoader` doing the loading.
*standard_tests* are the tests that would be loaded by default from the
module. It is common for test modules to only want to add or remove tests
from the standard set of tests.
The third argument is used when loading packages as part of test discovery.

A typical ``load_tests`` function that loads tests from a specific set of
:class:`TestCase` classes may look like::

    test_cases = (TestCase1, TestCase2, TestCase3)

    def load_tests(loader, tests, pattern):
        suite = TestSuite()
        for test_class in test_cases:
            tests = loader.loadTestsFromTestCase(test_class)
            suite.addTests(tests)
        return suite

If discovery is started in a directory containing a package, either from the
command line or by calling :meth:`TestLoader.discover`, then the package
:file:`__init__.py` will be checked for ``load_tests``.  If that function does
not exist, discovery will recurse into the package as though it were just
another directory.  Otherwise, discovery of the package's tests will be left up
to ``load_tests`` which is called with the following arguments::

    load_tests(loader, standard_tests, pattern)

This should return a :class:`TestSuite` representing all the tests
from the package. (``standard_tests`` will only contain tests
collected from :file:`__init__.py`.)

Because the pattern is passed into ``load_tests`` the package is free to
continue (and potentially modify) test discovery. A 'do nothing'
``load_tests`` function for a test package would look like::

    def load_tests(loader, standard_tests, pattern):
        # top level directory cached on loader instance
        this_dir = os.path.dirname(__file__)
        package_tests = loader.discover(start_dir=this_dir, pattern=pattern)
        standard_tests.addTests(package_tests)
        return standard_tests

.. versionchanged:: 3.5
   Discovery no longer checks package names for matching *pattern* due to the
   impossibility of package names matching the default pattern.



Class and Module Fixtures
-------------------------

Class and module level fixtures are implemented in :class:`TestSuite`. When
the test suite encounters a test from a new class then :meth:`tearDownClass`
from the previous class (if there is one) is called, followed by
:meth:`setUpClass` from the new class.

Similarly if a test is from a different module from the previous test then
``tearDownModule`` from the previous module is run, followed by
``setUpModule`` from the new module.

After all the tests have run the final ``tearDownClass`` and
``tearDownModule`` are run.

Note that shared fixtures do not play well with [potential] features like test
parallelization and they break test isolation. They should be used with care.

The default ordering of tests created by the unittest test loaders is to group
all tests from the same modules and classes together. This will lead to
``setUpClass`` / ``setUpModule`` (etc) being called exactly once per class and
module. If you randomize the order, so that tests from different modules and
classes are adjacent to each other, then these shared fixture functions may be
called multiple times in a single test run.

Shared fixtures are not intended to work with suites with non-standard
ordering. A ``BaseTestSuite`` still exists for frameworks that don't want to
support shared fixtures.

If there are any exceptions raised during one of the shared fixture functions
the test is reported as an error. Because there is no corresponding test
instance an ``_ErrorHolder`` object (that has the same interface as a
:class:`TestCase`) is created to represent the error. If you are just using
the standard unittest test runner then this detail doesn't matter, but if you
are a framework author it may be relevant.


setUpClass and tearDownClass
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

These must be implemented as class methods::

    import unittest

    class Test(unittest.TestCase):
        @classmethod
        def setUpClass(cls):
            cls._connection = createExpensiveConnectionObject()

        @classmethod
        def tearDownClass(cls):
            cls._connection.destroy()

If you want the ``setUpClass`` and ``tearDownClass`` on base classes called
then you must call up to them yourself. The implementations in
:class:`TestCase` are empty.

If an exception is raised during a ``setUpClass`` then the tests in the class
are not run and the ``tearDownClass`` is not run. Skipped classes will not
have ``setUpClass`` or ``tearDownClass`` run. If the exception is a
:exc:`SkipTest` exception then the class will be reported as having been skipped
instead of as an error.


setUpModule and tearDownModule
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

These should be implemented as functions::

    def setUpModule():
        createConnection()

    def tearDownModule():
        closeConnection()

If an exception is raised in a ``setUpModule`` then none of the tests in the
module will be run and the ``tearDownModule`` will not be run. If the exception is a
:exc:`SkipTest` exception then the module will be reported as having been skipped
instead of as an error.


Signal Handling
---------------

.. versionadded:: 3.2

The :option:`-c/--catch <unittest -c>` command-line option to unittest,
along with the ``catchbreak`` parameter to :func:`unittest.main()`, provide
more friendly handling of control-C during a test run. With catch break
behavior enabled control-C will allow the currently running test to complete,
and the test run will then end and report all the results so far. A second
control-c will raise a :exc:`KeyboardInterrupt` in the usual way.

The control-c handling signal handler attempts to remain compatible with code or
tests that install their own :const:`signal.SIGINT` handler. If the ``unittest``
handler is called but *isn't* the installed :const:`signal.SIGINT` handler,
i.e. it has been replaced by the system under test and delegated to, then it
calls the default handler. This will normally be the expected behavior by code
that replaces an installed handler and delegates to it. For individual tests
that need ``unittest`` control-c handling disabled the :func:`removeHandler`
decorator can be used.

There are a few utility functions for framework authors to enable control-c
handling functionality within test frameworks.

.. function:: installHandler()

   Install the control-c handler. When a :const:`signal.SIGINT` is received
   (usually in response to the user pressing control-c) all registered results
   have :meth:`~TestResult.stop` called.


.. function:: registerResult(result)

   Register a :class:`TestResult` object for control-c handling. Registering a
   result stores a weak reference to it, so it doesn't prevent the result from
   being garbage collected.

   Registering a :class:`TestResult` object has no side-effects if control-c
   handling is not enabled, so test frameworks can unconditionally register
   all results they create independently of whether or not handling is enabled.


.. function:: removeResult(result)

   Remove a registered result. Once a result has been removed then
   :meth:`~TestResult.stop` will no longer be called on that result object in
   response to a control-c.


.. function:: removeHandler(function=None)

   When called without arguments this function removes the control-c handler
   if it has been installed. This function can also be used as a test decorator
   to temporarily remove the handler whilst the test is being executed::

      @unittest.removeHandler
      def test_signal_handling(self):
          ...
