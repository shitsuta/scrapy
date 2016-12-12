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
``pip`` ( ``sudo`` などはありません) でパッケージをインストールできます

仮想環境を使い始めるには, `virtualenvのインストール手順`_ を参照してください. 
グローバルにインストールするには（グローバルにインストールすると実際に役立ちます）, 以下を実行してください::

    $ [sudo] pip install virtualenv

`ユーザーガイド`_を確認し virtualenv 環境を作成してください.

.. note::
    もし Linux または OS X を使用している場合, `virtualenvwrapper`_ という virtualenv をかんたんに作成できるツールがあります.

一度 virtualenv を作成すれば, 他の Python パッケージと同様に ``pip`` でインストールすることができます.
(あらかじめインストールする必要のあるPython以外の依存関係は :ref:`platform-specific guides <intro-install-platform-notes>` 
を参照してください).

Python virtualenvsはデフォルトでPython 2を使用するように、またはデフォルトでPython 3を使用するように作成できます.

* Python 3で Scrapy をインストールしたい場合は, Python 3 の virtualenv にインストールしてください.
* また, Python 2 で Scrapy をインストールしたい場合は, Python 2 の virtualenv にインストールしてください.

.. _virtualenv: https://virtualenv.pypa.io
.. _virtualenvのインストール手順: https://virtualenv.pypa.io/en/stable/installation/
.. _virtualenvwrapper: http://virtualenvwrapper.readthedocs.io/en/latest/install.html
.. _ユーザーガイド: https://virtualenv.pypa.io/en/stable/userguide/


.. _intro-install-platform-notes:

プラットフォーム別インストール手順
====================================

Windows
-------

* https://www.python.org/downloads/ から, Python 2.7 をインストールします

  Python実行可能ファイルと追加のスクリプトへのパスを含めるには, 環境変数の ``PATH`` を調整する必要があります. 
  ``PATH`` に Python のディレクトリパスを追加してください::

      C:\Python27\;C:\Python27\Scripts\;

  ``PATH`` PATHを更新するにはコマンドプロンプトを開き, 以下を実行します::

      c:\python27\python.exe c:\python27\tools\scripts\win_add2path.py

  コマンドプロンプトウィンドウを閉じてから再度開いて変更を有効にし, 次のコマンドを実行して Python のバージョンを確認します::

      python --version

* `pywin32` は http://sourceforge.net/projects/pywin32/ からインストールしてください.

  環境に合ったアーキテクチャ（win32またはamd64）をダウンロードしてください.
  
* *(バージョン 2.7.9 以下の Python が必要な限り)* `pip`_ で
  https://pip.pypa.io/en/latest/installing/ からインストールしてください.

  ``pip`` が正しくインストールされていることを確認するために, コマンドプロンプトを開き, 以下を実行します::

      pip --version

* この時点でPython 2.7と ``pip`` パッケージマネージャが動作しているはずです. Scrapyをインストールしましょう::

      pip install Scrapy

.. note::
     Python 3はWindowsではサポートされていません. これは、Scrapyのコア要件である Twisted が Windows 上での Python 3 をサポートしていないためです.

Ubuntu 12.04 以上
---------------------

Scrapyは現在, lxml, twisted, pyOpenSSLの最近の十分なバージョンでテストされており, 最近のUbuntuディストリビューションと互換性があります.
しかし, Ubuntuの以前のバージョンもサポートしてはいますが, Ubuntu 12.04 のように, TLS接続の潜在的な問題があります.

.. note::
    Ubuntuで提供されている ``python-scrapy`` パッケージは使用しないでください. 更新が遅く, 最新の Scrapy に追いつくのが遅くなります.
    
Ubuntu（またはUbuntuベース）システムにscrapyをインストールするには, これらの依存関係をインストールする必要があります::

    sudo apt-get install python-dev python-pip libxml2-dev libxslt1-dev zlib1g-dev libffi-dev libssl-dev

- ``python-dev``, ``zlib1g-dev``, ``libxml2-dev`` と ``libxslt1-dev``
  は ``lxml`` に必要です.
- ``libssl-dev`` と ``libffi-dev`` は ``cryptography`` に必要です.

Python 3 に Scrapy をインストールする場合は, Python 3 開発ヘッダーも必要です::

    sudo apt-get install python3 python3-dev

これらをインストールした後に, :ref:`virtualenv <intro-using-virtualenv>` の中で,
``pip`` で Scrapy をインストールすることができます::

    pip install scrapy

.. note::
    同じ non-python 依存関係を使って Debian Wheezy（7.0）以上で Scrapy をインストールすることができます.


Mac OS X
--------

Scrapy の依存関係をビルドするのには、Cコンパイラと開発ヘッダーが必要です. 
OS X では, これらは通常, Apple の Xcode 開発ツールによって提供されます. 
Xcode コマンドラインツールをインストールするには, ターミナルウィンドウを開き, 以下を実行します::

    xcode-select --install

``pip`` がシステムパッケージを更新しない `既知の問題 <https://github.com/pypa/pip/issues/2468>`_ があります.
Scrapy とその依存関係を正常にインストールするために, この問題に対処する必要があります.
これに対するいくつかの解決策があります:

* *(推奨)* システムの Python を **使用しないでください** . システムの残りの部分と競合しない新しいバージョンをインストールしてください.  `homebrew`_ のパッケージマネージャを使ってインストールを行う方法は次のとおりです:

  * http://brew.sh/ の指示に従って, `homebrew`_ をインストールします.
  * ``PATH`` 変数を更新して, システムパッケージを使用する前に homebrew パッケージを使用するようにしてください
    （デフォルトのシェルとして `zsh`_ を使用している場合は ``.bashrc`` を ``.zshrc`` に変更してください）::

      echo "export PATH=/usr/local/bin:/usr/local/sbin:$PATH" >> ~/.bashrc

  * ``.bashrc`` をリロードして、変更が行われたことを確認します::

      source ~/.bashrc

  * Python をインストールします::

      brew install python

  * Pythonの最新バージョンには ``pip`` が付属しているため, 別途インストールする必要はありません. もし, これが当てはまらない場合は, Pythonをアップグレードしてください::

      brew update; brew upgrade python

* *(オプション)* 独立したPython環境の中にScrapyをインストールする.

  この方法は, 上記の OS X の問題の回避策ですが, 依存関係を管理するための全体的な良い方法であり, 最初の方法を補完することができます.

  `virtualenv`_ は Python で仮想環境を作成するために使用できるツールです.
  開始するには
  http://docs.python-guide.org/en/latest/dev/virtualenvs/ のようなマニュアルを読むことをオススメします.

これらの回避策のいずれかを実行すると, Scrapy をインストールすることができます::

  pip install Scrapy


Anaconda
--------


Anacondaを使用することは、virtualenvを使用して ``pip`` でインストールする代わりの方法です.

.. note::

  Windowsユーザーの場合、または ``pip`` でインストールする際に問題が発生した場合は, この方法で Scrapy をインストールすることをお勧めします.

もし, `Anaconda`_ または `Miniconda`_ がすでにインストールされている場合, `conda-forge`_
コミュニティには Linux, Windows そして OS X のための最新パッケージが有ります.

``conda`` を用いてインストールするには, 以下を実行してください::

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
