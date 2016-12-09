.. _topics-telnetconsole:

==============
Telnet コンソール
==============

.. module:: scrapy.extensions.telnet
   :synopsis: The Telnet Console

Scrapyには、Scrapy実行プロセスを検査および制御するための組み込みのTelnetコンソールが付属しています.
elnetコンソールは、Scrapyプロセス内で実行されている通常のPythonシェルであるため、文字通りその中から何かを行うことができます.

Telnetコンソールは, デフォルトで有効になっている :ref:`組み込みScrapy拡張
<topics-extensions-ref>` ですが, 必要に応じて無効にすることもできます.
拡張機能の詳細については, 
:ref:`topics-extensions-ref-telnetconsole` を参照してください.

.. highlight:: none

Telnetコンソールにアクセスする方法
================================

Telnetコンソールは
:setting:`TELNETCONSOLE_PORT` 設定で定義されているTCPポートをリッスンします.
デフォルトでは ``6023`` に設定されています.
コンソールにアクセスするには::

    telnet localhost 6023
    >>>
    
Windowsにデフォルトでインストールされるtelnetプログラムと、ほとんどのLinuxディストリビューションが必要です.

Telnetコンソールで使用可能な変数
=========================================

elnetコンソールは、Scrapyプロセス内で動作する通常のPythonシェルに似ているので、新しいモジュールのインポートなど、何でもできます. 

しかし, telnetコンソールには, 便宜上いくつかのデフォルト変数が定義されています:

+----------------+-------------------------------------------------------------------+
| Shortcut       | Description                                                       |
+================+===================================================================+
| ``crawler``    | Scrapy クローラー (:class:`scrapy.crawler.Crawler` オブジェクト)         |
+----------------+-------------------------------------------------------------------+
| ``engine``     | Crawler.engine 属性                                               |
+----------------+-------------------------------------------------------------------+
| ``spider``     | アクティブスパイダー                                                    |
+----------------+-------------------------------------------------------------------+
| ``slot``       | エンジンスロット                                                       |
+----------------+-------------------------------------------------------------------+
| ``extensions`` | 拡張マネージャー (Crawler.extensions 属性)                            |
+----------------+-------------------------------------------------------------------+
| ``stats``      | 統計コレクター (Crawler.stats 属性)                                   |
+----------------+-------------------------------------------------------------------+
| ``settings``   | Scrapy設定オブジェクト (Crawler.settings 属性)                         |
+----------------+-------------------------------------------------------------------+
| ``est``        | エンジン状態のレポートを印刷する                                          |
+----------------+-------------------------------------------------------------------+
| ``prefs``      | メモリデバッグ用 ( :ref:`topics-leaks` を参照してください)                  |
+----------------+-------------------------------------------------------------------+
| ``p``          | `pprint.pprint`_ 機能のショートカット                                   |
+----------------+-------------------------------------------------------------------+
| ``hpy``        | メモリデバッグ用 ( :ref:`topics-leaks` を参照してください)                  |
+----------------+-------------------------------------------------------------------+

.. _pprint.pprint: https://docs.python.org/library/pprint.html#pprint.pprint

Telnetコンソールの使用例
=============================

Telnetコンソールで実行できるタスクの例を以下に示します:

エンジンステータスを表示する
----------------------

Scrapyエンジンの ``est()`` メソッドを使用すると、telnetコンソールを使用して状態をすばやく表示できます::

    telnet localhost 6023
    >>> est()
    Execution engine status

    time()-engine.start_time                        : 8.62972998619
    engine.has_capacity()                           : False
    len(engine.downloader.active)                   : 16
    engine.scraper.is_idle()                        : False
    engine.spider.name                              : followall
    engine.spider_is_idle(engine.spider)            : False
    engine.slot.closing                             : False
    len(engine.slot.inprogress)                     : 16
    len(engine.slot.scheduler.dqs or [])            : 0
    len(engine.slot.scheduler.mqs)                  : 92
    len(engine.scraper.slot.queue)                  : 0
    len(engine.scraper.slot.active)                 : 0
    engine.scraper.slot.active_size                 : 0
    engine.scraper.slot.itemproc_size               : 0
    engine.scraper.slot.needs_backout()             : False


Scrapyエンジンの一時停止, 再開, 停止
----------------------------------------

一時停止::

    telnet localhost 6023
    >>> engine.pause()
    >>>

再開::

    telnet localhost 6023
    >>> engine.unpause()
    >>>

停止::

    telnet localhost 6023
    >>> engine.stop()
    Connection closed by foreign host.

Telnetコンソールシグナル
======================

.. signal:: update_telnet_vars
.. function:: update_telnet_vars(telnet_vars)

    STelnetコンソールを開く直前に送信されます. 
    この信号に接続して、telnetローカル名前空間で使用できる変数を追加、削除、または更新することができます. 
    そのためには, ハンドラの ``telnet_vars`` を更新する必要があります.

    :param telnet_vars: telnet の辞書型変数
    :type telnet_vars: dict

Telnet設定
===============

telnetコンソールの動作を制御する設定です:

.. setting:: TELNETCONSOLE_PORT

TELNETCONSOLE_PORT
------------------

デフォルト: ``[6023, 6073]``

Telnetコンソールに使用するポート範囲. 
``None`` または ``0`` に設定すると, 動的に割り当てられたポートが使用されます.


.. setting:: TELNETCONSOLE_HOST

TELNETCONSOLE_HOST
------------------

デフォルト: ``'127.0.0.1'``

Telnetコンソールが監視すべきインターフェース

