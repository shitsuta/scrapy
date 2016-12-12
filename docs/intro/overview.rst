.. _intro-overview:

==================
Scrapy について
==================

Scrapyは, Webサイトのクロール, データマイニング, 情報処理, アーカイブなどの幅広い有用なアプリケーションに使用できる構造化データを抽出するためのアプリケーションフレームワークです.

Scrapyはもともと `Webスクレイピング`_ 用に設計されていましたが, API( `Amazon Associates Web Services`_ のような)または汎用Webクローラーとしてデータを抽出するためにも使用できます.


スパイダーの作成例
=================================

Scrapy がもたらすものを示すために、Scrapy Spiderの一例を、スパイダーを実行する最も簡単な方法を使って説明します.

ここでは、ページングを追っていきながらウェブサイト
http://quotes.toscrape.com から有名な引用を集めてくるスパイダーのコードを紹介します::

    import scrapy


    class QuotesSpider(scrapy.Spider):
        name = "quotes"
        start_urls = [
            'http://quotes.toscrape.com/tag/humor/',
        ]

        def parse(self, response):
            for quote in response.css('div.quote'):
                yield {
                    'text': quote.css('span.text::text').extract_first(),
                    'author': quote.xpath('span/small/text()').extract_first(),
                }

            next_page = response.css('li.next a::attr("href")').extract_first()
            if next_page is not None:
                next_page = response.urljoin(next_page)
                yield scrapy.Request(next_page, callback=self.parse)


``quotes_spider.py`` のような名前をつけ, 上記のコードをテキストファイルに保存し,
:command:`runspider` コマンドで実行してください::

    scrapy runspider quotes_spider.py -o quotes.json


実行完了後, ``quotes.json`` ファイルにJSON形式の引用リストができあがります. このファイルにはテキストと作者が含まれており、以下のようになっています (読みやすくするため, ここでは再フォーマットしています)::

    [{
        "author": "Jane Austen",
        "text": "\u201cThe person, be it gentleman or lady, who has not pleasure in a good novel, must be intolerably stupid.\u201d"
    },
    {
        "author": "Groucho Marx",
        "text": "\u201cOutside of a dog, a book is man's best friend. Inside of a dog it's too dark to read.\u201d"
    },
    {
        "author": "Steve Martin",
        "text": "\u201cA day without sunshine is like, you know, night.\u201d"
    },
    ...]


今何が起きたの?
-------------------

``scrapy runspider quotes_spider.py`` コマンドが実行されると, Scrapyはその内部のSpider定義を探して, クローラ・エンジンを通して実行しました.

クロールが開始されると ``start_urls`` で定義されたURL（この場合はユーモアカテゴリの引用符のURLのみ）にリクエストを行い, 
デフォルトのコールバックメソッドである ``parse`` に, Response オブジェクトを引数として渡します.
``parse`` コールバックの内部では, CSSセレクタを使用して引用要素をループし, 
抽出されたテキストと作成者でPythonディクテーションを生成し, 次のページへのリンクを探し, コールバックと同じ 
``parse`` メソッドを使用して次のリクエストをスケジュールします.

ここで、Scrapyの主な利点を1つ: リクエストはスケジュールされ, 
:ref:`非同期に処理されます <topics-architecture>`.  
つまり, Scrapyはリクエストが処理されるのを待つ必要はなく, その間に別のリクエストを送信したり, 他の処理を行うことができます. 
これは, リクエストが失敗した場合や, 処理中にエラーが発生した場合でも, 他のリクエストが続行できることを意味します.

これにより、非常に高速なクロールが可能になります（同時に複数の同時要求をフォールトトレラントな方法で送信できます）
また, Scrapyを使用すると, :ref:`いくつかの設定 <topics-settings-ref>` でクロールの公平性を制御できます. 
ドメインごとまたはIPごとに並行要求の量を制限し, 自動的にこれらを把握しようとする :ref:`自動調整拡張機能を使用する <topics-autothrottle>` など, 
各要求のダウンロード遅延を設定するなどの作業を行うことができます.

.. note::

    これは :ref:`フィードのエクスポート <topics-feed-exports>` を使用してJSONファイルを生成し, 
    エクスポート形式（XMLやCSVなど）やストレージバックエンド (例えば, FTP または `Amazon S3`_) を簡単に変更できます. 
    :ref:`アイテムパイプライン <topics-item-pipeline>` 作成してアイテムをデータベースに格納することもできます.


.. _topics-whatelse:

他には?
==========

YScrapyを使用してウェブサイトからアイテムを抽出して保存する方法を見てきましたが, これはごく表面的なものです. 
Scrapyは, スクレイピングを簡単かつ効率的にするための多くの強力な機能を提供します:

* 拡張CSSセレクタとXPath式を使用してHTML / XMLソースからデータを :ref:`選択して抽出する <topics-selectors>` 組み込みサポート, 正規表現を使用して抽出するヘルパーメソッド.

* CSSやXPath式を試してデータを集める :ref:`インタラクティブシェルコンソール <topics-shell>` （IPython対応）. スパイダーの作成やデバッグに非常に便利です.

* 複数の形式（JSON、CSV、XML）で :ref:`フィードのエクスポートを生成 <topics-feed-exports>` し, 複数のバックエンド（FTP, S3, ローカルファイルシステム）に格納するための組み込みサポート.

* 外国語、非標準、壊れたエンコーディング宣言を処理するための強力なエンコーディングサポートと自動検出.

* :ref:`強力な拡張性サポートにより <extending-scrapy>`,  :ref:`シグナル <topics-signals>` と明確に定義されたAPI (ミドルウェア, :ref:`拡張機能 <topics-extensions>`, 
  そして :ref:`パイプライン <topics-item-pipeline>`) を使用して独自の機能を作成することができます.

* さまざまな組み込み拡張機能とハンドリング用ミドルウェア:

  - cookie と session の操作
  - 圧縮, 認証, キャッシングなどのHTTP機能
  - user-agent の操作
  - robots.txt
  - クロールする深度制限
  - などなど
  
* Scrapyプロセス内で動作するPythonコンソールにフックするための :ref:`Telnet コンソール <topics-telnetconsole>` , クローラのイントロスペクションとデバッグ.

* さらに,  `サイトマップ`_ やXML / CSVフィードからサイトをクロールするための再利用可能なスパイダー,  スクラップしたアイテムに関連付けられた画像を（またはその他のメディア）
　:ref:`自動的にダウンロードするメディアパイプライン <topics-media-pipeline>`, キャッシングDNSリゾルバなど, 他にも機能がたくさんあります！
 
次は?
============

次のステップは, :ref:`Scrapy をインストール <intro-install>` し,
:ref:`チュートリアルに従って <intro-tutorial>` 本格的なScrapyプロジェクトを作成し, `コミュニティに参加する`_ 方法を学びます. あなたの興味に感謝！

.. _コミュニティに参加する: http://scrapy.org/community/
.. _Webスクレイピング: https://en.wikipedia.org/wiki/Web_scraping
.. _Amazon Associates Web Services: https://affiliate-program.amazon.com/gp/advertising/api/detail/main.html
.. _Amazon S3: https://aws.amazon.com/s3/
.. _サイトマップ: http://www.sitemaps.org
