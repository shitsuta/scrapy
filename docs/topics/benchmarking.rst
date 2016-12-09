.. _benchmarking:

============
ベンチマーク
============

.. versionadded:: 0.17

Scrapyには、ローカルのHTTPサーバーを生成し、最大限の速度でクロールする単純なベンチマークスイートが付属しています.  
このベンチマークの目的は、比較のための共通ベースラインを得るために、Scrapyがハードウェアでどのように機能するかを理解することです.  
これは何もせず, リンクをたどる単純なスパイダーを使用します.  
実行するには::

    scrapy bench

このような出力が表示されるはずです::

    2013-05-16 13:08:46-0300 [scrapy] INFO: Scrapy 0.17.0 started (bot: scrapybot)
    2013-05-16 13:08:47-0300 [scrapy] INFO: Spider opened
    2013-05-16 13:08:47-0300 [scrapy] INFO: Crawled 0 pages (at 0 pages/min), scraped 0 items (at 0 items/min)
    2013-05-16 13:08:48-0300 [scrapy] INFO: Crawled 74 pages (at 4440 pages/min), scraped 0 items (at 0 items/min)
    2013-05-16 13:08:49-0300 [scrapy] INFO: Crawled 143 pages (at 4140 pages/min), scraped 0 items (at 0 items/min)
    2013-05-16 13:08:50-0300 [scrapy] INFO: Crawled 210 pages (at 4020 pages/min), scraped 0 items (at 0 items/min)
    2013-05-16 13:08:51-0300 [scrapy] INFO: Crawled 274 pages (at 3840 pages/min), scraped 0 items (at 0 items/min)
    2013-05-16 13:08:52-0300 [scrapy] INFO: Crawled 343 pages (at 4140 pages/min), scraped 0 items (at 0 items/min)
    2013-05-16 13:08:53-0300 [scrapy] INFO: Crawled 410 pages (at 4020 pages/min), scraped 0 items (at 0 items/min)
    2013-05-16 13:08:54-0300 [scrapy] INFO: Crawled 474 pages (at 3840 pages/min), scraped 0 items (at 0 items/min)
    2013-05-16 13:08:55-0300 [scrapy] INFO: Crawled 538 pages (at 3840 pages/min), scraped 0 items (at 0 items/min)
    2013-05-16 13:08:56-0300 [scrapy] INFO: Crawled 602 pages (at 3840 pages/min), scraped 0 items (at 0 items/min)
    2013-05-16 13:08:57-0300 [scrapy] INFO: Closing spider (closespider_timeout)
    2013-05-16 13:08:57-0300 [scrapy] INFO: Crawled 666 pages (at 3840 pages/min), scraped 0 items (at 0 items/min)
    2013-05-16 13:08:57-0300 [scrapy] INFO: Dumping Scrapy stats:
        {'downloader/request_bytes': 231508,
         'downloader/request_count': 682,
         'downloader/request_method_count/GET': 682,
         'downloader/response_bytes': 1172802,
         'downloader/response_count': 682,
         'downloader/response_status_count/200': 682,
         'finish_reason': 'closespider_timeout',
         'finish_time': datetime.datetime(2013, 5, 16, 16, 8, 57, 985539),
         'log_count/INFO': 14,
         'request_depth_max': 34,
         'response_received_count': 682,
         'scheduler/dequeued': 682,
         'scheduler/dequeued/memory': 682,
         'scheduler/enqueued': 12767,
         'scheduler/enqueued/memory': 12767,
         'start_time': datetime.datetime(2013, 5, 16, 16, 8, 47, 676539)}
    2013-05-16 13:08:57-0300 [scrapy] INFO: Spider closed (closespider_timeout)

これは、Scrapyを実行しているハードウェアで毎分約3900ページをクロールできることを示しています.  
これはリンクをたどることを目的とした非常にシンプルなスパイダーであることに注意してください.  
カスタムスパイダーを作成すると, クロール速度が遅くなるほど多くの処理が行われます.  
どのくらい遅くなるかは、あなたのスパイダーがどれくらいのことをしているか, どの程度うまく書かれているかによって決まります.  

将来、他の一般的なシナリオをカバーするために, ベンチマークスイートにさらに多くのケースが追加されます.
