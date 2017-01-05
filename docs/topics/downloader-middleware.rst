.. _topics-downloader-middleware:

=====================
ダウンローダーミドルウェア
=====================

ダウンローダーミドルウェアは, Scrapy のリクエスト/レスポンス処理へフックするフレームワークです.  
Scrapy のリクエストとレスポンスをグローバルに変更するための軽量で低レベルのシステムです.

.. _topics-downloader-middleware-setting:

ダウンローダーミドルウェアの有効化
==================================

To activate a downloader middleware component, add it to the
:setting:`DOWNLOADER_MIDDLEWARES` setting, which is a dict whose keys are the
middleware class paths and their values are the middleware orders.

例::

    DOWNLOADER_MIDDLEWARES = {
        'myproject.middlewares.CustomDownloaderMiddleware': 543,
    }

The :setting:`DOWNLOADER_MIDDLEWARES` setting is merged with the
:setting:`DOWNLOADER_MIDDLEWARES_BASE` setting defined in Scrapy (and not meant
to be overridden) and then sorted by order to get the final sorted list of
enabled middlewares: the first middleware is the one closer to the engine and
the last is the one closer to the downloader. In other words,
the :meth:`~scrapy.downloadermiddlewares.DownloaderMiddleware.process_request`
method of each middleware will be invoked in increasing
middleware order (100, 200, 300, ...) and the :meth:`~scrapy.downloadermiddlewares.DownloaderMiddleware.process_response` method
of each middleware will be invoked in decreasing order.

To decide which order to assign to your middleware see the
:setting:`DOWNLOADER_MIDDLEWARES_BASE` setting and pick a value according to
where you want to insert the middleware. The order does matter because each
middleware performs a different action and your middleware could depend on some
previous (or subsequent) middleware being applied.

If you want to disable a built-in middleware (the ones defined in
:setting:`DOWNLOADER_MIDDLEWARES_BASE` and enabled by default) you must define it
in your project's :setting:`DOWNLOADER_MIDDLEWARES` setting and assign `None`
as its value.  For example, if you want to disable the user-agent middleware::

    DOWNLOADER_MIDDLEWARES = {
        'myproject.middlewares.CustomDownloaderMiddleware': 543,
        'scrapy.downloadermiddlewares.useragent.UserAgentMiddleware': None,
    }

Finally, keep in mind that some middlewares may need to be enabled through a
particular setting. See each middleware documentation for more info.

独自のダウンローダーミドルウェアの作成
======================================

各ミドルウェアコンポーネントは, 以下のメソッドの1つ以上を定義する Python クラスです:

.. module:: scrapy.downloadermiddlewares

.. class:: DownloaderMiddleware

   .. note::  ダウンローダーミドルウェアメソッドのいずれも, 遅延したものを返す可能性があります.

   .. method:: process_request(request, spider)

      This method is called for each request that goes through the download
      middleware.

      :meth:`process_request` should either: return ``None``, return a
      :class:`~scrapy.http.Response` object, return a :class:`~scrapy.http.Request`
      object, or raise :exc:`~scrapy.exceptions.IgnoreRequest`.

      If it returns ``None``, Scrapy will continue processing this request, executing all
      other middlewares until, finally, the appropriate downloader handler is called
      the request performed (and its response downloaded).

      If it returns a :class:`~scrapy.http.Response` object, Scrapy won't bother
      calling *any* other :meth:`process_request` or :meth:`process_exception` methods,
      or the appropriate download function; it'll return that response. The :meth:`process_response`
      methods of installed middleware is always called on every response.

      If it returns a :class:`~scrapy.http.Request` object, Scrapy will stop calling
      process_request methods and reschedule the returned request. Once the newly returned
      request is performed, the appropriate middleware chain will be called on
      the downloaded response.

      If it raises an :exc:`~scrapy.exceptions.IgnoreRequest` exception, the
      :meth:`process_exception` methods of installed downloader middleware will be called.
      If none of them handle the exception, the errback function of the request
      (``Request.errback``) is called. If no code handles the raised exception, it is
      ignored and not logged (unlike other exceptions).

      :param request: the request being processed
      :type request: :class:`~scrapy.http.Request` object

      :param spider: the spider for which this request is intended
      :type spider: :class:`~scrapy.spiders.Spider` object

   .. method:: process_response(request, response, spider)

      :meth:`process_response` should either: return a :class:`~scrapy.http.Response`
      object, return a :class:`~scrapy.http.Request` object or
      raise a :exc:`~scrapy.exceptions.IgnoreRequest` exception.

      If it returns a :class:`~scrapy.http.Response` (it could be the same given
      response, or a brand-new one), that response will continue to be processed
      with the :meth:`process_response` of the next middleware in the chain.

      If it returns a :class:`~scrapy.http.Request` object, the middleware chain is
      halted and the returned request is rescheduled to be downloaded in the future.
      This is the same behavior as if a request is returned from :meth:`process_request`.

      If it raises an :exc:`~scrapy.exceptions.IgnoreRequest` exception, the errback
      function of the request (``Request.errback``) is called. If no code handles the raised
      exception, it is ignored and not logged (unlike other exceptions).

      :param request: the request that originated the response
      :type request: is a :class:`~scrapy.http.Request` object

      :param response: the response being processed
      :type response: :class:`~scrapy.http.Response` object

      :param spider: the spider for which this response is intended
      :type spider: :class:`~scrapy.spiders.Spider` object

   .. method:: process_exception(request, exception, spider)

      Scrapy calls :meth:`process_exception` when a download handler
      or a :meth:`process_request` (from a downloader middleware) raises an
      exception (including an :exc:`~scrapy.exceptions.IgnoreRequest` exception)

      :meth:`process_exception` should return: either ``None``,
      a :class:`~scrapy.http.Response` object, or a :class:`~scrapy.http.Request` object.

      If it returns ``None``, Scrapy will continue processing this exception,
      executing any other :meth:`process_exception` methods of installed middleware,
      until no middleware is left and the default exception handling kicks in.

      If it returns a :class:`~scrapy.http.Response` object, the :meth:`process_response`
      method chain of installed middleware is started, and Scrapy won't bother calling
      any other :meth:`process_exception` methods of middleware.

      If it returns a :class:`~scrapy.http.Request` object, the returned request is
      rescheduled to be downloaded in the future. This stops the execution of
      :meth:`process_exception` methods of the middleware the same as returning a
      response would.

      :param request: the request that generated the exception
      :type request: is a :class:`~scrapy.http.Request` object

      :param exception: the raised exception
      :type exception: an ``Exception`` object

      :param spider: the spider for which this request is intended
      :type spider: :class:`~scrapy.spiders.Spider` object

.. _topics-downloader-middleware-ref:

ビルトインダウンローダーミドルウェアリファレンス
========================================

This page describes all downloader middleware components that come with
Scrapy. For information on how to use them and how to write your own downloader
middleware, see the :ref:`downloader middleware usage guide
<topics-downloader-middleware>`.

デフォルトで有効になっているコンポーネントの一覧（およびそのオーダー）については, 
:setting:`DOWNLOADER_MIDDLEWARES_BASE` 設定を参照してください.

.. _cookies-mw:

CookiesMiddleware
-----------------

.. module:: scrapy.downloadermiddlewares.cookies
   :synopsis: Cookies Downloader Middleware

.. class:: CookiesMiddleware

   This middleware enables working with sites that require cookies, such as
   those that use sessions. It keeps track of cookies sent by web servers, and
   send them back on subsequent requests (from that spider), just like web
   browsers do.

The following settings can be used to configure the cookie middleware:

* :setting:`COOKIES_ENABLED`
* :setting:`COOKIES_DEBUG`

.. reqmeta:: cookiejar

スパイダーごとに複数のCookieセッション
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. versionadded:: 0.15

There is support for keeping multiple cookie sessions per spider by using the
:reqmeta:`cookiejar` Request meta key. By default it uses a single cookie jar
(session), but you can pass an identifier to use different ones.

たとえば::

    for i, url in enumerate(urls):
        yield scrapy.Request(url, meta={'cookiejar': i},
            callback=self.parse_page)

Keep in mind that the :reqmeta:`cookiejar` meta key is not "sticky". You need to keep
passing it along on subsequent requests. For example::

    def parse_page(self, response):
        # do some processing
        return scrapy.Request("http://www.example.com/otherpage",
            meta={'cookiejar': response.meta['cookiejar']},
            callback=self.parse_other_page)

.. setting:: COOKIES_ENABLED

COOKIES_ENABLED
~~~~~~~~~~~~~~~

デフォルト: ``True``

クッキーミドルウェアを有効にするかどうか. 無効にすると, Webサーバーにクッキーは送信されません.

.. setting:: COOKIES_DEBUG

COOKIES_DEBUG
~~~~~~~~~~~~~

デフォルト: ``False``

If enabled, Scrapy will log all cookies sent in requests (ie. ``Cookie``
header) and all cookies received in responses (ie. ``Set-Cookie`` header).

Here's an example of a log with :setting:`COOKIES_DEBUG` enabled::

    2011-04-06 14:35:10-0300 [scrapy] INFO: Spider opened
    2011-04-06 14:35:10-0300 [scrapy] DEBUG: Sending cookies to: <GET http://www.diningcity.com/netherlands/index.html>
            Cookie: clientlanguage_nl=en_EN
    2011-04-06 14:35:14-0300 [scrapy] DEBUG: Received cookies from: <200 http://www.diningcity.com/netherlands/index.html>
            Set-Cookie: JSESSIONID=B~FA4DC0C496C8762AE4F1A620EAB34F38; Path=/
            Set-Cookie: ip_isocode=US
            Set-Cookie: clientlanguage_nl=en_EN; Expires=Thu, 07-Apr-2011 21:21:34 GMT; Path=/
    2011-04-06 14:49:50-0300 [scrapy] DEBUG: Crawled (200) <GET http://www.diningcity.com/netherlands/index.html> (referer: None)
    [...]


DefaultHeadersMiddleware
------------------------

.. module:: scrapy.downloadermiddlewares.defaultheaders
   :synopsis: Default Headers Downloader Middleware

.. class:: DefaultHeadersMiddleware

    This middleware sets all default requests headers specified in the
    :setting:`DEFAULT_REQUEST_HEADERS` setting.

DownloadTimeoutMiddleware
-------------------------

.. module:: scrapy.downloadermiddlewares.downloadtimeout
   :synopsis: Download timeout middleware

.. class:: DownloadTimeoutMiddleware

    This middleware sets the download timeout for requests specified in the
    :setting:`DOWNLOAD_TIMEOUT` setting or :attr:`download_timeout`
    spider attribute.

.. note::

    You can also set download timeout per-request using
    :reqmeta:`download_timeout` Request.meta key; this is supported
    even when DownloadTimeoutMiddleware is disabled.

HttpAuthMiddleware
------------------

.. module:: scrapy.downloadermiddlewares.httpauth
   :synopsis: HTTP Auth downloader middleware

.. class:: HttpAuthMiddleware

    このミドルウェアは,  `Basic access authentication`_ 
    （別名HTTP認証）を使用して, 特定のスパイダーから生成されたすべてのリクエストを認証します.

    特定のスパイダーからHTTP認証を有効にするには, これらのスパイダーの ``http_user`` 
    および ``http_pass`` 属性を設定します.

    例::

        from scrapy.spiders import CrawlSpider

        class SomeIntranetSiteSpider(CrawlSpider):

            http_user = 'someuser'
            http_pass = 'somepass'
            name = 'intranet.example.com'

            # .. 残りのスパイダーコードは省略されています ...

.. _Basic access authentication: https://en.wikipedia.org/wiki/Basic_access_authentication


HttpCacheMiddleware
-------------------

.. module:: scrapy.downloadermiddlewares.httpcache
   :synopsis: HTTP Cache downloader middleware

.. class:: HttpCacheMiddleware

    このミドルウェアは, すべてのHTTPリクエストとレスポンスに低レベルのキャッシュを提供します.
    これはキャッシュストレージバックエンドとキャッシュポリシーとを組み合わせなければなりません.

    2つのHTTPキャッシュストレージバックエンドを持つ Scrapy:

        * :ref:`httpcache-storage-fs`
        * :ref:`httpcache-storage-dbm`

    HTTPキャッシュストレージバックエンドは,  :setting:`HTTPCACHE_STORAGE`
    設定で変更できます. また, 独自のストレージバックエンドを実装することもできます.

    2つのHTTPキャッシュポリシーを持つ Scrapy:

        * :ref:`httpcache-policy-rfc2616`
        * :ref:`httpcache-policy-dummy`

    :setting:`HTTPCACHE_POLICY` 
    設定を使用してHTTPキャッシュポリシーを変更できます. あるいは独自のポリシーを実装することもできます.

    .. reqmeta:: dont_cache

    また,  :reqmeta:`dont_cache` メタキーを `True` とすると, すべてのポリシーで応答をキャッシュすることを避けることができます.

.. _httpcache-policy-dummy:

ダミーポリシー (デフォルト)
~~~~~~~~~~~~~~~~~~~~~~

このポリシーは, HTTP Cache-Control ディレクティブを意識していません. 
すべてのリクエストとそれに対応するレスポンスがキャッシュされます. 
同じリクエストが再び見られると, インターネットから何も転送せずにレスポンスが返されます.

ダミーポリシーは, スパイダーを素早くテストする（毎回ダウンロードを待たずに）, 
または, インターネット接続が利用できないときにスパイダーをオフラインで試すのに便利です. 
目標は, 以前に実行されたとおりにスパイダーの実行を「再生」できるようにすることです.

このポリシーを使用するには:

* :setting:`HTTPCACHE_POLICY` に ``scrapy.extensions.httpcache.DummyPolicy`` を設定します. 

.. _httpcache-policy-rfc2616:

RFC2616 ポリシー
~~~~~~~~~~~~~~

このポリシーは, HTTPキャッシュ制御の認識を備えた RFC2616 準拠の HTTP キャッシュを提供し, 
生産を目的とし, 変更なしのデータのダウンロードを避けるために連続実行で
使用します（帯域幅を節約し, クロールを高速化します）.

実装されているもの:

* `no-store` キャッシュ制御ディレクティブセットで, レスポンス/リクエストを格納しない
* 新しいレスポンスに対しても `no-cache` キャッシュコントロール指令が設定されている場合, キャッシュからの応答を提供しない
* `max-age` キャッシュ制御命令からフレッシュネスライフタイムを計算する
* `Expires` レスポンスヘッダーからフレッシュネスライフタイムを計算する
* `Last-Modified` レスポンスヘッダ（Firefoxで使用されるヒューリスティック）からフレッシュネスライフタイムを計算
* `Age` レスポンスヘッダから現在の年齢を計算する
* `Date` ヘッダから現在の年齢を計算する
* `Last-Modified` レスポンスヘッダに基づいて失効したレスポンスを再確認する
* `ETag` レスポンスヘッダーにもとづいて失効した応答を再検証する
* 受け取らなかったレスポンスの Date` ヘッダーを設定しない
* リクエストにおける `max-stale` キャッシュ制御命令をサポート

  これにより, スパイダーを完全なRFC2616キャッシュポリシーで構成することができますが, 
  HTTP仕様に準拠したままで, リクエストごとに再検証は行われません.

  例:

  `Cache-Control: max-stale=600` を追加して, 
  有効期限を超過したリクエストを600秒以下で受け入れるようにヘッダーに要求します.

  参照: RFC2616, 14.9.3

何が無くなったか:

* `Pragma: no-cache` サポート https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.9.1
* `Vary` ヘッダーサポート https://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html#sec13.6
* 更新または削除後の無効化 https://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html#sec13.10
* ... おそらく他にも ..

このポリシーを使用するには:

* :setting:`HTTPCACHE_POLICY` に ``scrapy.extensions.httpcache.RFC2616Policy`` を設定します. 

.. _httpcache-storage-fs:

ファイルシステムストレージバックエンド (デフォルト)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

ファイルシステムストレージバックエンドは, HTTPキャッシュミドルウェアで使用できます.

このストレージバックエンドを使用するには:

* :setting:`HTTPCACHE_STORAGE` に ``scrapy.extensions.httpcache.FilesystemCacheStorage`` を設定します. 

各 request/response のペアは, 次のファイルを含む別のディレクトリに格納されます:

 * ``request_body`` - the plain request body
 * ``request_headers`` - the request headers (in raw HTTP format)
 * ``response_body`` - the plain response body
 * ``response_headers`` - the request headers (in raw HTTP format)
 * ``meta`` - some metadata of this cache resource in Python ``repr()`` format
   (grep-friendly format)
 * ``pickled_meta`` - the same metadata in ``meta`` but pickled for more
   efficient deserialization

ディレクトリ名はリクエストフィンガープリント ( ``scrapy.utils.request.fingerprint`` を参照)から作成され, 
1つのレベルのサブディレクトリが, 同じディレクトリにあまりにも多くのファイルを作成することを
避けるために使用されます（多くのファイルシステムでは非効率的です）::

   /path/to/cache/dir/example.com/72/72811f648e718090f041317756c03adb0ada46c7

.. _httpcache-storage-dbm:

DBM ストレージバックエンド
~~~~~~~~~~~~~~~~~~~

.. versionadded:: 0.13

DBM_ ストレージバックエンドは, HTTPキャッシュミドルウェアでも使用できます.

デフォルトでは,  anydbm_ モジュールを使用しますが, 
:setting:`HTTPCACHE_DBM_MODULE` 設定で変更することができます.

このストレージバックエンドを使用するには:

* :setting:`HTTPCACHE_STORAGE` に ``scrapy.extensions.httpcache.DbmCacheStorage`` を設定します. 

.. _httpcache-storage-leveldb:

LevelDB ストレージバックエンド
~~~~~~~~~~~~~~~~~~~~~~~

.. versionadded:: 0.23

LevelDB_ ストレージバックエンドは, HTTPキャッシュミドルウェアでも使用できます.

このバックエンドは開発プロセスにはお勧めできません. 
これは, 同時に1つのプロセスしか LevelDB データベースにアクセスできないためです. 
そのため, 同じスパイダーに対して並列に Scrapy シェルを開くことはできません.

このストレージバックエンドを使用するには:

* :setting:`HTTPCACHE_STORAGE` に ``scrapy.extensions.httpcache.LeveldbCacheStorage`` を設定します
* ``pip install leveldb`` のようにして,  `LevelDB の Python バインディング`_ をインストールします

.. _LevelDB: https://github.com/google/leveldb
.. _LevelDB の Python バインディング: https://pypi.python.org/pypi/leveldb


HTTPCache middleware 設定
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

:class:`HttpCacheMiddleware` は, 次の設定で構成されています:

.. setting:: HTTPCACHE_ENABLED

HTTPCACHE_ENABLED
^^^^^^^^^^^^^^^^^

.. versionadded:: 0.11

デフォルト: ``False``

HTTPキャッシュを有効にするかどうか.

.. versionchanged:: 0.11
   0.11 より前では, キャッシュを有効にするために :setting:`HTTPCACHE_DIR` が使用されていました.

.. setting:: HTTPCACHE_EXPIRATION_SECS

HTTPCACHE_EXPIRATION_SECS
^^^^^^^^^^^^^^^^^^^^^^^^^

デフォルト: ``0``

キャッシュされたリクエストの有効期限（秒単位）.

この時間より古いキャッシュされたリクエストは再ダウンロードされます.  
0の場合, キャッシュされたリクエストは期限切れになりません.

.. versionchanged:: 0.11
   0.11 以前は, ``0`` はキャッシュされたリクエストが常に期限切れになることを意味しました.

.. setting:: HTTPCACHE_DIR

HTTPCACHE_DIR
^^^^^^^^^^^^^

デフォルト: ``'httpcache'``

（低レベル）HTTPキャッシュを格納するために使用するディレクトリ. 
空の場合, HTTPキャッシュは無効になります. 
相対パスが指定されている場合は, プロジェクトデータディレクトリに対して相対パスが使用されます. 
詳細は,  :ref:`topics-project-structure` を参照してください.

.. setting:: HTTPCACHE_IGNORE_HTTP_CODES

HTTPCACHE_IGNORE_HTTP_CODES
^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. versionadded:: 0.10

デフォルト: ``[]``

キャッシュしないレスポンスの HTTP コード.

.. setting:: HTTPCACHE_IGNORE_MISSING

HTTPCACHE_IGNORE_MISSING
^^^^^^^^^^^^^^^^^^^^^^^^

デフォルト: ``False``

有効にすると, キャッシュにないリクエストはダウンロードされずに無視されます.

.. setting:: HTTPCACHE_IGNORE_SCHEMES

HTTPCACHE_IGNORE_SCHEMES
^^^^^^^^^^^^^^^^^^^^^^^^

.. versionadded:: 0.10

デフォルト: ``['file']``

キャッシュしないレスポンスのURIスキーム.

.. setting:: HTTPCACHE_STORAGE

HTTPCACHE_STORAGE
^^^^^^^^^^^^^^^^^

デフォルト: ``'scrapy.extensions.httpcache.FilesystemCacheStorage'``

キャッシュストレージバックエンドを実装するクラス.

.. setting:: HTTPCACHE_DBM_MODULE

HTTPCACHE_DBM_MODULE
^^^^^^^^^^^^^^^^^^^^

.. versionadded:: 0.13

デフォルト: ``'anydbm'``

:ref:`DBMストレージバックエンド <httpcache-storage-dbm>` で使用するデータベースモジュール. 
この設定は, DBMバックエンド特有です.

.. setting:: HTTPCACHE_POLICY

HTTPCACHE_POLICY
^^^^^^^^^^^^^^^^

.. versionadded:: 0.18

デフォルト: ``'scrapy.extensions.httpcache.DummyPolicy'``

キャッシュポリシーを実装するクラス.

.. setting:: HTTPCACHE_GZIP

HTTPCACHE_GZIP
^^^^^^^^^^^^^^

.. versionadded:: 1.0

デフォルト: ``False``

有効にすると, キャッシュされたすべてのデータがgzipで圧縮されます. この設定はファイルシステムのバックエンド特有です.

.. setting:: HTTPCACHE_ALWAYS_STORE

HTTPCACHE_ALWAYS_STORE
^^^^^^^^^^^^^^^^^^^^^^

.. versionadded:: 1.1

デフォルト: ``False``

有効にすると, 無条件にページをキャッシュします.

スパイダーは,  `Cache-Control: max-stale` などを将来使用するために, 
すべてのレスポンスをキャッシュで利用できるようにすることができます. 
DummyPolicy はすべてのレスポンスをキャッシュしますが, 
それを再検証することはありません. 
また, 別のポリシーが望ましい場合もあります.

この設定は, 依然として `Cache-Control: no-store` ディレクティブを尊重します.
必要がない場合は, キャッシュミドルウェアにフィードしたレスポンスの Cache-Control ヘッダーから 
`no-store` を除外します.

.. setting:: HTTPCACHE_IGNORE_RESPONSE_CACHE_CONTROLS

HTTPCACHE_IGNORE_RESPONSE_CACHE_CONTROLS
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. versionadded:: 1.1

デフォルト: ``[]``

無視されるレスポンスのキャッシュ制御ディレクティブのリスト.

サイトはしばしば "no-store", "no-cache", "must-revalidate", などを設定しますが, 
スパイダーがそれらのディレクティブを尊重するならば生成できるトラフィックで動揺します. 
これにより, クロールしているサイトの重要でないことがわかっている 
Cache-Control ディレクティブを選択的に無視することができます.

スパイダーは実際に Cache-Control ディレクティブを必要としない限り, 
Cache-Control ディレクティブを発行しないので, リクエスト内のディレクティブはフィルタリングされません.

HttpCompressionMiddleware
-------------------------

.. module:: scrapy.downloadermiddlewares.httpcompression
   :synopsis: Http Compression Middleware

.. class:: HttpCompressionMiddleware

   このミドルウェアは, 圧縮された（gzip, deflate）トラフィックをWebサイトから送受信できるようにします.

HttpCompressionMiddleware 設定
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. setting:: COMPRESSION_ENABLED

COMPRESSION_ENABLED
^^^^^^^^^^^^^^^^^^^

デフォルト: ``True``

HttpCompressionMiddleware を有効にするかどうか.


HttpProxyMiddleware
-------------------

.. module:: scrapy.downloadermiddlewares.httpproxy
   :synopsis: Http Proxy Middleware

.. versionadded:: 0.8

.. reqmeta:: proxy

.. class:: HttpProxyMiddleware

   This middleware sets the HTTP proxy to use for requests, by setting the
   ``proxy`` meta value for :class:`~scrapy.http.Request` objects.

   Like the Python standard library modules `urllib`_ and `urllib2`_, it obeys
   the following environment variables:

   * ``http_proxy``
   * ``https_proxy``
   * ``no_proxy``

   You can also set the meta key ``proxy`` per-request, to a value like
   ``http://some_proxy_server:port``.

.. _urllib: https://docs.python.org/2/library/urllib.html
.. _urllib2: https://docs.python.org/2/library/urllib2.html

RedirectMiddleware
------------------

.. module:: scrapy.downloadermiddlewares.redirect
   :synopsis: Redirection Middleware

.. class:: RedirectMiddleware

   このミドルウェアは, 応答ステータスに基づいてリクエストのリダイレクトを処理します.

.. reqmeta:: redirect_urls

The urls which the request goes through (while being redirected) can be found
in the ``redirect_urls`` :attr:`Request.meta <scrapy.http.Request.meta>` key.

The :class:`RedirectMiddleware` can be configured through the following
settings (see the settings documentation for more info):

* :setting:`REDIRECT_ENABLED`
* :setting:`REDIRECT_MAX_TIMES`

.. reqmeta:: dont_redirect

If :attr:`Request.meta <scrapy.http.Request.meta>` has ``dont_redirect``
key set to True, the request will be ignored by this middleware.

If you want to handle some redirect status codes in your spider, you can
specify these in the ``handle_httpstatus_list`` spider attribute.

For example, if you want the redirect middleware to ignore 301 and 302
responses (and pass them through to your spider) you can do this::

    class MySpider(CrawlSpider):
        handle_httpstatus_list = [301, 302]

The ``handle_httpstatus_list`` key of :attr:`Request.meta
<scrapy.http.Request.meta>` can also be used to specify which response codes to
allow on a per-request basis. You can also set the meta key
``handle_httpstatus_all`` to ``True`` if you want to allow any response code
for a request.


RedirectMiddleware 設定
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. setting:: REDIRECT_ENABLED

REDIRECT_ENABLED
^^^^^^^^^^^^^^^^

.. versionadded:: 0.13

デフォルト: ``True``

RedirectMiddleware を有効にするかどうか.

.. setting:: REDIRECT_MAX_TIMES

REDIRECT_MAX_TIMES
^^^^^^^^^^^^^^^^^^

デフォルト: ``20``

1回のリクエストで実行されるリダイレクトの最大数.

MetaRefreshMiddleware
---------------------

.. class:: MetaRefreshMiddleware

   このミドルウェアは, メタリフレッシュhtmlタグに基づいてリクエストのリダイレクトを処理します.

The :class:`MetaRefreshMiddleware` can be configured through the following
settings (see the settings documentation for more info):

* :setting:`METAREFRESH_ENABLED`
* :setting:`METAREFRESH_MAXDELAY`

This middleware obey :setting:`REDIRECT_MAX_TIMES` setting, :reqmeta:`dont_redirect`
and :reqmeta:`redirect_urls` request meta keys as described for :class:`RedirectMiddleware`


MetaRefreshMiddleware 設定
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. setting:: METAREFRESH_ENABLED

METAREFRESH_ENABLED
^^^^^^^^^^^^^^^^^^^

.. versionadded:: 0.17

デフォルト: ``True``

MetaRefreshMiddleware を有効にするかどうか.

.. setting:: METAREFRESH_MAXDELAY

METAREFRESH_MAXDELAY
^^^^^^^^^^^^^^^^^^^^

デフォルト: ``100``

The maximum meta-refresh delay (in seconds) to follow the redirection.
Some sites use meta-refresh for redirecting to a session expired page, so we
restrict automatic redirection to the maximum delay.

RetryMiddleware
---------------

.. module:: scrapy.downloadermiddlewares.retry
   :synopsis: Retry Middleware

.. class:: RetryMiddleware

   A middleware to retry failed requests that are potentially caused by
   temporary problems such as a connection timeout or HTTP 500 error.

Failed pages are collected on the scraping process and rescheduled at the
end, once the spider has finished crawling all regular (non failed) pages.
Once there are no more failed pages to retry, this middleware sends a signal
(retry_complete), so other extensions could connect to that signal.

The :class:`RetryMiddleware` can be configured through the following
settings (see the settings documentation for more info):

* :setting:`RETRY_ENABLED`
* :setting:`RETRY_TIMES`
* :setting:`RETRY_HTTP_CODES`

.. reqmeta:: dont_retry

If :attr:`Request.meta <scrapy.http.Request.meta>` has ``dont_retry`` key
set to True, the request will be ignored by this middleware.

RetryMiddleware 設定
~~~~~~~~~~~~~~~~~~~~~~~~

.. setting:: RETRY_ENABLED

RETRY_ENABLED
^^^^^^^^^^^^^

.. versionadded:: 0.13

デフォルト: ``True``

RetryMiddleware を有効にするかどうか.

.. setting:: RETRY_TIMES

RETRY_TIMES
^^^^^^^^^^^

デフォルト: ``2``

最初のダウンロードに加えて, 再試行の最大回数.

.. setting:: RETRY_HTTP_CODES

RETRY_HTTP_CODES
^^^^^^^^^^^^^^^^

デフォルト: ``[500, 502, 503, 504, 408]``

Which HTTP response codes to retry. Other errors (DNS lookup issues,
connections lost, etc) are always retried.

In some cases you may want to add 400 to :setting:`RETRY_HTTP_CODES` because
it is a common code used to indicate server overload. It is not included by
default because HTTP specs say so.


.. _topics-dlmw-robots:

RobotsTxtMiddleware
-------------------

.. module:: scrapy.downloadermiddlewares.robotstxt
   :synopsis: robots.txt middleware

.. class:: RobotsTxtMiddleware

    This middleware filters out requests forbidden by the robots.txt exclusion
    standard.

    To make sure Scrapy respects robots.txt make sure the middleware is enabled
    and the :setting:`ROBOTSTXT_OBEY` setting is enabled.

.. reqmeta:: dont_obey_robotstxt

If :attr:`Request.meta <scrapy.http.Request.meta>` has
``dont_obey_robotstxt`` key set to True
the request will be ignored by this middleware even if
:setting:`ROBOTSTXT_OBEY` is enabled.


DownloaderStats
---------------

.. module:: scrapy.downloadermiddlewares.stats
   :synopsis: Downloader Stats Middleware

.. class:: DownloaderStats

   Middleware that stores stats of all requests, responses and exceptions that
   pass through it.

   To use this middleware you must enable the :setting:`DOWNLOADER_STATS`
   setting.

UserAgentMiddleware
-------------------

.. module:: scrapy.downloadermiddlewares.useragent
   :synopsis: User Agent Middleware

.. class:: UserAgentMiddleware

   スパイダーがデフォルトのユーザーエージェントをオーバーライドできるミドルウェア.

   In order for a spider to override the default user agent, its `user_agent`
   attribute must be set.

.. _ajaxcrawl-middleware:

AjaxCrawlMiddleware
-------------------

.. module:: scrapy.downloadermiddlewares.ajaxcrawl

.. class:: AjaxCrawlMiddleware

   Middleware that finds 'AJAX crawlable' page variants based
   on meta-fragment html tag. See
   https://developers.google.com/webmasters/ajax-crawling/docs/getting-started
   for more info.

   .. note::

       Scrapy finds 'AJAX crawlable' pages for URLs like
       ``'http://example.com/!#foo=bar'`` even without this middleware.
       AjaxCrawlMiddleware is necessary when URL doesn't contain ``'!#'``.
       This is often a case for 'index' or 'main' website pages.

AjaxCrawlMiddleware 設定
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. setting:: AJAXCRAWL_ENABLED

AJAXCRAWL_ENABLED
^^^^^^^^^^^^^^^^^

.. versionadded:: 0.21

デフォルト: ``False``

Whether the AjaxCrawlMiddleware will be enabled. You may want to
enable it for :ref:`broad crawls <topics-broad-crawls>`.

HttpProxyMiddleware 設定
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. setting:: HTTPPROXY_AUTH_ENCODING

HTTPPROXY_AUTH_ENCODING
^^^^^^^^^^^^^^^^^^^^^^^

デフォルト: ``"latin-1"``

The default encoding for proxy authentication on :class:`HttpProxyMiddleware`.


.. _DBM: https://en.wikipedia.org/wiki/Dbm
.. _anydbm: https://docs.python.org/2/library/anydbm.html
