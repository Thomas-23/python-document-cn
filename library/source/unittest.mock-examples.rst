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

我们在给我们的 mock 提供 'close' 方法上没有做任何事情。 访问 close 时就就创建了它。 因此在测试中如果 'close' 
还没有被调用，就去访问它 ，会创建它， 但是 :meth:`~Mock.assert_called_with` 将会抛出一个失败的异常。


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

我们可以使用 :data:`call` ，在 "chained call" 中构造一个调用集合， 然后后来就可以像下面这样轻松断言了:

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
可以将 ``side_effect`` 作为一个迭代器。当你作为迭代器上面设置了 ``side_effect`` 
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

The module name can be 'dotted', in the form ``package.module`` if needed:

    >>> @patch('package.module.ClassName.attribute', sentinel.attribute)
    ... def test():
    ...     from package.module import ClassName
    ...     assert ClassName.attribute == sentinel.attribute
    ...
    >>> test()

A nice pattern is to actually decorate test methods themselves:

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
这也就意味着是自底向上的，所以在上面的这个例子中 ``test_module.ClassName2`` 仿类将首先被传递进修饰方法。

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
如果你使用这种方式，它等同于你给每个以  "test" 方法上面单独的使用了这个修饰符.


.. _further-examples:

深入的例子
----------------


这里有一些更高级场景的例子。


仿造链式调用
~~~~~~~~~~~~~~~~~~~~~

Mocking chained calls is actually straightforward with mock once you
understand the :attr:`~Mock.return_value` attribute. When a mock is called for
the first time, or you fetch its ``return_value`` before it has been called, a
new :class:`Mock` is created.

This means that you can see how the object returned from a call to a mocked
object has been used by interrogating the ``return_value`` mock:

    >>> mock = Mock()
    >>> mock().foo(a=2, b=3)
    <Mock name='mock().foo()' id='...'>
    >>> mock.return_value.foo.assert_called_with(a=2, b=3)

From here it is a simple step to configure and then make assertions about
chained calls. Of course another alternative is writing your code in a more
testable way in the first place...

So, suppose we have some code that looks a little bit like this:

    >>> class Something:
    ...     def __init__(self):
    ...         self.backend = BackendProvider()
    ...     def method(self):
    ...         response = self.backend.get_endpoint('foobar').create_call('spam', 'eggs').start_call()
    ...         # more code

Assuming that ``BackendProvider`` is already well tested, how do we test
``method()``? Specifically, we want to test that the code section ``# more
code`` uses the response object in the correct way.

As this chain of calls is made from an instance attribute we can monkey patch
the ``backend`` attribute on a ``Something`` instance. In this particular case
we are only interested in the return value from the final call to
``start_call`` so we don't have much configuration to do. Let's assume the
object it returns is 'file-like', so we'll ensure that our response object
uses the builtin :func:`open` as its ``spec``.

To do this we create a mock instance as our mock backend and create a mock
response object for it. To set the response as the return value for that final
``start_call`` we could do this::

    mock_backend.get_endpoint.return_value.create_call.return_value.start_call.return_value = mock_response

We can do that in a slightly nicer way using the :meth:`~Mock.configure_mock`
method to directly set the return value for us:

    >>> something = Something()
    >>> mock_response = Mock(spec=open)
    >>> mock_backend = Mock()
    >>> config = {'get_endpoint.return_value.create_call.return_value.start_call.return_value': mock_response}
    >>> mock_backend.configure_mock(**config)

With these we monkey patch the "mock backend" in place and can make the real
call:

    >>> something.backend = mock_backend
    >>> something.method()

Using :attr:`~Mock.mock_calls` we can check the chained call with a single
assert. A chained call is several calls in one line of code, so there will be
several entries in ``mock_calls``. We can use :meth:`call.call_list` to create
this list of calls for us:

    >>> chained = call.get_endpoint('foobar').create_call('spam', 'eggs').start_call()
    >>> call_list = chained.call_list()
    >>> assert mock_backend.mock_calls == call_list


Partial mocking
~~~~~~~~~~~~~~~

In some tests I wanted to mock out a call to :meth:`datetime.date.today`
to return a known date, but I didn't want to prevent the code under test from
creating new date objects. Unfortunately :class:`datetime.date` is written in C, and
so I couldn't just monkey-patch out the static :meth:`date.today` method.

I found a simple way of doing this that involved effectively wrapping the date
class with a mock, but passing through calls to the constructor to the real
class (and returning real instances).

The :func:`patch decorator <patch>` is used here to
mock out the ``date`` class in the module under test. The :attr:`side_effect`
attribute on the mock date class is then set to a lambda function that returns
a real date. When the mock date class is called a real date will be
constructed and returned by ``side_effect``.

    >>> from datetime import date
    >>> with patch('mymodule.date') as mock_date:
    ...     mock_date.today.return_value = date(2010, 10, 8)
    ...     mock_date.side_effect = lambda *args, **kw: date(*args, **kw)
    ...
    ...     assert mymodule.date.today() == date(2010, 10, 8)
    ...     assert mymodule.date(2009, 6, 8) == date(2009, 6, 8)
    ...

Note that we don't patch :class:`datetime.date` globally, we patch ``date`` in the
module that *uses* it. See :ref:`where to patch <where-to-patch>`.

When ``date.today()`` is called a known date is returned, but calls to the
``date(...)`` constructor still return normal dates. Without this you can find
yourself having to calculate an expected result using exactly the same
algorithm as the code under test, which is a classic testing anti-pattern.

Calls to the date constructor are recorded in the ``mock_date`` attributes
(``call_count`` and friends) which may also be useful for your tests.

An alternative way of dealing with mocking dates, or other builtin classes,
is discussed in `this blog entry
<http://www.williamjohnbert.com/2011/07/how-to-unit-testing-in-django-with-mocking-and-patching/>`_.


Mocking a Generator Method
~~~~~~~~~~~~~~~~~~~~~~~~~~

A Python generator is a function or method that uses the :keyword:`yield` statement
to return a series of values when iterated over [#]_.

A generator method / function is called to return the generator object. It is
the generator object that is then iterated over. The protocol method for
iteration is :meth:`~container.__iter__`, so we can
mock this using a :class:`MagicMock`.

Here's an example class with an "iter" method implemented as a generator:

    >>> class Foo:
    ...     def iter(self):
    ...         for i in [1, 2, 3]:
    ...             yield i
    ...
    >>> foo = Foo()
    >>> list(foo.iter())
    [1, 2, 3]


How would we mock this class, and in particular its "iter" method?

To configure the values returned from the iteration (implicit in the call to
:class:`list`), we need to configure the object returned by the call to ``foo.iter()``.

    >>> mock_foo = MagicMock()
    >>> mock_foo.iter.return_value = iter([1, 2, 3])
    >>> list(mock_foo.iter())
    [1, 2, 3]

.. [#] There are also generator expressions and more `advanced uses
    <http://www.dabeaz.com/coroutines/index.html>`_ of generators, but we aren't
    concerned about them here. A very good introduction to generators and how
    powerful they are is: `Generator Tricks for Systems Programmers
    <http://www.dabeaz.com/generators/>`_.


Applying the same patch to every test method
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you want several patches in place for multiple test methods the obvious way
is to apply the patch decorators to every method. This can feel like unnecessary
repetition. For Python 2.6 or more recent you can use :func:`patch` (in all its
various forms) as a class decorator. This applies the patches to all test
methods on the class. A test method is identified by methods whose names start
with ``test``:

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

An alternative way of managing patches is to use the :ref:`start-and-stop`.
These allow you to move the patching into your ``setUp`` and ``tearDown`` methods.

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

If you use this technique you must ensure that the patching is "undone" by
calling ``stop``. This can be fiddlier than you might think, because if an
exception is raised in the setUp then tearDown is not called.
:meth:`unittest.TestCase.addCleanup` makes this easier:

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


Mocking Unbound Methods
~~~~~~~~~~~~~~~~~~~~~~~

Whilst writing tests today I needed to patch an *unbound method* (patching the
method on the class rather than on the instance). I needed self to be passed
in as the first argument because I want to make asserts about which objects
were calling this particular method. The issue is that you can't patch with a
mock for this, because if you replace an unbound method with a mock it doesn't
become a bound method when fetched from the instance, and so it doesn't get
self passed in. The workaround is to patch the unbound method with a real
function instead. The :func:`patch` decorator makes it so simple to
patch out methods with a mock that having to create a real function becomes a
nuisance.

If you pass ``autospec=True`` to patch then it does the patching with a
*real* function object. This function object has the same signature as the one
it is replacing, but delegates to a mock under the hood. You still get your
mock auto-created in exactly the same way as before. What it means though, is
that if you use it to patch out an unbound method on a class the mocked
function will be turned into a bound method if it is fetched from an instance.
It will have ``self`` passed in as the first argument, which is exactly what I
wanted:

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

If we don't use ``autospec=True`` then the unbound method is patched out
with a Mock instance instead, and isn't called with ``self``.


Checking multiple calls with mock
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

mock has a nice API for making assertions about how your mock objects are used.

    >>> mock = Mock()
    >>> mock.foo_bar.return_value = None
    >>> mock.foo_bar('baz', spam='eggs')
    >>> mock.foo_bar.assert_called_with('baz', spam='eggs')

If your mock is only being called once you can use the
:meth:`assert_called_once_with` method that also asserts that the
:attr:`call_count` is one.

    >>> mock.foo_bar.assert_called_once_with('baz', spam='eggs')
    >>> mock.foo_bar()
    >>> mock.foo_bar.assert_called_once_with('baz', spam='eggs')
    Traceback (most recent call last):
        ...
    AssertionError: Expected to be called once. Called 2 times.

Both ``assert_called_with`` and ``assert_called_once_with`` make assertions about
the *most recent* call. If your mock is going to be called several times, and
you want to make assertions about *all* those calls you can use
:attr:`~Mock.call_args_list`:

    >>> mock = Mock(return_value=None)
    >>> mock(1, 2, 3)
    >>> mock(4, 5, 6)
    >>> mock()
    >>> mock.call_args_list
    [call(1, 2, 3), call(4, 5, 6), call()]

The :data:`call` helper makes it easy to make assertions about these calls. You
can build up a list of expected calls and compare it to ``call_args_list``. This
looks remarkably similar to the repr of the ``call_args_list``:

    >>> expected = [call(1, 2, 3), call(4, 5, 6), call()]
    >>> mock.call_args_list == expected
    True


Coping with mutable arguments
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Another situation is rare, but can bite you, is when your mock is called with
mutable arguments. ``call_args`` and ``call_args_list`` store *references* to the
arguments. If the arguments are mutated by the code under test then you can no
longer make assertions about what the values were when the mock was called.

Here's some example code that shows the problem. Imagine the following functions
defined in 'mymodule'::

    def frob(val):
        pass

    def grob(val):
        "First frob and then clear val"
        frob(val)
        val.clear()

When we try to test that ``grob`` calls ``frob`` with the correct argument look
what happens:

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

One possibility would be for mock to copy the arguments you pass in. This
could then cause problems if you do assertions that rely on object identity
for equality.

Here's one solution that uses the :attr:`side_effect`
functionality. If you provide a ``side_effect`` function for a mock then
``side_effect`` will be called with the same args as the mock. This gives us an
opportunity to copy the arguments and store them for later assertions. In this
example I'm using *another* mock to store the arguments so that I can use the
mock methods for doing the assertion. Again a helper function sets this up for
me.

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

``copy_call_args`` is called with the mock that will be called. It returns a new
mock that we do the assertion on. The ``side_effect`` function makes a copy of
the args and calls our ``new_mock`` with the copy.

.. note::

    If your mock is only going to be used once there is an easier way of
    checking arguments at the point they are called. You can simply do the
    checking inside a ``side_effect`` function.

        >>> def side_effect(arg):
        ...     assert arg == {6}
        ...
        >>> mock = Mock(side_effect=side_effect)
        >>> mock({6})
        >>> mock(set())
        Traceback (most recent call last):
            ...
        AssertionError

An alternative approach is to create a subclass of :class:`Mock` or
:class:`MagicMock` that copies (using :func:`copy.deepcopy`) the arguments.
Here's an example implementation:

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

When you subclass ``Mock`` or ``MagicMock`` all dynamically created attributes,
and the ``return_value`` will use your subclass automatically. That means all
children of a ``CopyingMock`` will also have the type ``CopyingMock``.


Nesting Patches
~~~~~~~~~~~~~~~

Using patch as a context manager is nice, but if you do multiple patches you
can end up with nested with statements indenting further and further to the
right:

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

With unittest ``cleanup`` functions and the :ref:`start-and-stop` we can
achieve the same effect without the nested indentation. A simple helper
method, ``create_patch``, puts the patch in place and returns the created mock
for us:

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


Mocking a dictionary with MagicMock
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You may want to mock a dictionary, or other container object, recording all
access to it whilst having it still behave like a dictionary.

We can do this with :class:`MagicMock`, which will behave like a dictionary,
and using :data:`~Mock.side_effect` to delegate dictionary access to a real
underlying dictionary that is under our control.

When the :meth:`__getitem__` and :meth:`__setitem__` methods of our ``MagicMock`` are called
(normal dictionary access) then ``side_effect`` is called with the key (and in
the case of ``__setitem__`` the value too). We can also control what is returned.

After the ``MagicMock`` has been used we can use attributes like
:data:`~Mock.call_args_list` to assert about how the dictionary was used:

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

    An alternative to using ``MagicMock`` is to use ``Mock`` and *only* provide
    the magic methods you specifically want:

        >>> mock = Mock()
        >>> mock.__getitem__ = Mock(side_effect=getitem)
        >>> mock.__setitem__ = Mock(side_effect=setitem)

    A *third* option is to use ``MagicMock`` but passing in ``dict`` as the *spec*
    (or *spec_set*) argument so that the ``MagicMock`` created only has
    dictionary magic methods available:

        >>> mock = MagicMock(spec_set=dict)
        >>> mock.__getitem__.side_effect = getitem
        >>> mock.__setitem__.side_effect = setitem

With these side effect functions in place, the ``mock`` will behave like a normal
dictionary but recording the access. It even raises a :exc:`KeyError` if you try
to access a key that doesn't exist.

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

After it has been used you can make assertions about the access using the normal
mock methods and attributes:

    >>> mock.__getitem__.call_args_list
    [call('a'), call('c'), call('d'), call('b'), call('d')]
    >>> mock.__setitem__.call_args_list
    [call('b', 'fish'), call('d', 'eggs')]
    >>> my_dict
    {'a': 1, 'c': 3, 'b': 'fish', 'd': 'eggs'}


Mock subclasses and their attributes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

There are various reasons why you might want to subclass :class:`Mock`. One
reason might be to add helper methods. Here's a silly example:

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

The standard behaviour for ``Mock`` instances is that attributes and the return
value mocks are of the same type as the mock they are accessed on. This ensures
that ``Mock`` attributes are ``Mocks`` and ``MagicMock`` attributes are ``MagicMocks``
[#]_. So if you're subclassing to add helper methods then they'll also be
available on the attributes and return value mock of instances of your
subclass.

    >>> mymock.foo
    <MyMock name='mock.foo' id='...'>
    >>> mymock.foo.has_been_called()
    False
    >>> mymock.foo()
    <MyMock name='mock.foo()' id='...'>
    >>> mymock.foo.has_been_called()
    True

Sometimes this is inconvenient. For example, `one user
<https://code.google.com/p/mock/issues/detail?id=105>`_ is subclassing mock to
created a `Twisted adaptor
<http://twistedmatrix.com/documents/11.0.0/api/twisted.python.components.html>`_.
Having this applied to attributes too actually causes errors.

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


Mocking imports with patch.dict
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


Tracking order of calls and less verbose call assertions
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


More complex argument matching
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
