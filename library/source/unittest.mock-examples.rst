:mod:`unittest.mock` --- 开始 mock 之旅
========================================

.. moduleauthor:: Michael Foord <michael@python.org>
.. currentmodule:: unittest.mock

.. versionadded:: 3.3


.. _getting-started:

使用 Mock
----------

Mock 补丁方法
~~~~~~~~~~~~~~~~~~~~~

通常使用的 :class:`Mock` 对象包括:

* 给方法打补丁
* 记录方法在对象上的调用

你可能想要替换对象的某个方法，来检查这个方法是否被系统的其它部分以正确参数调用:

    >>> real = SomeClass()
    >>> real.method = MagicMock(name='method')
    >>> real.method(3, 4, 5, key='value')
    <MagicMock name='method()' id='...'>

一旦你的 mock 被使用了 (这个例子中是指 ``real.method`` )， 这个mock就包含了允许你进行断言的方法和属性.

.. note::

    在大多数例子中，:class:`Mock` 和 :class:`MagicMock` 类是可以互换的。  
    ``MagicMock`` 更加适合类， 默认使用它是明智的选择。

一旦一个 mock 被调用了， 它的 :attr:`~Mock.called` 属性将会设置程 ``True``. 
更重要的是我们可以使用方法 :meth:`~Mock.assert_called_with` 或者 :meth:`~Mock.assert_called_once_with` 
来检查方法是不是使用了正确的参数调用。

这个例子中测试了调用方法 ``ProductionClass().method`` 的结果是，调用了一次 ``something`` 方法:

    >>> class ProductionClass:
    ...     def method(self):
    ...         self.something(1, 2, 3)
    ...     def something(self, a, b, c):
    ...         pass
    ...
    >>> real = ProductionClass()
    >>> real.something = MagicMock()
    >>> real.method()
    >>> real.something.assert_called_once_with(1, 2, 3)



模仿对象上的方法调用
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

在最后这个例子中，我们直接在一个对象上补丁了一个方法，来检查这个方法是不是被正确调用了
另外一种普通的做法是传递一个对象到方法中 (或者系统中要测试的某部分)， 
然后检查这个对象是不是以正确方法使用。

下面这个简单 ``ProductionClass`` 类中还有一个 ``closer`` 方法. 如果它被一个对象调用，那么它就会调用这个对象的 
``close`` 方法。

    >>> class ProductionClass:
    ...     def closer(self, something):
    ...         something.close()
    ...

因此为了测试，我们需要传递的对象中包含 ``close`` 方法， 然后检查这个方法是不是被正确调用了。

    >>> real = ProductionClass()
    >>> mock = Mock()
    >>> real.closer(mock)
    >>> mock.close.assert_called_with()

我们在给我们的 mock 提供 'close' 方法上没有做任何事情。 访问 close 时就创建了它。 因此在测试中如果 'close' 
还没有被调用，就去访问它 ，这时会创建它， 但是 :meth:`~Mock.assert_called_with` 将会抛出一个失败的异常。


仿造类
~~~~~~~~~~~~~~~

一个常用的做法是在我们的测试代码中仿造类的实例。当你给一个类打补丁的时候，这个类被替换成了一个 mock.
实例的创建是通过 *调用这个类* . 也就是说，你可以通过查看仿造类的返回值来访问这个 "mock 实例"

在下面的例子中，我们有一个方法 ``some_function`` ，实例化了 ``Foo`` 并调用这个实例中的一方法。
调用 :func:`patch` 从而将 ``Foo`` 类替换成了一个 mock. 这个 ``Foo`` 实例是调用 mock 的结果，所以它可以通过
修改 mock 的  :attr:`~Mock.return_value` 来配置。

    >>> def some_function():
    ...     instance = module.Foo()
    ...     return instance.method()
    ...
    >>> with patch('module.Foo') as mock:
    ...     instance = mock.return_value
    ...     instance.method.return_value = 'the result'
    ...     result = some_function()
    ...     assert result == 'the result'


给你的 mocks 命名
~~~~~~~~~~~~~~~~~

给 mocks 提供一个名字非常有用。 这个名字会在 repr 中显示， 并且当测试失败信息中出现 mock
时，可以提供帮助。命名同样可以针对于 mock 的属性和方法:

    >>> mock = MagicMock(name='foo')
    >>> mock
    <MagicMock name='foo' id='...'>
    >>> mock.method
    <MagicMock name='foo.method' id='...'>


跟踪所有的调用
~~~~~~~~~~~~~~~~~~

经常，你想跟踪不止一个方法的调用。mock 有个 :attr:`~Mock.mock_calls` 
属性记录了所有的子属性的调用 - 也包括它们的子类.

    >>> mock = MagicMock()
    >>> mock.method()
    <MagicMock name='mock.method()' id='...'>
    >>> mock.attribute.method(10, x=53)
    <MagicMock name='mock.attribute.method()' id='...'>
    >>> mock.mock_calls
    [call.method(), call.attribute.method(10, x=53)]

如果你给 ``mock_calls`` 做了一个断言，任何非预期的方法被调用了，这个断言都会失败。
这非常有用，因为在断言的时候，你已经知道调用是你所期望的，你同样想检查他们调用的顺序
以及不会有多余的调用:

你使用对象 :data:`call` 构造了一个列表用来比较 ``mock_calls``:

    >>> expected = [call.method(), call.attribute.method(10, x=53)]
    >>> mock.mock_calls == expected
    True


设置返回值和属性
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

在 mock 对象上设置返回值很简单:

    >>> mock = Mock()
    >>> mock.return_value = 3
    >>> mock()
    3

当然你也可以在方法上这样做：

    >>> mock = Mock()
    >>> mock.method.return_value = 3
    >>> mock.method()
    3

返回值也可以在构造器中设置:

    >>> mock = Mock(return_value=3)
    >>> mock()
    3

如果你想在 mock 上设置一个属性，像下面这样做:

    >>> mock = Mock()
    >>> mock.x = 3
    >>> mock.x
    3

有时你想仿造一种很复杂的情况，例如 ``mock.connection.cursor().execute("SELECT 1")``. 
如果我们想在调用的时候返回一个列表， 我们需要配置这个结果为嵌套调用。

我们可以使用 :data:`call` ，在 "chained call" 中构造一个调用集合， 然后接下来就可以像下面这样轻松断言了:

    >>> mock = Mock()
    >>> cursor = mock.connection.cursor.return_value
    >>> cursor.execute.return_value = ['foo']
    >>> mock.connection.cursor().execute("SELECT 1")
    ['foo']
    >>> expected = call.connection.cursor().execute("SELECT 1").call_list()
    >>> mock.mock_calls
    [call.connection.cursor(), call.connection.cursor().execute('SELECT 1')]
    >>> mock.mock_calls == expected
    True

调用 ``.call_list()`` 将我们的调用对象转换为一个调用列表， 来描述调用链。


mocks 抛出异常
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

:attr:`~Mock.side_effect` 是一个有用的属性. 如果你在一个异常类中设置了它或者实例化，
当mock被调用时，这个异常将会被抛出。

    >>> mock = Mock(side_effect=Exception('Boom!'))
    >>> mock()
    Traceback (most recent call last):
      ...
    Exception: Boom!


边效应方法和迭代器
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``side_effect`` 同样可以设置在方法或者迭代器上面。一种使用情况是，
一个mock 被调用多次时，并且你希望每次调用都返回不同的值， 
可以将 ``side_effect`` 作为一个迭代器。当你在仿对象上面设置了 ``side_effect`` ，并把它作为迭代器时 
，每次调用 mock 返回的值将是从迭代器中取下一个的值:

    >>> mock = MagicMock(side_effect=[4, 5, 6])
    >>> mock()
    4
    >>> mock()
    5
    >>> mock()
    6


其他一些高级用法，例如依赖于具体哪个 mock 的调用动态改变返回值， ``side_effect`` 可以是一个函数。
这个函数作为一个 mock 以同样的参数被调用. 无论函数如何返回都和它的调用返回一样:

    >>> vals = {(1, 2): 1, (2, 3): 2}
    >>> def side_effect(*args):
    ...     return vals[args]
    ...
    >>> mock = MagicMock(side_effect=side_effect)
    >>> mock(1, 2)
    1
    >>> mock(2, 3)
    2


从一个已存在的对象中创建 Mock 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

在使用仿技术的时候，会有一个问题，与你测试的实现代码是你的 mocks 而不是你的真实代码。
假设你有一个类实现了 ``some_method`` 方法。在其他类中测试时， 你提供了这个对象的 mock
并且也提供了 ``some_method`` 。 如果你后来想要重构第一个类， 这时它已经没有 ``some_method`` 
- 然后对你的测试来说它是可以通过的，即使此时代码是有问题的!

:class:`Mock` 允许你提供一个对象，然后定制这个对象的 mock, 使用 *spec* 关键字参数。
访问这个 mock 所制定的对象中不存在的方法或者属性时，将会立即抛出属性错误。如果你改变定制
对象的实现，那么对于这个类的测试将会失败，因为在这些测试当中已经没有这个类的实例了。

    >>> mock = Mock(spec=SomeClass)
    >>> mock.old_method()
    Traceback (most recent call last):
       ...
    AttributeError: object has no attribute 'old_method'

使用定制对象能够精确的匹配对 mock 所做的调用，它不管是否参数传递的是位置还是命名参数::

   >>> def f(a, b, c): pass
   ...
   >>> mock = Mock(spec=f)
   >>> mock(1, 2, 3)
   <Mock name='mock()' id='140161580456576'>
   >>> mock.assert_called_with(a=1, b=2, c=3)

如果你想这种精确匹配可以同样在 mock 的方法调用时工作， :ref:`auto-speccing <auto-speccing>`.

如果你想更强模型的定制对象，来阻值任意属性的设置以及获取这些属性，你可以使用
*spec_set* 代替上面的 *spec*.


Patch 修饰符
----------------

.. note::

   :func:`patch` 很重要的一点是，你打补丁的对象是在他们所查的命名空间中的
   这很简单，你可以阅读快速指南 :ref:`where to patch <where-to-patch>` .

在一个测试中通常需要对类的属性或者模块属性打补丁， 例如修补内置类或模块中的类的实例。
模块和类是全局有效的，因此修补他们之后最后进行还原，否则它会对其他测试造成影响而导致一些很难诊断出来的问题。

基于此，mock 提供了三个非常方便的修饰符: :func:`patch`, :func:`patch.object` 和
:func:`patch.dict`. ``patch`` 使用了一个独立的字符串，来指定你要修补的属性，字符串的格式是这样
``package.module.Class.attribute`` . 它也可以选择性的使用一个值，来替换你想要的替换的属性（或者类以及其他).
'patch.object' 使用了一个对象以及想要修补的属性名字, 并加上可选的值来修补它。


``patch.object``:

    >>> original = SomeClass.attribute
    >>> @patch.object(SomeClass, 'attribute', sentinel.attribute)
    ... def test():
    ...     assert SomeClass.attribute == sentinel.attribute
    ...
    >>> test()
    >>> assert SomeClass.attribute == original

    >>> @patch('package.module.attribute', sentinel.attribute)
    ... def test():
    ...     from package.module import attribute
    ...     assert attribute is sentinel.attribute
    ...
    >>> test()

如果你要修补一个模块 (包括 :mod:`builtins`) 那么使用 :func:`patch`
代替 :func:`patch.object`:

    >>> mock = MagicMock(return_value=sentinel.file_handle)
    >>> with patch('builtins.open', mock):
    ...     handle = open('filename', 'r')
    ...
    >>> mock.assert_called_with('filename', 'r')
    >>> assert handle == sentinel.file_handle, "incorrect file handle returned"

模块的名字可以 '点号' 形式的, 如果需要可以是 ``package.module`` 这样形式的:

    >>> @patch('package.module.ClassName.attribute', sentinel.attribute)
    ... def test():
    ...     from package.module import ClassName
    ...     assert ClassName.attribute == sentinel.attribute
    ...
    >>> test()

一个好的的使用方式是修饰测试方法本身:

    >>> class MyTest(unittest2.TestCase):
    ...     @patch.object(SomeClass, 'attribute', sentinel.attribute)
    ...     def test_something(self):
    ...         self.assertEqual(SomeClass.attribute, sentinel.attribute)
    ...
    >>> original = SomeClass.attribute
    >>> MyTest('test_something').test_something()
    >>> assert SomeClass.attribute == original

如果你想使用一个 Mock 来修补, 你可以使用带有一个参数的 :func:`patch` 
(或者带两个参数的 :func:`patch.object` ). mock 将会为你创建，并且传递到测试函数 / 方法中:

    >>> class MyTest(unittest2.TestCase):
    ...     @patch.object(SomeClass, 'static_method')
    ...     def test_something(self, mock_method):
    ...         SomeClass.static_method()
    ...         mock_method.assert_called_with()
    ...
    >>> MyTest('test_something').test_something()

你可以叠加多个 patch 修饰，像下面这样:

    >>> class MyTest(unittest2.TestCase):
    ...     @patch('package.module.ClassName1')
    ...     @patch('package.module.ClassName2')
    ...     def test_something(self, MockClass2, MockClass1):
    ...         self.assertIs(package.module.ClassName1, MockClass1)
    ...         self.assertIs(package.module.ClassName2, MockClass2)
    ...
    >>> MyTest('test_something').test_something()

当你嵌套多个 patch 修饰符时，mock 传递给修饰方法的顺序与它修饰时一致。(与普通 *python* 修饰符使用是一样的)
这也就意味着是自底向上的，所以在上面的这个例子中 ``test_module.ClassName2`` 模仿类将首先被传递进修饰方法。

:func:`patch.dict` 是用来在一个字典中设置值的，这些值会一段时间内存在， 当测试结束它们被还原:

   >>> foo = {'key': 'value'}
   >>> original = foo.copy()
   >>> with patch.dict(foo, {'newkey': 'newvalue'}, clear=True):
   ...     assert foo == {'newkey': 'newvalue'}
   ...
   >>> assert foo == original

``patch``, ``patch.object`` 和 ``patch.dict`` 都可以使用上下文管理。

当在某个地方，你使用 :func:`patch` 为你创建了一个 mock, 你可以获得这个 mock 的引用, 然后使用 with 语句里
的  "as" 格式:

    >>> class ProductionClass:
    ...     def method(self):
    ...         pass
    ...
    >>> with patch.object(ProductionClass, 'method') as mock_method:
    ...     mock_method.return_value = None
    ...     real = ProductionClass()
    ...     real.method(1, 2, 3)
    ...
    >>> mock_method.assert_called_with(1, 2, 3)


作为一种替代， ``patch``, ``patch.object`` 和 ``patch.dict`` 能够可以用于类修饰符
如果你使用这种方式，它等同于你给每个名字以  "test" 开始的方法上面单独的使用了这个修饰符.


.. _further-examples:

深入的例子
----------------


这里有一些更高级场景的例子。


仿造链式调用
~~~~~~~~~~~~~~~~~~~~~

当你理解了 :attr:`~Mock.return_value` 属性时，使用 mock 仿造链式调用实际上是非常直截了当的. 
当一个 mock 被首次调用时，或者在它被调用前你去获取它的 ``return_value`` 时，会创建一个新的
:class:`Mock` 类.

这就意味着你可以看到对象从一个仿对象调用的返回，这个仿对象已经通过查询 ``return_value``  mock
被使用了:

    >>> mock = Mock()
    >>> mock().foo(a=2, b=3)
    <Mock name='mock().foo()' id='...'>
    >>> mock.return_value.foo.assert_called_with(a=2, b=3)

这里只是一个简单的配置步骤，然后使用一个关于链式调用的断言. 
当然另外可选的方式是，首先考虑下你编写的代码的可测性...

因此, 假设我们有下面这样一些代码:

    >>> class Something:
    ...     def __init__(self):
    ...         self.backend = BackendProvider()
    ...     def method(self):
    ...         response = self.backend.get_endpoint('foobar').create_call('spam', 'eggs').start_call()
    ...         # more code

假设 ``BackendProvider`` 已经通过测试, 那我们怎么指定测试
``method()`` ? , 我们想要以正确的方式，使用 response 对象测试代码段 ``# more code`` 。

由于这个链式调用，是由一个实例属性产生的,我们可以在 ``Something`` 实例上鼓捣一下 ``backend`` 
属性。在这种特殊的情况下，我仅仅感兴趣的是来自最后调用的 ``start_call``。
所以我们没有做更多的配置. 让我们假设这个返回是一个 '类似-文件' 形式的，这样我们将可以确保我们的 response
对象使用的是指定的内置 :func:`open` 方法.

为了达到目地，我们创建了一个 mock 实例， 来作为 mock 后台
然后给它创建了一个 response 对象。为了将最后调用的 ``start_call`` 的返回值设置为 response 
我们可以这样做::

    mock_backend.get_endpoint.return_value.create_call.return_value.start_call.return_value = mock_response

我们也可以使用更加优雅的方式，使用 :meth:`~Mock.configure_mock`
方法直接给我们设置返回值:

    >>> something = Something()
    >>> mock_response = Mock(spec=open)
    >>> mock_backend = Mock()
    >>> config = {'get_endpoint.return_value.create_call.return_value.start_call.return_value': mock_response}
    >>> mock_backend.configure_mock(**config)

在这之后，我们弄了一个 "mock backend" 补丁，它可以使用真实调用:

    >>> something.backend = mock_backend
    >>> something.method()

使用 :attr:`~Mock.mock_calls` ， 我们可以通过一个 assert 检查链式调用. 
一个链式调用是一行代码有几个调用, 所以会在 ``mock_calls`` 中有几个调用实体. 
我们可以使用 :meth:`call.call_list` 来为我们创建一个调用列表:

    >>> chained = call.get_endpoint('foobar').create_call('spam', 'eggs').start_call()
    >>> call_list = chained.call_list()
    >>> assert mock_backend.mock_calls == call_list


局部仿造
~~~~~~~~~~~~~~~

在有些测试中，我们想要仿造出一个 :meth:`datetime.date.today` 调用来返回一个已知日期,
但是我们不想阻止被测试代码创建一个新的 date 对象. 
不幸的是， :class:`datetime.date` 使用 C 编写的, 所以我们无法简单的弄出一个 
:meth:`date.today` 静态方法.

我找到了一种简单的方式来做这个事，将 Date 有效的加入到仿造的包裹类, 
除此，还可以传递给真实类的构造器并返回真实的实例.

在测试模块中，这里使用 :func:`patch decorator <patch>` 修饰器来仿造出 ``date`` 
类. 在 mock 上的 :attr:`side_effect` 属性设置为一个匿名函数，它返回一个真实的 date.
当这个仿造 date 类被调用时，会构造一个真实的日期，然后通过 ``side_effect`` 返回.

    >>> from datetime import date
    >>> with patch('mymodule.date') as mock_date:
    ...     mock_date.today.return_value = date(2010, 10, 8)
    ...     mock_date.side_effect = lambda *args, **kw: date(*args, **kw)
    ...
    ...     assert mymodule.date.today() == date(2010, 10, 8)
    ...     assert mymodule.date(2009, 6, 8) == date(2009, 6, 8)
    ...

注意我们并没有全局的修补 :class:`datetime.date` , 我们只在使用的模块中修补 ``date`` . 
请查看 :ref:`where to patch <where-to-patch>`.

当 ``date.today()`` 被调用时，会返回一个已知日期，但是实际去调用
``date(...)`` 构造器时，仍然会返回正常的日期。不这样的话，在你的测试的代码中，
你可以使用相同的逻辑找到你自己期望计算得到的结果, 这是一个经典的反模式测试.

调用日期构造器会记录到 ``mock_date`` 属性中
(``call_count`` 和其他) 在你的测试中也许非常有用.

一个可选的方式来处理仿造日期或者其它内置类，在
`这篇博文 <http://www.williamjohnbert.com/2011/07/how-to-unit-testing-in-django-with-mocking-and-patching/>`_
中有这方面的讨论.


仿造一个生成器函数
~~~~~~~~~~~~~~~~~~~~~~~~~~

一个 Python 生成器是一个在迭代的时候，使用 :keyword:`yield` 语句返回一系列值的函数或者方法 [#]_.

一个生成器函数或者方法在调用时，返回一个生成器对象. 在这上面可以进行迭代。用于迭代的协议方法
是 :meth:`~container.__iter__` , 因此我们可以使用 :class:`MagicMock` 仿造它.

这里是一个类子，类有一个通过生成器实现的迭代 "iter" 方法:

    >>> class Foo:
    ...     def iter(self):
    ...         for i in [1, 2, 3]:
    ...             yield i
    ...
    >>> foo = Foo()
    >>> list(foo.iter())
    [1, 2, 3]


我们如何仿造这个类, 尤其是它的 "iter" 方法?

为了配置由迭代返回的值 ( 隐式的调用了 :class:`list`), 
我们需要通过调用  ``foo.iter()`` 来配置对象的返回.

    >>> mock_foo = MagicMock()
    >>> mock_foo.iter.return_value = iter([1, 2, 3])
    >>> list(mock_foo.iter())
    [1, 2, 3]

.. [#] 这里有一些生成器表达式和更多 `高级使用
    <http://www.dabeaz.com/coroutines/index.html>`_  生成器使用说明, 在我们这里并没有过多的关注它们.
    一个最好的关于生成器的介绍以及它的用处，可以看: 
    `针对系统编程的生成器技巧 <http://www.dabeaz.com/generators/>`_.


将同一个补丁应用到每个测试方法
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

如果你想要多个补丁应用在不同测试方法，前面的方法是在每个方法上使用一个修饰器。
心想应该有方法可以去掉这种重复的事情。在 Python 2.6 以后版本，你可以使用
:func:`patch` 的另一种不同的形式，作为类修饰器。，这将这些补丁应用到所有的测试
方法上。区分测试方法的手段就是看方法名以 ``test`` 开头:

    >>> @patch('mymodule.SomeClass')
    ... class MyTest(TestCase):
    ...
    ...     def test_one(self, MockSomeClass):
    ...         self.assertIs(mymodule.SomeClass, MockSomeClass)
    ...
    ...     def test_two(self, MockSomeClass):
    ...         self.assertIs(mymodule.SomeClass, MockSomeClass)
    ...
    ...     def not_a_test(self):
    ...         return 'something'
    ...
    >>> MyTest('test_one').test_one()
    >>> MyTest('test_two').test_two()
    >>> MyTest('test_two').not_a_test()
    'something'

一个可选的管理补丁的方式是使用 :ref:`start-and-stop` , 它们允许你在 
``setUp`` 和 ``tearDown`` 方法里移除补丁.

    >>> class MyTest(TestCase):
    ...     def setUp(self):
    ...         self.patcher = patch('mymodule.foo')
    ...         self.mock_foo = self.patcher.start()
    ...
    ...     def test_foo(self):
    ...         self.assertIs(mymodule.foo, self.mock_foo)
    ...
    ...     def tearDown(self):
    ...         self.patcher.stop()
    ...
    >>> MyTest('test_foo').run()

如果你使用这种技术，你必须确保补丁是通过调用 ``stop`` 取消的(undone),
这可能比你预期的要麻烦些，因为如果一个异常在 setUp 抛出，那么 tearDown 将不会被调用。
:meth:`unittest.TestCase.addCleanup` 使其变得简单:

    >>> class MyTest(TestCase):
    ...     def setUp(self):
    ...         patcher = patch('mymodule.foo')
    ...         self.addCleanup(patcher.stop)
    ...         self.mock_foo = patcher.start()
    ...
    ...     def test_foo(self):
    ...         self.assertIs(mymodule.foo, self.mock_foo)
    ...
    >>> MyTest('test_foo').run()


仿造非绑定方法
~~~~~~~~~~~~~~~~~~~~~~~

有时在写测试的时候，我需要给一个 *未绑定的方法* 添加补丁(修补类方法而不是实例方法). 
我需要 self 作为第一个参数传递进去，因为我需要判断那个对象调用了这个特殊的方法. 
这里的问题是，你无法使用 mock 给它打补丁， 因为如果使用了 mock 替换了一个未绑定的方法
当你从实例获取它时，它不会成为一个绑定方法。这样就无法将 self 传递进去了.
正确的做法是使用真实的函数来替换这个未绑定方法。
:func:`patch` 修饰器使用 mock 补出一个方法是非常简单的，而创建一个真实的函数将非常烦人.

如果你给 patch 传递 ``autospec=True`` ，那么将会以一个真实的函数对象作为补丁。
这个方法与它替换的方法有着相同的签名， 但背后确实际上委托给一个 mock . 这样你仍然像前面那样
获得一个自动创建的 mock。它整个过程的意思就是，如果你在类上面给一个未绑定的方法打补丁，
那么你从一个实例中去获取它时，这个 mock 函数会转换成一个绑定的方法。它将传递  ``self``  
进第一个参数，这正式我想要的:

    >>> class Foo:
    ...   def foo(self):
    ...     pass
    ...
    >>> with patch.object(Foo, 'foo', autospec=True) as mock_foo:
    ...   mock_foo.return_value = 'foo'
    ...   foo = Foo()
    ...   foo.foo()
    ...
    'foo'
    >>> mock_foo.assert_called_once_with(foo)

如果你不使用 ``autospec=True`` 那么未绑定方法将补充一个 Mock 实例来替代， 它将会被 ``self`` 调用。


使用 mock 检查多次调用
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

mock 有一个漂亮的,用来给你的仿对象怎么被使用的断言.

    >>> mock = Mock()
    >>> mock.foo_bar.return_value = None
    >>> mock.foo_bar('baz', spam='eggs')
    >>> mock.foo_bar.assert_called_with('baz', spam='eggs')

如果你的 mock 仅被调用一次，你可以使用
:meth:`assert_called_once_with` 方法，也可以用 :attr:`call_count` 等于 1 来断言.

    >>> mock.foo_bar.assert_called_once_with('baz', spam='eggs')
    >>> mock.foo_bar()
    >>> mock.foo_bar.assert_called_once_with('baz', spam='eggs')
    Traceback (most recent call last):
        ...
    AssertionError: Expected to be called once. Called 2 times.

``assert_called_with`` 和 ``assert_called_once_with`` 它两都是以最近的调用来做断言的. 
如果你的 mock 将要被调用多次， 而你又想断言所有的调用, 你可以使用
:attr:`~Mock.call_args_list`:

    >>> mock = Mock(return_value=None)
    >>> mock(1, 2, 3)
    >>> mock(4, 5, 6)
    >>> mock()
    >>> mock.call_args_list
    [call(1, 2, 3), call(4, 5, 6), call()]

:data:`call` 帮助方法使得对这些调用做断言简单。你可以构建一个期望的调用列表来与 
``call_args_list`` 来做比较. 这看起来非常地与 ``call_args_list`` 的 repr 输出相似:

    >>> expected = [call(1, 2, 3), call(4, 5, 6), call()]
    >>> mock.call_args_list == expected
    True


拷贝可变参数
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

另外一种情况很少见，但遇到你会很痛苦，那就是
在 调用 mock 时使用可变参数。 ``call_args`` 和 ``call_args_list`` 存储参数的引用
. 如果在你的测试代码中参数可变，那么你将不能在给 mock 调用的值进行断言了.

下面的例子说明了这个问题. 想象着下面的函数定义在 'mymodule' 中::

    def frob(val):
        pass

    def grob(val):
        "First frob and then clear val"
        frob(val)
        val.clear()

当我们尝试测试 ``grob`` ,它使用正常的参数调用了 ``frob`` ，像下面这样:

    >>> with patch('mymodule.frob') as mock_frob:
    ...     val = {6}
    ...     mymodule.grob(val)
    ...
    >>> val
    set()
    >>> mock_frob.assert_called_with({6})
    Traceback (most recent call last):
        ...
    AssertionError: Expected: (({6},), {})
    Called with: ((set(),), {})

一种可能是，mock 拷贝传递进去的参数. 它可能导致一个问题，如果你想的断言依赖于对象标识.

这里有一种解决方法是使用 :attr:`side_effect` 函数. 如果你给 mock 提供了一个 ``side_effect`` 
方法， ``side_effect`` 将会作为 mock 调用同样的参数. 这就给我们拷贝这些参数提供了机会，并为后面的
断言进行储存。在这个例子中，我使用另外一个 mock 存储这些参数，这样我可以使用 mock 的方法来
进行断言。 同样的我给自己设置了一个帮助函数.

    >>> from copy import deepcopy
    >>> from unittest.mock import Mock, patch, DEFAULT
    >>> def copy_call_args(mock):
    ...     new_mock = Mock()
    ...     def side_effect(*args, **kwargs):
    ...         args = deepcopy(args)
    ...         kwargs = deepcopy(kwargs)
    ...         new_mock(*args, **kwargs)
    ...         return DEFAULT
    ...     mock.side_effect = side_effect
    ...     return new_mock
    ...
    >>> with patch('mymodule.frob') as mock_frob:
    ...     new_mock = copy_call_args(mock_frob)
    ...     val = {6}
    ...     mymodule.grob(val)
    ...
    >>> new_mock.assert_called_with({6})
    >>> new_mock.call_args
    call({6})

``copy_call_args`` 被 mock 调用，它将会被调用到. 它返回一个新的 mock 用来做断言. 
``side_effect`` 函数为我们的参数做了拷贝，然后使用拷贝调用我们的 ``new_mock`` .

.. note::

    如果你的 mock 将仅仅被使用一次，这个时候有一种简单的方法来检查它们调用参数。
    你可以简单的在  ``side_effect``  函数中来做检查.

        >>> def side_effect(arg):
        ...     assert arg == {6}
        ...
        >>> mock = Mock(side_effect=side_effect)
        >>> mock({6})
        >>> mock(set())
        Traceback (most recent call last):
            ...
        AssertionError

另一种可选的途径是创建一个 :class:`Mock` 或者
:class:`MagicMock` 子类，它们通过 :func:`copy.deepcopy` 拷贝这些参数.
这里有一个实现的例子:

    >>> from copy import deepcopy
    >>> class CopyingMock(MagicMock):
    ...     def __call__(self, *args, **kwargs):
    ...         args = deepcopy(args)
    ...         kwargs = deepcopy(kwargs)
    ...         return super(CopyingMock, self).__call__(*args, **kwargs)
    ...
    >>> c = CopyingMock(return_value=None)
    >>> arg = set()
    >>> c(arg)
    >>> arg.add(1)
    >>> c.assert_called_with(set())
    >>> c.assert_called_with(arg)
    Traceback (most recent call last):
        ...
    AssertionError: Expected call: mock({1})
    Actual call: mock(set())
    >>> c.foo
    <CopyingMock name='mock.foo' id='...'>

当你有 ``Mock`` 或者 ``MagicMock`` 的子类时，它们的属性是自动创建的，并且  ``return_value`` 
会自动化应用在你的子类上。这就意味着所有 ``CopyingMock`` 的子类也会有 ``CopyingMock`` 的类型.


嵌套修补
~~~~~~~~~~~~~~~

将 patch 作为一个上下文使用是非常不错的, 这样如果你想用多个补丁时,
你可以在接下来的嵌套语句中使用多个这样的语句:

    >>> class MyTest(TestCase):
    ...
    ...     def test_foo(self):
    ...         with patch('mymodule.Foo') as mock_foo:
    ...             with patch('mymodule.Bar') as mock_bar:
    ...                 with patch('mymodule.Spam') as mock_spam:
    ...                     assert mymodule.Foo is mock_foo
    ...                     assert mymodule.Bar is mock_bar
    ...                     assert mymodule.Spam is mock_spam
    ...
    >>> original = mymodule.Foo
    >>> MyTest('test_foo').test_foo()
    >>> assert mymodule.Foo is original

使用 unittest ``cleanup`` 函数 和 :ref:`start-and-stop` 我们可以获得同样的效果，而不需要嵌套缩进。
一个简单的帮助方法 ``create_patch``, 将这个 patch 放到正确的位置上，然后
它将为我们返回创建的 mock :

    >>> class MyTest(TestCase):
    ...
    ...     def create_patch(self, name):
    ...         patcher = patch(name)
    ...         thing = patcher.start()
    ...         self.addCleanup(patcher.stop)
    ...         return thing
    ...
    ...     def test_foo(self):
    ...         mock_foo = self.create_patch('mymodule.Foo')
    ...         mock_bar = self.create_patch('mymodule.Bar')
    ...         mock_spam = self.create_patch('mymodule.Spam')
    ...
    ...         assert mymodule.Foo is mock_foo
    ...         assert mymodule.Bar is mock_bar
    ...         assert mymodule.Spam is mock_spam
    ...
    >>> original = mymodule.Foo
    >>> MyTest('test_foo').run()
    >>> assert mymodule.Foo is original


使用 MagicMock 仿造字典
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

你可能想要仿造一个字典，或者其他容器对象, 记录所有访问的同时也保持这字典的行为.

我们可以使用 :class:`MagicMock` 来做这个事, 它的行为像字典，然后使用 :data:`~Mock.side_effect` 
来代理字典访问一个真实的潜在的字典，它是在我们控制之下的.

当我们的 ``MagicMock``  :meth:`__getitem__` 和 :meth:`__setitem__` 方法被调用时(正常的字典访问)
那么 ``side_effect`` 作为 key 被调用 ( 在这种情况下，使用 ``__setitem__`` 设置值). 
我们还可以控制它的返回.

在使用 ``MagicMock`` 之后，我们可以使用像
:data:`~Mock.call_args_list` 这样的属性来断言字典是如何被使用的:

    >>> my_dict = {'a': 1, 'b': 2, 'c': 3}
    >>> def getitem(name):
    ...      return my_dict[name]
    ...
    >>> def setitem(name, val):
    ...     my_dict[name] = val
    ...
    >>> mock = MagicMock()
    >>> mock.__getitem__.side_effect = getitem
    >>> mock.__setitem__.side_effect = setitem

.. note::

    另一种可选是不使用 ``MagicMock`` 而使用 ``Mock`` , 这样将仅提供你想要的魔法方法:

        >>> mock = Mock()
        >>> mock.__getitem__ = Mock(side_effect=getitem)
        >>> mock.__setitem__ = Mock(side_effect=setitem)

    第三个选择是使用 ``MagicMock`` 但是将 ``dict`` 作为 *spec*
    或者 *spec_set* 参数传递进去，这样 ``MagicMock`` 的创建仅仅只有字典的魔法方法可以访问:

        >>> mock = MagicMock(spec_set=dict)
        >>> mock.__getitem__.side_effect = getitem
        >>> mock.__setitem__.side_effect = setitem

放置这些边界效应的函数到正确位置之后, ``mock``  的行为将会像普通的字典那样记录访问. 
如果你访问一个不存在的 key ，它也会抛出 :exc:`KeyError` .

    >>> mock['a']
    1
    >>> mock['c']
    3
    >>> mock['d']
    Traceback (most recent call last):
        ...
    KeyError: 'd'
    >>> mock['b'] = 'fish'
    >>> mock['d'] = 'eggs'
    >>> mock['b']
    'fish'
    >>> mock['d']
    'eggs'

使用它之后，你可以使用正常的方法和属性对其进行断言:

    >>> mock.__getitem__.call_args_list
    [call('a'), call('c'), call('d'), call('b'), call('d')]
    >>> mock.__setitem__.call_args_list
    [call('b', 'fish'), call('d', 'eggs')]
    >>> my_dict
    {'a': 1, 'c': 3, 'b': 'fish', 'd': 'eggs'}


仿造子类和它们的属性
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

有各种不同的理由使用你想要用 :class:`Mock` 子类. 一个理由就是增加一个帮助方法. 
这里有个简单的例子:

    >>> class MyMock(MagicMock):
    ...     def has_been_called(self):
    ...         return self.called
    ...
    >>> mymock = MyMock(return_value=None)
    >>> mymock
    <MyMock id='...'>
    >>> mymock.has_been_called()
    False
    >>> mymock()
    >>> mymock.has_been_called()
    True

所有 ``Mock`` 实例的标准行为，如 mock 的属性和返回值都与访问 mock 时一样。这样
就确保了 ``Mock`` 属性都是 ``Mocks`` 的并且 ``MagicMock`` 属性都是 ``MagicMocks``
[#]_ 因此如果你在子类中增加了一个帮助方法，
那么它们在你的的子类可以作为属性或者返回值使用.

    >>> mymock.foo
    <MyMock name='mock.foo' id='...'>
    >>> mymock.foo.has_been_called()
    False
    >>> mymock.foo()
    <MyMock name='mock.foo()' id='...'>
    >>> mymock.foo.has_been_called()
    True

有时这是非常不方便的, `一个用户 <https://code.google.com/p/mock/issues/detail?id=105>`_ 
是一个 mock 子类用来创建一个 `Twisted 适配器 <http://twistedmatrix.com/documents/11.0.0/api/twisted.python.components.html>`_.
在应用这个属性时将导致一个错误发生.

``Mock`` (in all its flavours) uses a method called ``_get_child_mock`` to create
these "sub-mocks" for attributes and return values. You can prevent your
subclass being used for attributes by overriding this method. The signature is
that it takes arbitrary keyword arguments (``**kwargs``) which are then passed
onto the mock constructor:

    >>> class Subclass(MagicMock):
    ...     def _get_child_mock(self, **kwargs):
    ...         return MagicMock(**kwargs)
    ...
    >>> mymock = Subclass()
    >>> mymock.foo
    <MagicMock name='mock.foo' id='...'>
    >>> assert isinstance(mymock, Subclass)
    >>> assert not isinstance(mymock.foo, Subclass)
    >>> assert not isinstance(mymock(), Subclass)

.. [#] An exception to this rule are the non-callable mocks. Attributes use the
    callable variant because otherwise non-callable mocks couldn't have callable
    methods.


使用 patch.dict 仿造导入
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

One situation where mocking can be hard is where you have a local import inside
a function. These are harder to mock because they aren't using an object from
the module namespace that we can patch out.

Generally local imports are to be avoided. They are sometimes done to prevent
circular dependencies, for which there is *usually* a much better way to solve
the problem (refactor the code) or to prevent "up front costs" by delaying the
import. This can also be solved in better ways than an unconditional local
import (store the module as a class or module attribute and only do the import
on first use).

That aside there is a way to use ``mock`` to affect the results of an import.
Importing fetches an *object* from the :data:`sys.modules` dictionary. Note that it
fetches an *object*, which need not be a module. Importing a module for the
first time results in a module object being put in `sys.modules`, so usually
when you import something you get a module back. This need not be the case
however.

This means you can use :func:`patch.dict` to *temporarily* put a mock in place
in :data:`sys.modules`. Any imports whilst this patch is active will fetch the mock.
When the patch is complete (the decorated function exits, the with statement
body is complete or ``patcher.stop()`` is called) then whatever was there
previously will be restored safely.

Here's an example that mocks out the 'fooble' module.

    >>> mock = Mock()
    >>> with patch.dict('sys.modules', {'fooble': mock}):
    ...    import fooble
    ...    fooble.blob()
    ...
    <Mock name='mock.blob()' id='...'>
    >>> assert 'fooble' not in sys.modules
    >>> mock.blob.assert_called_once_with()

As you can see the ``import fooble`` succeeds, but on exit there is no 'fooble'
left in :data:`sys.modules`.

This also works for the ``from module import name`` form:

    >>> mock = Mock()
    >>> with patch.dict('sys.modules', {'fooble': mock}):
    ...    from fooble import blob
    ...    blob.blip()
    ...
    <Mock name='mock.blob.blip()' id='...'>
    >>> mock.blob.blip.assert_called_once_with()

With slightly more work you can also mock package imports:

    >>> mock = Mock()
    >>> modules = {'package': mock, 'package.module': mock.module}
    >>> with patch.dict('sys.modules', modules):
    ...    from package.module import fooble
    ...    fooble()
    ...
    <Mock name='mock.module.fooble()' id='...'>
    >>> mock.module.fooble.assert_called_once_with()


跟踪调用顺序和更少的断言输出
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The :class:`Mock` class allows you to track the *order* of method calls on
your mock objects through the :attr:`~Mock.method_calls` attribute. This
doesn't allow you to track the order of calls between separate mock objects,
however we can use :attr:`~Mock.mock_calls` to achieve the same effect.

Because mocks track calls to child mocks in ``mock_calls``, and accessing an
arbitrary attribute of a mock creates a child mock, we can create our separate
mocks from a parent one. Calls to those child mock will then all be recorded,
in order, in the ``mock_calls`` of the parent:

    >>> manager = Mock()
    >>> mock_foo = manager.foo
    >>> mock_bar = manager.bar

    >>> mock_foo.something()
    <Mock name='mock.foo.something()' id='...'>
    >>> mock_bar.other.thing()
    <Mock name='mock.bar.other.thing()' id='...'>

    >>> manager.mock_calls
    [call.foo.something(), call.bar.other.thing()]

We can then assert about the calls, including the order, by comparing with
the ``mock_calls`` attribute on the manager mock:

    >>> expected_calls = [call.foo.something(), call.bar.other.thing()]
    >>> manager.mock_calls == expected_calls
    True

If ``patch`` is creating, and putting in place, your mocks then you can attach
them to a manager mock using the :meth:`~Mock.attach_mock` method. After
attaching calls will be recorded in ``mock_calls`` of the manager.

    >>> manager = MagicMock()
    >>> with patch('mymodule.Class1') as MockClass1:
    ...     with patch('mymodule.Class2') as MockClass2:
    ...         manager.attach_mock(MockClass1, 'MockClass1')
    ...         manager.attach_mock(MockClass2, 'MockClass2')
    ...         MockClass1().foo()
    ...         MockClass2().bar()
    ...
    <MagicMock name='mock.MockClass1().foo()' id='...'>
    <MagicMock name='mock.MockClass2().bar()' id='...'>
    >>> manager.mock_calls
    [call.MockClass1(),
     call.MockClass1().foo(),
     call.MockClass2(),
     call.MockClass2().bar()]

If many calls have been made, but you're only interested in a particular
sequence of them then an alternative is to use the
:meth:`~Mock.assert_has_calls` method. This takes a list of calls (constructed
with the :data:`call` object). If that sequence of calls are in
:attr:`~Mock.mock_calls` then the assert succeeds.

    >>> m = MagicMock()
    >>> m().foo().bar().baz()
    <MagicMock name='mock().foo().bar().baz()' id='...'>
    >>> m.one().two().three()
    <MagicMock name='mock.one().two().three()' id='...'>
    >>> calls = call.one().two().three().call_list()
    >>> m.assert_has_calls(calls)

Even though the chained call ``m.one().two().three()`` aren't the only calls that
have been made to the mock, the assert still succeeds.

Sometimes a mock may have several calls made to it, and you are only interested
in asserting about *some* of those calls. You may not even care about the
order. In this case you can pass ``any_order=True`` to ``assert_has_calls``:

    >>> m = MagicMock()
    >>> m(1), m.two(2, 3), m.seven(7), m.fifty('50')
    (...)
    >>> calls = [call.fifty('50'), call(1), call.seven(7)]
    >>> m.assert_has_calls(calls, any_order=True)


匹配更加复杂的参数
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Using the same basic concept as :data:`ANY` we can implement matchers to do more
complex assertions on objects used as arguments to mocks.

Suppose we expect some object to be passed to a mock that by default
compares equal based on object identity (which is the Python default for user
defined classes). To use :meth:`~Mock.assert_called_with` we would need to pass
in the exact same object. If we are only interested in some of the attributes
of this object then we can create a matcher that will check these attributes
for us.

You can see in this example how a 'standard' call to ``assert_called_with`` isn't
sufficient:

    >>> class Foo:
    ...     def __init__(self, a, b):
    ...         self.a, self.b = a, b
    ...
    >>> mock = Mock(return_value=None)
    >>> mock(Foo(1, 2))
    >>> mock.assert_called_with(Foo(1, 2))
    Traceback (most recent call last):
        ...
    AssertionError: Expected: call(<__main__.Foo object at 0x...>)
    Actual call: call(<__main__.Foo object at 0x...>)

A comparison function for our ``Foo`` class might look something like this:

    >>> def compare(self, other):
    ...     if not type(self) == type(other):
    ...         return False
    ...     if self.a != other.a:
    ...         return False
    ...     if self.b != other.b:
    ...         return False
    ...     return True
    ...

And a matcher object that can use comparison functions like this for its
equality operation would look something like this:

    >>> class Matcher:
    ...     def __init__(self, compare, some_obj):
    ...         self.compare = compare
    ...         self.some_obj = some_obj
    ...     def __eq__(self, other):
    ...         return self.compare(self.some_obj, other)
    ...

Putting all this together:

    >>> match_foo = Matcher(compare, Foo(1, 2))
    >>> mock.assert_called_with(match_foo)

The ``Matcher`` is instantiated with our compare function and the ``Foo`` object
we want to compare against. In ``assert_called_with`` the ``Matcher`` equality
method will be called, which compares the object the mock was called with
against the one we created our matcher with. If they match then
``assert_called_with`` passes, and if they don't an :exc:`AssertionError` is raised:

    >>> match_wrong = Matcher(compare, Foo(3, 4))
    >>> mock.assert_called_with(match_wrong)
    Traceback (most recent call last):
        ...
    AssertionError: Expected: ((<Matcher object at 0x...>,), {})
    Called with: ((<Foo object at 0x...>,), {})

With a bit of tweaking you could have the comparison function raise the
:exc:`AssertionError` directly and provide a more useful failure message.

As of version 1.5, the Python testing library `PyHamcrest
<https://pypi.python.org/pypi/PyHamcrest>`_ provides similar functionality,
that may be useful here, in the form of its equality matcher
(`hamcrest.library.integration.match_equality
<http://pythonhosted.org/PyHamcrest/integration.html#hamcrest.library.integration.match_equality.match_equality>`_).
