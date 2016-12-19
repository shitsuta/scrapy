.. _topics-spider-middleware:

=================
スパイダーミドルウェア
=================

スパイダーミドルウェアは, Scrapyのスパイダー処理機構へのフックのフレームワークで, スパイダーに送信された応答を処理し, 
:ref:`topics-spiders` から生成されたリクエストとアイテムを処理するカスタム機能をプラグインできます.

.. _topics-spider-middleware-setting:

スパイダーミドルウェアの有効化
==============================

スパイダーミドルウェアコンポーネントをアクティブにするには, 
:setting:`SPIDER_MIDDLEWARES` 設定に追加します. 
これは, キーがミドルウェアクラスのパスであり, その値がミドルウェアオーダーです.

例::

    SPIDER_MIDDLEWARES = {
        'myproject.middlewares.CustomSpiderMiddleware': 543,
    }

:setting:`SPIDER_MIDDLEWARES` 設定は, Scrapyで定義された
:setting:`SPIDER_MIDDLEWARES_BASE` 設定にマージされます（上書きされるという意味ではありません）. 
次に, 使用可能なミドルウェアの最終的なリストを取得するために, 並べ替えられます. 
値の小さいミドルウェアはエンジンに近いもので,  大きいものはスパイダーに近いです. 言い換えれば,
:meth:`~scrapy.spidermiddlewares.SpiderMiddleware.process_spider_input`
メソッドは, ミドルウェアの順番が増加する (100, 200, 300, ...), ように呼び出され, 各ミドルウェアの
:meth:`~scrapy.spidermiddlewares.SpiderMiddleware.process_spider_output` 
メソッドが降順で呼び出されます.

ミドルウェアに割り当てる順序を決定するには, 
:setting:`SPIDER_MIDDLEWARES_BASE` 設定を参照し, ミドルウェアを挿入する場所に応じて値を選択します. 
各ミドルウェアが異なるアクションを実行し, ミドルウェアが適用されている以前の
（または後続の）ミドルウェアに依存する可能性があるため, 順序は重要です.

組み込みミドルウェア ( :setting:`SPIDER_MIDDLEWARES_BASE` で定義され, デフォルトで有効になっているもの) 
を無効にするには, プロジェクトの :setting:`SPIDER_MIDDLEWARES` 設定で定義し, 
その値にNoneを割り当てる必要があります.  たとえば, off-site ミドルウェアを無効にする場合は::

    SPIDER_MIDDLEWARES = {
        'myproject.middlewares.CustomSpiderMiddleware': 543,
        'scrapy.spidermiddlewares.offsite.OffsiteMiddleware': None,
    }

最後に, 特定の設定によっていくつかのミドルウェアを有効にする必要があるかもしれないことに留意してください. 
詳細は各ミドルウェアのドキュメントを参照してください.

独自のスパイダーミドルウェアの作成
==================================

各ミドルウェアコンポーネントは, 以下のメソッドの1つ以上を定義する Python クラスです:

.. module:: scrapy.spidermiddlewares

.. class:: SpiderMiddleware

    .. method:: process_spider_input(response, spider)

        This method is called for each response that goes through the spider
        middleware and into the spider, for processing.

        :meth:`process_spider_input` should return ``None`` or raise an
        exception.

        If it returns ``None``, Scrapy will continue processing this response,
        executing all other middlewares until, finally, the response is handed
        to the spider for processing.

        If it raises an exception, Scrapy won't bother calling any other spider
        middleware :meth:`process_spider_input` and will call the request
        errback.  The output of the errback is chained back in the other
        direction for :meth:`process_spider_output` to process it, or
        :meth:`process_spider_exception` if it raised an exception.

        :param response: the response being processed
        :type response: :class:`~scrapy.http.Response` object

        :param spider: the spider for which this response is intended
        :type spider: :class:`~scrapy.spiders.Spider` object


    .. method:: process_spider_output(response, result, spider)

        This method is called with the results returned from the Spider, after
        it has processed the response.

        :meth:`process_spider_output` must return an iterable of
        :class:`~scrapy.http.Request`, dict or :class:`~scrapy.item.Item` 
        objects.

        :param response: the response which generated this output from the
          spider
        :type response: :class:`~scrapy.http.Response` object

        :param result: the result returned by the spider
        :type result: an iterable of :class:`~scrapy.http.Request`, dict
          or :class:`~scrapy.item.Item` objects

        :param spider: the spider whose result is being processed
        :type spider: :class:`~scrapy.spiders.Spider` object


    .. method:: process_spider_exception(response, exception, spider)

        This method is called when when a spider or :meth:`process_spider_input`
        method (from other spider middleware) raises an exception.

        :meth:`process_spider_exception` should return either ``None`` or an
        iterable of :class:`~scrapy.http.Response`, dict or
        :class:`~scrapy.item.Item` objects.

        If it returns ``None``, Scrapy will continue processing this exception,
        executing any other :meth:`process_spider_exception` in the following
        middleware components, until no middleware components are left and the
        exception reaches the engine (where it's logged and discarded).

        If it returns an iterable the :meth:`process_spider_output` pipeline
        kicks in, and no other :meth:`process_spider_exception` will be called.

        :param response: the response being processed when the exception was
          raised
        :type response: :class:`~scrapy.http.Response` object

        :param exception: the exception raised
        :type exception: `Exception`_ object

        :param spider: the spider which raised the exception
        :type spider: :class:`~scrapy.spiders.Spider` object

    .. method:: process_start_requests(start_requests, spider)

        .. versionadded:: 0.15

        This method is called with the start requests of the spider, and works
        similarly to the :meth:`process_spider_output` method, except that it
        doesn't have a response associated and must return only requests (not
        items).

        It receives an iterable (in the ``start_requests`` parameter) and must
        return another iterable of :class:`~scrapy.http.Request` objects.

        .. note:: When implementing this method in your spider middleware, you
           should always return an iterable (that follows the input one) and
           not consume all ``start_requests`` iterator because it can be very
           large (or even unbounded) and cause a memory overflow. The Scrapy
           engine is designed to pull start requests while it has capacity to
           process them, so the start requests iterator can be effectively
           endless where there is some other condition for stopping the spider
           (like a time limit or item/page count).

        :param start_requests: the start requests
        :type start_requests: an iterable of :class:`~scrapy.http.Request`

        :param spider: the spider to whom the start requests belong
        :type spider: :class:`~scrapy.spiders.Spider` object


.. _Exception: https://docs.python.org/2/library/exceptions.html#exceptions.Exception


.. _topics-spider-middleware-ref:

ビルトインスパイダーミドルウェアリファレンス
====================================

このページでは, Scrapyに付属するすべてのスパイダーミドルウェアコンポーネントについて説明します. 
それらの使用方法と独自のスパイダーミドルウェアの作成方法については,  
:ref:`スパイダーミドルウェア使用方法ガイド <topics-spider-middleware>` を参照してください.

デフォルトで有効になっているコンポーネントの一覧（およびそのオーダー）については, 
:setting:`SPIDER_MIDDLEWARES_BASE` 設定を参照してください.

DepthMiddleware
---------------

.. module:: scrapy.spidermiddlewares.depth
   :synopsis: Depth Spider Middleware

.. class:: DepthMiddleware

   DepthMiddleware は, スクレイプされているサイト内の各リクエストの深さを追跡するために使用されるスクレイプミドルウェアです. 
   これは, スクレイピングなのどの最大深さを制限するために使用することができます.

   :class:`DepthMiddleware` は以下の設定で設定することができます（詳細については各設定を参照してください）:

      * :setting:`DEPTH_LIMIT` - クロールできる最大の深さ. ゼロの場合, 制限は課されません.
      * :setting:`DEPTH_STATS` - 深度統計を収集するかどうか.
      * :setting:`DEPTH_PRIORITY` - リクエストを深さに基づいて優先順位付けするかどうか.

HttpErrorMiddleware
-------------------

.. module:: scrapy.spidermiddlewares.httperror
   :synopsis: HTTP Error Spider Middleware

.. class:: HttpErrorMiddleware

    Filter out unsuccessful (erroneous) HTTP responses so that spiders don't
    have to deal with them, which (most of the time) imposes an overhead,
    consumes more resources, and makes the spider logic more complex.

According to the `HTTP standard`_, successful responses are those whose
status codes are in the 200-300 range.

.. _HTTP standard: https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html

If you still want to process response codes outside that range, you can
specify which response codes the spider is able to handle using the
``handle_httpstatus_list`` spider attribute or
:setting:`HTTPERROR_ALLOWED_CODES` setting.

For example, if you want your spider to handle 404 responses you can do
this::

    class MySpider(CrawlSpider):
        handle_httpstatus_list = [404]

.. reqmeta:: handle_httpstatus_list

.. reqmeta:: handle_httpstatus_all

The ``handle_httpstatus_list`` key of :attr:`Request.meta
<scrapy.http.Request.meta>` can also be used to specify which response codes to
allow on a per-request basis. You can also set the meta key ``handle_httpstatus_all``
to ``True`` if you want to allow any response code for a request.

Keep in mind, however, that it's usually a bad idea to handle non-200
responses, unless you really know what you're doing.

For more information see: `HTTP Status Code Definitions`_.

.. _HTTP Status Code Definitions: https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html

HttpErrorMiddleware 設定
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. setting:: HTTPERROR_ALLOWED_CODES

HTTPERROR_ALLOWED_CODES
^^^^^^^^^^^^^^^^^^^^^^^

Default: ``[]``

このリストに含まれる200以外のステータスコードを含むすべての応答を渡します.

.. setting:: HTTPERROR_ALLOW_ALL

HTTPERROR_ALLOW_ALL
^^^^^^^^^^^^^^^^^^^

Default: ``False``

ステータスコードに関係なくすべての応答を渡します.

OffsiteMiddleware
-----------------

.. module:: scrapy.spidermiddlewares.offsite
   :synopsis: Offsite Spider Middleware

.. class:: OffsiteMiddleware

   Filters out Requests for URLs outside the domains covered by the spider.

   This middleware filters out every request whose host names aren't in the
   spider's :attr:`~scrapy.spiders.Spider.allowed_domains` attribute.
   All subdomains of any domain in the list are also allowed.
   E.g. the rule ``www.example.org`` will also allow ``bob.www.example.org``
   but not ``www2.example.com`` nor ``example.com``.

   When your spider returns a request for a domain not belonging to those
   covered by the spider, this middleware will log a debug message similar to
   this one::

      DEBUG: Filtered offsite request to 'www.othersite.com': <GET http://www.othersite.com/some/page.html>

   To avoid filling the log with too much noise, it will only print one of
   these messages for each new domain filtered. So, for example, if another
   request for ``www.othersite.com`` is filtered, no log message will be
   printed. But if a request for ``someothersite.com`` is filtered, a message
   will be printed (but only for the first request filtered).

   If the spider doesn't define an
   :attr:`~scrapy.spiders.Spider.allowed_domains` attribute, or the
   attribute is empty, the offsite middleware will allow all requests.

   If the request has the :attr:`~scrapy.http.Request.dont_filter` attribute
   set, the offsite middleware will allow the request even if its domain is not
   listed in allowed domains.


RefererMiddleware
-----------------

.. module:: scrapy.spidermiddlewares.referer
   :synopsis: Referer Spider Middleware

.. class:: RefererMiddleware

   リクエストの ``Referer`` ヘッダーに, レスポンスのURLに基づいてヘッダーを挿入します.

RefererMiddleware 設定
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. setting:: REFERER_ENABLED

REFERER_ENABLED
^^^^^^^^^^^^^^^

.. versionadded:: 0.15

デフォルト: ``True``

リファラーミドルウェアを有効にするかどうか.

UrlLengthMiddleware
-------------------

.. module:: scrapy.spidermiddlewares.urllength
   :synopsis: URL Length Spider Middleware

.. class:: UrlLengthMiddleware

   URL が URLLENGTH_LIMIT より長い場合, リクエストをフィルタリングします
   
   The :class:`UrlLengthMiddleware` は, 以下の設定によって構成することができます（詳細については, 設定ドキュメントを参照してください）:

      * :setting:`URLLENGTH_LIMIT` - クロールするURLで許可されるURLの最大長.

