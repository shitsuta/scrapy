.. _topics-downloader-middleware:

=====================
ダウンローダーミドルウェア
=====================

The downloader middleware is a framework of hooks into Scrapy's
request/response processing.  It's a light, low-level system for globally
altering Scrapy's requests and responses.

.. _topics-downloader-middleware-setting:

ダウンローダーミドルウェアの有効化
==================================

To activate a downloader middleware component, add it to the
:setting:`DOWNLOADER_MIDDLEWARES` setting, which is a dict whose keys are the
middleware class paths and their values are the middleware orders.

Here's an example::

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

Each middleware component is a Python class that defines one or
more of the following methods:

.. module:: scrapy.downloadermiddlewares

.. class:: DownloaderMiddleware

   .. note::  Any of the downloader middleware methods may also return a deferred.

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

For a list of the components enabled by default (and their orders) see the
:setting:`DOWNLOADER_MIDDLEWARES_BASE` setting.

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

Multiple cookie sessions per spider
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. versionadded:: 0.15

There is support for keeping multiple cookie sessions per spider by using the
:reqmeta:`cookiejar` Request meta key. By default it uses a single cookie jar
(session), but you can pass an identifier to use different ones.

For example::

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

    This middleware authenticates all requests generated from certain spiders
    using `Basic access authentication`_ (aka. HTTP auth).

    To enable HTTP authentication from certain spiders, set the ``http_user``
    and ``http_pass`` attributes of those spiders.

    例::

        from scrapy.spiders import CrawlSpider

        class SomeIntranetSiteSpider(CrawlSpider):

            http_user = 'someuser'
            http_pass = 'somepass'
            name = 'intranet.example.com'

            # .. rest of the spider code omitted ...

.. _Basic access authentication: https://en.wikipedia.org/wiki/Basic_access_authentication


HttpCacheMiddleware
-------------------

.. module:: scrapy.downloadermiddlewares.httpcache
   :synopsis: HTTP Cache downloader middleware

.. class:: HttpCacheMiddleware

    This middleware provides low-level cache to all HTTP requests and responses.
    It has to be combined with a cache storage backend as well as a cache policy.

    2つのHTTPキャッシュストレージバックエンドを持つ Scrapy:

        * :ref:`httpcache-storage-fs`
        * :ref:`httpcache-storage-dbm`

    You can change the HTTP cache storage backend with the :setting:`HTTPCACHE_STORAGE`
    setting. Or you can also implement your own storage backend.

    2つのHTTPキャッシュポリシーを持つ Scrapy:

        * :ref:`httpcache-policy-rfc2616`
        * :ref:`httpcache-policy-dummy`

    You can change the HTTP cache policy with the :setting:`HTTPCACHE_POLICY`
    setting. Or you can also implement your own policy.

    .. reqmeta:: dont_cache

    You can also avoid caching a response on every policy using :reqmeta:`dont_cache` meta key equals `True`.

.. _httpcache-policy-dummy:

Dummy policy (default)
~~~~~~~~~~~~~~~~~~~~~~

This policy has no awareness of any HTTP Cache-Control directives.
Every request and its corresponding response are cached.  When the same
request is seen again, the response is returned without transferring
anything from the Internet.

The Dummy policy is useful for testing spiders faster (without having
to wait for downloads every time) and for trying your spider offline,
when an Internet connection is not available. The goal is to be able to
"replay" a spider run *exactly as it ran before*.

このポリシーを使用するには:

* :setting:`HTTPCACHE_POLICY` to ``scrapy.extensions.httpcache.DummyPolicy``


.. _httpcache-policy-rfc2616:

RFC2616 policy
~~~~~~~~~~~~~~

This policy provides a RFC2616 compliant HTTP cache, i.e. with HTTP
Cache-Control awareness, aimed at production and used in continuous
runs to avoid downloading unmodified data (to save bandwidth and speed up crawls).

実装されているもの:

* Do not attempt to store responses/requests with `no-store` cache-control directive set
* Do not serve responses from cache if `no-cache` cache-control directive is set even for fresh responses
* Compute freshness lifetime from `max-age` cache-control directive
* Compute freshness lifetime from `Expires` response header
* Compute freshness lifetime from `Last-Modified` response header (heuristic used by Firefox)
* Compute current age from `Age` response header
* Compute current age from `Date` header
* Revalidate stale responses based on `Last-Modified` response header
* Revalidate stale responses based on `ETag` response header
* Set `Date` header for any received response missing it
* Support `max-stale` cache-control directive in requests

  This allows spiders to be configured with the full RFC2616 cache policy,
  but avoid revalidation on a request-by-request basis, while remaining
  conformant with the HTTP spec.

  例:

  Add `Cache-Control: max-stale=600` to Request headers to accept responses that
  have exceeded their expiration time by no more than 600 seconds.

  See also: RFC2616, 14.9.3

what is missing:

* `Pragma: no-cache` support https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.9.1
* `Vary` header support https://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html#sec13.6
* Invalidation after updates or deletes https://www.w3.org/Protocols/rfc2616/rfc2616-sec13.html#sec13.10
* ... probably others ..

このポリシーを使用するには:

* :setting:`HTTPCACHE_POLICY` to ``scrapy.extensions.httpcache.RFC2616Policy``


.. _httpcache-storage-fs:

Filesystem storage backend (default)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

ファイルシステムストレージバックエンドは, HTTPキャッシュミドルウェアで使用できます.

このストレージバックエンドを使用するには:

* :setting:`HTTPCACHE_STORAGE` to ``scrapy.extensions.httpcache.FilesystemCacheStorage``

Each request/response pair is stored in a different directory containing
the following files:

 * ``request_body`` - the plain request body
 * ``request_headers`` - the request headers (in raw HTTP format)
 * ``response_body`` - the plain response body
 * ``response_headers`` - the request headers (in raw HTTP format)
 * ``meta`` - some metadata of this cache resource in Python ``repr()`` format
   (grep-friendly format)
 * ``pickled_meta`` - the same metadata in ``meta`` but pickled for more
   efficient deserialization

The directory name is made from the request fingerprint (see
``scrapy.utils.request.fingerprint``), and one level of subdirectories is
used to avoid creating too many files into the same directory (which is
inefficient in many file systems). An example directory could be::

   /path/to/cache/dir/example.com/72/72811f648e718090f041317756c03adb0ada46c7

.. _httpcache-storage-dbm:

DBM storage backend
~~~~~~~~~~~~~~~~~~~

.. versionadded:: 0.13

A DBM_ storage backend is also available for the HTTP cache middleware.

By default, it uses the anydbm_ module, but you can change it with the
:setting:`HTTPCACHE_DBM_MODULE` setting.

このストレージバックエンドを使用するには:

* :setting:`HTTPCACHE_STORAGE` to ``scrapy.extensions.httpcache.DbmCacheStorage``

.. _httpcache-storage-leveldb:

LevelDB storage backend
~~~~~~~~~~~~~~~~~~~~~~~

.. versionadded:: 0.23

A LevelDB_ storage backend is also available for the HTTP cache middleware.

This backend is not recommended for development because only one process can
access LevelDB databases at the same time, so you can't run a crawl and open
the scrapy shell in parallel for the same spider.

In order to use this storage backend:

* set :setting:`HTTPCACHE_STORAGE` to ``scrapy.extensions.httpcache.LeveldbCacheStorage``
* install `LevelDB python bindings`_ like ``pip install leveldb``

.. _LevelDB: https://github.com/google/leveldb
.. _leveldb python bindings: https://pypi.python.org/pypi/leveldb


HTTPCache middleware settings
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The :class:`HttpCacheMiddleware` can be configured through the following
settings:

.. setting:: HTTPCACHE_ENABLED

HTTPCACHE_ENABLED
^^^^^^^^^^^^^^^^^

.. versionadded:: 0.11

デフォルト: ``False``

Whether the HTTP cache will be enabled.

.. versionchanged:: 0.11
   Before 0.11, :setting:`HTTPCACHE_DIR` was used to enable cache.

.. setting:: HTTPCACHE_EXPIRATION_SECS

HTTPCACHE_EXPIRATION_SECS
^^^^^^^^^^^^^^^^^^^^^^^^^

デフォルト: ``0``

Expiration time for cached requests, in seconds.

Cached requests older than this time will be re-downloaded. If zero, cached
requests will never expire.

.. versionchanged:: 0.11
   Before 0.11, zero meant cached requests always expire.

.. setting:: HTTPCACHE_DIR

HTTPCACHE_DIR
^^^^^^^^^^^^^

デフォルト: ``'httpcache'``

The directory to use for storing the (low-level) HTTP cache. If empty, the HTTP
cache will be disabled. If a relative path is given, is taken relative to the
project data dir. For more info see: :ref:`topics-project-structure`.

.. setting:: HTTPCACHE_IGNORE_HTTP_CODES

HTTPCACHE_IGNORE_HTTP_CODES
^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. versionadded:: 0.10

デフォルト: ``[]``

これらのHTTPコードでレスポンスをキャッシュしないでください.

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

これらのURIスキームでレスポンスをキャッシュしないでください.

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

The database module to use in the :ref:`DBM storage backend
<httpcache-storage-dbm>`. This setting is specific to the DBM backend.

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

If enabled, will compress all cached data with gzip.
This setting is specific to the Filesystem backend.

.. setting:: HTTPCACHE_ALWAYS_STORE

HTTPCACHE_ALWAYS_STORE
^^^^^^^^^^^^^^^^^^^^^^

.. versionadded:: 1.1

デフォルト: ``False``

有効にすると, 無条件にページをキャッシュします.

A spider may wish to have all responses available in the cache, for
future use with `Cache-Control: max-stale`, for instance. The
DummyPolicy caches all responses but never revalidates them, and
sometimes a more nuanced policy is desirable.

This setting still respects `Cache-Control: no-store` directives in responses.
If you don't want that, filter `no-store` out of the Cache-Control headers in
responses you feedto the cache middleware.

.. setting:: HTTPCACHE_IGNORE_RESPONSE_CACHE_CONTROLS

HTTPCACHE_IGNORE_RESPONSE_CACHE_CONTROLS
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. versionadded:: 1.1

デフォルト: ``[]``

無視されるレスポンスのキャッシュ制御ディレクティブのリスト.

Sites often set "no-store", "no-cache", "must-revalidate", etc., but get
upset at the traffic a spider can generate if it respects those
directives. This allows to selectively ignore Cache-Control directives
that are known to be unimportant for the sites being crawled.

We assume that the spider will not issue Cache-Control directives
in requests unless it actually needs them, so directives in requests are
not filtered.

HttpCompressionMiddleware
-------------------------

.. module:: scrapy.downloadermiddlewares.httpcompression
   :synopsis: Http Compression Middleware

.. class:: HttpCompressionMiddleware

   This middleware allows compressed (gzip, deflate) traffic to be
   sent/received from web sites.

HttpCompressionMiddleware Settings
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. setting:: COMPRESSION_ENABLED

COMPRESSION_ENABLED
^^^^^^^^^^^^^^^^^^^

デフォルト: ``True``

Compressionミドルウェアを有効にするかどうか.


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

   このミドルウェアは, 応答ステータスに基づいて要求のリダイレクトを処理します.

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


RedirectMiddleware settings
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. setting:: REDIRECT_ENABLED

REDIRECT_ENABLED
^^^^^^^^^^^^^^^^

.. versionadded:: 0.13

デフォルト: ``True``

リダイレクトミドルウェアを有効にするかどうか.

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


MetaRefreshMiddleware settings
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. setting:: METAREFRESH_ENABLED

METAREFRESH_ENABLED
^^^^^^^^^^^^^^^^^^^

.. versionadded:: 0.17

デフォルト: ``True``

Meta Refreshミドルウェアを有効にするかどうか.

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

RetryMiddleware Settings
~~~~~~~~~~~~~~~~~~~~~~~~

.. setting:: RETRY_ENABLED

RETRY_ENABLED
^^^^^^^^^^^^^

.. versionadded:: 0.13

デフォルト: ``True``

リトライミドルウェアを有効にするかどうか.

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

AjaxCrawlMiddleware Settings
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. setting:: AJAXCRAWL_ENABLED

AJAXCRAWL_ENABLED
^^^^^^^^^^^^^^^^^

.. versionadded:: 0.21

デフォルト: ``False``

Whether the AjaxCrawlMiddleware will be enabled. You may want to
enable it for :ref:`broad crawls <topics-broad-crawls>`.

HttpProxyMiddleware settings
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. setting:: HTTPPROXY_AUTH_ENCODING

HTTPPROXY_AUTH_ENCODING
^^^^^^^^^^^^^^^^^^^^^^^

デフォルト: ``"latin-1"``

The default encoding for proxy authentication on :class:`HttpProxyMiddleware`.


.. _DBM: https://en.wikipedia.org/wiki/Dbm
.. _anydbm: https://docs.python.org/2/library/anydbm.html
