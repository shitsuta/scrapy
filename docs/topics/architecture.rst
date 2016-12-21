.. _topics-architecture:

=====================
アーキテクチャの概要
=====================

このドキュメントでは, Scrapyのアーキテクチャとそのコンポーネントがどのように相互作用するかについて説明します.

概要
========

次の図は, Scrapy アーキテクチャの概要とそのコンポーネントと, システム内部で実行されるデータフローの概要を示しています（緑色の矢印）. 
コンポーネントの簡単な説明は, それらの詳細な情報については, 以下のリンクを参照してください. データフローも以下で説明します.

.. _data-flow:

データフロー
=========

.. image:: _images/scrapy_architecture_02.png
   :width: 700
   :height: 470
   :alt: Scrapy architecture

Scrapy のデータフローは実行エンジンによって制御され, 以下のようになります:

1. :ref:`Engine <component-engine>` は, :ref:`Spider <component-spiders>` からクロールする最初のリクエストを取得します.

2. :ref:`Engine <component-engine>` は, :ref:`Scheduler <component-scheduler>` 
   内のリクエストをスケジュールし, 次にクロールするリクエストを要求します.

3. :ref:`Scheduler <component-scheduler>` は, 次のリクエストを :ref:`Engine <component-engine>` に渡します.

4. :ref:`Engine <component-engine>` は, :ref:`Downloader Middlewares <component-downloader-middleware>` 
   を経由して :ref:`Downloader <component-downloader>` にリクエストを送信します
   (:meth:`~scrapy.downloadermiddlewares.DownloaderMiddleware.process_request` を参照してください).

5. ページのダウンロードが終了すると, 
   :ref:`Downloader <component-downloader>` はそのページでレスポンスを生成し, 
   :ref:`Downloader Middlewares <component-downloader-middleware>` 
   (:meth:`~scrapy.downloadermiddlewares.DownloaderMiddleware.process_response` を参照してください) を経由して 
   :ref:`Engine <component-engine>` に送信します.

6. :ref:`Engine <component-engine>` は, レスポンスを :ref:`Downloader <component-downloader>` 
   から受信し, :ref:`Spider Middleware <component-spider-middleware>`
   (:meth:`~scrapy.spidermiddlewares.SpiderMiddleware.process_spider_input` を参照してください)
   を経由する処理のために :ref:`Spider <component-spiders>` に送信します.

7. :ref:`Spider <component-spiders>` はレスポンスを処理し, 
   :ref:`Spider Middleware <component-spider-middleware>` 
   (:meth:`~scrapy.spidermiddlewares.SpiderMiddleware.process_spider_output` を参照してください)
   を経由して, 収集したアイテムと新しいリクエストを :ref:`Engine <component-engine>` に返します.

8. The :ref:`Engine <component-engine>` は, 処理されたアイテムを
   :ref:`Item Pipelines <component-pipelines>` に渡し, 処理されたリクエストを
   :ref:`Scheduler <component-scheduler>` に渡した後, 次のクロールリクエストを要求します.

9. このプロセスは, :ref:`Scheduler <component-scheduler>` からの要求がなくなるまで（ステップ1から）繰り返されます.

コンポーネント
==========

.. _component-engine:

Scrapy エンジン
-------------

エンジンは, システムのすべてのコンポーネント間のデータフローを制御し, 
特定のアクションが発生したときにイベントをトリガーします. 
詳細については, 上記の :ref:`データフロー <data-flow>` のセクションを参照してください.

.. _component-scheduler:

スケジューラー
---------

スケジューラは, エンジンからリクエストを受信し, 
エンジンが要求したときに後で（エンジンにも）それらを供給するためにそれらのキューを実行します.

.. _component-downloader:

ダウンローダー
----------

ダウンローダーは, Web ページを取得してエンジンに供給し, そのエンジンをスパイダーにフィードします.

.. _component-spiders:

スパイダー
-------

スパイダーとは, Scrapy ユーザーがレスポンスを解析し, それらから収集したアイテム
を抽出するためのカスタムクラスです. 詳細は :ref:`topics-spiders` を参照してください.

.. _component-pipelines:

アイテムパイプライン
-------------

アイテムパイプラインは, アイテムがスパイダーによって抽出されるとアイテムの処理を行います. 
典型的なタスクには, クレンジング, 検証, 永続性（アイテムをデータベースに格納するなど）が含まれます. 
詳細は,  :ref:`topics-item-pipeline` を参照してください.

.. _component-downloader-middleware:

ダウンローダーミドルウェア
----------------------

ダウンローダーミドルウェアは, エンジンとダウンローダーの間に位置し, エンジンからダウンローダーに渡されたリクエストと, 
ダウンローダーからエンジンに渡すレスポンスを処理する特定のフックです.

次のいずれかを実行する必要がある場合は, ダウンローダーミドルウェアを使用します:

* ダウンローダに送信される直前のリクエスト（つまり, Scrapy がリクエストをウェブサイトに送信する直前）を処理する
* 受信したレスポンスをスパイダーに渡す前に変更する
* 受け取ったレスポンスをスパイダーに渡す代わりに新しいリクエストを送信する
* ウェブページを取得せずにスパイダーにレスポンスを渡す
* いくつかのリクエストを実行しない

詳細については,  :ref:`topics-downloader-middleware` を参照してください.

.. _component-spider-middleware:

スパイダーミドルウェア
------------------

スパイダーミドルウェアは, エンジンとスパイダーの間に位置し, 
スパイダーの入力（レスポンス）と出力（アイテムとリクエスト）を処理する特定のフックです.

次のいずれかを実行する必要がある場合は, スパイダーミドルウェアを使用します:

* スパイダーコールバックの後処理出力 - リクエストまたはアイテムの変更/追加/削除
* start_requests の後処理
* スパイダーの例外処理
* レスポンスの内容に基づいてリクエストの一部をコールバックする代わりにerrbackを呼び出す.

詳細については,  :ref:`topics-spider-middleware` を参照してください.

イベントドリブンネットワーキング
=======================

Scrapy は, Python の有名なイベント駆動型ネットワークフレームワークである `Twisted`_ 
で書かれています. したがって, 非同期コードを使用して同時実行性を実現しています.

非同期プログラミングと Twisted の詳細については, これらのリンクを参照してください:

* `Introduction to Deferreds in Twisted`_
* `Twisted - hello, asynchronous programming`_
* `Twisted Introduction - Krondo`_

.. _Twisted: https://twistedmatrix.com/trac/
.. _Introduction to Deferreds in Twisted: https://twistedmatrix.com/documents/current/core/howto/defer-intro.html
.. _Twisted - hello, asynchronous programming: http://jessenoller.com/2009/02/11/twisted-hello-asynchronous-programming/
.. _Twisted Introduction - Krondo: http://krondo.com/an-introduction-to-asynchronous-programming-and-twisted/
