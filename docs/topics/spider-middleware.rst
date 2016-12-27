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

        このメソッドは, 処理のためにスパイダーミドルウェアを経由してスパイダーに入る各レスポンスに対して呼び出されます.

        :meth:`process_spider_input` は必ず ``None`` を返すか, または例外を発生させなければいけません.

        ``None`` を返すと, Scrapy はこのレスポンスの処理を続行し,
        最終的にレスポンスが処理のためにスパイダーに渡されるまで他のすべてのミドルウェアを実行します.

        例外が発生した場合, Scrapy は他のミドルウェアの :meth:`process_spider_input` 
        メソッド呼び出しを無視して errback リクエストを呼び出します. 
        errback の出力は, :meth:`process_spider_output` で処理されるか, エラーが発生した場合は
        :meth:`process_spider_exception` メソッドにチェーンされて戻ります.

        :param response: 処理されているレスポンス
        :type response: :class:`~scrapy.http.Response` オブジェクト
        
        :param spider: このレスポンスを処理しているスパイダー
        :type spider: :class:`~scrapy.spiders.Spider` オブジェクト
        

    .. method:: process_spider_output(response, result, spider)

        このメソッドは, レスポンスを処理した後にスパイダーから返された結果にたいして呼び出されます.

        :meth:`process_spider_output` は必ずイテラブルな :class:`~scrapy.http.Request`, 
        ``dict`` または :class:`~scrapy.item.Item` オブジェクトのいずれかを返す必要があります.

        :param response: スパイダーによって出力されたレスポンス
        :type response: :class:`~scrapy.http.Response` オブジェクト
        
        :param result: スパイダーによって返された結果
        :type result: イテラブルな :class:`~scrapy.http.Request`, ``dict`` または :class:`~scrapy.item.Item` オブジェクト
        
        :param spider: 結果が処理されているスパイダー
        :type spider: :class:`~scrapy.spiders.Spider` オブジェクト
        

    .. method:: process_spider_exception(response, exception, spider)

        このメソッドは, スパイダーまたは :meth:`process_spider_input` メソッド 
        (他のスパイダーミドルウェアのもの) が例外を発生させたときに呼び出されます.

        :meth:`process_spider_exception` は ``None`` , イテラブルな 
        :class:`~scrapy.http.Response`, ``dict`` または
        :class:`~scrapy.item.Item` オブジェクトを返さなければいけません.

        ``None`` を返すと, Scrapy は, ミドルウェアコンポーネントが残っていない, かつ例外がエンジンに到達するまで, 
        この例外の処理を続け, 次のミドルウェアコンポーネントで :meth:`process_spider_exception` を実行します. 
        
        イテラブルを返すと :meth:`process_spider_output` パイプラインが起動し, 
        ほかの :meth:`process_spider_exception` は呼び出されません.

        :param response: 例外が発生したときに処理されるレスポンス
        :type response: :class:`~scrapy.http.Response` オブジェクト
        
        :param exception: 発生した例外
        :type exception: `Exception`_ オブジェクト
        
        :param spider: 例外が発生したスパイダー
        :type spider: :class:`~scrapy.spiders.Spider` オブジェクト
        
    .. method:: process_start_requests(start_requests, spider)

        .. versionadded:: 0.15

        このメソッドはスパイダーの開始要求とともに呼び出され, :meth:`process_spider_output` 
        メソッドと同様に動作します. ただし, レスポンスが関連付けられておらず, リクエスト（項目ではない）のみを返さなければなりません.

        イテラブル ( ``start_requests`` パラメーター無い) を受取り, 
        イテラブルな :class:`~scrapy.http.Request` オブジェを返さなければいけません. 

        .. note:: このメソッドをスパイダー・ミドルウェアに実装する場合は, 常にiterableを返し, 
            ``start_requests`` をすべて消費しないようにする必要があります. 
            非常に大きい（または制限なし）ことがあり, メモリーがオーバーフローする可能性があるからです. 
            Scrapy エンジンは, 開始要求を処理する能力を持っている間に開始要求を引き出すように設計されているため, 
            開始要求イテレータは, スパイダーを停止するためのその他の条件（時間制限や項目/ページ数など）がある場合には効果的に無限になります.

        :param start_requests: スタートリクエスト
        :type start_requests: イテラブルな :class:`~scrapy.http.Request`

        :param spider: スタートリクエストが属するスパイダー
        :type spider: :class:`~scrapy.spiders.Spider` オブジェクト
        

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

    スパイダーが対処する必要がないように, 失敗した（誤った）HTTP レスポンスを除外します. 
    ほとんどの場合, オーバーヘッドがかかり, より多くのリソースを消費し, スパイダーロジックがより複雑になります.

`HTTP standard`_ によれば, 成功したレスポンスは, ステータスコードが200〜300の範囲にあるものです.

.. _HTTP standard: https://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html

その範囲外のレスポンスコードを処理したい場合は, 
``handle_httpstatus_list`` 属性または
:setting:`HTTPERROR_ALLOWED_CODES` 設定を使用して, スパイダーが処理できるレスポンスコードを指定できます.

たとえば, スパイダーが 404 レスポンスを処理するようにするには, 以下のようにします::

    class MySpider(CrawlSpider):
        handle_httpstatus_list = [404]

.. reqmeta:: handle_httpstatus_list

.. reqmeta:: handle_httpstatus_all

:attr:`Request.meta<scrapy.http.Request.meta>` の  ``handle_httpstatus_list`` 
キーを使用して, リクエストごとに許可するレスポンスコードを指定することもできます. 
リクエストにすべてのレスポンスコードを許可させる場合は, meta キー ``handle_httpstatus_all``
を ``True`` に設定することもできます.

しかし, あなた自信が何を行いたいかを本当に理解していない限り, 
200以外の回答をむやみに処理することは, 通常は悪い考えです.

詳細については,  `HTTP Status Code Definitions`_ を参照してください.

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

   スパイダーがカバーするドメイン外のURLに対するリクエストをフィルタリングします.

   このミドルウェアは, スパイダーの :attr:`~scrapy.spiders.Spider.allowed_domains` 
   属性にないホスト名を持つすべてのリクエストをフィルタリングします.
   リスト内の任意のドメインのすべてのサブドメインも許可されます.
   例えば, ルールに ``www.example.org`` が指定されている場合でも ``bob.www.example.org`` は許可されますが, 
   ``www2.example.com`` や ``example.com`` は許可されません.

   スパイダーが許可されていないドメインのリクエストを返すと, このミドルウェアはこのようなデバッグメッセージを記録します::

      DEBUG: Filtered offsite request to 'www.othersite.com': <GET http://www.othersite.com/some/page.html>

   あまりにも多くのノイズをログに書き込まないようにするため, フィルタリングされた新しいドメインごとにこれらのメッセージの1つのみを出力します. 
   たとえば,  ``www.othersite.com`` の別のリクエストがフィルタリングされた場合, ログメッセージは出力されません. 
   しかし,  ``someothersite.com`` のリクエストがフィルタリングされると, メッセージが出力されます（最初のリクエストがフィルタリングされた場合のみ）.

   スパイダーが :attr:`~scrapy.spiders.Spider.allowed_domains` 属性を定義していないか, 
   属性が空の場合, オフサイトミドルウェアはすべてのリクエストを許可します.

   リクエストに :attr:`~scrapy.http.Request.dont_filter` 属性が設定されている場合, 
   オフサイトミドルウェアはドメインが許可リストになくてもリクエストを許可します.


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

