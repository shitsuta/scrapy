.. _intro-install:

==================
インストールガイド
==================

Scrapy のインストール
=================

Scrapy は Python 2.7 と Python 3.3 以上で実行できます
(Python 3 がまだサポートされていない Windows を除いて).

Pythonパッケージのインストールに慣れている場合は, PyPIからScrapyとその依存関係をインストールすることができます::

    pip install Scrapy

:ref:`専用のvirtualenv <intro-using-virtualenv>` にScrapyをインストールして、システムパッケージとの衝突を避けることを強くお勧めします

詳細およびプラットフォームの詳細については, 以下を参照してください.


知りたいこと
----------------------------

crapyは純粋なPythonで書かれており、いくつかの主要なPythonパッケージ（他のものの中でも）に依存しています:

* `lxml`_, 効率的なXMLとHTMLパーサー.
* `parsel`_, lxmlの上に書かれたHTML / XMLデータ抽出ライブラリ.
* `w3lib`_, URLやWebページのエンコーディングを扱うための多目的ヘルパー.
* `twisted`_, 非同期ネットワーキングフレームワーク.
* `cryptography`_ と `pyOpenSSL`_, さまざまなネットワークレベルのセキュリティニーズに対処する.

Scrapyがテストされる最小バージョンは:

* Twisted 14.0
* lxml 3.4
* pyOpenSSL 0.14

Scrapyはこれらのパッケージの古いバージョンで動作するかもしれませんが, テストされていないため動作し続けることは保証されません.

これらのパッケージ自体は, プラットフォームに応じて追加のインストール手順が必要な非Pythonパッケージに依存しています.
以下の :ref:`プラットフォーム固有のガイド <intro-install-platform-notes>` を確認してください.

これらの依存関係に関連する問題が発生した場合は, それぞれのインストール手順を参照してください:

* `lxml installation`_
* `cryptography installation`_

.. _lxml installation: http://lxml.de/installation.html
.. _cryptography installation: https://cryptography.io/en/latest/installation/


.. _intro-using-virtualenv:

仮想環境を使用する（推奨）
-----------------------------------------

TL;DR: すべてのプラットフォームで仮想環境内にScrapyをインストールすることをお勧めします.

Pythonパッケージは、グローバル（a.k.aシステム全体）またはユーザスペースにインストールできます。私たちは、Scrapyをシステム全体にインストールすることはお勧めしません.

代わりに、いわゆる "仮想環境" (`virtualenv`_) 内にScrapyをインストールすることをお勧めします.
Virtualenvsを使用すると, 既にインストールされているPythonシステムパッケージと競合することなく（システムツールやスクリプトの一部が壊れる可能性があります）, 
``pip``（ ``sudo`` などはありません）でパッケージをインストールできます

仮想環境を使い始めるには, `virtualenvのインストール手順`_ を参照してください. 
それをグローバルにインストールするには（グローバルにインストールすると実際に役立ちます）、これは実行の問題です::

    $ [sudo] pip install virtualenv

Check this `ユーザーガイド`_ on how to create your virtualenv.

.. note::
    If you use Linux or OS X, `virtualenvwrapper`_ is a handy tool to create virtualenvs.

Once you have created a virtualenv, you can install scrapy inside it with ``pip``,
just like any other Python package.
(See :ref:`platform-specific guides <intro-install-platform-notes>`
below for non-Python dependencies that you may need to install beforehand).

Python virtualenvs can be created to use Python 2 by default, or Python 3 by default.

* If you want to install scrapy with Python 3, install scrapy within a Python 3 virtualenv.
* And if you want to install scrapy with Python 2, install scrapy within a Python 2 virtualenv.

.. _virtualenv: https://virtualenv.pypa.io
.. _virtualenvのインストール手順: https://virtualenv.pypa.io/en/stable/installation/
.. _virtualenvwrapper: http://virtualenvwrapper.readthedocs.io/en/latest/install.html
.. _ユーザーガイド: https://virtualenv.pypa.io/en/stable/userguide/


.. _intro-install-platform-notes:

プラットフォーム固有のインストールメモ
====================================

Windows
-------

* Install Python 2.7 from https://www.python.org/downloads/

  You need to adjust ``PATH`` environment variable to include paths to
  the Python executable and additional scripts. The following paths need to be
  added to ``PATH``::

      C:\Python27\;C:\Python27\Scripts\;

  To update the ``PATH`` open a Command prompt and run::

      c:\python27\python.exe c:\python27\tools\scripts\win_add2path.py

  Close the command prompt window and reopen it so changes take effect, run the
  following command and check it shows the expected Python version::

      python --version

* Install `pywin32` from http://sourceforge.net/projects/pywin32/

  Be sure you download the architecture (win32 or amd64) that matches your system

* *(Only required for Python<2.7.9)* Install `pip`_ from
  https://pip.pypa.io/en/latest/installing/

  Now open a Command prompt to check ``pip`` is installed correctly::

      pip --version

* At this point Python 2.7 and ``pip`` package manager must be working, let's
  install Scrapy::

      pip install Scrapy

.. note::
     Python 3 is not supported on Windows. This is because Scrapy core requirement Twisted does not support
     Python 3 on Windows.

Ubuntu 12.04 以上
---------------------

Scrapy is currently tested with recent-enough versions of lxml,
twisted and pyOpenSSL, and is compatible with recent Ubuntu distributions.
But it should support older versions of Ubuntu too, like Ubuntu 12.04,
albeit with potential issues with TLS connections.

**Don't** use the ``python-scrapy`` package provided by Ubuntu, they are
typically too old and slow to catch up with latest Scrapy.


To install scrapy on Ubuntu (or Ubuntu-based) systems, you need to install
these dependencies::

    sudo apt-get install python-dev python-pip libxml2-dev libxslt1-dev zlib1g-dev libffi-dev libssl-dev

- ``python-dev``, ``zlib1g-dev``, ``libxml2-dev`` and ``libxslt1-dev``
  are required for ``lxml``
- ``libssl-dev`` and ``libffi-dev`` are required for ``cryptography``

If you want to install scrapy on Python 3, you’ll also need Python 3 development headers::

    sudo apt-get install python3 python3-dev

Inside a :ref:`virtualenv <intro-using-virtualenv>`,
you can install Scrapy with ``pip`` after that::

    pip install scrapy

.. note::
    The same non-python dependencies can be used to install Scrapy in Debian
    Wheezy (7.0) and above.


Mac OS X
--------

Building Scrapy's dependencies requires the presence of a C compiler and
development headers. On OS X this is typically provided by Apple’s Xcode
development tools. To install the Xcode command line tools open a terminal
window and run::

    xcode-select --install

There's a `known issue <https://github.com/pypa/pip/issues/2468>`_ that
prevents ``pip`` from updating system packages. This has to be addressed to
successfully install Scrapy and its dependencies. Here are some proposed
solutions:

* *(Recommended)* **Don't** use system python, install a new, updated version
  that doesn't conflict with the rest of your system. Here's how to do it using
  the `homebrew`_ package manager:

  * Install `homebrew`_ following the instructions in http://brew.sh/

  * Update your ``PATH`` variable to state that homebrew packages should be
    used before system packages (Change ``.bashrc`` to ``.zshrc`` accordantly
    if you're using `zsh`_ as default shell)::

      echo "export PATH=/usr/local/bin:/usr/local/sbin:$PATH" >> ~/.bashrc

  * Reload ``.bashrc`` to ensure the changes have taken place::

      source ~/.bashrc

  * Install python::

      brew install python

  * Latest versions of python have ``pip`` bundled with them so you won't need
    to install it separately. If this is not the case, upgrade python::

      brew update; brew upgrade python

* *(Optional)* Install Scrapy inside an isolated python environment.

  This method is a workaround for the above OS X issue, but it's an overall
  good practice for managing dependencies and can complement the first method.

  `virtualenv`_ is a tool you can use to create virtual environments in python.
  We recommended reading a tutorial like
  http://docs.python-guide.org/en/latest/dev/virtualenvs/ to get started.

After any of these workarounds you should be able to install Scrapy::

  pip install Scrapy


Anaconda
--------


Using Anaconda is an alternative to using a virtualenv and installing with ``pip``.

.. note::

  For Windows users, or if you have issues installing through ``pip``, this is
  the recommended way to install Scrapy.

If you already have `Anaconda`_ or `Miniconda`_ installed, the `conda-forge`_
community have up-to-date packages for Linux, Windows and OS X.

To install Scrapy using ``conda``, run::

  conda install -c conda-forge scrapy

.. _Python: https://www.python.org/
.. _pip: https://pip.pypa.io/en/latest/installing/
.. _Control Panel: https://www.microsoft.com/resources/documentation/windows/xp/all/proddocs/en-us/sysdm_advancd_environmnt_addchange_variable.mspx
.. _lxml: http://lxml.de/
.. _parsel: https://pypi.python.org/pypi/parsel
.. _w3lib: https://pypi.python.org/pypi/w3lib
.. _twisted: https://twistedmatrix.com/
.. _cryptography: https://cryptography.io/
.. _pyOpenSSL: https://pypi.python.org/pypi/pyOpenSSL
.. _setuptools: https://pypi.python.org/pypi/setuptools
.. _AUR Scrapy package: https://aur.archlinux.org/packages/scrapy/
.. _homebrew: http://brew.sh/
.. _zsh: http://www.zsh.org/
.. _Scrapinghub: http://scrapinghub.com
.. _Anaconda: http://docs.continuum.io/anaconda/index
.. _Miniconda: http://conda.pydata.org/docs/install/quick.html
.. _conda-forge: https://conda-forge.github.io/
