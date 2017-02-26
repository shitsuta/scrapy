.. _faq:

よくある質問
==========================

.. _faq-scrapy-bs-cmp:

ScrapyとBeautifulSoupまたはlxmlとの比較
-------------------------------------------------

`BeautifulSoup`_ と `lxml`_ はHTMLとXMLを解析するためのライブラリです. Scrapyは, Webサイトをクロールし, Webサイトからデータを抽出するWebスパイダーを作成するためのアプリケーションフレームワークです.

Scrapy には, データを抽出するためのメカニズム ( :ref:`selectors <topics-selectors>` と呼ばれる) が組み込まれていますが,
使い方が分かりやすい場合は `BeautifulSoup`_
(または `lxml`_) をかんたんに使用することができます. 
結局のところ, Pythonコードからインポートして使用できるライブラリを用いて解析しているだけです.

つまり,  `BeautifulSoup`_ (または `lxml`_) と Scrapy を比較するのは `jinja2`_ と `Django`_ を比較するのと同じことです.

.. _BeautifulSoup: http://www.crummy.com/software/BeautifulSoup/
.. _lxml: http://lxml.de/
.. _jinja2: http://jinja.pocoo.org/
.. _Django: https://www.djangoproject.com/

Scrapy で BeautifulSoup を使うことができますか?
------------------------------------

使用できます.
:ref:`上記 <faq-scrapy-bs-cmp>` のように, `BeautifulSoup`_ を Scrapy のコールバックでHTMLレスポンスを解析するために使用できます.
レスポンスのボディを ``BeautifulSoup`` オブジェクトにフィードし, そこから必要なデータを抽出するだけです.

以下は, BeautifulSoup API を使用したスパイダーの例です. HTMLパーサーとして ``lxml`` を使用しています::


    from bs4 import BeautifulSoup
    import scrapy


    class ExampleSpider(scrapy.Spider):
        name = "example"
        allowed_domains = ["example.com"]
        start_urls = (
            'http://www.example.com/',
        )

        def parse(self, response):
            # use lxml to get decent HTML parsing speed
            soup = BeautifulSoup(response.text, 'lxml')
            yield {
                "url": response.url,
                "title": soup.h1.string
            }

.. note::

    ``BeautifulSoup`` は複数の HTML/XML パーサーをサポートしています.
    `BeautifulSoup 公式ドキュメント`_ を参照してください.

.. _BeautifulSoup 公式ドキュメント: https://www.crummy.com/software/BeautifulSoup/bs4/doc/#specifying-the-parser-to-use

.. _faq-python-versions:

Scrapy がサポートしているのは Python のどのバージョンですか?
-----------------------------------------

Scrapy は Python 2.7 と Python 3.3+　での動作が確認されています。
Python 2.6　は　Scrapy 0.20　からサポート対象から外されています。
Scrapy 1.1 から　Python 3 サポートが開始されています。

.. note::
    Windows では　Python 3 はサポートされていません。
    
Did Scrapy "steal" X from Django?
---------------------------------

Probably, but we don't like that word. We think Django_ is a great open source
project and an example to follow, so we've used it as an inspiration for
Scrapy.

We believe that, if something is already done well, there's no need to reinvent
it. This concept, besides being one of the foundations for open source and free
software, not only applies to software but also to documentation, procedures,
policies, etc. So, instead of going through each problem ourselves, we choose
to copy ideas from those projects that have already solved them properly, and
focus on the real problems we need to solve.

We'd be proud if Scrapy serves as an inspiration for other projects. Feel free
to steal from us!

Scrapy はHTTPプロキシで動作しますか?
-----------------------------------

はい. HTTPプロキシダウンローダミドルウェアを介してHTTPプロキシのサポートが提供されています（Scrapy 0.8以降）. 
:class:`~scrapy.downloadermiddlewares.httpproxy.HttpProxyMiddleware` を参照してください.

異なるページの属性を持つアイテムをスクラップする方法はありますか?
------------------------------------------------------------

:ref:`topics-request-response-ref-request-callback-arguments` を参照してください.


Scrapy がクラッシュする: No module named win32api
----------------------------------------------------------

 `Twisted のバグ`_ のために, `pywin32`_ をインストールする必要があります.

.. _pywin32: https://sourceforge.net/projects/pywin32/
.. _Twisted のバグ: https://twistedmatrix.com/trac/ticket/3707

スパイダーでユーザーログインをシミュレートする方法はありますか?
---------------------------------------------

:ref:`topics-request-response-ref-request-userlogin` 参照してください.

.. _faq-bfo-dfo:

Scrapyは幅優先, 深さ優先どちらでクロールしますか?
--------------------------------------------------------

By default, Scrapy uses a `LIFO`_ queue for storing pending requests, which
basically means that it crawls in `DFO order`_. This order is more convenient
in most cases. If you do want to crawl in true `BFO order`_, you can do it by
setting the following settings::

    DEPTH_PRIORITY = 1
    SCHEDULER_DISK_QUEUE = 'scrapy.squeues.PickleFifoDiskQueue'
    SCHEDULER_MEMORY_QUEUE = 'scrapy.squeues.FifoMemoryQueue'

私の Scrapy のクローラにはメモリリークがあります。どうしたら良いですか?
--------------------------------------------------

:ref:`topics-leaks` を参照してください.

また, Pythonには, 
:ref:`topics-leaks-without-leaks` 
で説明されているメモリリークの問題があります.

どうしたらScrapyの消費メモリを少なくすることができますか?
------------------------------------------

直前の質問を参照してください.

スパイダーで基本的なHTTP認証を使用することはできますか?
--------------------------------------------------

はい, :class:`~scrapy.downloadermiddlewares.httpauth.HttpAuthMiddleware` を確認してください.

どのようにして英語のかわりに, 私の母国語でページをダウンロードするのですか？
------------------------------------------------------------------------

Try changing the default `Accept-Language`_ request header by overriding the
:setting:`DEFAULT_REQUEST_HEADERS` setting.

.. _Accept-Language: https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.4

Scrapyプロジェクトの例はどこにありますか?
----------------------------------------------

:ref:`intro-examples` を参照してください.

プロジェクトを作成せずにスパイダーを実行することはできますか?
----------------------------------------------

はい.  :command:`runspider` コマンドを使用することで可能です. 例えば,  ``my_spider.py`` ファイルがすでに作成されているのであれば, 以下のように実行することができます::

    scrapy runspider my_spider.py

詳細は, :command:`runspider` コマンドを参照してください.

 "Filtered offsite request"メッセージが表示されます. どうすれば修正できますか?
--------------------------------------------------------------

Those messages (logged with ``DEBUG`` level) don't necessarily mean there is a
problem, so you may not need to fix them.

Those messages are thrown by the Offsite Spider Middleware, which is a spider
middleware (enabled by default) whose purpose is to filter out requests to
domains outside the ones covered by the spider.

詳細については, :class:`~scrapy.spidermiddlewares.offsite.OffsiteMiddleware` を参照してください.

プロダクションでScrapyクローラーを導入するための推奨される方法はなんですか?
---------------------------------------------------------------------

:ref:`topics-deploy` 参照してください.

大量のエクスポートにJSONを使用することはできますか?
---------------------------------

It'll depend on how large your output is. See :ref:`this warning
<json-with-large-data>` in :class:`~scrapy.exporters.JsonItemExporter`
documentation.

シグナルハンドラから（Twised）遅延を返すことはできますか?
------------------------------------------------------

Some signals support returning deferreds from their handlers, others don't. See
the :ref:`topics-signals-ref` to know which ones.

応答ステータスコード999は何を意味しますか？
---------------------------------------------

999 is a custom response status code used by Yahoo sites to throttle requests.
Try slowing down the crawling speed by using a download delay of ``2`` (or
higher) in your spider::

    class MySpider(CrawlSpider):

        name = 'myspider'

        download_delay = 2

        # [ ... rest of the spider code ... ]

Or by setting a global download delay in your project with the
:setting:`DOWNLOAD_DELAY` setting.

スパイダーのデバッグで ``pdb.set_trace()`` メソッドを呼ぶことはできますか?
-------------------------------------------------------------

Yes, but you can also use the Scrapy shell which allows you to quickly analyze
(and even modify) the response being processed by your spider, which is, quite
often, more useful than plain old ``pdb.set_trace()``.

詳細については,  :ref:`topics-shell-inspect-response` を参照してください.

スクレイピングしたデータを JSON/CSV/XML ファイルとして出力する簡単な方法はなんですか?
-------------------------------------------------------------------

JSONファイルで出力する::

    scrapy crawl myspider -o items.json

CSVファイルで出力する::

    scrapy crawl myspider -o items.csv

XMLファイルで出力する::

    scrapy crawl myspider -o items.xml

より詳細な情報は :ref:`topics-feed-exports` を参照してください.

いくつかのフォームで使用される ``__VIEWSTATE`` パラメーターは一体何ですか?
----------------------------------------------------------------------

The ``__VIEWSTATE`` parameter is used in sites built with ASP.NET/VB.NET. For
more info on how it works see `this page`_. Also, here's an `example spider`_
which scrapes one of these sites.

.. _this page: http://search.cpan.org/~ecarroll/HTML-TreeBuilderX-ASP_NET-0.09/lib/HTML/TreeBuilderX/ASP_NET.pm
.. _example spider: https://github.com/AmbientLighter/rpn-fas/blob/master/fas/spiders/rnp.py

大きな XML/CSV データフィードを解析する最適な方法は何ですか?
----------------------------------------------------

Parsing big feeds with XPath selectors can be problematic since they need to
build the DOM of the entire feed in memory, and this can be quite slow and
consume a lot of memory.

In order to avoid parsing all the entire feed at once in memory, you can use
the functions ``xmliter`` and ``csviter`` from ``scrapy.utils.iterators``
module. In fact, this is what the feed spiders (see :ref:`topics-spiders`) use
under the cover.

Scrapy は自動的にクッキーを管理しますか？
-----------------------------------------

はい, Scrapy はサーバーから送信されたCookieを受信して追跡し, 通常のWebブラウザーと同様に後続のリクエストでそれらを送信します.

詳細については,  :ref:`topics-request-response` と :ref:`cookies-mw` を参照してください.

Scrapyから送受信されるクッキーを確認するにはどうすればよいですか？
--------------------------------------------------------------

:setting:`COOKIES_DEBUG` 設定を有効化してください.

スパイダーに止めるように指示するにはどうすればよいですか？
-------------------------------------------

:exc:`~scrapy.exceptions.CloseSpider` エクセプションをコールバックから発生させます. 詳細については :exc:`~scrapy.exceptions.CloseSpider` を参照してください.

私のScrapyボットが禁止されるのを防ぐには？
----------------------------------------------------

:ref:`bans` 参照してください.

スパイダーの引数や設定を使用してスパイダーを構成する必要がありますか?
-----------------------------------------------------------------

Both :ref:`spider arguments <spiderargs>` and :ref:`settings <topics-settings>`
can be used to configure your spider. There is no strict rule that mandates to
use one or the other, but settings are more suited for parameters that, once
set, don't change much, while spider arguments are meant to change more often,
even on each spider run and sometimes are required for the spider to run at all
(for example, to set the start url of a spider).

To illustrate with an example, assuming you have a spider that needs to log
into a site to scrape data, and you only want to scrape data from a certain
section of the site (which varies each time). In that case, the credentials to
log in would be settings, while the url of the section to scrape would be a
spider argument.

XML文書をスクラップしていて、XPathセレクタがアイテムを返さない
--------------------------------------------------------------------------

ネームスペースを削除する必要があるかもしれません. :ref:`removing-namespaces` 参照してください.

.. _user agents: https://en.wikipedia.org/wiki/User_agent
.. _LIFO: https://en.wikipedia.org/wiki/Stack_(abstract_data_type)
.. _DFO order: https://en.wikipedia.org/wiki/Depth-first_search
.. _BFO order: https://en.wikipedia.org/wiki/Breadth-first_search
