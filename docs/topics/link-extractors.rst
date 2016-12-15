.. _topics-link-extractors:

===============
LinkExtractor
===============

LinkExtractorは, 最終的に追跡されるウェブページ (:class:`scrapy.http.Response` オブジェクト) からリンクを抽出することのみを目的とするオブジェクトです.

``scrapy.linkextractors import LinkExtractor`` によって Scrapy で有効化されますが, シンプルなインターフェースを実装することで, 独自のカスタム LinkExtractor を作成してニーズに合わせることができます.

すべてのリンク抽出プログラムが持つ唯一のパブリックメソッドは ``extract_links`` です.
これは,  :class:`~scrapy.http.Response` オブジェクトを受け取り
:class:`scrapy.link.Link` オブジェクトのリストを返します. 
LinkExtractor は一度インスタンス化されることを意図されており,  ``extract_links`` メソッドは異なる応答で数回呼び出され, 続くリンクを抽出します.

LinkExtractor は, 一連のルールを通じて :class:`~scrapy.spiders.CrawlSpider`
クラス (Scrapyで利用可能) で使用されますが,
:class:`~scrapy.spiders.CrawlSpider` からサブクラス化しない場合でも、スパイダーで使用することができます.


.. _topics-link-extractors-ref:

ビルトイン LinkExtractor リファレンス
==================================

.. module:: scrapy.linkextractors
   :synopsis: Link extractors classes

Scrapy にバンドルされたリンク抽出クラスは, 
:mod:`scrapy.linkextractors` モジュールとして提供されています.

デフォルトのリンク抽出プログラムは ``LinkExtractor`` で, 
:class:`~.LxmlLinkExtractor` と同じです::

    from scrapy.linkextractors import LinkExtractor

古いScrapyバージョンでは他のリンク抽出クラスが使用されていましたが, 現在は推奨されていません.

LxmlLinkExtractor
-----------------

.. module:: scrapy.linkextractors.lxmlhtml
   :synopsis: lxml's HTMLParser-based link extractors


.. class:: LxmlLinkExtractor(allow=(), deny=(), allow_domains=(), deny_domains=(), deny_extensions=None, restrict_xpaths=(), restrict_css=(), tags=('a', 'area'), attrs=('href',), canonicalize=True, unique=True, process_value=None)

    LxmlLinkExtractor は, 便利なフィルタリングオプションを備えた推奨リンク抽出ツールです. lxmlの堅牢なHTMLパーサーを使用して実装されています.

    :param allow: リンクを抽出するために（絶対）URLが一致しなければならない単一の正規表現（または正規表現のリスト）. 
    指定されていない（または空）場合, すべてのリンクに一致します.
    :type allow: 正規表現（またはそのリスト）

    :param deny: 除外する（絶対）URLが一致しなければならない（つまり抽出されない）単一の正規表現（または正規表現のリスト）. 
    これは, ``allow`` パラメータよりも優先されます. 指定されていない場合（または空の場合）, リンクを除外しません. 
    :type deny: 正規表現（またはそのリスト）

    :param allow_domains:リンクを抽出するために考慮されるドメインを含む文字列の単一の値, またはリスト
    :type allow_domains: 文字列またはリスト
    
    :param deny_domains: リンクを抽出するために考慮されないドメインを含む文字列の単一の値、またはリスト
    :type deny_domains: 文字列またはリスト

    :param deny_extensions: リンクを抽出するときに無視すべき拡張子を含む単一の値または文字列のリスト.
        指定されていない場合は,  `scrapy.linkextractors`_ パッケージで定義されている
        デフォルトの ``IGNORED_EXTENSIONS`` の値になります.
    :type deny_extensions: リスト
    
    :param restrict_xpaths: は, XPath（またはXPathのリスト）であり, そこからリンクを抽出する応答内の領域を定義します. 
        指定すると, それらのXPathによって選択されたテキストのみがリンクのためにスキャンされます. 下記の例を参照してください.
    :type restrict_xpaths: 文字列またはリスト

    :param restrict_css: リンクの抽出元となる応答内の領域を定義するCSSセレクタ（またはセレクタのリスト）.
        ``restrict_xpaths`` と同じ動作をします.
    :type restrict_css: 文字列またはリスト

    :param tags: リンクを抽出するときに考慮するタグまたはタグのリスト. デフォルトは ``('a', 'area')`` タグです.
    :type tags: 文字列またはリスト

    :param attrs: 抽出するリンクを探すときに考慮する属性または属性のリスト（ ``tags`` パラメータで指定されたタグのみ）. デフォルトは ``('href',)``
    :type attrs: リスト
    
    :param canonicalize: 抽出された各URLを正規化します (w3lib.url.canonicalize_urlを使用). デフォルトは ``True``.
    :type canonicalize: boolean

    :param unique: 抽出されたリンクに重複フィルタリングを適用するかどうか.
    :type unique: boolean

    :param process_value: タグから抽出された各値とスキャンされた属性を受け取り, 値を修正して新しい値を返す, 
        または ``None`` を返してリンクを完全に無視する関数.
        指定されていない場合, ``process_value`` のデフォルトは ``lambda x: x``.

        .. highlight:: html

        たとえば, このコードからリンクを抽出するには::

            <a href="javascript:goToPage('../other/page.html'); return false">Link text</a>

        .. highlight:: python

        ``process_value`` で次の関数を使用することができます::

            def process_value(value):
                m = re.search("javascript:goToPage\('(.*?)'", value)
                if m:
                    return m.group(1)

    :type process_value: callable

.. _scrapy.linkextractors: https://github.com/scrapy/scrapy/blob/master/scrapy/linkextractors/__init__.py
