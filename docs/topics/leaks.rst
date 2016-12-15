.. _topics-leaks:

======================
メモリリークのデバッグ
======================

Scrapyでは, リクエスト, レスポンス, アイテムなどのオブジェクトの寿命は制限されています. 
作成され, しばらく使用された後, 最終的に破棄されます.

全オブジェクトの中でも, Requestはおそらく最も寿命の長いものです. 
スケジューラのキューで待機してから処理するまでです. 
詳細は,  :ref:`topics-architecture` を参照してください.

これらの Scrapy オブジェクトは（かなり長い）存続期間を持つため, 
適切に解放されずメモリに蓄積され, 「メモリリーク」を引き起こすリスクが常にあります.

メモリリークをデバッグするのに役立つように, Scrapyには :ref:`trackref <topics-leaks-trackrefs>` 
というオブジェクト参照をトラッキングするためのメカニズムが組み込まれています. 
さらに詳細なメモリデバッグには :ref:`Guppy <topics-leaks-guppy>` 
というサードパーティのライブラリを使用することもできます.  
どちらのツールも :ref:`Telnet コンソール <topics-telnetconsole>` から使用する必要があります.

メモリリークの一般的な原因
=============================

頻繁に, Scrapy 開発者はリクエストで参照されるオブジェクト（たとえば, :attr:`~scrapy.http.Request.meta` 
属性または要求コールバック関数を使用して）を渡し, 
これらの参照されたオブジェクトの有効期間をリクエストの存続期間として効果的に制限します. 
これは, 今のところScrapyプロジェクトでのメモリリークの最も一般的な原因であり, 
新規参入者がデバッグするのは非常に難しいものです.

大きなプロジェクトでは, 一般的にスパイダーは様々な人によって書かれており, 
そのスパイダーのいくつかは「リーク」の原因になりえ, 他の（よく書かれた）スパイダーの部分に同時に影響を及ぼし, 
全体的なクロールプロセスに影響を及ぼす可能性があります.

また, （以前に割り当てられた）リソースを適切に解放していない場合は, 
作成したカスタムミドルウェア, パイプライン, または拡張機能からリークが発生する可能性があります. 
たとえば,  :signal:`spider_opened` にリソースを割り当てても
:signal:`spider_closed` でリソースを解放しないと, 
:ref:`プロセスごとに複数のスパイダーを実行 <run-multiple-spiders>` していると問題が発生する可能性があります. 

リクエストが多すぎ?
------------------

デフォルトでは, Scrapyはリクエストのキューをメモリ内に保持します. 
これは,  :class:`~scrapy.http.Request` オブジェクトおよび
（例えば, :attr:`~scrapy.http.Request.meta` 内の）
リクエスト属性内で参照されるすべてのオブジェクトを含みます. 
必ずしもリークとはいえませんが, これには多くのメモリを必要とします. 
:ref:`永続的なジョブ・キュー <topics-jobs>` を有効にすると, メモリー使用量を制御できます.

.. _topics-leaks-trackrefs:

``trackref`` でメモリリークのデバッグをする
========================================

:mod:`trackref` は, メモリリークの最も一般的なケースをデバッグするためにScrapyによって提供されるモジュールです. 
これは, 基本的にすべての実際のリクエスト, レスポンス, アイテム, およびセレクタオブジェクトへの参照を追跡します.

Telnet コンソールに入り,  :func:`~scrapy.utils.trackref.print_live_refs` 
関数のエイリアスである ``prefs()`` 関数を使って, 
現在生きているオブジェクトの数（上記のクラスのうちどれか）を調べることができます::

    telnet localhost 6023

    >>> prefs()
    Live References

    ExampleSpider                       1   oldest: 15s ago
    HtmlResponse                       10   oldest: 1s ago
    Selector                            2   oldest: 0s ago
    FormRequest                       878   oldest: 7s ago

ご覧のとおり, このレポートには, 各クラスの中で最も古いオブジェクトの「年齢」も表示されます. 
プロセスごとに複数のスパイダーを実行している場合は, 最も古いリクエスト, またはレスポンスを調べることで, 
どのスパイダーがリークを起こしているか把握できます.
Telnetコンソールから :func:`~scrapy.utils.trackref.get_oldest` 関数を使用して, 
各クラスの最も古いオブジェクトを取得できます.

どのオブジェクトが追跡されるの?
--------------------------

``trackrefs``によって追跡されるオブジェクトは, すべてこれらのクラス（およびそのすべてのサブクラス）のものです:

* :class:`scrapy.http.Request`
* :class:`scrapy.http.Response`
* :class:`scrapy.item.Item`
* :class:`scrapy.selector.Selector`
* :class:`scrapy.spiders.Spider`

実際の例
--------------

仮想のメモリリークの具体例を見てみましょう. このような行を持つスパイダーがいくつかあるとします::

    return Request("http://www.somenastyspider.com/product.php?pid=%d" % product_id,
        callback=self.parse, meta={referer: response})

この行は, リクエスト中にレスポンスの参照を渡しています. 
このレスポンスの参照は, レスポンスの存続期間とリクエストの存続期間とを結びつけ, 
メモリリークを引き起こします.

``trackref`` ツールを使用して, 原因を発見する方法を見てみましょう.

クローラが数分間実行され, メモリ使用量が大きく増えたことがわかったら, 
Telnet コンソールに入り, 生きている参照を確認します::

    >>> prefs()
    Live References

    SomenastySpider                     1   oldest: 15s ago
    HtmlResponse                     3890   oldest: 265s ago
    Selector                            2   oldest: 0s ago
    Request                          3878   oldest: 250s ago

レスポンスはリクエストと比較して, 短い寿命でなければならないので, 
非常に多くの生きているレスポンスがあること（そしてそれらは古くなっています）は間違いなく疑わしいです. 
レスポンスの数とリクエストの数がほぼ同じなので, 何らかの原因で結び付けられているように見えます. 
この結果から, スパイダーのコードをチェックして, リークを生成している厄介な行を発見することができます（リクエスト内でレスポンス参照を渡す）.

生きているオブジェクトに関する追加情報が役立つ場合があります. 最も古いリクエストを確認しましょう::

    >>> from scrapy.utils.trackref import get_oldest
    >>> r = get_oldest('HtmlResponse')
    >>> r.url
    'http://www.somenastyspider.com/product.php?pid=123'

最も古いものだけを取得するのではなく, すべてのオブジェクトを繰り返し処理したい場合は 
:func:`scrapy.utils.trackref.iter_all` 関数を使用します::

    >>> from scrapy.utils.trackref import iter_all
    >>> [r.url for r in iter_all('HtmlResponse')]
    ['http://www.somenastyspider.com/product.php?pid=123',
     'http://www.somenastyspider.com/product.php?pid=584',
    ...

スパイダーが多すぎる?
-----------------

プロジェクトの並列実行数が多すぎると,
:func:`prefs()` の出力を読みにくくなる可能性があります.
このため, この関数には, 特定のクラス（およびすべてのサブクラス）を無視できる ``ignore`` 引数があります. 
たとえば, これはスパイダーの参照を表示しません::

    >>> from scrapy.spiders import Spider
    >>> prefs(ignore=Spider)

.. module:: scrapy.utils.trackref
   :synopsis: Track references of live objects

scrapy.utils.trackref モジュール
------------------------------

:mod:`~scrapy.utils.trackref` モジュールで利用できる関数は次のとおりです.

.. class:: object_ref

    ``trackref`` モジュールを使用してライブインスタンスをトラッキングする場合は, （オブジェクトではなく）このクラスから継承します.

.. function:: print_live_refs(class_name, ignore=NoneType)

    生きている参照のレポートをクラス名でグループ化して出力します.

    :param ignore: 与えられた場合, 指定されたクラス（またはクラスのタプル）からのすべてのオブジェクトは無視されます.
    :type ignore: class または classe のタプル
    
.. function:: get_oldest(class_name)

    指定されたクラス名で生存しているすべてのオブジェクトのイテレータを返します. 見つからない場合は ``None`` を返します. 
    まず,  :func:`print_live_refs` を使用して, クラス名ごとに追跡されたすべてのライブオブジェクトのリストを取得します.

.. function:: iter_all(class_name)

    指定されたクラス名で生存しているすべてのオブジェクトのイテレータを返します. 見つからない場合は
    ``None`` を返します. まず,  :func:`print_live_refs` を使用して, 
    クラス名ごとに追跡されたすべてのライブオブジェクトのリストを取得します.

.. _topics-leaks-guppy:

Guppy でメモリリークのデバッグをする
=================================

``trackref`` はメモリリークを追跡する非常に便利なメカニズムを提供しますが, 
メモリリーク（リクエスト, レスポンス, アイテム, セレクタ）の原因となる可能性の高いオブジェクトのみを追跡します. 
しかし, メモリリークが他の（多かれ少なかれわかりにくい）オブジェクトから来る場合もあります. 
``trackref`` を使ってリークを見つけることができない場合は, もう一つのリソース,
`Guppy ライブラリ`_ があります.

.. _Guppy ライブラリ: https://pypi.python.org/pypi/guppy

``pip`` を使用している場合は, 次のコマンドでGuppyをインストールできます::

    pip install guppy

また, Telnet コンソールには, Guppy ヒープオブジェクトにアクセスするための組み込みショートカット（hpy）
が付属しています. 以下は, Guppyを使ってヒープ内で利用可能なすべてのPythonオブジェクトを表示する例です::

    >>> x = hpy.heap()
    >>> x.bytype
    Partition of a set of 297033 objects. Total size = 52587824 bytes.
     Index  Count   %     Size   % Cumulative  % Type
         0  22307   8 16423880  31  16423880  31 dict
         1 122285  41 12441544  24  28865424  55 str
         2  68346  23  5966696  11  34832120  66 tuple
         3    227   0  5836528  11  40668648  77 unicode
         4   2461   1  2222272   4  42890920  82 type
         5  16870   6  2024400   4  44915320  85 function
         6  13949   5  1673880   3  46589200  89 types.CodeType
         7  13422   5  1653104   3  48242304  92 list
         8   3735   1  1173680   2  49415984  94 _sre.SRE_Pattern
         9   1209   0   456936   1  49872920  95 scrapy.http.headers.Headers
    <1676 more rows. Type e.g. '_.more' to view.>

ほとんどのスペースは ``dict`` によって使用されていることがわかります. 
これらの ``dict`` がどの属性から参照されているかを確認したい場合は, 以下のようにします::

    >>> x.bytype[0].byvia
    Partition of a set of 22307 objects. Total size = 16423880 bytes.
     Index  Count   %     Size   % Cumulative  % Referred Via:
         0  10982  49  9416336  57   9416336  57 '.__dict__'
         1   1820   8  2681504  16  12097840  74 '.__dict__', '.func_globals'
         2   3097  14  1122904   7  13220744  80
         3    990   4   277200   2  13497944  82 "['cookies']"
         4    987   4   276360   2  13774304  84 "['cache']"
         5    985   4   275800   2  14050104  86 "['meta']"
         6    897   4   251160   2  14301264  87 '[2]'
         7      1   0   196888   1  14498152  88 "['moduleDict']", "['modules']"
         8    672   3   188160   1  14686312  89 "['cb_kwargs']"
         9     27   0   155016   1  14841328  90 '[1]'
    <333 more rows. Type e.g. '_.more' to view.>

ご覧のとおり, Guppy モジュールは非常に強力ですが, Python についての深い知識も必要です. Guppyの詳細については, 
`Guppy ドキュメント`_ を参照してください.

.. _Guppy ドキュメント: http://guppy-pe.sourceforge.net/

.. _topics-leaks-without-leaks:

漏れのない漏れ
===================

場合によっては, Scrapy プロセスのメモリ使用量が増加するだけで, 減少しないことがあります. 
この場合残念なことに, Scrapy または, あなたのプロジェクトどちらにも, メモリリークが発生する可能性があります. 
これは（あまりよく知られていない）Python の問題で, Python がリリースしたメモリをオペレーティングシステムに返さないことにより発生します. 
詳細については, 以下を参照してください:

* `Python Memory Management <http://www.evanjones.ca/python-memory.html>`_
* `Python Memory Management Part 2 <http://www.evanjones.ca/python-memory-part2.html>`_
* `Python Memory Management Part 3 <http://www.evanjones.ca/python-memory-part3.html>`_

 `この記事`_で詳しく述べられている Evan Jones が提案した改良点は, 
Python 2.5でマージされましたが, これは問題が軽減されただけで, 完全に修正されたわけではありません. 
以下は, 記事の引用です:

    *Unfortunately, this patch can only free an arena if there are no more
    objects allocated in it anymore. This means that fragmentation is a large
    issue. An application could have many megabytes of free memory, scattered
    throughout all the arenas, but it will be unable to free any of it. This is
    a problem experienced by all memory allocators. The only way to solve it is
    to move to a compacting garbage collector, which is able to move objects in
    memory. This would require significant changes to the Python interpreter.*

.. _この記事: http://www.evanjones.ca/memoryallocator/

メモリ消費を合理的に保つために, ジョブを複数の小さなジョブに分割するか, 
:ref:`永続的なジョブ・キュー <topics-jobs>` を有効にし, スパイダーを停止/開始するすることが望ましいです.
