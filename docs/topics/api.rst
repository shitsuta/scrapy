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

The Extension Manager is responsible for loading and keeping track of installed
extensions and it's configured through the :setting:`EXTENSIONS` setting which
contains a dictionary of all available extensions and their order similar to
how you :ref:`configure the downloader middlewares
<topics-downloader-middleware-setting>`.

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

    Dictionary that sets the key name and priority level of the default
    settings priorities used in Scrapy.

    Each item defines a settings entry point, giving it a code name for
    identification and an integer priority. Greater priorities take more
    precedence over lesser ones when setting and retrieving values in the
    :class:`~scrapy.settings.Settings` class.

    .. highlight:: python

    ::

        SETTINGS_PRIORITIES = {
            'default': 0,
            'command': 10,
            'project': 20,
            'spider': 30,
            'cmdline': 40,
        }

    For a detailed explanation on each settings sources, see:
    :ref:`topics-settings`.

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

    This class is in charge of retrieving and handling the spider classes
    defined across the project.

    Custom spider loaders can be employed by specifying their path in the
    :setting:`SPIDER_LOADER_CLASS` project setting. They must fully implement
    the :class:`scrapy.interfaces.ISpiderLoader` interface to guarantee an
    errorless execution.

    .. method:: from_settings(settings)

       This class method is used by Scrapy to create an instance of the class.
       It's called with the current project settings, and it loads the spiders
       found recursively in the modules of the :setting:`SPIDER_MODULES`
       setting.

       :param settings: project settings
       :type settings: :class:`~scrapy.settings.Settings` instance

    .. method:: load(spider_name)

       Get the Spider class with the given name. It'll look into the previously
       loaded spiders for a spider class with name `spider_name` and will raise
       a KeyError if not found.

       :param spider_name: spider class name
       :type spider_name: str

    .. method:: list()

       Get the names of the available spiders in the project.

    .. method:: find_by_request(request)

       List the spiders' names that can handle the given request. Will try to
       match the request's url against the domains of the spiders.

       :param request: queried request
       :type request: :class:`~scrapy.http.Request` instance

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

There are several Stats Collectors available under the
:mod:`scrapy.statscollectors` module and they all implement the Stats
Collector API defined by the :class:`~scrapy.statscollectors.StatsCollector`
class (which they all inherit from).

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

        Increment the value of the given stats key, by the given count,
        assuming the start value given (when it's not set).

    .. method:: max_value(key, value)

        Set the given value for the given key only if current value for the
        same key is lower than value. If there is no current value for the
        given key, the value is always set.

    .. method:: min_value(key, value)

        Set the given value for the given key only if current value for the
        same key is greater than value. If there is no current value for the
        given key, the value is always set.

    .. method:: clear_stats()

        すべての統計情報をクリアします.

    次のメソッドは, 統計収集APIの一部ではなく, カスタム統計コレクタを実装するときに使用されます:

    .. method:: open_spider(spider)

        統計収集のために指定されたスパイダーを開きます.

    .. method:: close_spider(spider)

        指定されたスパイダーを閉じます。これが呼び出された後, 特定の統計情報にアクセスまたは収集することはできません.
        
.. _reactor: https://twistedmatrix.com/documents/current/core/howto/reactor-basics.html
