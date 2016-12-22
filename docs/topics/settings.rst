.. _topics-settings:

========
設定
========

Scrapyの設定では, コア, 拡張機能, パイプライン, スパイダー自体を含むすべての Scrapy コンポーネントの動作をカスタマイズすることができます.

設定のインフラストラクチャは, コードで値を取得するために使用できる, キーと値のマッピングのグローバルな名前空間を提供します. 
この設定は, 以下で説明するさまざまなメカニズムによって設定できます.

この設定は, 現在アクティブなScrapyプロジェクトを選択するためのメカニズムでもあります.

使用可能なビルトイン設定のリストについては,  :ref:`topics-settings-ref` を参照してください.

.. _topics-settings-module-envvar:

設定を指定する
========================

Scrapyを使用するときは, 環境変数 ``SCRAPY_SETTINGS_MODULE`` を使用して, 設定を Scrapy に教えてください.

``SCRAPY_SETTINGS_MODULE`` の値はPythonのパス構文でなければなりません, e.g.
``myproject.settings``. 設定モジュールは, Pythonの `インポートサーチパス`_ 上にある必要があります.

.. _インポートサーチパス: https://docs.python.org/2/tutorial/modules.html#the-module-search-path

設定を入力する
=======================

設定は, それぞれ異なる優先順位を持つ異なるメカニズムを使用しています. 以下は優先順位の高い順にリストされています:

 1. コマンドラインオプション（優先度高）
 2. スパイダーごとの設定
 3. プロジェクト設定モジュール
 4. コマンドごとのデフォルト設定
 5. デフォルトのグローバル設定（優先度低）

これら設定の優先度は内部的に処理されますが, API呼び出しを使用して手動で処理することが可能です. 
:ref:`topics-api-settings` を参照してください.

これらのメカニズムは, 以下でより詳細に説明します.

1. コマンドラインオプション
-----------------------

コマンドラインで提供される引数は, 他のオプションよりも優先されます. 
``-s`` (または ``--set``) コマンドラインオプションを使用して, 1つ（またはそれ以上）の設定を明示的に上書きすることができます.

.. highlight:: sh

例::

    scrapy crawl myspider -s LOG_FILE=scrapy.log

2. スパイダーごとの設定
----------------------

スパイダー ( :ref:`topics-spiders` を参照) は, 独自の設定を定義して優先順位を上げることができます. 
これは,  :attr:`~scrapy.spiders.Spider.custom_settings` 属性に設定することで可能です::

    class MySpider(scrapy.Spider):
        name = 'myspider'

        custom_settings = {
            'SOME_SETTING': 'some value',
        }

3. プロジェクト設定モジュール
--------------------------

プロジェクト設定モジュールは, Scrapyプロジェクトの標準設定ファイルです. 
カスタム設定の大部分が設定されます. 
標準の Scrapy プロジェクトでは, プロジェクト用に作成された ``settings.py`` ファイルの設定を追加または変更することになります.

4. コマンドごとのデフォルト設定
-------------------------------

各 :doc:`Scrapy ツール </topics/commands>` コマンドには独自のデフォルト設定があり, 
グローバルなデフォルト設定を上書きします. 
これらのカスタムコマンド設定は,  ``default_settings`` 属性で指定されます.

5. デフォルトのグローバル設定
--------------------------

グローバルなデフォルト設定は ``scrapy.settings.default_settings``
モジュールにあり,  :ref:`topics-settings-ref` に記載されています.

設定にアクセスする方法
======================

.. highlight:: python

スパイダーでは,  設定は ``self.settings`` で取得することができます::

    class MySpider(scrapy.Spider):
        name = 'myspider'
        start_urls = ['http://example.com']

        def parse(self, response):
            print("Existing settings: %s" % self.settings.attributes.keys())

.. note::
   ``settings`` 属性は, スパイダーが初期化された後, 
   ベース Spider クラスに設定されます. 
   初期化の前に設定を使用する場合（たとえば, スパイダーの ``__init__()`` メソッド）, 
   :meth:`~scrapy.spiders.Spider.from_crawler` メソッドをオーバーライドする必要があります.

設定は, 拡張機能, ミドルウェアおよびアイテムパイプラインの ``from_crawler`` メソッドに渡される, 
クローラの :attr:`scrapy.crawler.Crawler.settings` 属性を介してアクセスできます. :

    class MyExtension(object):
        def __init__(self, log_is_enabled=False):
            if log_is_enabled:
                print("log is enabled!")

        @classmethod
        def from_crawler(cls, crawler):
            settings = crawler.settings
            return cls(settings.getbool('LOG_ENABLED'))

設定オブジェクトは ``dict'' (例： ``settings['LOG_ENABLED']`` ) のように使用できますが, 
:class:`~scrapy.settings.Settings` API で提供されるメソッドの1つを使用して, 
タイプエラーを回避するために必要な形式で設定を抽出することをお勧めします.

名前を設定する理由
===========================

設定名には通常, 構成するコンポーネントの接頭辞が付いています. 
例えば, 架空の ``robots.txt`` 拡張子の適切な設定名は
``ROBOTSTXT_ENABLED``, ``ROBOTSTXT_OBEY``, ``ROBOTSTXT_CACHEDIR`` などです.


.. _topics-settings-ref:

ビルトイン設定リファレンス
===========================

ここでは, アルファベット順に使用可能なすべての Scrapy 設定のリストと, デフォルト値, 適用されるスコープが示されています.

使用可能な場合スコープは, 特定のコンポーネントに関連付けられていれば, 設定が使用されている場所を示します. 
その場合, そのコンポーネントのモジュール, 通常は拡張モジュール, ミドルウェアまたはパイプラインが表示されます. 
また, 設定を有効にするためにコンポーネントを有効にする必要があることも意味します.

.. setting:: AWS_ACCESS_KEY_ID

AWS_ACCESS_KEY_ID
-----------------

デフォルト: ``None``

:ref:`S3フィードストレージバックエンド <topics-feed-storage-s3>` など,   `Amazon Web services`_ のアクセスを必要とするコードで使用されるAWSアクセスキー.

.. setting:: AWS_SECRET_ACCESS_KEY

AWS_SECRET_ACCESS_KEY
---------------------

デフォルト: ``None``

:ref:`S3フィードストレージバックエンド <topics-feed-storage-s3>` など,  `Amazon Web services`_ へのアクセスを必要とするコードで使用されるAWS秘密鍵.

.. setting:: BOT_NAME

BOT_NAME
--------

デフォルト: ``'scrapybot'``

このScrapyプロジェクトによって実装されたボットの名前（プロジェクト名とも呼ばれます）. 
これは, デフォルトで User-Agent, ロギングに使用されます.

:command:`startproject` コマンドを使用してプロジェクトを作成すると, プロジェクト名が自動的に入力されます.

.. setting:: CONCURRENT_ITEMS

CONCURRENT_ITEMS
----------------

デフォルト: ``100``

アイテムプロセッサ ( :ref:`アイテムパイプライン <topics-item-pipeline>` とも呼ばれます) 
で並列処理する同時アイテムの最大数（応答あたり）.

.. setting:: CONCURRENT_REQUESTS

CONCURRENT_REQUESTS
-------------------

デフォルト: ``16``

Scrapyダウンローダによって実行される並行（つまり同時の）リクエストの最大数.

.. setting:: CONCURRENT_REQUESTS_PER_DOMAIN

CONCURRENT_REQUESTS_PER_DOMAIN
------------------------------

デフォルト: ``8``

単一のドメインに対して実行される並行（つまり同時）リクエストの最大数.

:ref:`topics-autothrottle` と 
:setting:`AUTOTHROTTLE_TARGET_CONCURRENCY` オプションも参照してください.


.. setting:: CONCURRENT_REQUESTS_PER_IP

CONCURRENT_REQUESTS_PER_IP
--------------------------

デフォルト: ``0``

単一の IP に対して実行される並行（つまり同時）要求の最大数. 
0以外の場合, :setting:`CONCURRENT_REQUESTS_PER_DOMAIN` 設定は無視され, 
代わりにこの設定が使用されます. つまり, ドメインごとではなく, IPごとに並行処理の制限が適用されます.

この設定は,  :setting:`DOWNLOAD_DELAY` 及び 
:ref:`topics-autothrottle`: にも影響します.  :setting:`CONCURRENT_REQUESTS_PER_IP`
がゼロ以外の場合, ドメインごとではなくIPごとにダウンロード遅延が強制されます.


.. setting:: DEFAULT_ITEM_CLASS

DEFAULT_ITEM_CLASS
------------------

デフォルト: ``'scrapy.item.Item'``

:ref:`Scrapy shell <topics-shell>` でアイテムをインスタンス化するために使用されるデフォルトクラスです.

.. setting:: DEFAULT_REQUEST_HEADERS

DEFAULT_REQUEST_HEADERS
-----------------------

デフォルト::

    {
        'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
        'Accept-Language': 'en',
    }

Scrapy HTTP Request に使用されるデフォルトのヘッダー. これらは, 
:class:`~scrapy.downloadermiddlewares.defaultheaders.DefaultHeadersMiddleware` 
に設定されています.

.. setting:: DEPTH_LIMIT

DEPTH_LIMIT
-----------

デフォルト: ``0``

スコープ: ``scrapy.spidermiddlewares.depth.DepthMiddleware``

どのサイトでもクロールできる最大の深さ. ゼロの場合, 制限は課されません.

.. setting:: DEPTH_PRIORITY

DEPTH_PRIORITY
--------------

デフォルト: ``0``

スコープ: ``scrapy.spidermiddlewares.depth.DepthMiddleware``

深さに基づいてリクエストの優先度を調整するために使用される整数:

- 0（デフォルト）の場合, 深度からの優先調整は行われません
- **正の値は優先度を下げます. つまり, 深度の高い要求が後で処理されます.**   
  これは, 幅優先のクロール（BFO）を行うときによく使用されます
- 負の値は優先度を増加させます. すなわち, より深い深度要求がより早く処理されます（DFO）

BFO または DFO のチューニングに関しては :ref:`faq-bfo-dfo` を参照してください.

.. note::

    この設定は, 他の優先度設定である :setting:`REDIRECT_PRIORITY_ADJUST`
    及び :setting:`RETRY_PRIORITY_ADJUST` と比較して, 優先度を調整します.

.. setting:: DEPTH_STATS

DEPTH_STATS
-----------

デフォルト: ``True``

スコープ: ``scrapy.spidermiddlewares.depth.DepthMiddleware``

最大深度統計を収集するかどうか.

.. setting:: DEPTH_STATS_VERBOSE

DEPTH_STATS_VERBOSE
-------------------

デフォルト: ``False``

スコープ: ``scrapy.spidermiddlewares.depth.DepthMiddleware``

冗長な深さ統計を収集するかどうか. これを有効にすると, 各深さのリクエスト数が統計情報に収集されます.

.. setting:: DNSCACHE_ENABLED

DNSCACHE_ENABLED
----------------

デフォルト: ``True``

DNSインメモリキャッシュを有効にするかどうか.

.. setting:: DNSCACHE_SIZE

DNSCACHE_SIZE
-------------

デフォルト: ``10000``

DNSのインメモリキャッシュサイズ.

.. setting:: DNS_TIMEOUT

DNS_TIMEOUT
-----------

デフォルト: ``60``

DNSクエリを処理するための秒単位タイムアウト. フロートがサポートされています.

.. setting:: DOWNLOADER

DOWNLOADER
----------

デフォルト: ``'scrapy.core.downloader.Downloader'``

クロールに使用するダウンローダ.

.. setting:: DOWNLOADER_HTTPCLIENTFACTORY

DOWNLOADER_HTTPCLIENTFACTORY
----------------------------

デフォルト: ``'scrapy.core.downloader.webclient.ScrapyHTTPClientFactory'``

HTTP / 1.0接続（ ``HTTP10DownloadHandler`` の場合）に使用する, 
Twisted の ``protocol.ClientFactory`` クラスを定義します. 

.. note::

    HTTP/1.0 is rarely used nowadays so you can safely ignore this setting,
    unless you use Twisted<11.1, or if you really want to use HTTP/1.0
    and override :setting:`DOWNLOAD_HANDLERS_BASE` for ``http(s)`` scheme
    accordingly, i.e. to
    ``'scrapy.core.downloader.handlers.http.HTTP10DownloadHandler'``.

.. setting:: DOWNLOADER_CLIENTCONTEXTFACTORY

DOWNLOADER_CLIENTCONTEXTFACTORY
-------------------------------

デフォルト: ``'scrapy.core.downloader.contextfactory.ScrapyClientContextFactory'``

使用する ContextFactory へのクラスパスを表します.

ContextFactory は,  SSL / TLS コンテキストの Twisted の用語で, 
使用するTLS / SSLプロトコルのバージョン, 証明書の検証の有無, 
クライアント側の認証（およびその他のさまざまなもの）の有効化を定義します. 

.. note::

    Scrapy default context factory **does NOT perform remote server
    certificate verification**. This is usually fine for web scraping.

    If you do need remote server certificate verification enabled,
    Scrapy also has another context factory class that you can set,
    ``'scrapy.core.downloader.contextfactory.BrowserLikeContextFactory'``,
    which uses the platform's certificates to validate remote endpoints.
    **This is only available if you use Twisted>=14.0.**

カスタム ContextFactory を使用する場合は, 
init で ``method`` パラメータを受け入れるようにしてください
（これは ``OpenSSL.SSL`` メソッドの :setting:`DOWNLOADER_CLIENT_TLS_METHOD` のマッピングです）. 

.. setting:: DOWNLOADER_CLIENT_TLS_METHOD

DOWNLOADER_CLIENT_TLS_METHOD
----------------------------

デフォルト: ``'TLS'``

この設定を使用して, デフォルトの HTTP/1.1 ダウンローダが使用する TLS/SSL 方式をカスタマイズします.

この設定は, これらの文字列値のいずれかでなければなりません:

- ``'TLS'``: OpenSSLの ``TLS_method()`` (a.k.a ``SSLv23_method()``), 
  にマップされています. これにより, プラットフォームでサポートされている最高位から始まる
  プロトコルネゴシエーションが可能になります; **デフォルト, 推奨**
- ``'TLSv1.0'``: この値を指定すると, HTTPS接続はTLSバージョン1.0を使用します.  Scrapy < 1.1 の動作が必要な場合にこれを設定します
- ``'TLSv1.1'``: TLS バージョン 1.1 を強制します
- ``'TLSv1.2'``: TLS バージョン 1.2 を強制します
- ``'SSLv3'``: SSL バージョン 3 を強制します (**非推奨**)

.. note::

    PyOpenSSL >= 0.13, Twisted >= 0.13 以上を使用することをお勧めします（出来れば Twisted> = 14.0）.

.. setting:: DOWNLOADER_MIDDLEWARES

DOWNLOADER_MIDDLEWARES
----------------------

デフォルト:: ``{}``

あなたのプロジェクトで有効になっているダウンローダミドルウェアとその注文を含む辞書. 
詳細については :ref:`topics-downloader-middleware-setting` を参照してください.

.. setting:: DOWNLOADER_MIDDLEWARES_BASE

DOWNLOADER_MIDDLEWARES_BASE
---------------------------

デフォルト::

    {
        'scrapy.downloadermiddlewares.robotstxt.RobotsTxtMiddleware': 100,
        'scrapy.downloadermiddlewares.httpauth.HttpAuthMiddleware': 300,
        'scrapy.downloadermiddlewares.downloadtimeout.DownloadTimeoutMiddleware': 350,
        'scrapy.downloadermiddlewares.defaultheaders.DefaultHeadersMiddleware': 400,
        'scrapy.downloadermiddlewares.useragent.UserAgentMiddleware': 500,
        'scrapy.downloadermiddlewares.retry.RetryMiddleware': 550,
        'scrapy.downloadermiddlewares.ajaxcrawl.AjaxCrawlMiddleware': 560,
        'scrapy.downloadermiddlewares.redirect.MetaRefreshMiddleware': 580,
        'scrapy.downloadermiddlewares.httpcompression.HttpCompressionMiddleware': 590,
        'scrapy.downloadermiddlewares.redirect.RedirectMiddleware': 600,
        'scrapy.downloadermiddlewares.cookies.CookiesMiddleware': 700,
        'scrapy.downloadermiddlewares.httpproxy.HttpProxyMiddleware': 750,
        'scrapy.downloadermiddlewares.stats.DownloaderStats': 850,
        'scrapy.downloadermiddlewares.httpcache.HttpCacheMiddleware': 900,
    }

Scrapyでデフォルトで有効になっているダウンローダミドルウェアを含む辞書. 
ローオーダーはエンジンに近く, ハイオーダーはダウンローダーに近くなっています. 
プロジェクトでこの設定を変更しないでください. 代わりに 
:setting:`DOWNLOADER_MIDDLEWARES` を変更してください. 詳細については, 
:ref:`topics-downloader-middleware-setting` を参照してください.

.. setting:: DOWNLOADER_STATS

DOWNLOADER_STATS
----------------

デフォルト: ``True``

ダウンローダの統計情報収集を有効にするかどうか.

.. setting:: DOWNLOAD_DELAY

DOWNLOAD_DELAY
--------------

デフォルト: ``0``

ダウンローダが同じWebサイトから連続したページをダウンロードするまで待機する時間（秒）. 
これは, サーバに負荷がかかることを避けるために, クロール速度を抑えるために使用できます.  
10進数がサポートされています. 例::

    DOWNLOAD_DELAY = 0.25    # 250 ms of delay
    
この設定は,  :setting:`RANDOMIZE_DOWNLOAD_DELAY` 
設定の影響を受けます(デフォルトで有効). 既定では, 
Scrapyは要求間の固定時間を待機しませんが, 0.5 * :setting:`DOWNLOAD_DELAY` から 1.5 * :setting:`DOWNLOAD_DELAY` 
までのランダムな間隔を使用します.

:setting:`CONCURRENT_REQUESTS_PER_IP` がゼロ以外の場合, 遅延はドメインごとではなくIPアドレスごとに適用されます.

``download_delay`` スパイダー属性を設定することで, スパイダーごとにこの設定を変更することもできます.

.. setting:: DOWNLOAD_HANDLERS

DOWNLOAD_HANDLERS
-----------------

デフォルト: ``{}``

プロジェクトで有効になっているリクエストダウンローダーハンドラを含む ``dict``.
形式のサンプルについては :setting:`DOWNLOAD_HANDLERS_BASE` を参照してください.

.. setting:: DOWNLOAD_HANDLERS_BASE

DOWNLOAD_HANDLERS_BASE
----------------------

デフォルト::

    {
        'file': 'scrapy.core.downloader.handlers.file.FileDownloadHandler',
        'http': 'scrapy.core.downloader.handlers.http.HTTPDownloadHandler',
        'https': 'scrapy.core.downloader.handlers.http.HTTPDownloadHandler',
        's3': 'scrapy.core.downloader.handlers.s3.S3DownloadHandler',
        'ftp': 'scrapy.core.downloader.handlers.ftp.FTPDownloadHandler',
    }


デフォルトで Scrapy で有効になっているリクエストハンドラを含む ``dict`` . プロジェクトでこの設定を変更しないでください. 代わりに 
:setting:`DOWNLOAD_HANDLERS` を変更してください.

これらのダウンロードハンドラのいずれかを無効にするには, 
:setting:`DOWNLOAD_HANDLERS` の URI スキームにNoneを割り当てます. 
たとえば, 組み込みのFTPハンドラを無効にするには（置き換えずに）, これを settings.py に記述子ます::

    DOWNLOAD_HANDLERS = {
        'ftp': None,
    }

.. setting:: DOWNLOAD_TIMEOUT

DOWNLOAD_TIMEOUT
----------------

デフォルト: ``180``

ダウンローダーがタイムアウトするまで待機する時間（秒単位）. 

.. note::

    このタイムアウトは,  :attr:`download_timeout`
    スパイダ属性と :reqmeta:`download_timeout`
    Request.meta キーを使用したリクエストごとに設定できます.

.. setting:: DOWNLOAD_MAXSIZE

DOWNLOAD_MAXSIZE
----------------

デフォルト: `1073741824` (1024MB)

ダウンローダがダウンロードする最大応答サイズ（バイト単位）.

無効にする場合は, 0に設定します.

.. reqmeta:: download_maxsize

.. note::

    このサイズは,  :attr:`download_maxsize` Request.metaキーを使用して, 
    :attr:`download_maxsize` スパイダ属性とper-requestを使用してスパイダごとに設定できます. 

    この機能には Twisted >= 11.1 が必要です.

.. setting:: DOWNLOAD_WARNSIZE

DOWNLOAD_WARNSIZE
-----------------

デフォルト: `33554432` (32MB)

ダウンローダが警告を開始するレスポンスサイズ（バイト単位）.

無効にする場合は、0に設定します.

.. note::

    このサイズは、 :attr:`download_warnsize` Request.metaキーを使用して、
    :reqmeta:`download_warnsize` スパイダ属性およびper-requestを使用してスパイダごとに設定できます。

    これには、 Twisted >= 11.1 が必要です.

.. setting:: DUPEFILTER_CLASS

DUPEFILTER_CLASS
----------------

デフォルト: ``'scrapy.dupefilters.RFPDupeFilter'``

重複リクエストを検出してフィルタリングするために使用されるクラス.

デフォルトのフィルタ (``RFPDupeFilter``) は、 
``scrapy.utils.request.request_fingerprint`` 関数を使用してリクエストフィンガープリントに基づいてフィルタリングします. 
重複をチェックする方法を変更するには、 ``RFPDupeFilter`` をサブクラス化し、
``request_fingerprint`` メソッドをオーバーライドすることができます. 
このメソッドは :class:`~scrapy.http.Request` オブジェクトを受け入れ、そのフィンガープリント（文字列）を返す必要があります.

.. setting:: DUPEFILTER_DEBUG

DUPEFILTER_DEBUG
----------------

デフォルト: ``False``

デフォルトでは, ``RFPDupeFilter`` は最初の重複要求のみを記録します.
:setting:`DUPEFILTER_DEBUG` を ``True`` に設定すると、すべての重複リクエストが記録されます.

.. setting:: EDITOR

EDITOR
------

デフォルト: `環境依存`

:command:`edit` マンドでスパイダーを編集するために使用するエディター. 
``EDITOR`` 環境変数が設定されていると、デフォルトで使用されます。
それ以外の場合、デフォルトは ``vi`` Unixシステムの場合）または IDLE エディター（Windowsの場合）になります.

.. setting:: EXTENSIONS

EXTENSIONS
----------

デフォルト:: ``{}``

プロジェクトで有効になっている拡張機能と、そのオーダーを含む ``dict``.

.. setting:: EXTENSIONS_BASE

EXTENSIONS_BASE
---------------

デフォルト::

    {
        'scrapy.extensions.corestats.CoreStats': 0,
        'scrapy.extensions.telnet.TelnetConsole': 0,
        'scrapy.extensions.memusage.MemoryUsage': 0,
        'scrapy.extensions.memdebug.MemoryDebugger': 0,
        'scrapy.extensions.closespider.CloseSpider': 0,
        'scrapy.extensions.feedexport.FeedExporter': 0,
        'scrapy.extensions.logstats.LogStats': 0,
        'scrapy.extensions.spiderstate.SpiderState': 0,
        'scrapy.extensions.throttle.AutoThrottle': 0,
    }

Scrapy のデフォルトで利用可能な拡張機能とそのオーダーを含む ``dict`` . 
この設定には、すべての安定したビルトイン拡張機能が含まれています。
そのうちのいくつかは設定によって有効にする必要があることに注意してください.

詳細については、 :ref:`拡張機能ユーザーガイド  <topics-extensions>`
及び :ref:`使用可能な拡張機能の一覧 <topics-extensions-ref>` を参照してください.


.. setting:: FEED_TEMPDIR

FEED_TEMPDIR
------------

Feed Temp ディレクトリでは、 
:ref:`FTP フィードストレージ <topics-feed-storage-ftp>` と
:ref:`Amazon S3 <topics-feed-storage-s3>` にアップロードする前に、
クローラの一時ファイルを保存するカスタムフォルダを設定できます.


.. setting:: ITEM_PIPELINES

ITEM_PIPELINES
--------------

デフォルト: ``{}``

使用するアイテムパイプラインとその注文を含む ``dict`` 。
順序値は任意ですが、0〜1000の範囲で定義するのが通例です。
下位の注文は高次のオーダーの前に処理されます.

例::

   ITEM_PIPELINES = {
       'mybot.pipelines.validate.ValidateMyItem': 300,
       'mybot.pipelines.validate.StoreMyItem': 800,
   }

.. setting:: ITEM_PIPELINES_BASE

ITEM_PIPELINES_BASE
-------------------

デフォルト: ``{}``

Scrapy でデフォルトで有効になっているパイプラインを含む ``dict`` 。
プロジェクトでこの設定を変更しないでください。
代わりに :setting:`ITEM_PIPELINES` を変更してください

.. setting:: LOG_ENABLED

LOG_ENABLED
-----------

デフォルト: ``True``

ロギングを有効にするかどうか.

.. setting:: LOG_ENCODING

LOG_ENCODING
------------

デフォルト: ``'utf-8'``

ロギングに使用するエンコーディング.

.. setting:: LOG_FILE

LOG_FILE
--------

デフォルト: ``None``

ログ出力に使用するファイル名. ``None`` の場合, 標準エラーが使用されます.

.. setting:: LOG_FORMAT

LOG_FORMAT
----------

デフォルト: ``'%(asctime)s [%(name)s] %(levelname)s: %(message)s'``

String for formatting log messsages. Refer to the `Python logging documentation`_ for the whole list of available
placeholders.

.. _Python logging documentation: https://docs.python.org/2/library/logging.html#logrecord-attributes

.. setting:: LOG_DATEFORMAT

LOG_DATEFORMAT
--------------

デフォルト: ``'%Y-%m-%d %H:%M:%S'``

日付/時刻の書式設定の文字列,  :setting:`LOG_FORMAT` の ``%(asctime)s`` のプレースホルダの展開。
使用可能なディレクティブの一覧については、 `Python datetime documentation`_ を参照してください.

.. _Python datetime documentation: https://docs.python.org/2/library/datetime.html#strftime-and-strptime-behavior

.. setting:: LOG_LEVEL

LOG_LEVEL
---------

デフォルト: ``'DEBUG'``

グに記録する最小レベル。使用可能なレベルは、CRITICAL、ERROR、WARNING、INFO、DEBUGです. 
詳細は :ref:`topics-logging` を参照してください.

.. setting:: LOG_STDOUT

LOG_STDOUT
----------

デフォルト: ``False``

``True`` の場合, プロセスのすべての標準出力（およびエラー）がログにリダイレクトされます. 
たとえば、 ``print 'hello'`` を実行すると、Scrapyログに表示されます.

.. setting:: MEMDEBUG_ENABLED

MEMDEBUG_ENABLED
----------------

デフォルト: ``False``

メモリデバッグを有効にするかどうか.

.. setting:: MEMDEBUG_NOTIFY

MEMDEBUG_NOTIFY
---------------

デフォルト: ``[]``

メモリデバッグが有効になっている場合、この設定が空でなければ、指定されたアドレスにメモリレポートが送信されます。
そうでない場合、レポートはログに書き込まれます.

例::

    MEMDEBUG_NOTIFY = ['user@example.com']

.. setting:: MEMUSAGE_ENABLED

MEMUSAGE_ENABLED
----------------

デフォルト: ``False``

スコープ: ``scrapy.extensions.memusage``

メモリ使用量の拡張を有効にして、メモリ制限を超えた場合に Scrapy プロセスをシャットダウンするかどうか。
また、その場合は電子メールで通知されます.

:ref:`topics-extensions-ref-memusage` を参照してください.

.. setting:: MEMUSAGE_LIMIT_MB

MEMUSAGE_LIMIT_MB
-----------------

デフォルト: ``0``

スコープ: ``scrapy.extensions.memusage``

(MEMUSAGE_ENABLEDがTrueの場合）Scrapy をシャットダウンする前に許容される
メモリの最大量（メガバイト単位）。ゼロの場合、チェックは実行されません.

:ref:`topics-extensions-ref-memusage` を参照してください.

.. setting:: MEMUSAGE_CHECK_INTERVAL_SECONDS

MEMUSAGE_CHECK_INTERVAL_SECONDS
-------------------------------

.. versionadded:: 1.1

デフォルト: ``60.0``

スコープ: ``scrapy.extensions.memusage``

:ref:`メモリ使用量拡張機能 <topics-extensions-ref-memusage>` は、
:setting:`MEMUSAGE_LIMIT_MB` および :setting:`MEMUSAGE_WARNING_MB` 
によって設定された制限と現在のメモリ使用量を、一定の時間間隔でチェックします.

これは、これらの間隔の長さを秒単位で設定します.

:ref:`topics-extensions-ref-memusage` を参照してください.

.. setting:: MEMUSAGE_NOTIFY_MAIL

MEMUSAGE_NOTIFY_MAIL
--------------------

デフォルト: ``False``

スコープ: ``scrapy.extensions.memusage``

モリー制限に達したことを通知する電子メールのリスト.

例::

    MEMUSAGE_NOTIFY_MAIL = ['user@example.com']

:ref:`topics-extensions-ref-memusage` を参照してください.

.. setting:: MEMUSAGE_REPORT

MEMUSAGE_REPORT
---------------

デフォルト: ``False``

スコープ: ``scrapy.extensions.memusage``

各スパイダーが閉じられた後にメモリ使用量レポートを送信するかどうか.

:ref:`topics-extensions-ref-memusage` を参照してください.

.. setting:: MEMUSAGE_WARNING_MB

MEMUSAGE_WARNING_MB
-------------------

デフォルト: ``0``

スコープ: ``scrapy.extensions.memusage``

通知する警告電子メールを送信する前に許容されるメモリの最大量（メガバイト単位）。
ゼロの場合、警告は生成されません.

.. setting:: NEWSPIDER_MODULE

NEWSPIDER_MODULE
----------------

デフォルト: ``''``

:command:`genspider` コマンドを使用して新しいスパイダーを作成する場所.

例::

    NEWSPIDER_MODULE = 'mybot.spiders_dev'

.. setting:: RANDOMIZE_DOWNLOAD_DELAY

RANDOMIZE_DOWNLOAD_DELAY
------------------------

デフォルト: ``True``

有効にすると、同じWebサイトからリクエストを取得する間、Scrapyはランダムな時間
(0.5 * :setting:`DOWNLOAD_DELAY` から 1.5 * :setting:`DOWNLOAD_DELAY` の間まで) 
待機します.

このランダム化は、リクエスト間の時間から統計的に有意な類似性を探し、
リクエストを分析するサイトによってクローラが検出される（およびその後ブロックされる）可能性を低下させます.

ランダム化ポリシーは、 `wget`_ ``--random-wait`` オプションで使用されるものと同じです.

:setting:`DOWNLOAD_DELAY` が 0（デフォルト）の場合、このオプションは無効です.

.. _wget: http://www.gnu.org/software/wget/manual/wget.html

.. setting:: REACTOR_THREADPOOL_MAXSIZE

REACTOR_THREADPOOL_MAXSIZE
--------------------------

デフォルト: ``10``

Twisted Reactor スレッドプールサイズの最大限度。
これは、さまざまな Scrapy コンポーネントで使用される汎用スレッドプールです。
スレッド化された DNS リゾルバ、BlockingFeedStorage、S3 FilesStore などです。
ブロッキング IO が不十分で問題が発生している場合は、この値を大きくしてください。

.. setting:: REDIRECT_MAX_TIMES

REDIRECT_MAX_TIMES
------------------

デフォルト: ``20``

要求をリダイレクトできる最大時間を定義します。
この最大値の後、要求の応答はそのまま返されます。
私達は、Firefoxのデフォルト値を同じタスクに使用しました.

.. setting:: REDIRECT_PRIORITY_ADJUST

REDIRECT_PRIORITY_ADJUST
------------------------

デフォルト: ``+2``

スコープ: ``scrapy.downloadermiddlewares.redirect.RedirectMiddleware``

元のリクエストに対するリダイレクトリクエストの優先度を調整します。

- **正の優先度調整（デフォルト）はより高い優先度を意味します.**
- 負の優先順位調整は低い優先順位を意味します.

.. setting:: RETRY_PRIORITY_ADJUST

RETRY_PRIORITY_ADJUST
---------------------

デフォルト: ``-1``

スコープ: ``scrapy.downloadermiddlewares.retry.RetryMiddleware``

オリジナルのリクエストに対するリトライリクエストの優先度を調整する:

- 正の優先度調整はより高い優先度を意味します.
- **負の優先順位調整（デフォルト）は低い優先順位を意味します.**

.. setting:: ROBOTSTXT_OBEY

ROBOTSTXT_OBEY
--------------

デフォルト: ``False``

スコープ: ``scrapy.downloadermiddlewares.robotstxt``

有効にすると、Scrapy は robots.txt ポリシーを尊重します。詳細については、
:ref:`topics-dlmw-robots` を参照してください.

.. note::

    歴史的な理由からデフォルト値は ``False`` ですが、このオプションは 
    ``scrapy startproject`` コマンドで生成される settings.py ファイルではデフォルトで有効になっています.

.. setting:: SCHEDULER

SCHEDULER
---------

デフォルト: ``'scrapy.core.scheduler.Scheduler'``

クロールに使用するスケジューラ.

.. setting:: SCHEDULER_DEBUG

SCHEDULER_DEBUG
---------------

デフォルト: ``False``

Setting to ``True`` will log debug information about the requests scheduler.
This currently logs (only once) if the requests cannot be serialized to disk.
Stats counter (``scheduler/unserializable``) tracks the number of times this happens.

Example entry in logs::

    1956-01-31 00:00:00+0800 [scrapy] ERROR: Unable to serialize request:
    <GET http://example.com> - reason: cannot serialize <Request at 0x9a7c7ec>
    (type Request)> - no more unserializable requests will be logged
    (see 'scheduler/unserializable' stats counter)


.. setting:: SCHEDULER_DISK_QUEUE

SCHEDULER_DISK_QUEUE
--------------------

デフォルト: ``'scrapy.squeues.PickleLifoDiskQueue'``

Type of disk queue that will be used by scheduler. Other available types are
``scrapy.squeues.PickleFifoDiskQueue``, ``scrapy.squeues.MarshalFifoDiskQueue``,
``scrapy.squeues.MarshalLifoDiskQueue``.

.. setting:: SCHEDULER_MEMORY_QUEUE

SCHEDULER_MEMORY_QUEUE
----------------------
デフォルト: ``'scrapy.squeues.LifoMemoryQueue'``

Type of in-memory queue used by scheduler. Other available type is:
``scrapy.squeues.FifoMemoryQueue``.

.. setting:: SCHEDULER_PRIORITY_QUEUE

SCHEDULER_PRIORITY_QUEUE
------------------------
デフォルト: ``'queuelib.PriorityQueue'``

Type of priority queue used by scheduler.

.. setting:: SPIDER_CONTRACTS

SPIDER_CONTRACTS
----------------

デフォルト:: ``{}``

A dict containing the spider contracts enabled in your project, used for
testing spiders. For more info see :ref:`topics-contracts`.

.. setting:: SPIDER_CONTRACTS_BASE

SPIDER_CONTRACTS_BASE
---------------------

デフォルト::

    {
        'scrapy.contracts.default.UrlContract' : 1,
        'scrapy.contracts.default.ReturnsContract': 2,
        'scrapy.contracts.default.ScrapesContract': 3,
    }

A dict containing the scrapy contracts enabled by default in Scrapy. You should
never modify this setting in your project, modify :setting:`SPIDER_CONTRACTS`
instead. For more info see :ref:`topics-contracts`.

You can disable any of these contracts by assigning ``None`` to their class
path in :setting:`SPIDER_CONTRACTS`. E.g., to disable the built-in
``ScrapesContract``, place this in your ``settings.py``::

    SPIDER_CONTRACTS = {
        'scrapy.contracts.default.ScrapesContract': None,
    }

.. setting:: SPIDER_LOADER_CLASS

SPIDER_LOADER_CLASS
-------------------

デフォルト: ``'scrapy.spiderloader.SpiderLoader'``

The class that will be used for loading spiders, which must implement the
:ref:`topics-api-spiderloader`.

.. setting:: SPIDER_MIDDLEWARES

SPIDER_MIDDLEWARES
------------------

デフォルト:: ``{}``

A dict containing the spider middlewares enabled in your project, and their
orders. For more info see :ref:`topics-spider-middleware-setting`.

.. setting:: SPIDER_MIDDLEWARES_BASE

SPIDER_MIDDLEWARES_BASE
-----------------------

デフォルト::

    {
        'scrapy.spidermiddlewares.httperror.HttpErrorMiddleware': 50,
        'scrapy.spidermiddlewares.offsite.OffsiteMiddleware': 500,
        'scrapy.spidermiddlewares.referer.RefererMiddleware': 700,
        'scrapy.spidermiddlewares.urllength.UrlLengthMiddleware': 800,
        'scrapy.spidermiddlewares.depth.DepthMiddleware': 900,
    }

A dict containing the spider middlewares enabled by default in Scrapy, and
their orders. Low orders are closer to the engine, high orders are closer to
the spider. For more info see :ref:`topics-spider-middleware-setting`.

.. setting:: SPIDER_MODULES

SPIDER_MODULES
--------------

デフォルト: ``[]``

A list of modules where Scrapy will look for spiders.

Example::

    SPIDER_MODULES = ['mybot.spiders_prod', 'mybot.spiders_dev']

.. setting:: STATS_CLASS

STATS_CLASS
-----------

デフォルト: ``'scrapy.statscollectors.MemoryStatsCollector'``

The class to use for collecting stats, who must implement the
:ref:`topics-api-stats`.

.. setting:: STATS_DUMP

STATS_DUMP
----------

デフォルト: ``True``

Dump the :ref:`Scrapy stats <topics-stats>` (to the Scrapy log) once the spider
finishes.

For more info see: :ref:`topics-stats`.

.. setting:: STATSMAILER_RCPTS

STATSMAILER_RCPTS
-----------------

デフォルト: ``[]`` (empty list)

Send Scrapy stats after spiders finish scraping. See
:class:`~scrapy.extensions.statsmailer.StatsMailer` for more info.

.. setting:: TELNETCONSOLE_ENABLED

TELNETCONSOLE_ENABLED
---------------------

デフォルト: ``True``

A boolean which specifies if the :ref:`telnet console <topics-telnetconsole>`
will be enabled (provided its extension is also enabled).

.. setting:: TELNETCONSOLE_PORT

TELNETCONSOLE_PORT
------------------

デフォルト: ``[6023, 6073]``

The port range to use for the telnet console. If set to ``None`` or ``0``, a
dynamically assigned port is used. For more info see
:ref:`topics-telnetconsole`.

.. setting:: TEMPLATES_DIR

TEMPLATES_DIR
-------------

デフォルト: ``templates`` dir inside scrapy module

The directory where to look for templates when creating new projects with
:command:`startproject` command and new spiders with :command:`genspider`
command.

The project name must not conflict with the name of custom files or directories
in the ``project`` subdirectory.


.. setting:: URLLENGTH_LIMIT

URLLENGTH_LIMIT
---------------

デフォルト: ``2083``

Scope: ``spidermiddlewares.urllength``

The maximum URL length to allow for crawled URLs. For more information about
the default value for this setting see: http://www.boutell.com/newfaq/misc/urllength.html

.. setting:: USER_AGENT

USER_AGENT
----------

デフォルト: ``"Scrapy/VERSION (+http://scrapy.org)"``

The default User-Agent to use when crawling, unless overridden.


Settings documented elsewhere:
------------------------------

The following settings are documented elsewhere, please check each specific
case to see how to enable and use them.

.. settingslist::


.. _Amazon web services: https://aws.amazon.com/
.. _breadth-first order: https://en.wikipedia.org/wiki/Breadth-first_search
.. _depth-first order: https://en.wikipedia.org/wiki/Depth-first_search
