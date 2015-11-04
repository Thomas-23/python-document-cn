
.. _tut-venv:

*********************************
虚拟环境和包
*********************************

介绍
============

在 Python 应用程序中经常会用到非标准库的模块和包（译者注：即其他人开发的程序）
。应用程序有时需要某个类库的特殊版本，因为应用程序可能需要某个指定的bug被修复了的类库
或者这个应用程序可能使用了老的版本的类库接口

这就意味着仅安装一个 Python 将不能满足每个应用程序的需求， 如果应用程序A需要某个指定模块的 1.0 版本， 
但是应用程序B则需要这个模块的 2.0 版本， 那么这个需求存在冲突，
因为无论安装版本 1.0 还是 2.0 都将导致一个应用程序无法正常运行。

解决这个问题的办法是创建一个 :term:`虚拟环境` （英文简称 "virtualenv"），
它是一个自包含的目录，里面有安装指定版本的 Python ，外加一些额外的包 （译者注：在虚拟环境下安装的其他非标准库）

不同的应用程序可以使用不同的虚拟环境。要解决先前例子中的冲突需求，应用程序 A 可以有它自己的安装了版本 1.0 的虚拟环境
而应用程序 B 可以有另外一个版本 2.0 的虚拟环境。如果应用程序B需要一个类库升级到版本 3.0 ，那么这个升级过程将不会影响应用程序
A的环境。


创建虚拟环境
=============================

被用来创建和管理虚拟环境的脚本叫做 :program:`pyvenv` 。 :program:`pyvenv` 脚本通常会安装在你可以使用的最近版本的 Python 中；
这个脚本安装完也带有版本号，因此如果在你的系统中有多个版本的 Python ， 你可以通过执行 ``pyvenv-3.4`` 或者
你希望的其他版本的 :program:`pyvenv` 来选择一个特殊版本的 Python 。

要想创建一个虚拟环境，需要在你想放置虚拟环境的地方即某个目录下，执行 :program:`pyvenv` ，后面跟上目录路径::

   pyvenv tutorial-env

如果 ``tutorial-env`` 目录不存在， 这样就会创建这个目录，
并且在这个目录下面生成一份拷贝的 Python 解释器，标准库和各种支持文件。

一旦你创建了一个虚拟环境，你需要激活它。

在 Windows 上， 运行::

  tutorial-env/Scripts/activate

在 Unix  和 MacOS 上， 运行::

  source tutorial-env/bin/activate

（这个脚本是基于 bash shell 写的。 如果你使用的是 :program:`csh` 或者 :program:`fish` shell ，
你需要使用 ``activate.csh`` 或 ``activate.fish`` 来代替前面命令中的 ``activate`` ） 

激活了虚拟环境后， 你将会看到 shell 的提示符显示当前你所应用的虚拟环境，并且在你运行 ``python`` 时，
当前环境将会变成你安装的指定版本 Python 。例如::

  -> source ~/envs/tutorial-env/bin/activate
  (tutorial-env) -> python
  Python 3.4.3+ (3.4:c7b9645a6f35+, May 22 2015, 09:31:25)
    ...
  >>> import sys
  >>> sys.path
  ['', '/usr/local/lib/python34.zip', ...,
  '~/envs/tutorial-env/lib/python3.4/site-packages']
  >>>


使用 pip 管理包
==========================

一旦你已经激活了一个虚拟环境， 你可以通过 :program:`pip` 程序安装，升级和移除包。默认 ``pip`` 会
从 Python 官方类库 <https://pypi.python.org/pypi> 中安装包， 你可以通过浏览器浏览 Python 官方类库，
或者你可以使用 ``pip`` 的定制搜索功能::

  (tutorial-env) -> pip search astronomy
  skyfield               - Elegant astronomy for Python
  gary                   - Galactic astronomy and gravitational dynamics.
  novas                  - The United States Naval Observatory NOVAS astronomy library
  astroobs               - Provides astronomy ephemeris to plan telescope observations
  PyAstronomy            - A collection of astronomy related tools for Python.
  ...

``pip`` 有一些子命令: "search"，"install"，"uninstall"，"freeze"，等。  
（详情，请查看完整的 ``pip`` 手册 `安装 Python 模块`_ 。）

你可以指定一个包的名字来安装最近版本包::

  -> pip install novas
  Collecting novas
    Downloading novas-3.1.1.3.tar.gz (136kB)
  Installing collected packages: novas
    Running setup.py install for novas
  Successfully installed novas-3.1.1.3

你可以安装一个指定版本的包，只要在 ``==`` 后面加上包名的版本号::

  -> pip install requests==2.6.0
  Collecting requests==2.6.0
    Using cached requests-2.6.0-py2.py3-none-any.whl
  Installing collected packages: requests
  Successfully installed requests-2.6.0

如果继续运行上面的命令，将不执行任何操作， ``pip`` 会提示你 request 2.6.0 已经安装，
你可以通过提供一个不同的版本号来获取其他版本或者你也可以运行 ``pip install --upgrade`` 来升级这个包到最新版本::

  -> pip install --upgrade requests
  Collecting requests
  Installing collected packages: requests
    Found existing installation: requests 2.6.0
      Uninstalling requests-2.6.0:
        Successfully uninstalled requests-2.6.0
  Successfully installed requests-2.7.0

在 ``pip uninstall`` 后面跟一个或者多个包的名字，将会从这个虚拟环境中移除这些包。

``pip show`` 用来显示一个包的详细信息::

  (tutorial-env) -> pip show requests
  ---
  Metadata-Version: 2.0
  Name: requests
  Version: 2.7.0
  Summary: Python HTTP for Humans.
  Home-page: http://python-requests.org
  Author: Kenneth Reitz
  Author-email: me@kennethreitz.com
  License: Apache 2.0
  Location: /Users/akuchling/envs/tutorial-env/lib/python3.4/site-packages
  Requires:

``pip list`` 用来显示当前虚拟环境下安装的所有包::

  (tutorial-env) -> pip list
  novas (3.1.1.3)
  numpy (1.9.2)
  pip (7.0.3)
  requests (2.7.0)
  setuptools (16.0)

``pip freeze`` 将会生成一个类似于安装包的列表，但是在输出的文本格式中不包含 ``pip install`` ，一个普遍的
做法是将这个列表放到一个 ``requirements.txt`` 文件中::

  (tutorial-env) -> pip freeze > requirements.txt
  (tutorial-env) -> cat requirements.txt
  novas==3.1.1.3
  numpy==1.9.2
  requests==2.7.0

这个 ``requirements.txt`` 文件将会被提交到版本控制（译者注：通过git，svn等的工具可以进行代码版本的控制）中，
并作为应用程序的一部分。这样用户可以通过 ``install -r`` 安装所有需要的包::

  -> pip install -r requirements.txt
  Collecting novas==3.1.1.3 (from -r requirements.txt (line 1))
    ...
  Collecting numpy==1.9.2 (from -r requirements.txt (line 2))
    ...
  Collecting requests==2.7.0 (from -r requirements.txt (line 3))
    ...
  Installing collected packages: novas, numpy, requests
    Running setup.py install for novas
  Successfully installed novas-3.1.1.3 numpy-1.9.2 requests-2.7.0

``pip`` 有非常多的选项。详情请参考 ``pip`` 完整的指导手册 `安装 Python 模块`_ 。 如果你自己写了
一个包，想把它放到 Python 官方类库中，请参考 `发布 Python 模块`_ 。


.. _安装 Python 模块: https://docs.python.org/3/installing/index.html#installing-index
.. _发布 Python 模块: https://docs.python.org/3/distributing/index.html#distributing-index
