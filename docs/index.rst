.. _topics-index:

==============================
Scrapy |version| ドキュメント
==============================

このドキュメントには、Scrapyについて知っておくべきすべてが含まれています。

ヘルプ
============

トラブルですか? 私達が助けます!

* :doc:`FAQ <faq>` を試してください -- いくつかのよくある質問への答えがあります.
* 具体的な情報をお探しですか？ :ref:`genindex` か :ref:`modindex` を試してください.
* `StackOverflow using the scrapy tag`_ で質問するか, 答えを探してください.
* `archives of the scrapy-users mailing list`_, か
  `post a question`_ で情報を探してください.
* `#scrapy IRC channel`_ で質問してください.
* Scrapy のバグは 私達の `issue tracker`_ に報告してください.

.. _archives of the scrapy-users mailing list: https://groups.google.com/forum/#!forum/scrapy-users
.. _post a question: https://groups.google.com/forum/#!forum/scrapy-users
.. _StackOverflow using the scrapy tag: https://stackoverflow.com/tags/scrapy
.. _#scrapy IRC channel: irc://irc.freenode.net/scrapy
.. _issue tracker: https://github.com/scrapy/scrapy/issues


最初のステップ
===========

.. toctree::
   :caption: 最初のステップ 
   :hidden:

   intro/overview
   intro/install
   intro/tutorial
   intro/examples

:doc:`intro/overview`
    Scrapy がどのようにしてあなたを手助けするかを理解する.

:doc:`intro/install`
    コンピューターに Scraoy をインストールする方法.

:doc:`intro/tutorial`
    最初の Scrapy プロジェクトを作成する.

:doc:`intro/examples`
    あらかじめ作成された Scrapy プロジェクトで遊ぶことでさらに学ぶ.

.. _section-basics:

基本概念
==============

.. toctree::
   :caption: 基本概念
   :hidden:

   topics/commands
   topics/spiders
   topics/selectors
   topics/items
   topics/loaders
   topics/shell
   topics/item-pipeline
   topics/feed-exports
   topics/request-response
   topics/link-extractors
   topics/settings
   topics/exceptions


:doc:`topics/commands`
    Scrapy プロジェクトの管理に使用するコマンドラインツールについて学ぶ.

:doc:`topics/spiders`
    ウェブサイトをクロールするためのルールを書く.

:doc:`topics/selectors`
    XPathを使用してWebページからデータを抽出する.

:doc:`topics/shell`
    インタラクティブな環境で抽出コードをテストする.

:doc:`topics/items`
    スクレイプしたいデータを定義する.

:doc:`topics/loaders`
    抽出したデータをアイテムに埋め込む.

:doc:`topics/item-pipeline`
    後処理してスクラップしたデータを保存する.

:doc:`topics/feed-exports`
    さまざまなフォーマットとストレージを使用してスクラップしたデータを出力する.

:doc:`topics/request-response`
    HTTP要求と応答を表すために使用されるクラスを理解する.

:doc:`topics/link-extractors`
    ページから続くリンクを抽出するための便利なクラス.

:doc:`topics/settings`
    Scrapyを設定方法を学び, :ref:`利用可能な設定 <topics-settings-ref>` をすべて見る.

:doc:`topics/exceptions`
    使用可能な例外とその意味をすべて表示する.


Built-in サービス
=================

.. toctree::
   :caption: Built-in サービス
   :hidden:

   topics/logging
   topics/stats
   topics/email
   topics/telnetconsole
   topics/webservice

:doc:`topics/logging`
    Pythonの組み込みログをScrapyで使用する方法を学ぶ.

:doc:`topics/stats`
    スクレイピングクローラに関する統計情報を収集する.

:doc:`topics/email`
    特定のイベントが発生したときに電子メール通知を送信する.

:doc:`topics/telnetconsole`
    組み込みのPythonコンソールを使用して実行中のクローラを検査する.

:doc:`topics/webservice`
    Webサービスを使用してクローラを監視および制御する.


特定の問題の解決
=========================

.. toctree::
   :caption: 特定の問題の解決
   :hidden:

   faq
   topics/debug
   topics/contracts
   topics/practices
   topics/broad-crawls
   topics/firefox
   topics/firebug
   topics/leaks
   topics/media-pipeline
   topics/deploy
   topics/autothrottle
   topics/benchmarking
   topics/jobs

:doc:`faq`
    最もよく寄せられる質問への回答を得る.

:doc:`topics/debug`
    スパイダーの一般的な問題をデバッグする方法を学ぶ.

:doc:`topics/contracts`
    スパイダーをテストのために使用する方法を学ぶ.

:doc:`topics/practices`
    いくつかの Scrapy の共通プラクティスを理解する.

:doc:`topics/broad-crawls`
    多くのドメインを並行してクロールするための調整.

:doc:`topics/firefox`
    Firefoxといくつかの便利なアドオンを使用してスクラップする方法を学ぶ.

:doc:`topics/firebug`
    Firebugを使って効率的にスクレイプする方法を学ぶ.

:doc:`topics/leaks`
    クローラでメモリリークを見つけて取り除く方法を学ぶ.

:doc:`topics/media-pipeline`
    スクラップしたアイテムに関連するファイルや画像をダウンロードする.

:doc:`topics/deploy`
    Scrapyスパイダーをデプロイしてリモートサーバーで実行する.

:doc:`topics/autothrottle`
    負荷に基づいて動的にクロール速度を調整する.

:doc:`topics/benchmarking`
    あなたのハードウェアでScrapyがどのように機能するかを調べる.

:doc:`topics/jobs`
    大きなスパイダーのクロールを一時停止して再開する方法を学ぶ.

.. _extending-scrapy:

Scrapy を拡張する
================

.. toctree::
   :caption: Scrapy を拡張する
   :hidden:

   topics/architecture
   topics/downloader-middleware
   topics/spider-middleware
   topics/extensions
   topics/api
   topics/signals
   topics/exporters


:doc:`topics/architecture`
    Scrapyアーキテクチャを理解する.

:doc:`topics/downloader-middleware`
    ページのリクエスト数とダウンロードのカスタマイズする.

:doc:`topics/spider-middleware`
    スパイダーの入力と出力をカスタマイズする.

:doc:`topics/extensions`
    カスタム機能でScrapyを拡張する.
    
:doc:`topics/api`
    拡張機能やミドルウェアでそれを使ってScrapy機能を拡張する.
    
:doc:`topics/signals`
    利用可能なすべてのシグナルとそれらを操作する方法を見る.

:doc:`topics/exporters`
    スクラップしたアイテムをファイルにすばやくエクスポートする (XML, CSV, etc).


残りのすべて
============

.. toctree::
   :caption: 残りのすべて
   :hidden:

   news
   contributing
   versioning

:doc:`news`
    最近のScrapyのバージョンで何が変わったのか見る.

:doc:`contributing`
    Scrapyプロジェクトに貢献する方法を学ぶ.

:doc:`versioning`
    ScrapyのバージョニングとAPIの安定性を理解する.
