.. _topics-commands:

=================
コマンドラインツール
=================

.. versionadded:: 0.10

ここでは, 「コマンド」または「Scrapyコマンド」と呼ばれるサブコマンドと区別するために, 「Scrapy ツール」と呼ばれる ``scrapy`` コマンドラインツールを使用して Scrapy を制御します.

Scrapyツールは, 複数の目的で複数のコマンドを提供し, それぞれが異なる引数とオプションのセットを受け入れます.

( ``scrapy deploy`` マンドは1.0で削除され,  ``scrapyd-deploy`` が採用されました. `Deploying your project`_ を参照してください.)

.. _topics-config-settings:

環境設定
======================

Scrapy は標準的な以下の場所から ``scrapy.cfg`` ファイルの設定パラメータを探します:

1. ``/etc/scrapy.cfg`` または ``c:\scrapy\scrapy.cfg`` (system-wide),
2. グローバルセッティングのための ``~/.config/scrapy.cfg`` (``$XDG_CONFIG_HOME``) と ``~/.scrapy.cfg`` (``$HOME``) 
   (user-wide), 
3. プロジェクトルート内にある ``scrapy.cfg`` (次のセクションを参照してください).

これらのファイルからの設定は, 表示された優先順位でマージされます. 
ユーザ定義の値はシステム全体のデフォルトよりも優先され, プロジェクト全体の設定は定義されている場合は他のすべての設定を上書きします.

また, Scrapy はいくつかの環境変数を理解し, 設定することができます. 現在, 以下のものが存在します:

* ``SCRAPY_SETTINGS_MODULE`` ( :ref:`topics-settings-module-envvar` を参照してください)
* ``SCRAPY_PROJECT``
* ``SCRAPY_PYTHON_SHELL`` ( :ref:`topics-shell` を参照してください)

.. _topics-project-structure:

Scrapyプロジェクトのデフォルト構造
====================================

コマンドラインツールとそのサブコマンドについて解説する前に, まずScrapyプロジェクトのディレクトリ構造について理解しておきましょう.

変更することはできますが, すべての Scrapy プロジェクトはデフォルトで同じファイル構造, もしくはこれに似た構造になっています::

   scrapy.cfg
   myproject/
       __init__.py
       items.py
       pipelines.py
       settings.py
       spiders/
           __init__.py
           spider1.py
           spider2.py
           ...

``scrapy.cfg`` ファイルが存在するディレクトリは, 
**プロジェクトルートディレクトリ** と呼ばれます. 
このファイルには、プロジェクト設定を定義するpythonモジュールの名前が含まれています。次に例を示します::

    [settings]
    default = myproject.settings

``scrapy`` ツールを使う
=========================

引数を指定せずにScrapyツールを実行すると, いくつかの使用方法のヘルプと使用可能なコマンドが表示されます::

    Scrapy X.Y - no active project

    Usage:
      scrapy <command> [options] [args]

    Available commands:
      crawl         Run a spider
      fetch         Fetch a URL using the Scrapy downloader
    [...]

あなたがScrapyプロジェクトの中にいる場合, 最初の行は現在アクティブなプロジェクトを出力します. 
上記の例では, プロジェクトの外から実行されました. 
プロジェクトの中から実行すると、次のような内容が出力されます::

    Scrapy X.Y - project: myproject

    Usage:
      scrapy <command> [options] [args]

    [...]
    
プロジェクトの作成
-----------------

まず,  ``scrapy`` ツールで最初に行うことは, あなたのScrapyプロジェクトを作成することです::

    scrapy startproject myproject [project_dir]

これにより, ``project_dir`` ディレクトリの下に Scrapy プロジェクトが作成されます.
``project_dir`` が指定されていない場合, ``project_dir`` は ``myproject`` と同じになります.

次に, 新しいプロジェクトディレクトリ中に移動します::

    cd project_dir

これで,  ``scrapy`` コマンドを使用してそこからプロジェクトを管理および制御する準備が整いました.

プロジェクトの制御
--------------------

プロジェクトの中から ``scrapy`` ツールを使用して, プロジェクトを制御および管理します.

例えば, 新しいスパイダーを作成するには::

    scrapy genspider mydomain mydomain.com

一部のScrapyコマンド ( :command:`crawl` など) は, Scrapyプロジェクト内から実行する必要があります. 
どのコマンドをプロジェクト内から実行する必要があるかについての詳細は, 
以下の :ref:`コマンドリファレンス <topics-commands-ref>` を参照してください.

また、いくつかのコマンドは、プロジェクトの中から実行する際に, 少し違う振る舞いをすることがあります. 
たとえば、フェッチされたURLが特定のスパイダーに関連付けられている場合, 
``fetch`` コマンドは spider-overridden ビヘイビア（``user-agent`` 属性を上書きする user_agent など）を使用します. 
``fetch`` コマンドは, スパイダーがページをどのようにダウンロードしているかを確認するために使用されるため, 意図的に行っています.

.. _topics-commands-ref:

利用可能なコマンド
=======================

このセクションでは, 使用可能な組み込みコマンドのリストと, 使用例を示します. それぞれのコマンドについての詳細は, 以下のコマンドでいつでも確認できます::

    scrapy <command> -h

または, 使用可能なすべてのコマンドは, 以下で確認できます::

    scrapy -h

コマンドは, アクティブなScrapyプロジェクトなしでのみ動作するコマンド（グローバルコマンド）と, プロジェクト内から実行するコマンドの動作が若干異なる場合があります（プロジェクトオーバーライド設定を使用するため）.

グローバルコマンド:

* :command:`startproject`
* :command:`genspider`
* :command:`settings`
* :command:`runspider`
* :command:`shell`
* :command:`fetch`
* :command:`view`
* :command:`version`

プロジェクト下でのみ使用可能なコマンド:

* :command:`crawl`
* :command:`check`
* :command:`list`
* :command:`edit`
* :command:`parse`
* :command:`bench`

.. command:: startproject

startproject
------------

* シンタックス: ``scrapy startproject <project_name> [project_dir]``
* プロジェクトに必要か: *no*

 ``project_dir`` ディレクトリ下に ``project_name`` という名前の新しい Scrapy プロジェクトを作成します.
もし,  ``project_dir`` が指定されていない場合, プロジェクト名と同じ名前の ``project_dir`` が作成されます.

使用例::

    $ scrapy startproject myproject

.. command:: genspider

genspider
---------

* シンタックス: ``scrapy genspider [-t template] <name> <domain>``
* プロジェクトに必要か: *no*

プロジェクト内から呼び出された場合は, 現在のフォルダまたは現在のプロジェクトの``spiders`` フォルダに新しいスパイダーを作成します. 
``<name>`` パラメータはスパイダの名前として設定され,  ``<domain>`` はスパイダーの ``allowed_domains`` および``start_urls`` 属性を生成するために使用されます.

使用例::

    $ scrapy genspider -l
    Available templates:
      basic
      crawl
      csvfeed
      xmlfeed

    $ scrapy genspider example example.com
    Created spider 'example' using template 'basic'

    $ scrapy genspider -t crawl scrapyorg scrapy.org
    Created spider 'scrapyorg' using template 'crawl'

これはあらかじめ定義されたテンプレートに基づいてスパイダーを作成する便利なショートカットコマンドですが, スパイダーを作成する唯一の方法ではありません. 
このコマンドを使用する代わりに, スパイダーのソースコードファイルを自分で作成することもできます.

.. command:: crawl

crawl
-----

* シンタックス: ``scrapy crawl <spider>``
* プロジェクトに必要か: *yes*

スパイダーを使用してクロールを始める.

使用例::

    $ scrapy crawl myspider
    [ ... myspider starts crawling ... ]


.. command:: check

check
-----

* シンタックス: ``scrapy check [-l] <spider>``
* プロジェクトに必要か: *yes*

コントラクトチェックを実行する.

使用例::

    $ scrapy check -l
    first_spider
      * parse
      * parse_item
    second_spider
      * parse
      * parse_item

    $ scrapy check
    [FAILED] first_spider:parse_item
    >>> 'RetailPricex' field is missing

    [FAILED] first_spider:parse
    >>> Returned 92 requests, expected 0..4

.. command:: list

list
----

* シンタックス: ``scrapy list``
* プロジェクトに必要か: *yes*

現在のプロジェクトで使用可能なすべてのスパイダーを一覧表示します. 出力は, 1行に1つのスパイダーです.

使用例::

    $ scrapy list
    spider1
    spider2

.. command:: edit

edit
----

* シンタックス: ``scrapy edit <spider>``
* プロジェクトに必要か: *yes*

:setting:`EDITOR` 設定で定義されたエディタを使用して, 指定されたスパイダーを編集します.

このコマンドは, 便利なショートカットとしてのみ提供されています. 開発者はもちろん, ツールやIDEを自由に選択して, スパイダーを作成・デバッグできます.

使用例::

    $ scrapy edit spider1

.. command:: fetch

fetch
-----

* シンタックス: ``scrapy fetch <url>``
* プロジェクトに必要か: *no*

Scrapy ダウンローダーを使用してURLからダウンロードし, その内容を標準出力に書き出します.

このコマンドの興味深い点は, スパイダーがどのようにダウンロードするかをページから取得することです. たとえば, スパイダーがUser Agentを上書きする ``USER_AGENT``
属性を持っている場合は, それを使用します.

このコマンドは, あなたのスパイダーが特定のページをどのようにフェッチするかを "見る" ために使うことができます.

プロジェクトの外で使用される場合は, スパイダーごとの特定の動作は適用されず, デフォルトのScrapyダウンローダ設定を使用します.

使用例::

    $ scrapy fetch --nolog http://www.example.com/some/page.html
    [ ... html content here ... ]

    $ scrapy fetch --nolog --headers http://www.example.com/
    {'Accept-Ranges': ['bytes'],
     'Age': ['1263   '],
     'Connection': ['close     '],
     'Content-Length': ['596'],
     'Content-Type': ['text/html; charset=UTF-8'],
     'Date': ['Wed, 18 Aug 2010 23:59:46 GMT'],
     'Etag': ['"573c1-254-48c9c87349680"'],
     'Last-Modified': ['Fri, 30 Jul 2010 15:30:18 GMT'],
     'Server': ['Apache/2.2.3 (CentOS)']}

.. command:: view

view
----

* シンタックス: ``scrapy view <url>``
* プロジェクトに必要か: *no*

Scrapyスパイダーがそれを "見る" ようにブラウザでURLを開きます.
スパイダーは通常のユーザーとは違うページを表示することがあるので, スパイダーが何を見ているかを確認し, 期待通りのものかどうかを確認することができます.

使用例::

    $ scrapy view http://www.example.com/some/page.html
    [ ... browser starts ... ]

.. command:: shell

shell
-----

* シンタックス: ``scrapy shell [url]``
* プロジェクトに必要か: *no*

指定されたURL（指定されている場合）またはURLが指定されていない場合は空のScrapyシェルを開始します. 
また, UNIX形式のローカルファイルパスをサポートしています. 
``./`` または ``../`` を接頭辞とした相対パス, もしくは絶対パスです.
詳細については,  :ref:`topics-shell` を参照してください.

使用例::

    $ scrapy shell http://www.example.com/some/page.html
    [ ... scrapy shell starts ... ]

.. command:: parse

parse
-----

* シンタックス: ``scrapy parse <url> [options]``
* プロジェクトに必要か: *yes*

指定されたURLを取得し、それをスパイダーで処理・解析します.  ``--callback`` オプションで渡されたメソッドを使用します. 指定されていない場合は ``parse`` メソッドを使用します.

サポートされているオプション:

* ``--spider=SPIDER``: スパイダーの自動検出をバイパスし, 特定のスパイダーを強制的に使用する

* ``--a NAME=VALUE``: スパイダー引数を設定する（繰り返してもよい）

* ``--callback`` または ``-c``: レスポンスを解析するためのコールバックとして使用するspiderメソッド

* ``--pipelines``: パイプラインを通じてアイテムを処理する

* ``--rules`` または ``-r``:  :class:`~scrapy.spiders.CrawlSpider` のルールを使用して, レスポンスの解析に使用するコールバック (i.e. spider メソッド) を検出する

* ``--noitems``: スクレイピングしたアイテムを表示しない

* ``--nolinks``: 抽出されたリンクを表示しない

* ``--nocolour``: 出力の色分けを行わない

* ``--depth`` または ``-d``: 要求を再帰的に追跡する深さレベル（デフォルト：1）

* ``--verbose`` または ``-v``: 各深度レベルの情報を表示する

使用例::

    $ scrapy parse http://www.example.com/ -c parse_item
    [ ... scrapy log lines crawling example.com spider ... ]

    >>> STATUS DEPTH LEVEL 1 <<<
    # Scraped Items  ------------------------------------------------------------
    [{'name': u'Example item',
     'category': u'Furniture',
     'length': u'12 cm'}]

    # Requests  -----------------------------------------------------------------
    []


.. command:: settings

settings
--------

* シンタックス: ``scrapy settings [options]``
* プロジェクトに必要か: *no*

Scrapy設定の値を取得します.

プロジェクト内で使用されている場合はプロジェクト設定値が表示され, そうでない場合はその設定のデフォルトのScrapy値が表示されます.

使用例::

    $ scrapy settings --get BOT_NAME
    scrapybot
    $ scrapy settings --get DOWNLOAD_DELAY
    0

.. command:: runspider

runspider
---------

* シンタックス: ``scrapy runspider <spider_file.py>``
* プロジェクトに必要か: *no*

プロジェクトを作成せずに, Pythonファイルに含まれているスパイダーを実行します.

使用例::

    $ scrapy runspider myspider.py
    [ ... spider starts crawling ... ]

.. command:: version

version
-------

* シンタックス: ``scrapy version [-v]``
* プロジェクトに必要か: *no*

Scrapy のバージョンを表示します.  ``-v`` と一緒に使用すると, バグレポートに便利な Python, Twisted, そしてプラットフォームの情報も表示されます.

.. command:: bench

bench
-----

.. versionadded:: 0.17

* シンタックス: ``scrapy bench``
* プロジェクトに必要か: *no*

かんたんなベンチマークテストを実行します. :ref:`benchmarking` を参照してください.

カスタムプロジェクトコマンド
=======================

:setting:`COMMANDS_MODULE` 設定を使用してカスタムプロジェクトコマンドを追加することができます. 
コマンドの実装方法の例については, `scrapy/コマンド`_ の Scrapy コマンドを参照してください.

.. _scrapy/コマンド: https://github.com/scrapy/scrapy/tree/master/scrapy/commands
.. setting:: COMMANDS_MODULE

COMMANDS_MODULE
---------------

初期値: ``''`` (空文字列)

カスタムのScrapyコマンドを検索するためのモジュール. これは Scrapy プロジェクトのカスタムコマンドを追加するために使用されます.

例::

    COMMANDS_MODULE = 'mybot.commands'

.. _Deploying your project: http://scrapyd.readthedocs.org/en/latest/deploy.html

setup.pyエントリポイントを介してコマンドを登録する
-------------------------------------------

.. note:: これは実験的な機能なので注意してください.

``scrapy.commands`` ファイルのエントリポイントに,  ``setup.py`` セクションを追加することで, 外部ライブラリから Scrapy コマンドを追加することもできます.

次の例では,  ``my_command`` コマンドを追加しています::

  from setuptools import setup, find_packages

  setup(name='scrapy-mymodule',
    entry_points={
      'scrapy.commands': [
        'my_command=my_scrapy_module.commands:MyCommand',
      ],
    },
   )
