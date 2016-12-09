.. _topics-autothrottle:

======================
AutoThrottle 拡張機能
======================

これは、ScrapyサーバーとクロールしているWebサイトの両方の負荷に基づいてクロール速度を自動的に調整する拡張機能です.

設計目標
============

1. 既定のダウンロード遅延ゼロを使用する代わりに、サイトをより良くする
2. 自動的に最適なクロール速度に調整し, ユーザーはダウンロード遅延をチューニングして最適なものを見つける必要がない.
   ユーザーは許可する同時要求の最大数を指定するだけで, 残りの部分は拡張機能で処理する.

.. _autothrottle-algorithm:

使い方
============

AutoThrottle拡張機能はダウンロード遅延を動的に調整して、スパイダーが 
:setting:`AUTOTHROTTLE_TARGET_CONCURRENCY` の同時リクエストを各リモートWebサイトに送信します.

ダウンロード待ち時間を使用して遅延を計算します. 
主な考え方は次のとおりです: サーバが応答するのに ``待ち時間`` が必要な場合, クライアントは
``N`` 個の要求を並行して処理するために, 各 ``待ち時間/ N`` 秒間に要求を送信する必要があります.

Instead of adjusting the delays one can just set a small fixed
download delay and impose hard limits on concurrency using
:setting:`CONCURRENT_REQUESTS_PER_DOMAIN` or
:setting:`CONCURRENT_REQUESTS_PER_IP` options. It will provide a similar
effect, but there are some important differences:

* because the download delay is small there will be occasional bursts
  of requests;
* often non-200 (error) responses can be returned faster than regular
  responses, so with a small download delay and a hard concurrency limit
  crawler will be sending requests to server faster when server starts to
  return errors. But this is an opposite of what crawler should do - in case
  of errors it makes more sense to slow down: these errors may be caused by
  the high request rate.

AutoThrottle doesn't have these issues.

スロットルアルゴリズム
====================

オートスロットルアルゴリズムは、次のルールに基づいてダウンロードの遅延を調整します:

1. spiders always start with a download delay of
   :setting:`AUTOTHROTTLE_START_DELAY`;
2. when a response is received, the target download delay is calculated as
   ``latency / N`` where ``latency`` is a latency of the response,
   and ``N`` is :setting:`AUTOTHROTTLE_TARGET_CONCURRENCY`.
3. download delay for next requests is set to the average of previous
   download delay and the target download delay;
4. latencies of non-200 responses are not allowed to decrease the delay;
5. download delay can't become less than :setting:`DOWNLOAD_DELAY` or greater
   than :setting:`AUTOTHROTTLE_MAX_DELAY`

.. note:: The AutoThrottle extension honours the standard Scrapy settings for
   concurrency and delay. This means that it will respect
   :setting:`CONCURRENT_REQUESTS_PER_DOMAIN` and
   :setting:`CONCURRENT_REQUESTS_PER_IP` options and
   never set a download delay lower than :setting:`DOWNLOAD_DELAY`.

.. _download-latency:

In Scrapy, the download latency is measured as the time elapsed between
establishing the TCP connection and receiving the HTTP headers.

Note that these latencies are very hard to measure accurately in a cooperative
multitasking environment because Scrapy may be busy processing a spider
callback, for example, and unable to attend downloads. However, these latencies
should still give a reasonable estimate of how busy Scrapy (and ultimately, the
server) is, and this extension builds on that premise.

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

より多くの情報は :ref:`autothrottle-algorithm` を確認してください.

.. setting:: AUTOTHROTTLE_ENABLED

AUTOTHROTTLE_ENABLED
~~~~~~~~~~~~~~~~~~~~

初期値: ``False``

Enables the AutoThrottle extension.

.. setting:: AUTOTHROTTLE_START_DELAY

AUTOTHROTTLE_START_DELAY
~~~~~~~~~~~~~~~~~~~~~~~~

初期値: ``5.0``

The initial download delay (in seconds).

.. setting:: AUTOTHROTTLE_MAX_DELAY

AUTOTHROTTLE_MAX_DELAY
~~~~~~~~~~~~~~~~~~~~~~

初期値: ``60.0``

The maximum download delay (in seconds) to be set in case of high latencies.

.. setting:: AUTOTHROTTLE_TARGET_CONCURRENCY

AUTOTHROTTLE_TARGET_CONCURRENCY
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. versionadded:: 1.1

初期値: ``1.0``

Average number of requests Scrapy should be sending in parallel to remote
websites.

By default, AutoThrottle adjusts the delay to send a single
concurrent request to each of the remote websites. Set this option to
a higher value (e.g. ``2.0``) to increase the throughput and the load on remote
servers. A lower ``AUTOTHROTTLE_TARGET_CONCURRENCY`` value
(e.g. ``0.5``) makes the crawler more conservative and polite.

Note that :setting:`CONCURRENT_REQUESTS_PER_DOMAIN`
and :setting:`CONCURRENT_REQUESTS_PER_IP` options are still respected
when AutoThrottle extension is enabled. This means that if
``AUTOTHROTTLE_TARGET_CONCURRENCY`` is set to a value higher than
:setting:`CONCURRENT_REQUESTS_PER_DOMAIN` or
:setting:`CONCURRENT_REQUESTS_PER_IP`, the crawler won't reach this number
of concurrent requests.

At every given time point Scrapy can be sending more or less concurrent
requests than ``AUTOTHROTTLE_TARGET_CONCURRENCY``; it is a suggested
value the crawler tries to approach, not a hard limit.

.. setting:: AUTOTHROTTLE_DEBUG

AUTOTHROTTLE_DEBUG
~~~~~~~~~~~~~~~~~~

初期値: ``False``

Enable AutoThrottle debug mode which will display stats on every response
received, so you can see how the throttling parameters are being adjusted in
real time.
