.. _topics-settings:

========
設定
========

Scrapyの設定では, コア, 拡張機能, パイプライン, スパイダー自体を含むすべての Scrapy コンポーネントの動作をカスタマイズすることができます.

設定のインフラストラクチャは, コードで値を取得するために使用できる, キーと値のマッピングのグローバルな名前空間を提供します. 
この設定は、以下で説明するさまざまなメカニズムによって設定できます.

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

プロジェクト設定モジュールは、Scrapyプロジェクトの標準設定ファイルです. 
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
    The ``settings`` attribute is set in the base Spider class after the spider
    is initialized.  If you want to use the settings before the initialization
    (e.g., in your spider's ``__init__()`` method), you'll need to override the
    :meth:`~scrapy.spiders.Spider.from_crawler` method.

Settings can be accessed through the :attr:`scrapy.crawler.Crawler.settings`
attribute of the Crawler that is passed to ``from_crawler`` method in
extensions, middlewares and item pipelines::

    class MyExtension(object):
        def __init__(self, log_is_enabled=False):
            if log_is_enabled:
                print("log is enabled!")

        @classmethod
        def from_crawler(cls, crawler):
            settings = crawler.settings
            return cls(settings.getbool('LOG_ENABLED'))

The settings object can be used like a dict (e.g.,
``settings['LOG_ENABLED']``), but it's usually preferred to extract the setting
in the format you need it to avoid type errors, using one of the methods
provided by the :class:`~scrapy.settings.Settings` API.

名前を設定する理由
===========================

設定名には通常, 構成するコンポーネントの接頭辞が付いています. 
例えば, 架空の ``robots.txt`` 拡張子の適切な設定名は
``ROBOTSTXT_ENABLED``, ``ROBOTSTXT_OBEY``, ``ROBOTSTXT_CACHEDIR`` などです.


.. _topics-settings-ref:

ビルトイン設定リファレンス
===========================

ここでは, アルファベット順に使用可能なすべてのScrapy設定のリストと, デフォルト値, 適用されるスコープが示されています.

The scope, where available, shows where the setting is being used, if it's tied
to any particular component. In that case the module of that component will be
shown, typically an extension, middleware or pipeline. It also means that the
component must be enabled in order for the setting to have any effect.

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
これは、デフォルトで User-Agent, ロギングに使用されます.

:command:`startproject` コマンドを使用してプロジェクトを作成すると, プロジェクト名が自動的に入力されます.

.. setting:: CONCURRENT_ITEMS

CONCURRENT_ITEMS
----------------

デフォルト: ``100``

Maximum number of concurrent items (per response) to process in parallel in the
Item Processor (also known as the :ref:`Item Pipeline <topics-item-pipeline>`).

.. setting:: CONCURRENT_REQUESTS

CONCURRENT_REQUESTS
-------------------

デフォルト: ``16``

The maximum number of concurrent (ie. simultaneous) requests that will be
performed by the Scrapy downloader.

.. setting:: CONCURRENT_REQUESTS_PER_DOMAIN

CONCURRENT_REQUESTS_PER_DOMAIN
------------------------------

デフォルト: ``8``

The maximum number of concurrent (ie. simultaneous) requests that will be
performed to any single domain.

See also: :ref:`topics-autothrottle` and its
:setting:`AUTOTHROTTLE_TARGET_CONCURRENCY` option.


.. setting:: CONCURRENT_REQUESTS_PER_IP

CONCURRENT_REQUESTS_PER_IP
--------------------------

デフォルト: ``0``

The maximum number of concurrent (ie. simultaneous) requests that will be
performed to any single IP. If non-zero, the
:setting:`CONCURRENT_REQUESTS_PER_DOMAIN` setting is ignored, and this one is
used instead. In other words, concurrency limits will be applied per IP, not
per domain.

This setting also affects :setting:`DOWNLOAD_DELAY` and
:ref:`topics-autothrottle`: if :setting:`CONCURRENT_REQUESTS_PER_IP`
is non-zero, download delay is enforced per IP, not per domain.


.. setting:: DEFAULT_ITEM_CLASS

DEFAULT_ITEM_CLASS
------------------

デフォルト: ``'scrapy.item.Item'``

The default class that will be used for instantiating items in the :ref:`the
Scrapy shell <topics-shell>`.

.. setting:: DEFAULT_REQUEST_HEADERS

DEFAULT_REQUEST_HEADERS
-----------------------

デフォルト::

    {
        'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
        'Accept-Language': 'en',
    }

The default headers used for Scrapy HTTP Requests. They're populated in the
:class:`~scrapy.downloadermiddlewares.defaultheaders.DefaultHeadersMiddleware`.

.. setting:: DEPTH_LIMIT

DEPTH_LIMIT
-----------

デフォルト: ``0``

スコープ: ``scrapy.spidermiddlewares.depth.DepthMiddleware``

The maximum depth that will be allowed to crawl for any site. If zero, no limit
will be imposed.

.. setting:: DEPTH_PRIORITY

DEPTH_PRIORITY
--------------

デフォルト: ``0``

スコープ: ``scrapy.spidermiddlewares.depth.DepthMiddleware``

An integer that is used to adjust the request priority based on its depth:

- if zero (default), no priority adjustment is made from depth
- **a positive value will decrease the priority, i.e. higher depth
  requests will be processed later** ; this is commonly used when doing
  breadth-first crawls (BFO)
- a negative value will increase priority, i.e., higher depth requests
  will be processed sooner (DFO)

See also: :ref:`faq-bfo-dfo` about tuning Scrapy for BFO or DFO.

.. note::

    This setting adjusts priority **in the opposite way** compared to
    other priority settings :setting:`REDIRECT_PRIORITY_ADJUST`
    and :setting:`RETRY_PRIORITY_ADJUST`.

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

Whether to collect verbose depth stats. If this is enabled, the number of
requests for each depth is collected in the stats.

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

Defines a Twisted ``protocol.ClientFactory``  class to use for HTTP/1.0
connections (for ``HTTP10DownloadHandler``).

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

Represents the classpath to the ContextFactory to use.

Here, "ContextFactory" is a Twisted term for SSL/TLS contexts, defining
the TLS/SSL protocol version to use, whether to do certificate verification,
or even enable client-side authentication (and various other things).

.. note::

    Scrapy default context factory **does NOT perform remote server
    certificate verification**. This is usually fine for web scraping.

    If you do need remote server certificate verification enabled,
    Scrapy also has another context factory class that you can set,
    ``'scrapy.core.downloader.contextfactory.BrowserLikeContextFactory'``,
    which uses the platform's certificates to validate remote endpoints.
    **This is only available if you use Twisted>=14.0.**

If you do use a custom ContextFactory, make sure it accepts a ``method``
parameter at init (this is the ``OpenSSL.SSL`` method mapping
:setting:`DOWNLOADER_CLIENT_TLS_METHOD`).

.. setting:: DOWNLOADER_CLIENT_TLS_METHOD

DOWNLOADER_CLIENT_TLS_METHOD
----------------------------

デフォルト: ``'TLS'``

Use this setting to customize the TLS/SSL method used by the default
HTTP/1.1 downloader.

This setting must be one of these string values:

- ``'TLS'``: maps to OpenSSL's ``TLS_method()`` (a.k.a ``SSLv23_method()``),
  which allows protocol negotiation, starting from the highest supported
  by the platform; **default, recommended**
- ``'TLSv1.0'``: this value forces HTTPS connections to use TLS version 1.0 ;
  set this if you want the behavior of Scrapy<1.1
- ``'TLSv1.1'``: forces TLS version 1.1
- ``'TLSv1.2'``: forces TLS version 1.2
- ``'SSLv3'``: forces SSL version 3 (**not recommended**)

.. note::

    We recommend that you use PyOpenSSL>=0.13 and Twisted>=0.13
    or above (Twisted>=14.0 if you can).

.. setting:: DOWNLOADER_MIDDLEWARES

DOWNLOADER_MIDDLEWARES
----------------------

デフォルト:: ``{}``

A dict containing the downloader middlewares enabled in your project, and their
orders. For more info see :ref:`topics-downloader-middleware-setting`.

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

A dict containing the downloader middlewares enabled by default in Scrapy. Low
orders are closer to the engine, high orders are closer to the downloader. You
should never modify this setting in your project, modify
:setting:`DOWNLOADER_MIDDLEWARES` instead.  For more info see
:ref:`topics-downloader-middleware-setting`.

.. setting:: DOWNLOADER_STATS

DOWNLOADER_STATS
----------------

デフォルト: ``True``

Whether to enable downloader stats collection.

.. setting:: DOWNLOAD_DELAY

DOWNLOAD_DELAY
--------------

デフォルト: ``0``

The amount of time (in secs) that the downloader should wait before downloading
consecutive pages from the same website. This can be used to throttle the
crawling speed to avoid hitting servers too hard. Decimal numbers are
supported.  Example::

    DOWNLOAD_DELAY = 0.25    # 250 ms of delay

This setting is also affected by the :setting:`RANDOMIZE_DOWNLOAD_DELAY`
setting (which is enabled by default). By default, Scrapy doesn't wait a fixed
amount of time between requests, but uses a random interval between 0.5 * :setting:`DOWNLOAD_DELAY` and 1.5 * :setting:`DOWNLOAD_DELAY`.

When :setting:`CONCURRENT_REQUESTS_PER_IP` is non-zero, delays are enforced
per ip address instead of per domain.

You can also change this setting per spider by setting ``download_delay``
spider attribute.

.. setting:: DOWNLOAD_HANDLERS

DOWNLOAD_HANDLERS
-----------------

デフォルト: ``{}``

A dict containing the request downloader handlers enabled in your project.
See :setting:`DOWNLOAD_HANDLERS_BASE` for example format.

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


A dict containing the request download handlers enabled by default in Scrapy.
You should never modify this setting in your project, modify
:setting:`DOWNLOAD_HANDLERS` instead.

You can disable any of these download handlers by assigning ``None`` to their
URI scheme in :setting:`DOWNLOAD_HANDLERS`. E.g., to disable the built-in FTP
handler (without replacement), place this in your ``settings.py``::

    DOWNLOAD_HANDLERS = {
        'ftp': None,
    }

.. setting:: DOWNLOAD_TIMEOUT

DOWNLOAD_TIMEOUT
----------------

デフォルト: ``180``

The amount of time (in secs) that the downloader will wait before timing out.

.. note::

    This timeout can be set per spider using :attr:`download_timeout`
    spider attribute and per-request using :reqmeta:`download_timeout`
    Request.meta key.

.. setting:: DOWNLOAD_MAXSIZE

DOWNLOAD_MAXSIZE
----------------

デフォルト: `1073741824` (1024MB)

The maximum response size (in bytes) that downloader will download.

If you want to disable it set to 0.

.. reqmeta:: download_maxsize

.. note::

    This size can be set per spider using :attr:`download_maxsize`
    spider attribute and per-request using :reqmeta:`download_maxsize`
    Request.meta key.

    This feature needs Twisted >= 11.1.

.. setting:: DOWNLOAD_WARNSIZE

DOWNLOAD_WARNSIZE
-----------------

デフォルト: `33554432` (32MB)

The response size (in bytes) that downloader will start to warn.

If you want to disable it set to 0.

.. note::

    This size can be set per spider using :attr:`download_warnsize`
    spider attribute and per-request using :reqmeta:`download_warnsize`
    Request.meta key.

    This feature needs Twisted >= 11.1.

.. setting:: DUPEFILTER_CLASS

DUPEFILTER_CLASS
----------------

デフォルト: ``'scrapy.dupefilters.RFPDupeFilter'``

The class used to detect and filter duplicate requests.

The default (``RFPDupeFilter``) filters based on request fingerprint using
the ``scrapy.utils.request.request_fingerprint`` function. In order to change
the way duplicates are checked you could subclass ``RFPDupeFilter`` and
override its ``request_fingerprint`` method. This method should accept
scrapy :class:`~scrapy.http.Request` object and return its fingerprint
(a string).

.. setting:: DUPEFILTER_DEBUG

DUPEFILTER_DEBUG
----------------

デフォルト: ``False``

By default, ``RFPDupeFilter`` only logs the first duplicate request.
Setting :setting:`DUPEFILTER_DEBUG` to ``True`` will make it log all duplicate requests.

.. setting:: EDITOR

EDITOR
------

デフォルト: `depends on the environment`

The editor to use for editing spiders with the :command:`edit` command. It
defaults to the ``EDITOR`` environment variable, if set. Otherwise, it defaults
to ``vi`` (on Unix systems) or the IDLE editor (on Windows).

.. setting:: EXTENSIONS

EXTENSIONS
----------

デフォルト:: ``{}``

A dict containing the extensions enabled in your project, and their orders.

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

A dict containing the extensions available by default in Scrapy, and their
orders. This setting contains all stable built-in extensions. Keep in mind that
some of them need to be enabled through a setting.

For more information See the :ref:`extensions user guide  <topics-extensions>`
and the :ref:`list of available extensions <topics-extensions-ref>`.


.. setting:: FEED_TEMPDIR

FEED_TEMPDIR
------------

The Feed Temp dir allows you to set a custom folder to save crawler
temporary files before uploading with :ref:`FTP feed storage <topics-feed-storage-ftp>` and
:ref:`Amazon S3 <topics-feed-storage-s3>`.


.. setting:: ITEM_PIPELINES

ITEM_PIPELINES
--------------

デフォルト: ``{}``

A dict containing the item pipelines to use, and their orders. Order values are
arbitrary, but it is customary to define them in the 0-1000 range. Lower orders
process before higher orders.

Example::

   ITEM_PIPELINES = {
       'mybot.pipelines.validate.ValidateMyItem': 300,
       'mybot.pipelines.validate.StoreMyItem': 800,
   }

.. setting:: ITEM_PIPELINES_BASE

ITEM_PIPELINES_BASE
-------------------

デフォルト: ``{}``

A dict containing the pipelines enabled by default in Scrapy. You should never
modify this setting in your project, modify :setting:`ITEM_PIPELINES` instead.

.. setting:: LOG_ENABLED

LOG_ENABLED
-----------

デフォルト: ``True``

Whether to enable logging.

.. setting:: LOG_ENCODING

LOG_ENCODING
------------

デフォルト: ``'utf-8'``

The encoding to use for logging.

.. setting:: LOG_FILE

LOG_FILE
--------

デフォルト: ``None``

File name to use for logging output. If ``None``, standard error will be used.

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

String for formatting date/time, expansion of the ``%(asctime)s`` placeholder
in :setting:`LOG_FORMAT`. Refer to the `Python datetime documentation`_ for the whole list of available
directives.

.. _Python datetime documentation: https://docs.python.org/2/library/datetime.html#strftime-and-strptime-behavior

.. setting:: LOG_LEVEL

LOG_LEVEL
---------

デフォルト: ``'DEBUG'``

Minimum level to log. Available levels are: CRITICAL, ERROR, WARNING,
INFO, DEBUG. For more info see :ref:`topics-logging`.

.. setting:: LOG_STDOUT

LOG_STDOUT
----------

デフォルト: ``False``

If ``True``, all standard output (and error) of your process will be redirected
to the log. For example if you ``print 'hello'`` it will appear in the Scrapy
log.

.. setting:: MEMDEBUG_ENABLED

MEMDEBUG_ENABLED
----------------

デフォルト: ``False``

Whether to enable memory debugging.

.. setting:: MEMDEBUG_NOTIFY

MEMDEBUG_NOTIFY
---------------

デフォルト: ``[]``

When memory debugging is enabled a memory report will be sent to the specified
addresses if this setting is not empty, otherwise the report will be written to
the log.

Example::

    MEMDEBUG_NOTIFY = ['user@example.com']

.. setting:: MEMUSAGE_ENABLED

MEMUSAGE_ENABLED
----------------

デフォルト: ``False``

Scope: ``scrapy.extensions.memusage``

Whether to enable the memory usage extension that will shutdown the Scrapy
process when it exceeds a memory limit, and also notify by email when that
happened.

See :ref:`topics-extensions-ref-memusage`.

.. setting:: MEMUSAGE_LIMIT_MB

MEMUSAGE_LIMIT_MB
-----------------

デフォルト: ``0``

Scope: ``scrapy.extensions.memusage``

The maximum amount of memory to allow (in megabytes) before shutting down
Scrapy  (if MEMUSAGE_ENABLED is True). If zero, no check will be performed.

See :ref:`topics-extensions-ref-memusage`.

.. setting:: MEMUSAGE_CHECK_INTERVAL_SECONDS

MEMUSAGE_CHECK_INTERVAL_SECONDS
-------------------------------

.. versionadded:: 1.1

デフォルト: ``60.0``

Scope: ``scrapy.extensions.memusage``

The :ref:`Memory usage extension <topics-extensions-ref-memusage>`
checks the current memory usage, versus the limits set by
:setting:`MEMUSAGE_LIMIT_MB` and :setting:`MEMUSAGE_WARNING_MB`,
at fixed time intervals.

This sets the length of these intervals, in seconds.

See :ref:`topics-extensions-ref-memusage`.

.. setting:: MEMUSAGE_NOTIFY_MAIL

MEMUSAGE_NOTIFY_MAIL
--------------------

デフォルト: ``False``

Scope: ``scrapy.extensions.memusage``

A list of emails to notify if the memory limit has been reached.

Example::

    MEMUSAGE_NOTIFY_MAIL = ['user@example.com']

See :ref:`topics-extensions-ref-memusage`.

.. setting:: MEMUSAGE_REPORT

MEMUSAGE_REPORT
---------------

デフォルト: ``False``

Scope: ``scrapy.extensions.memusage``

Whether to send a memory usage report after each spider has been closed.

See :ref:`topics-extensions-ref-memusage`.

.. setting:: MEMUSAGE_WARNING_MB

MEMUSAGE_WARNING_MB
-------------------

デフォルト: ``0``

Scope: ``scrapy.extensions.memusage``

The maximum amount of memory to allow (in megabytes) before sending a warning
email notifying about it. If zero, no warning will be produced.

.. setting:: NEWSPIDER_MODULE

NEWSPIDER_MODULE
----------------

デフォルト: ``''``

Module where to create new spiders using the :command:`genspider` command.

Example::

    NEWSPIDER_MODULE = 'mybot.spiders_dev'

.. setting:: RANDOMIZE_DOWNLOAD_DELAY

RANDOMIZE_DOWNLOAD_DELAY
------------------------

デフォルト: ``True``

If enabled, Scrapy will wait a random amount of time (between 0.5 * :setting:`DOWNLOAD_DELAY` and 1.5 * :setting:`DOWNLOAD_DELAY`) while fetching requests from the same
website.

This randomization decreases the chance of the crawler being detected (and
subsequently blocked) by sites which analyze requests looking for statistically
significant similarities in the time between their requests.

The randomization policy is the same used by `wget`_ ``--random-wait`` option.

If :setting:`DOWNLOAD_DELAY` is zero (default) this option has no effect.

.. _wget: http://www.gnu.org/software/wget/manual/wget.html

.. setting:: REACTOR_THREADPOOL_MAXSIZE

REACTOR_THREADPOOL_MAXSIZE
--------------------------

デフォルト: ``10``

The maximum limit for Twisted Reactor thread pool size. This is common
multi-purpose thread pool used by various Scrapy components. Threaded
DNS Resolver, BlockingFeedStorage, S3FilesStore just to name a few. Increase
this value if you're experiencing problems with insufficient blocking IO.

.. setting:: REDIRECT_MAX_TIMES

REDIRECT_MAX_TIMES
------------------

デフォルト: ``20``

Defines the maximum times a request can be redirected. After this maximum the
request's response is returned as is. We used Firefox default value for the
same task.

.. setting:: REDIRECT_PRIORITY_ADJUST

REDIRECT_PRIORITY_ADJUST
------------------------

デフォルト: ``+2``

Scope: ``scrapy.downloadermiddlewares.redirect.RedirectMiddleware``

Adjust redirect request priority relative to original request:

- **a positive priority adjust (default) means higher priority.**
- a negative priority adjust means lower priority.

.. setting:: RETRY_PRIORITY_ADJUST

RETRY_PRIORITY_ADJUST
---------------------

デフォルト: ``-1``

Scope: ``scrapy.downloadermiddlewares.retry.RetryMiddleware``

Adjust retry request priority relative to original request:

- a positive priority adjust means higher priority.
- **a negative priority adjust (default) means lower priority.**

.. setting:: ROBOTSTXT_OBEY

ROBOTSTXT_OBEY
--------------

デフォルト: ``False``

Scope: ``scrapy.downloadermiddlewares.robotstxt``

If enabled, Scrapy will respect robots.txt policies. For more information see
:ref:`topics-dlmw-robots`.

.. note::

    While the default value is ``False`` for historical reasons,
    this option is enabled by default in settings.py file generated
    by ``scrapy startproject`` command.

.. setting:: SCHEDULER

SCHEDULER
---------

デフォルト: ``'scrapy.core.scheduler.Scheduler'``

The scheduler to use for crawling.

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
