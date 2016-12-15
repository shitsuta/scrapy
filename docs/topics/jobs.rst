.. _topics-jobs:

=================================
ジョブ: クロールの一時停止と再開
=================================

大規模なサイトでは, クロールを一時停止してから後で再開できるようにすることが望ましい場合があります.

Scrapy は, 以下の機能により, この機能を使用できます:

* スケジュールされた要求をディスクに保存するスケジューラ

* ディスクに訪問要求を継続する重複フィルタ

* バッチ間で永続的なスパイダー状態（キーと値のペア）を維持する拡張機能

ジョブディレクトリ
=============

永続性サポートを有効にするには,  ``JOBDIR`` 設定を使用してジョブディレクトリを定義するだけです. 
このディレクトリは, 単一のジョブ（つまり, スパイダ実行）の状態を保持するために必要なすべてのデータを格納するためのディレクトリです. 
このディレクトリは, 単一のジョブの状態を格納するために使用されるため, 
異なるスパイダ, または同じスパイダの異なるジョブ・実行間で共有されてはならないことに注意することが重要です.

使用方法
=============

持続性がサポートされているスパイダーを起動するには, 次のように実行します::

    scrapy crawl somespider -s JOBDIR=crawls/somespider-1

その後, いつでも（Ctrl-Cを押すかシグナルを送信することによって）スパイダーを安全に停止し, 
後で同じコマンドを発行して再開することができます::

    scrapy crawl somespider -s JOBDIR=crawls/somespider-1

バッチ間で永続的な状態を維持する
========================================

一時停止/再開バッチの間にスパイダーの状態を維持したい場合があります. 
これには,  ``spider.state`` 属性を使用できます. これは ``dict`` でなければなりません. 
スパイダーの起動と停止時に, ジョブディレクトリからその属性をシリアル化, 
格納, ロードするための拡張機能が組み込まれています.

以下は, スパイダーの状態を使用するコールバックの例です（簡潔にするために他のスパイダーコードは省略されています）::

    def parse_item(self, response):
        # parse item here
        self.state['items_count'] = self.state.get('items_count', 0) + 1

持続性の落とし穴
===================

Scrapyの永続性サポートを使用できるようにするには, 次の点に注意してください.

Cookieの有効期限
------------------

クッキーの有効期限が切れる可能性があります. したがって, スパイダーをすぐに再開しないと, 
スケジュールされたリクエストはすぐに機能しなくなる可能性があります. 
スパイダーがクッキーに依存していない場合, これは問題にはなりません.

シリアル化を要求する
---------------------

永続性を機能させるためには, リクエストが ``pickle`` モジュールによって直列化可能でなければならないため, 
リクエストが直列化可能であることを確認する必要があります.

最も一般的な問題は, 永続化できない要求コールバックに対して ``lambda`` 関数を使用することです.
語
例えば, これはうまくいきません::

    def some_callback(self, response):
        somearg = 'test'
        return scrapy.Request('http://www.example.com', callback=lambda r: self.other_callback(r, somearg))

    def other_callback(self, response, somearg):
        print "the argument passed is:", somearg

しかし, これはうまくいきます::

    def some_callback(self, response):
        somearg = 'test'
        return scrapy.Request('http://www.example.com', callback=self.other_callback, meta={'somearg': somearg})

    def other_callback(self, response):
        somearg = response.meta['somearg']
        print "the argument passed is:", somearg

シリアライズできなかったリクエストを記録する場合は, プロジェクトの設定ページで
:setting:`SCHEDULER_DEBUG` 設定を ``True`` に設定します.
デフォルトでは ``False`` です.

.. _pickle: http://docs.python.org/library/pickle.html
