.. _topics-exceptions:

==========
エクセプション
==========

.. module:: scrapy.exceptions
   :synopsis: Scrapy exceptions

.. _topics-exceptions-ref:

ビルトインエクセプションリファレンス
=============================

Scrapyに含まれるすべての例外のリストとその使用法.

DropItem
--------

.. exception:: DropItem

アイテムの処理を停止するためにアイテムパイプラインステージで発生させる必要がある例外. 
詳細については、項目 :ref:`topics-item-pipeline` を参照してください.

CloseSpider
-----------

.. exception:: CloseSpider(reason='cancelled')

    この例外は、スパイダーのコールバックからクローズ/ストップを要求することができます. サポートされている引数:

    :param reason: クローズした理由
    :type reason: str

例::

    def parse_page(self, response):
        if 'Bandwidth exceeded' in response.body:
            raise CloseSpider('bandwidth_exceeded')

IgnoreRequest
-------------

.. exception:: IgnoreRequest

この例外は、スケジューラまたは任意のダウンローダミドルウェアによって, リクエストを無視すべきであることを示すために発生させることができます.

NotConfigured
-------------

.. exception:: NotConfigured

この例外は, 一部のコンポーネントによって, それらが無効のままであることを示すために発生させることができます. コンポーネントは以下を含んでいます:

 * 拡張機能
 * アイテムパイプライン
 * ダウンローダミドルウェア
 * スパイダーミドルウェア
 
コンポーネントの ``__init__`` メソッドで例外を発生させる必要があります.

NotSupported
------------

.. exception:: NotSupported

この例外は、サポートされていない機能を示すために発生します.

