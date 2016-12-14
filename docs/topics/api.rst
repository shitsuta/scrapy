.. _topics-api:

========
コア API
========

.. versionadded:: 0.15

このセクションでは, ScrapyコアAPIについて説明します. 拡張APIやミドルウェアの開発者向けです.

.. _topics-api-crawler:

クローラー API
===========

Scrapy API の主なエントリポイントは,  ``from_crawler`` メソッドを使用して拡張機能に渡される 
:class:`~scrapy.crawler.Crawler` オブジェクトです. 
このオブジェクトはすべての Scrapy コアコンポーネントへのアクセスを提供し, 
拡張機能がそれらにアクセスして機能を Scrapy に引き込む唯一の方法です.

.. module:: scrapy.crawler
   :synopsis: The Scrapy crawler

拡張機能マネージャーは, インストールされている拡張機能の読み込みと追跡を担当しており, 使用可能なすべての拡張機能の辞書と, 
:ref:`ダウンローダーミドルウェアの設定 <topics-downloader-middleware-setting>` 
方法と同様の命令を含む :setting:`EXTENSIONS` 設定によって構成されています.

.. class:: Crawler(spidercls, settings)

    Crawler オブジェクトは, 
    :class:`scrapy.spiders.Spider` サブクラスと
    :class:`scrapy.settings.Settings` オブジェクトでインスタンス化する必要があります.

    .. attribute:: settings

        クローラの設定マネージャ.

        これは, このクローラーのScrapy設定にアクセスするために拡張機能とミドルウェアによって使用されます.

        詳細については,  :ref:`topics-settings` を参照してください.

        APIについては, :class:`~scrapy.settings.Settings` クラスを参照してください.

    .. attribute:: signals

        クローラーの信号マネージャ.

        これは, 拡張機能とミドルウェアがScrapy機能にフックするために使用されます.

        詳細については, :ref:`topics-signals` を参照してください.

        APIについては, :class:`~scrapy.signalmanager.SignalManager` クラスを参照してください.

    .. attribute:: stats

        クローラーの統計コレクタ.

        これは, エクステンションとミドルウェアから自分の行動の統計情報を記録したり, 
        他の拡張機能によって収集された統計情報にアクセスするために使用されます.

        詳細については :ref:`topics-stats` を参照してください.

        API については :class:`~scrapy.statscollectors.StatsCollector` クラスを参照してください.

    .. attribute:: extensions

        有効な拡張機能を追跡する拡張マネージャー.

        ほとんどの拡張機能はこの属性にアクセスする必要はありません.

        拡張機能の紹介とScrapyの利用可能な拡張機能のリストについては, :ref:`topics-extensions` を参照してください.

    .. attribute:: engine

        実行エンジン. スケジューラ, ダウンローダ, およびスパイダ間のコアクロールロジックを調整します.

        Scrapy エンジンにアクセスして, ダウンローダとスケジューラの動作を検査または変更したい場合にしようできます. 
        ただし, これは高度な使い方であり, APIはまだ安定していません.
        
    .. attribute:: spider

        Spider currently being crawled. 
        クローラの構築中に提供されるspiderクラスのインスタンスであり, :meth:`crawl` メソッドで指定された引数の後に作成されます.

    .. method:: crawl(\*args, \**kwargs)

        クローラー開始時に, 指定された `args` および `kwargs` 引数を使用してスパイダーをインスタンス化し, 実行エンジンの動作を設定します.

        クロールが終了したときに発生する遅延を返します.

.. autoclass:: CrawlerRunner
   :members:

.. autoclass:: CrawlerProcess
   :show-inheritance:
   :members:
   :inherited-members:

.. _topics-api-settings:

設定 API
============

.. module:: scrapy.settings
   :synopsis: Settings manager

.. attribute:: SETTINGS_PRIORITIES

    Scrapyで使用されるデフォルトの設定優先度のキー名と優先度を辞書形式で設定します.

    各項目は設定エントリポイントを定義し, 
    識別のためのコード名と整数の優先順位を与えます. 
    :class:`~scrapy.settings.Settings` クラスで値を設定したり取得したりするときは, 優先度が高いほど優先度が低くなります.

    .. highlight:: python

    ::

        SETTINGS_PRIORITIES = {
            'default': 0,
            'command': 10,
            'project': 20,
            'spider': 30,
            'cmdline': 40,
        }

    各設定の詳細については, 
    :ref:`topics-settings` を参照してください.

.. autofunction:: get_settings_priority

.. autoclass:: Settings
   :show-inheritance:
   :members:

.. autoclass:: BaseSettings
   :members:

.. _topics-api-spiderloader:

スパイダーローダー API
==================

.. module:: scrapy.loader
   :synopsis: The spider loader

.. class:: SpiderLoader

    このクラスは, プロジェクト全体で定義されたスパイダークラスの取得と処理を担当します.

    :setting:`SPIDER_LOADER_CLASS` プロジェクト設定でパスを指定することで, カスタムスパイダーローダーを使用できます. 
    エラーのない実行を保証するために :class:`scrapy.interfaces.ISpiderLoader` インタフェースを完全に実装する必要があります.

    .. method:: from_settings(settings)

       このクラスメソッドは, クラスのインスタンスを作成するためにScrapyによって使用されます.
       プロジェクト設定で呼び出され,  :setting:`SPIDER_MODULES` 
       設定で指定したモジュール内で再帰的に見つかったスパイダーをロードします.

       :param settings: プロジェクト設定
       :type settings: :class:`~scrapy.settings.Settings` インスタンス
       
    .. method:: load(spider_name)

       Spider クラスを指定された名前で取得します. 以前に読み込まれたスパイダーのうち, 
       `spider_name` という名前の Spider クラスが検索され, 見つからなければ KeyError が発生します.

       :param spider_name: Spider クラス名
       :type spider_name: str

    .. method:: list()

       プロジェクトで利用可能なスパイダーの名前を取得する.

    .. method:: find_by_request(request)

       指定されたリクエストを処理できるスパイダーの名前をリストします. 要求のURLとスパイダーのドメインを照合しようとします.

       :param request: 問い合わされたリクエスト
       :type request: :class:`~scrapy.http.Request` インスタンス
       
.. _topics-api-signals:

シグナル API
===========

.. automodule:: scrapy.signalmanager
    :synopsis: The signal manager
    :members:
    :undoc-members:

.. _topics-api-stats:

統計コレクター API
===================

:mod:`scrapy.statscollectors` モジュールの下に利用可能ないくつかの統計コレクタがあり, それらはすべて
:class:`~scrapy.statscollectors.StatsCollector` 
クラスで定義されたStatsコレクタAPIを実装しています（すべて継承しています）.

.. module:: scrapy.statscollectors
   :synopsis: Stats Collectors

.. class:: StatsCollector

    .. method:: get_value(key, default=None)

        指定されたstatsキーの値を返します. 存在しない場合はdefaultを返します.

    .. method:: get_stats()

        現在実行中のスパイダーからすべての統計情報を dict として取得します.

    .. method:: set_value(key, value)

        与えられた stats キーに与えられた値を設定する.

    .. method:: set_stats(stats)

        ``stats`` 引数に渡された dict で現在の統計をオーバーライドします.

    .. method:: inc_value(key, count=1, start=0)

        指定された統計値キーの値を与えられた数だけ増やします（設定されていない場合は, 与えられた開始値を仮定する）.

    .. method:: max_value(key, value)

        指定されたキーの現在の値が ``value`` よりも小さい場合にのみ, 指定されたキーの値に ``value`` を設定します. 
        指定されたキーに現在の値がない場合, 値は常に設定されます. 

    .. method:: min_value(key, value)

        指定されたキーの現在の値が ``value`` より大きい場合にのみ, 指定されたキーの値に ``value`` を設定します. 
        指定されたキーに現在の値がない場合, 値は常に設定されます.

    .. method:: clear_stats()

        すべての統計情報をクリアします.

    次のメソッドは, 統計収集APIの一部ではなく, カスタム統計コレクタを実装するときに使用されます:

    .. method:: open_spider(spider)

        統計収集のために指定されたスパイダーを開きます.

    .. method:: close_spider(spider)

        指定されたスパイダーを閉じます. これが呼び出された後, 特定の統計情報にアクセスまたは収集することはできません.
        
.. _reactor: https://twistedmatrix.com/documents/current/core/howto/reactor-basics.html
