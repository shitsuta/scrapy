.. _topics-spiders:

=======
スパイダー
=======

パイダーとは, クロールの実行方法（リンクをたどる方法）やページから構造化データを抽出する方法（アイテムのスクレイピングなど）を含む, 
特定のサイト（またはサイトのグループ）のスクレイピング方法を定義するクラスです. 
言い換えれば, スパイダー は, 特定のサイト（場合によってはサイトのグループ）のページをクロールして
解析するためのカスタム動作を定義する場所です.

スパイダーの場合, スクレイピングのライフサイクルは以下のようになります:

1. 最初のURLをクロールする最初のリクエストを生成し, それらのリクエストからダウンロードされたレスポンスで呼び出されるコールバック関数を指定します.

   実行する最初のリクエストは, 
   :meth:`~scrapy.spiders.Spider.start_requests` メソッドを呼び出すことによって取得されます. 
   このメソッドは, デフォルトで
   :attr:`~scrapy.spiders.Spider.start_urls` で指定されたURLと
   :class:`~scrapy.http.Request` のコールバック関数として呼ばれる :attr:`~scrapy.spiders.Spider.parse` メソッドで, 
   :class:`~scrapy.http.Request` を生成します.

2. コールバック関数では, レスポンス（Webページ）を解析し, 抽出されたデータ, 
   :class:`~scrapy.item.Item` オブジェクト, :class:`~scrapy.http.Request` オブジェクト, 
   またはこれらのオブジェクトの反復可能なものを返します
   これらのリクエストには, コールバックが含まれ, 
   Scrapyによってダウンロードされ, その後指定されたコールバックによってリクエストが処理されます.

3. コールバック関数では,  :ref:`topics-selectors` 
   を使用してページの内容を解析します（ただし, BeautifulSoup, lxmlなどの任意のメカニズムを使用することもできます）.

4. 最後に, スパイダーから返されたアイテムは, 
   通常, データベース（一部の :ref:`アイテムパイプライン <topics-item-pipeline>` 内）に永続化されるか, 
   または :ref:`topics-feed-exports` を使用してファイルに書き込まれます.

このサイクルはどんな種類のスパイダーにも（多かれ少なかれ）適用されますが, さまざまな種類のデフォルトのスパイダーが Scrapy にバンドルされています. これらのタイプについてはここで説明します.

.. module:: scrapy.spiders
   :synopsis: Spiders base class, spider manager and spider middleware

.. _topics-spiders-ref:

scrapy.Spider
=============

.. class:: Spider()
   
   これは最もシンプルなスパイダーで, 他のすべてのスパイダーが継承しなければならないものです（Scrapyにバンドルされたスパイダー, あなた自身で作成したスパイダーを含む）.
   特別な機能は提供しません. 
   :attr:`start_urls` 属性からリクエストを送信し, スパイダーの ``parse`` メソッドを, 
   レスポンス結果ごとに呼び出す :meth:`start_requests` メソッドの実装を提供するだけです.

   .. attribute:: name
      
      スパイダーの名前を定義する文字列. 名前は, スパイダーが Scrapy によってどのように配置（インスタンス化）されているか判別するために, ユニークでなければなりません. 
      ただし, 同じスパイダーのインスタンスは一つだけ作成可能で, 複数インスタンス化することはできません. 
      これは最も重要なスパイダー属性であり, 必須です.
      
      スパイダーが単一のドメインをスクラップする場合, 一般的には,  `TLD`_ の有無にかかわらず, ドメイン名と同じの名前を付けます. 
      したがって, たとえば, ``mywebsite.com`` をクロールするスパイダーには,  ``mywebsite`` という名前をつけます.

   .. note:: Python 2 では, ASCIIのみでなければなりません.
       
   .. attribute:: allowed_domains
      
      このスパイダーがクロールできるドメインを含む文字列のオプションのリスト. 
      :class:`~scrapy.spidermiddlewares.offsite.OffsiteMiddleware` 
      が有効になっている場合, このリスト（またはそのサブドメイン）で指定された
      ドメイン名に属していないURLに対するリクエストは追跡されません.

   .. attribute:: start_urls
      
      特定のURLが指定されていない場合, スパイダーがクロールを開始するURLのリスト. 
      したがって, ダウンロードされる最初のページはここにリストされたページになります. 
      後続のURLは, 開始URLに含まれるデータから順番に生成されます.

   .. attribute:: custom_settings
      
      このスパイダーを実行するときにオーバーライドされるプロジェクトの設定の辞書. 
      インスタンス化前に設定が更新されるため, クラス属性として定義する必要があります.
      
      使用可能なビルトイン設定のリストについては, 
      :ref:`topics-settings-ref` を参照してください.

   .. attribute:: crawler
      
      この属性は, クラスを初期化した後の :meth:`from_crawler` クラスメソッドによって設定され, 
      このスパイダーインスタンスがバインドされている
      :class:`~scrapy.crawler.Crawler` オブジェクトへのリンクになります.
      
      クローラは, 単一エントリアクセス（エクステンション, ミドルウェア, シグナルマネージャなど）のために, 
      プロジェクト内の多くのコンポーネントをカプセル化します. 
      詳細については :ref:`topics-api-crawler` を参照してください.

   .. attribute:: settings
      
      このスパイダーを実行するための設定. これは
      :class:`~scrapy.settings.Settings` インスタンスです.  
      このトピックの詳細な紹介については, 
      :ref:`topics-settings` を参照してください.

   .. attribute:: logger
      
      スパイダーの :attr:`name` で作成された Python ロガー. 
      :ref:`topics-logging-from-spiders` で説明しているように, 
      これを使ってログメッセージを送信することができます.

   .. method:: from_crawler(crawler, \*args, \**kwargs) 
      
      これは Scrapy があなたのスパイダーを作成するために使用するクラスメソッドです.
      
      デフォルトの実装は :meth:`__init__` メソッドへのプロキシとして機能し, 
      与えられた引数 `args` と名前付き引数 `kwargs` を呼び出すので, 
      これを直接オーバーライドする必要はないでしょう.
      
      それにもかかわらず, このメソッドは :attr:`crawler` と :attr:`settings`
      属性を新しいインスタンスに設定し, 後でスパイダのコード内でアクセスできるようにします.
      
      :param crawler: スパイダーにバインドされるクローラー 
      :type crawler: :class:`~scrapy.crawler.Crawler` インスタンス
       
      :param args: :meth:`__init__` メソッドに渡される引数
      :type args: list
      
      :param kwargs: :meth:`__init__` メソッドに渡されるキーワード引数
      :type kwargs: dict

   .. method:: start_requests()
      
      このメソッドは, スパイダーの最初のクロールリクエストで繰り返し可能な値を返す必要があります.
      
      これは, 特定のURLが指定されずにスパイダーを開いてスクレイピングするときに, 
      Scrapy によって呼び出されるメソッドです. 特定のURLが指定されている場合, 
      代わりに :meth:`make_requests_from_url` がリクエストを作成するために使用されます. 
      このメソッドはScrapyから1回だけ呼び出されるため, 安全にジェネレータとして実装することができます.
      
      デフォルトの実装では,  :meth:`make_requests_from_url` を使用して, 
      :attr:`start_urls` 内の各URLのリクエストを生成します.
      
      ドメインのスクレイピングを開始するために使用されるリクエストを変更したい場合は, これをオーバーライドすることができます. 
      たとえば, POSTリクエストを使用してログインする必要がある場合は::

           class MySpider(scrapy.Spider):
               name = 'myspider'

               def start_requests(self):
                   return [scrapy.FormRequest("http://www.example.com/login",
                                              formdata={'user': 'john', 'pass': 'secret'},
                                              callback=self.logged_in)]

               def logged_in(self, response):
                   # here you would extract links to follow and return Requests for
                   # each of them, with another callback
                   pass

   .. method:: make_requests_from_url(url)

       AURLを受け取って、 :class:`~scrapy.http.Request` オブジェクト
       （または :class:`~scrapy.http.Request` オブジェクトのリスト) を返すメソッド. 
       このメソッドは、:meth:`start_requests` メソッドで初期リクエストを作成するために使用され、
       通常は URL をリクエストに変換するために使用されます.

       オーバーライドされない限り、このメソッドは、callback関数として :meth:`parse`
       メソッドを使用し, パラメータを有効にしてリクエストを返します
       (詳細は :class:`~scrapy.http.Request` クラスを参照してください).

   .. method:: parse(response)

       これは、リクエストがコールバックを指定していないときに、
       ダウンロードされたレスポンスを処理するために Scrapy に使用されるデフォルトのコールバックです.
       
       ``parse`` ソッドは、レスポンスを処理し、スクレイピングされたデータおよび/または、
       より多くのURLを返すことを担当します。
       その他のリクエストコールバックは :class:`Spider` クラスと同じ要件を持ちます.
       
       このメソッドおよび他のRequestコールバックは、 イテレータブルな :class:`~scrapy.http.Request` 
       および/または、 ``dict`` または、
       :class:`~scrapy.item.Item` オブジェクトを返さなければなりません.

       :param response: パースするレスポンス
       :type response: :class:`~scrapy.http.Response`

   .. method:: log(message, [level, component])

       スパイダーの :attr:`logger` を介してログメッセージを送信し、下位互換性を保つために保管されたラッパー. 
       詳細については :ref:`topics-logging-from-spiders` を参照してください.

   .. method:: closed(reason)

       が閉じたときに呼び出されます。このメソッドは、
       :signal:`spider_closed` シグナルを送信するための signals.connect() メソッドのショートカットを提供します.

例を見てみましょう::

    import scrapy


    class MySpider(scrapy.Spider):
        name = 'example.com'
        allowed_domains = ['example.com']
        start_urls = [
            'http://www.example.com/1.html',
            'http://www.example.com/2.html',
            'http://www.example.com/3.html',
        ]

        def parse(self, response):
            self.logger.info('A response from %s just arrived!', response.url)
            
単一のコールバックから複数のリクエストとアイテムを返します::

    import scrapy

    class MySpider(scrapy.Spider):
        name = 'example.com'
        allowed_domains = ['example.com']
        start_urls = [
            'http://www.example.com/1.html',
            'http://www.example.com/2.html',
            'http://www.example.com/3.html',
        ]

        def parse(self, response):
            for h3 in response.xpath('//h3').extract():
                yield {"title": h3}

            for url in response.xpath('//a/@href').extract():
                yield scrapy.Request(url, callback=self.parse)

:attr:`~.start_urls` の代わりに :meth:`~.start_requests` を直接使うことができます. 
:ref:`topics-items` を使用することで多くの構造体にデータを与えることができます::

    import scrapy
    from myproject.items import MyItem

    class MySpider(scrapy.Spider):
        name = 'example.com'
        allowed_domains = ['example.com']

        def start_requests(self):
            yield scrapy.Request('http://www.example.com/1.html', self.parse)
            yield scrapy.Request('http://www.example.com/2.html', self.parse)
            yield scrapy.Request('http://www.example.com/3.html', self.parse)

        def parse(self, response):
            for h3 in response.xpath('//h3').extract():
                yield MyItem(title=h3)

            for url in response.xpath('//a/@href').extract():
                yield scrapy.Request(url, callback=self.parse)

.. _spiderargs:

スパイダーの引数
================

スパイダーは行動を変更する引数を受け取ることができます. 
スパイダー引数の一般的な用途の1つは, 
開始URLを定義するか, サイトの特定のセクションにクロールを制限することですが, 
スパイダーの機能を構成するために使用できます.

スパイダーの引数は, ``-a`` オプションを使用して :command:`crawl` コマンドに渡されます. 例えば::

    scrapy crawl myspider -a category=electronics

スパイダーは `__init__` メソッドで引数にアクセスできます::

    import scrapy

    class MySpider(scrapy.Spider):
        name = 'myspider'

        def __init__(self, category=None, *args, **kwargs):
            super(MySpider, self).__init__(*args, **kwargs)
            self.start_urls = ['http://www.example.com/categories/%s' % category]
            # ...

スパイダーの引数は, Scrapydの ``schedule.json`` API を介して渡すこともできます.
`Scrapyd documentation`_ を参照してください.

.. _builtin-spiders:

一般的なスパイダー
===============

Scrapy には, スパイダーをサブクラス化するために使用できる, いくつかの有用なスパイダーがあります. 
その目的は, 特定のルールに基づいてサイトのすべてのリンクをたどったり, 
`Sitemaps`_ からクロールしたり, XML / CSV フィードを解析するなど, 
いくつかの一般的なスクラップケースに対して便利な機能を提供することです

以下のスパイダーで使用されているサンプルについては, 
``myproject.items`` モジュールで宣言された ``TestItem`` を持つプロジェクトがあると仮定します::

    import scrapy

    class TestItem(scrapy.Item):
        id = scrapy.Field()
        name = scrapy.Field()
        description = scrapy.Field()


.. currentmodule:: scrapy.spiders

CrawlSpider
-----------

.. class:: CrawlSpider

   これは定期的なウェブサイトをクロールするために最も一般的に使用されるスパイダーです. 
   一連のルールを定義してリンクをたどるための便利なメカニズムを提供します.
   特定のウェブサイトやプロジェクトには最適ではないかもしれませんが, 
   いくつかのケースでは十分に一般的なので, 
   より多くのカスタム機能のために必要に応じて上書きしたり, 独自のスパイダーを実装することができます.

   Spiderから継承した属性（指定する必要がある）を除いて, このクラスは新しい属性をサポートします:

   .. attribute:: rules
   
      1つ（または複数）の :class:`Rule` オブジェクトのリストです. 
      各 :class:`Rule` は, サイトをクロールするための特定の動作を定義します. 
      Rules オブジェクトについては以下で説明します. 
      複数のルールが同じリンクに一致する場合, この属性で定義されている順序に従って, 最初のリンクが使用されます.

   このスパイダーは, オーバーライド可能なメソッドも公開しています:

   .. method:: parse_start_url(response)
   
      このメソッドは, start_urlsレスポンスに対して呼び出されます. これは初期応答を解析することを可能にし, 
      :class:`~scrapy.item.Item` オブジェクト, :class:`~scrapy.http.Request` 
      オブジェクト, またはそれらのいずれかを含む iterable を返さなければなりません.

Rule
~~~~~~~~~~~~~~

.. class:: Rule(link_extractor, callback=None, cb_kwargs=None, follow=None, process_links=None, process_request=None)

   ``link_extractor`` は, クロールされた各ページからリンクを抽出する方法を定義する
   :ref:`Link Extractor <topics-link-extractors>` オブジェクトです.

   ``callback`` は抽出されたリンクごとに呼び出される, 
   呼び出し可能または文字列です（この場合, その名前を持つスパイダーオブジェクトのメソッドが使用されます）. 
   このコールバックは, 最初の引数としてレスポンスを受け取り,  :class:`~scrapy.item.Item` オブジェクトおよび/または
   :class:`~scrapy.http.Request` オブジェクト（またはそれらのサブクラス）を含むリストを返す必要があります.

   .. warning:: 
      クロールスパイダールールを作成するときは, :class:`CrawlSpider` は ``parse`` メソッド自体を使用してロジックを実装するため, 
      ``parse`` メソッドをコールバックとして使用しないでください. ``parse`` メソッドをオーバーライドすると, CrawlSpider が機能しなくなります. 
       
   ``cb_kwargs`` は, コールバック関数に渡すキーワード引数を含む ``dict`` です.

   ``follow`` は, このルールで抽出された各レスポンスからリンクをたどるかどうかを指定する bool 値です. 
   ``callback`` が None の場合,  ``follow`` のデフォルトは ``True`` になります. 
   それ以外の場合は ``False`` にデフォルト設定されます

   ``process_links`` は呼び出し可能です. 指定された ``link_extractor`` 
   を使用して各レスポンスから抽出されたリンクのリストごとに呼び出される文字列
   （その名前のスパイダーオブジェクトのメソッドが使用されます）です. 
   これは, 主にフィルタリングの目的で使用されます. 
   
   ``process_request`` は呼び出し可能です. 
   このルールで抽出されたリクエストごとに呼び出され, 
   リクエストを返す必要があります（リクエストをフィルタにかけるには, 
   その名前を持つスパイダオブジェクトのメソッドが呼び出されます）.

CrawlSpider の設定例
~~~~~~~~~~~~~~~~~~~

ルールを使用したCrawlSpiderの例を見てみましょう::

    import scrapy
    from scrapy.spiders import CrawlSpider, Rule
    from scrapy.linkextractors import LinkExtractor

    class MySpider(CrawlSpider):
        name = 'example.com'
        allowed_domains = ['example.com']
        start_urls = ['http://www.example.com']

        rules = (
            # 'category.php' に一致するリンクを抽出する ('subsection.php' とは一致しません)
            # そして, それらのリンクをたどります (なぜならコールバックはデフォルトで follow=True ではないからです).
            Rule(LinkExtractor(allow=('category\.php', ), deny=('subsection\.php', ))),

            # 'allow' と一致するリンクを抽出し, スパイダーの parse_item() メソッドで解析します
            Rule(LinkExtractor(allow=('item\.php', )), callback='parse_item'),
        )

        def parse_item(self, response):
            self.logger.info('Hi, this is an item page! %s', response.url)
            item = scrapy.Item()
            item['id'] = response.xpath('//td[@id="item_id"]/text()').re(r'ID: (\d+)')
            item['name'] = response.xpath('//td[@id="item_name"]/text()').extract()
            item['description'] = response.xpath('//td[@id="item_description"]/text()').extract()
            return item


このスパイダーは, example.comのホームページをクロールし, カテゴリリンクとアイテムリンクを収集し, 
後者を ``parse_item`` メソッドで解析します. 
各アイテムのレスポンスでは, XPathを使用してHTMLからいくつかのデータが抽出され, 
:class:`~scrapy.item.Item` にそのデータをいれています.

XMLFeedSpider
-------------

.. class:: XMLFeedSpider

    XMLFeedSpider は、XML フィードを特定のノード名で繰り返し解析するために設計されています. 
    イテレーター は  ``iternodes``, ``xml``, 及び ``html`` から選択することができます. 
    ``xml`` と ``html`` のイテレーターは、解析するために一度に ``DOM`` 全体を生成するので、
    パフォーマンス上の理由から ``iternode`` イテレーターを使用することをお勧めします. 
    しかし、イテレータとして ``html`` を使用すると、だめなマークアップで書かれた ``XML`` を解析するときに便利です.

    イテレータとタグ名を設定するには、次のクラス属性を定義する必要があります:

    .. attribute:: iterator

        使用するイテレータを定義する文字列:

           - ``'iternodes'`` - 正規表現に基づく高速なイテレータ

           - ``'html'`` - :class:`~scrapy.selector.Selector` を使用するイテレータ. 
             これは DOM 解析を使用しており、大きなフィードの問題となる可能性のあるすべての DOM をメモリにロードする必要があることに注意してください

           - ``'xml'`` - :class:`~scrapy.selector.Selector` を使用するイテレータ. 
             これは DOM 解析を使用しており、大きなフィードの問題となる可能性のあるすべての DOM をメモリにロードする必要があることに注意してください 

        デフォルト: ``'iternodes'``.

    .. attribute:: itertag

        反復処理するノード（または要素）の名前の文字列. 例::

            itertag = 'product'

    .. attribute:: namespaces

        このスパイダーで処理される、文書で利用可能な名前空間を定義する ``(prefix, uri)`` タプルのリスト. 
        ``prefix`` と ``uri`` は、 :meth:`~scrapy.selector.Selector.register_namespace` 
        メソッドを使って名前空間を自動的に登録するために使われます.

        :attr:`itertag` 属性に名前空間を持つノードを指定できます.

        例::

            class YourSpider(XMLFeedSpider):

                namespaces = [('n', 'http://www.sitemaps.org/schemas/sitemap/0.9')]
                itertag = 'n:url'
                # ...

    これらの属性とは別に、このスパイダーは次の無効化可能なメソッドも持っています:

    .. method:: adapt_response(response)

        A method that receives the response as soon as it arrives from the spider
        middleware, before the spider starts parsing it. It can be used to modify
        the response body before parsing it. This method receives a response and
        also returns a response (it could be the same or another one).

    .. method:: parse_node(response, selector)

        This method is called for the nodes matching the provided tag name
        (``itertag``).  Receives the response and an
        :class:`~scrapy.selector.Selector` for each node.  Overriding this
        method is mandatory. Otherwise, you spider won't work.  This method
        must return either a :class:`~scrapy.item.Item` object, a
        :class:`~scrapy.http.Request` object, or an iterable containing any of
        them.

    .. method:: process_results(response, results)

        This method is called for each result (item or request) returned by the
        spider, and it's intended to perform any last time processing required
        before returning the results to the framework core, for example setting the
        item IDs. It receives a list of results and the response which originated
        those results. It must return a list of results (Items or Requests).


XMLFeedSpider の例
~~~~~~~~~~~~~~~~~~~~~

これらのスパイダーはかなり使いやすくいです. 一例を見てみましょう::

    from scrapy.spiders import XMLFeedSpider
    from myproject.items import TestItem

    class MySpider(XMLFeedSpider):
        name = 'example.com'
        allowed_domains = ['example.com']
        start_urls = ['http://www.example.com/feed.xml']
        iterator = 'iternodes'  # This is actually unnecessary, since it's the default value
        itertag = 'item'

        def parse_node(self, response, node):
            self.logger.info('Hi, this is a <%s> node!: %s', self.itertag, ''.join(node.extract()))

            item = TestItem()
            item['id'] = node.xpath('@id').extract()
            item['name'] = node.xpath('name').extract()
            item['description'] = node.xpath('description').extract()
            return item

Basically what we did up there was to create a spider that downloads a feed from
the given ``start_urls``, and then iterates through each of its ``item`` tags,
prints them out, and stores some random data in an :class:`~scrapy.item.Item`.

CSVFeedSpider
-------------

.. class:: CSVFeedSpider

   This spider is very similar to the XMLFeedSpider, except that it iterates
   over rows, instead of nodes. The method that gets called in each iteration
   is :meth:`parse_row`.

   .. attribute:: delimiter

       CSVファイルの各フィールドの区切り文字を含む文字列. デフォルトは ``','`` (コンマ).

   .. attribute:: quotechar

       A string with the enclosure character for each field in the CSV file
       Defaults to ``'"'`` (quotation mark).

   .. attribute:: headers

       ファイルからフィールドを抽出するために使用されるCSVファイルのフィードに含まれる行のリスト.

   .. method:: parse_row(response, row)

       Receives a response and a dict (representing each row) with a key for each
       provided (or detected) header of the CSV file.  This spider also gives the
       opportunity to override ``adapt_response`` and ``process_results`` methods
       for pre- and post-processing purposes.

CSVFeedSpider の例
~~~~~~~~~~~~~~~~~~~~~

:class:`CSVFeedSpider` を使用した, 前の例に似た例を見てみましょう::

    from scrapy.spiders import CSVFeedSpider
    from myproject.items import TestItem

    class MySpider(CSVFeedSpider):
        name = 'example.com'
        allowed_domains = ['example.com']
        start_urls = ['http://www.example.com/feed.csv']
        delimiter = ';'
        quotechar = "'"
        headers = ['id', 'name', 'description']

        def parse_row(self, response, row):
            self.logger.info('Hi, this is a row!: %r', row)

            item = TestItem()
            item['id'] = row['id']
            item['name'] = row['name']
            item['description'] = row['description']
            return item


SitemapSpider
-------------

.. class:: SitemapSpider

    SitemapSpider では,  `Sitemaps`_ を使ってURLを発見することでサイトをクロールできます.

    これはネストされたサイトマップをサポートし,  `robots.txt`_ からサイトマップのURLを見つけることもできます.

    .. attribute:: sitemap_urls

        クロールしたいサイトマップを指すURLのリスト.

        サイトマップのURLを抽出するために `robots.txt`_ を指定することもできます.

    .. attribute:: sitemap_rules

        A list of tuples ``(regex, callback)`` where:

        * ``regex`` is a regular expression to match urls extracted from sitemaps.
          ``regex`` can be either a str or a compiled regex object.

        * callback is the callback to use for processing the urls that match
          the regular expression. ``callback`` can be a string (indicating the
          name of a spider method) or a callable.

        例えば::

            sitemap_rules = [('/product/', 'parse_product')]

        ルールは順番に適用され, 一致する最初のルールのみが使用されます.

        If you omit this attribute, all urls found in sitemaps will be
        processed with the ``parse`` callback.

    .. attribute:: sitemap_follow

        A list of regexes of sitemap that should be followed. This is is only
        for sites that use `Sitemap index files`_ that point to other sitemap
        files.

        デフォルトでは, すべてのサイトマップに従います.

    .. attribute:: sitemap_alternate_links

        Specifies if alternate links for one ``url`` should be followed. These
        are links for the same website in another language passed within
        the same ``url`` block.

        例えば::

            <url>
                <loc>http://example.com/</loc>
                <xhtml:link rel="alternate" hreflang="de" href="http://example.com/de"/>
            </url>

        With ``sitemap_alternate_links`` set, this would retrieve both URLs. With
        ``sitemap_alternate_links`` disabled, only ``http://example.com/`` would be
        retrieved.

        デフォルトでは ``sitemap_alternate_links`` は無効化されています.


SitemapSpider の例
~~~~~~~~~~~~~~~~~~~~~~

かんたんな例: サイトマップを利用して見つけたURLを, すべて ``parse`` コールバックによって処理する::

    from scrapy.spiders import SitemapSpider

    class MySpider(SitemapSpider):
        sitemap_urls = ['http://www.example.com/sitemap.xml']

        def parse(self, response):
            pass # ... scrape item here ...

特定のコールバックと, いくつかの別のコールバックを持つ他のURLを処理する::

    from scrapy.spiders import SitemapSpider

    class MySpider(SitemapSpider):
        sitemap_urls = ['http://www.example.com/sitemap.xml']
        sitemap_rules = [
            ('/product/', 'parse_product'),
            ('/category/', 'parse_category'),
        ]

        def parse_product(self, response):
            pass # ... scrape product ...

        def parse_category(self, response):
            pass # ... scrape category ...

 `robots.txt`_ ファイルによって定義されたサイトマップと, 一部に ``/sitemap_shop`` を含むURLのみを処理する::

    from scrapy.spiders import SitemapSpider

    class MySpider(SitemapSpider):
        sitemap_urls = ['http://www.example.com/robots.txt']
        sitemap_rules = [
            ('/shop/', 'parse_shop'),
        ]
        sitemap_follow = ['/sitemap_shops']

        def parse_shop(self, response):
            pass # ... scrape shop here ...

SitemapSpiderと他のURLのソースを結合する::

    from scrapy.spiders import SitemapSpider

    class MySpider(SitemapSpider):
        sitemap_urls = ['http://www.example.com/robots.txt']
        sitemap_rules = [
            ('/shop/', 'parse_shop'),
        ]

        other_urls = ['http://www.example.com/about']

        def start_requests(self):
            requests = list(super(MySpider, self).start_requests())
            requests += [scrapy.Request(x, self.parse_other) for x in self.other_urls]
            return requests

        def parse_shop(self, response):
            pass # ... scrape shop here ...

        def parse_other(self, response):
            pass # ... scrape other here ...

.. _Sitemaps: http://www.sitemaps.org
.. _Sitemap index files: http://www.sitemaps.org/protocol.html#index
.. _robots.txt: http://www.robotstxt.org/
.. _TLD: https://en.wikipedia.org/wiki/Top-level_domain
.. _Scrapyd documentation: http://scrapyd.readthedocs.org/en/latest/
