.. _topics-practices:

================
一般的なプラクティス
================

このセクションでは, Scrapyを使用する際の一般的な方法について説明します. 
これらは, 多くの話題を網羅しており, 他の特定のセクションにはいるのはよくありません.

.. _run-from-script:

スクリプトから Scrapy を実行する
========================

``scrapy crawl`` コマンドで Scrapy を実行する従来の方法に加え, 
:ref:`API <topics-api>` を利用してスクリプトから Scrapy を実行することができます.

Scrapy は Twisted 非同期ネットワーキングライブラリの上に構築されているので, Twisted リアクタ内で実行する必要があります.

スパイダーを実行するために使用できる最初のユーティリティーは
:class:`scrapy.crawler.CrawlerProcess` です. このクラスは, 
Twisted リアクターを開始し, ロギング・シャットダウンハンドラを設定します. 
このクラスは, すべてのScrapyコマンドで使用されるクラスです.

ここで, 1つのスパイダーを実行する方法の例を示します.

::

    import scrapy
    from scrapy.crawler import CrawlerProcess

    class MySpider(scrapy.Spider):
        # 独自のスパイダー定義
        ...

    process = CrawlerProcess({
        'USER_AGENT': 'Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 5.1)'
    })

    process.crawl(MySpider)
    process.start() # クロールが終了するまでスクリプトはここでブロックされます

:class:`~scrapy.crawler.CrawlerProcess` ドキュメントをチェックして, 使用法の詳細を確認してください.

プロジェクトの内部にいる場合は, プロジェクト内のコンポーネントをインポートするために使用できる追加のヘルパーがいくつかあります. 
名前を渡すスパイダーを
:class:`~scrapy.crawler.CrawlerProcess` に自動的にインポートし,  
``get_project_settings`` を使用してプロジェクト設定で :class:`~scrapy.settings.Settings`
インスタンスを取得することができます.

これは,  `testspiders`_ プロジェクトを例とし, 実行する方法の実例です.

::

    from scrapy.crawler import CrawlerProcess
    from scrapy.utils.project import get_project_settings

    process = CrawlerProcess(get_project_settings())

    # 'followall' はプロジェクトのスパイダーの名前です.
    process.crawl('followall', domain='scrapinghub.com')
    process.start() # クロールが終了するまでスクリプトはここでブロックされます

クロールプロセスをより詳細に制御できるScrapyユーティリティとして :class:`scrapy.crawler.CrawlerRunner` があります. 
このクラスは, いくつかのヘルパーをカプセル化して複数のクローラーを実行するかんたんなラッパーですが, 
既存のリアクターを開始したり干渉したりすることはありません.

このクラスを使用して, リアクターはスパイダーをスケジュールした後に明示的に実行する必要があります. 
アプリケーションがすでにTwistedを使用していて, 同じリアクターでScrapyを実行する場合は, 
 :class:`~scrapy.crawler.CrawlerProcess` ではなく, 
 :class:`~scrapy.crawler.CrawlerRunner` を使用することをお勧めします.

スパイダーが完成した後, Twistedリアクターを手動でシャットダウンする必要があります. 
これは, :meth:`CrawlerRunner.crawl <scrapy.crawler.CrawlerRunner.crawl>` 
メソッドによって返された遅延にコールバックを追加することで実現できます.

MySpiderの実行が終了した後, コールバックとともにリアクターを手動で停止する, 使用例を示します.

::

    from twisted.internet import reactor
    import scrapy
    from scrapy.crawler import CrawlerRunner
    from scrapy.utils.log import configure_logging

    class MySpider(scrapy.Spider):
        # 独自のスパイダー定義
        ...

    configure_logging({'LOG_FORMAT': '%(levelname)s: %(message)s'})
    runner = CrawlerRunner()

    d = runner.crawl(MySpider)
    d.addBoth(lambda _: reactor.stop())
    reactor.run() # クロールが終了するまでスクリプトはここでブロックされます

.. seealso:: `Twisted Reactor Overview`_.

.. _run-multiple-spiders:

同じプロセスで複数のスパイダーを実行する
============================================

デフォルトでは, Scrapy は ``scrapy crawl`` を実行するときにプロセスごとに1つのスパイダーを実行します. 
ただし, Scrapy は :ref:`内部 API <topics-api>` を使用することでプロセスごとに複数のスパイダーを実行できます.

以下は, 複数のスパイダーを同時に実行する例です:

::

    import scrapy
    from scrapy.crawler import CrawlerProcess

    class MySpider1(scrapy.Spider):
        # 一番目の独自のスパイダーの定義
        ...

    class MySpider2(scrapy.Spider):
        # 二番目の独自のスパイダーの定義
        ...

    process = CrawlerProcess()
    process.crawl(MySpider1)
    process.crawl(MySpider2)
    process.start() # すべてのクロールジョブが終了するまでスクリプトはここでブロックされます

:class:`~scrapy.crawler.CrawlerRunner` を使用した同様の例です:

::

    import scrapy
    from twisted.internet import reactor
    from scrapy.crawler import CrawlerRunner
    from scrapy.utils.log import configure_logging

    class MySpider1(scrapy.Spider):
        # 一番目の独自のスパイダーの定義
        ...

    class MySpider2(scrapy.Spider):
        # 二番目の独自のスパイダーの定義
        ...

    configure_logging()
    runner = CrawlerRunner()
    runner.crawl(MySpider1)
    runner.crawl(MySpider2)
    d = runner.join()
    d.addBoth(lambda _: reactor.stop())

    reactor.run() # すべてのクロールジョブが終了するまで, スクリプトはここでブロックされます

同様の例ですが, 遅延を連鎖させてスパイダーを順番に実行しています:

::

    from twisted.internet import reactor, defer
    from scrapy.crawler import CrawlerRunner
    from scrapy.utils.log import configure_logging

    class MySpider1(scrapy.Spider):
        # 一番目の独自のスパイダーの定義
        ...

    class MySpider2(scrapy.Spider):
        # 二番目の独自のスパイダーの定義
        ...

    configure_logging()
    runner = CrawlerRunner()

    @defer.inlineCallbacks
    def crawl():
        yield runner.crawl(MySpider1)
        yield runner.crawl(MySpider2)
        reactor.stop()

    crawl()
    reactor.run() # 最後のクロールコールが終了するまで, スクリプトはここでブロックされます
    
.. seealso:: :ref:`run-from-script`.

.. _distributed-crawls:

分散クロール
==================

Scrapy は, 配布（マルチサーバー）方式でクロールを実行するための組み込み機能を提供していません. 
ただし, クロールを配布する方法はいくつかあり, その方法は配布方法によって異なります.

スパイダーがたくさんある場合, 負荷を分散させる明白な方法は, 多くのScrapydインスタンスをセットアップし, スパイダーをその中で実行することです.

多くのマシンで単一の（大きな）スパイダーを実行する場合は, 通常はクロールするURLを分割して別々のスパイダーに送信します. 
具体的な例を次に示します:

まず, クロールするURLのリストを用意して, 別々のファイル/URLに入れます::

    http://somedomain.com/urls-to-crawl/spider1/part1.list
    http://somedomain.com/urls-to-crawl/spider1/part2.list
    http://somedomain.com/urls-to-crawl/spider1/part3.list

次に, 3つのScrapydサーバーでスパイダーを実行します. スパイダーは,
(spider) 引数 ``part`` にクロールするパーティションの番号を渡します::

    curl http://scrapy1.mycompany.com:6800/schedule.json -d project=myproject -d spider=spider1 -d part=1
    curl http://scrapy2.mycompany.com:6800/schedule.json -d project=myproject -d spider=spider1 -d part=2
    curl http://scrapy3.mycompany.com:6800/schedule.json -d project=myproject -d spider=spider1 -d part=3

.. _bans:

BANされることを回避する
=======================

いくつかのウェブサイトでは, ボットがWebサイトをクロールするのを防ぐために, さまざまな洗練された手段を実装しています. 
これらの措置を回避することは非常に困難なことがあり, 特別なインフラストラクチャが必要な場合があります. 
ご不明な点がある場合は, `商用サポート`_ にお問い合わせください.

 これらの種類のサイトを扱う際に留意すべきヒントをいくつか紹介します:

* ユーザーエージェントを, よく知られているブラウザのプールからローテーションします（Googleのリストを取得するにはGoogleを使用します）
* 一部のサイトでは, クッキーを使用してボットの動作を特定する場合があるため, クッキーを無効にする ( :setting:`COOKIES_ENABLED` を参照してください).
* ダウンロード遅延 (2 or higher) を使用する.  :setting:`DOWNLOAD_DELAY` 設定を参照してください.
* 可能であれば, サイトに直接アクセスするのではなく, `Google cache`_ を使用してページを取得する
* IPプールをローテーションさせ使用します。たとえば, 無料の `Tor project`_ や
  `ProxyMesh`_ のような有料サービスです. また, あなた自身のプロキシを添付できるスーパープロキシである `scrapoxy`_ のようなオープンソースのプロジェクトが有ります.
* 内部的に禁止を回避する高度に分散されたダウンローダを使用するので, クリーンなページの解析に集中することができます. 
  そのようなダウンローダの一例に `Crawlera`_ があります.
  
それでもあなたのボットが禁止されるのを防ぐことができない場合は,  `商用サポート`_ に連絡することを検討してください.

.. _Tor project: https://www.torproject.org/
.. _商用サポート: http://scrapy.org/support/
.. _ProxyMesh: http://proxymesh.com/
.. _Google cache: http://www.googleguide.com/cached_pages.html
.. _testspiders: https://github.com/scrapinghub/testspiders
.. _Twisted Reactor Overview: https://twistedmatrix.com/documents/current/core/howto/reactor-basics.html
.. _Crawlera: http://scrapinghub.com/crawlera
.. _scrapoxy: http://scrapoxy.io/
