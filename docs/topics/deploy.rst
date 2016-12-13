.. _topics-deploy:

=================
スパイダーのデプロイ
=================

このセクションでは, Scrapy のスパイダーを定期的に実行するためのさまざまなオプションについて説明します. 
あなたのローカルマシンで Scrapy のスパイダーを実行するは,（早期の）開発段階にとっては非常に便利ですが, 
ロングランニングスパイダーを実行したり, スパイダーを継続的に稼動させたりするには現実的ではありません. 
ここでは, Scrapy のスパイダーをデプロイするためのソリューションを提案します.

Scrapyのスパイダーを展開するためのオプティカルな選択肢は:

* :ref:`Scrapyd <deploy-scrapyd>` (open source)
* :ref:`Scrapy Cloud <deploy-scrapy-cloud>` (cloud-based)

.. _deploy-scrapyd:

Scrapyd サーバーへのデプロイ
=============================

`Scrapyd`_ は Scrapy のスパイダーを実行するためのオープンソースのアプリケーションです. 
これは, HTTP API を備えたサーバーを提供し, Scrapy スパイダーの実行および監視することができます.

スパイダーを Scrapyd にデプロイするには,  `scrapyd-client`_ パッケージが提供する scrapyd-deploy ツールを使用します. 
詳細については,  `scrapyd-deploy documentation`_ を参照してください.

Scrapyd は何人かの Scrapy 開発者によって維持されています.

.. _deploy-scrapy-cloud:

Scrapy Cloud へのデプロイ
=========================

`Scrapy Cloud`_ はScrapyの背後にある `Scrapinghub`_,
のホスティング型のクラウドベースのサービスです.

Scrapy Cloud は, サーバーのセットアップと監視の必要性を排除し, 
スパイダーを管理し, スクラップされたアイテム, ログ, 統計情報を確認するための優れたUIを提供します.

Scrapy Cloud にスパイダーをデプロイするには,  `shub`_ コマンドラインツールを使用します. 
詳細については,  `Scrapy Cloud documentation`_ を参照してください.

Scrapy Cloud はScrapydと互換性があり, 必要に応じてそれらの間を切り替えることができます. 
``scrapyd-deploy`` と同様に ``scrapy.cfg`` ファイルから読みこまれます.

.. _Scrapyd: https://github.com/scrapy/scrapyd
.. _Deploying your project: https://scrapyd.readthedocs.org/en/latest/deploy.html
.. _Scrapy Cloud: http://scrapinghub.com/scrapy-cloud/
.. _scrapyd-client: https://github.com/scrapy/scrapyd-client
.. _shub: http://doc.scrapinghub.com/shub.html
.. _scrapyd-deploy documentation: http://scrapyd.readthedocs.org/en/latest/deploy.html
.. _Scrapy Cloud documentation: http://doc.scrapinghub.com/scrapy-cloud.html
.. _Scrapinghub: http://scrapinghub.com/
