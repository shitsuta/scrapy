.. _topics-autothrottle:

======================
AutoThrottle 拡張機能
======================

これは, ScrapyサーバーとクロールしているWebサイトの両方の負荷に基づいてクロール速度を自動的に調整する拡張機能です.

設計目標
============

1. 既定のダウンロード遅延ゼロを使用する代わりに, サイトをより良くする
2. 自動的に最適なクロール速度に調整し, ユーザーはダウンロード遅延をチューニングして最適なものを見つける必要がない.  
   ユーザーは許可する同時要求の最大数を指定するだけで, 残りの部分は拡張機能で処理する.

.. _autothrottle-algorithm:

使い方
============

AutoThrottle拡張機能はダウンロード遅延を動的に調整して, スパイダーが 
:setting:`AUTOTHROTTLE_TARGET_CONCURRENCY` の同時リクエストを各リモートWebサイトに送信します.

ダウンロード待ち時間を使用して遅延を計算します. 
主な考え方は次のとおりです: サーバが応答するのに ``待ち時間`` が必要な場合, クライアントは
``N`` 個の要求を並行して処理するために, 各 ``待ち時間/ N`` 秒間に要求を送信する必要があります.

遅延を調整する代わりに, 一定のダウンロード遅延を設定し, 
:setting:`CONCURRENT_REQUESTS_PER_DOMAIN` または
:setting:`CONCURRENT_REQUESTS_PER_IP` オプションを使用して並行性に厳しい制限を課すことができます. 
同様の効果が得られますが, いくつかの重要な違いがあります:

* ダウンロードの遅延が小さいため, 時折リクエストが激しくなります.
* 多くの場合, non-200 (エラー) レスポンスは通常の応答より速く返される可能性があるため, 
  サーバーがエラーを返すようになると, ダウンロードの遅延が少なく, 並行処理の制限が厳しいため, クローラはサーバーに要求を高速に送信します. 
  しかし, これはクローラが行うべきこととは反対です. エラーの場合, 遅くするのがより理にかなっています.

オートスロットルにはこれらの問題はありません.

スロットルアルゴリズム
====================

オートスロットルアルゴリズムは, 次のルールに基づいてダウンロードの遅延を調整します:

1. スパイダーは常に 
   :setting:`AUTOTHROTTLE_START_DELAY` のダウンロード遅延で始まります.
2. レスポンスが受信されると, ターゲットダウンロード遅延は,  ``latency`` がレスポンスのレイテンシである
   ``latency / N``として計算され, 
   ``N`` は :setting:`AUTOTHROTTLE_TARGET_CONCURRENCY` です.
3. 次の要求のダウンロード遅延は, 前回のダウンロード遅延と目標のダウンロード遅延の平均値に設定されます.
4. レイテンシの non-200 レスポンス待ち時間は遅延を減少させることができない.
5. ダウンロードの遅延が :setting:`DOWNLOAD_DELAY` より小さくなることも,
    :setting:`AUTOTHROTTLE_MAX_DELAY` より大きくなることもできません.
    
.. note:: AutoThrottle拡張機能は, 標準的なScrapyの設定の並行性と遅延を優先します. 
   This means that it will respect
   :setting:`CONCURRENT_REQUESTS_PER_DOMAIN` オプションと, 
   :setting:`CONCURRENT_REQUESTS_PER_IP` オプションを尊重し, ダウンロード遅延を
   :setting:`DOWNLOAD_DELAY` よりも低く設定しないことを意味します.

.. _download-latency:

Scrapyでは, ダウンロードの待ち時間は, TCP接続を確立してからHTTPヘッダーを受信するまでの経過時間として測定されます.

これらのレイテンシは, Scrapyがスパイダーコールバックを処理するのに忙しく, 
ダウンロードに参加できないために, 協調的なマルチタスク環境では正確に測定することが非常に難しいことに注意してください. 
しかし, これらのレイテンシは, Scrapy（そして最終的にはサーバー）がどの程度忙しいかについての妥当な見積もりを与えるはずであり, 
この拡張はその前提に基づいています.

設定
========

オートスロットルエクステンションを制御するための設定以下です:

* :setting:`AUTOTHROTTLE_ENABLED`
* :setting:`AUTOTHROTTLE_START_DELAY`
* :setting:`AUTOTHROTTLE_MAX_DELAY`
* :setting:`AUTOTHROTTLE_DEBUG`
* :setting:`CONCURRENT_REQUESTS_PER_DOMAIN`
* :setting:`CONCURRENT_REQUESTS_PER_IP`
* :setting:`DOWNLOAD_DELAY`

詳細については,  :ref:`autothrottle-algorithm` を参照してください.

.. setting:: AUTOTHROTTLE_ENABLED

AUTOTHROTTLE_ENABLED
~~~~~~~~~~~~~~~~~~~~

デフォルト: ``False``

オートスロットル拡張機能を有効にする.

.. setting:: AUTOTHROTTLE_START_DELAY

AUTOTHROTTLE_START_DELAY
~~~~~~~~~~~~~~~~~~~~~~~~

デフォルト: ``5.0``

最初のダウンロードの遅延（秒単位）

.. setting:: AUTOTHROTTLE_MAX_DELAY

AUTOTHROTTLE_MAX_DELAY
~~~~~~~~~~~~~~~~~~~~~~

デフォルト: ``60.0``

レイテンシが高い場合に設定される最大ダウンロード遅延（秒単位）

.. setting:: AUTOTHROTTLE_TARGET_CONCURRENCY

AUTOTHROTTLE_TARGET_CONCURRENCY
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. versionadded:: 1.1

デフォルト: ``1.0``

リモートWebサイトと並行して送信する必要があるリクエストの平均数.

デフォルトでは, AutoThrottleは, 1つの同時リクエストを各リモートWebサイトに送信する遅延を調整します. 
このオプションをより高い値（たとえば ``2.0`` ）に設定すると, リモートサーバーのスループットと負荷が増加します. 
``AUTOTHROTTLE_TARGET_CONCURRENCY`` が小さいほど（ ``0.5`` など）, クローラはより控えめで丁寧なものになります.

AutoThrottle拡張機能が有効な場合,  :setting:`CONCURRENT_REQUESTS_PER_DOMAIN`
および :setting:`CONCURRENT_REQUESTS_PER_IP` oオプションは引き続き考慮されます. This means that if
``AUTOTHROTTLE_TARGET_CONCURRENCY`` が 
:setting:`CONCURRENT_REQUESTS_PER_DOMAIN` または
:setting:`CONCURRENT_REQUESTS_PER_IP` より高い値に設定されていると, クローラはこの数の同時要求に達しません.

与えられたすべての時点で, Scrapyは ``AUTOTHROTTLE_TARGET_CONCURRENCY`` よりも多かれ少なかれ並行した要求を送ることができます. 
クローラがアプローチしようとする推奨値であり, ハードな制限ではありません.

.. setting:: AUTOTHROTTLE_DEBUG

AUTOTHROTTLE_DEBUG
~~~~~~~~~~~~~~~~~~

デフォルト: ``False``

受信したすべてのレスポンスの統計情報を表示するAutoThrottleデバッグモードを有効にすると, 調整パラメータがリアルタイムでどのように調整されているかがわかります.
