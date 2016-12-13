.. _topics-exporters:

===============
アイテムエクスポーター
===============

.. module:: scrapy.exporters
   :synopsis: Item Exporters

アイテムを集めたら, そのアイテムを永続化またはエクスポートして, 
他のアプリケーションでデータを使用することがしばしばあります. 
つまり, 結局のところ, スクレイピング工程の全目的です.

この目的のため, Scrapy は, XML, CSV, JSON などのさまざまな出力形式に対しアイテムエクスポーターのコレクションを提供します.

アイテムエクスポーターを使う
====================

あなたが急いでいて, アイテムエクスポータを使用してスクレイピングしたデータを出力したい場合は, 
:ref:`topics-feed-exports` を参照してください. 
そうではなく, アイテムエクスポータがどのように機能するかを知りたい, もしくは
（デフォルトのエクスポートでカバーされていない）カスタム機能をもっと必要とする場合は, 以下をお読みください.

アイテムエクスポーターを使用するには, 必要な引数でインスタンス化する必要があります. 
各エクスポーターは異なる引数を必要とするため, 
in :ref:`topics-exporters-reference` を参照してください. 
エクスポーターのインスタンスを作成した後は, 以下の作業を行う必要があります:

1. エクスポートプロセスの開始を知らせるために :meth:`~BaseItemExporter.start_exporting` メソッドを呼び出します.

2. エクスポートする各アイテムの :meth:`~BaseItemExporter.export_item` メソッドを呼び出します.

3. 最後に :meth:`~BaseItemExporter.finish_exporting` メソッドを呼び出してエクスポートプロセスの終了を知らせます.

ここで :doc:`アイテムパイプライン <item-pipeline>` を使用して, 
スクレイピングしたアイテムを異なるファイルにエクスポートするアイテムパイプライン（スパイダーごとに1つ）の例を紹介します::

   from scrapy import signals
   from scrapy.exporters import XmlItemExporter

   class XmlExportPipeline(object):

       def __init__(self):
           self.files = {}

        @classmethod
        def from_crawler(cls, crawler):
            pipeline = cls()
            crawler.signals.connect(pipeline.spider_opened, signals.spider_opened)
            crawler.signals.connect(pipeline.spider_closed, signals.spider_closed)
            return pipeline

       def spider_opened(self, spider):
           file = open('%s_products.xml' % spider.name, 'w+b')
           self.files[spider] = file
           self.exporter = XmlItemExporter(file)
           self.exporter.start_exporting()

       def spider_closed(self, spider):
           self.exporter.finish_exporting()
           file = self.files.pop(spider)
           file.close()

       def process_item(self, item, spider):
           self.exporter.export_item(item)
           return item


.. _topics-exporters-field-serialization:

アイテムフィールドのシリアル化
============================

デフォルトでは, フィールド値は未変更のシリアル化ライブラリに渡され, 
シリアル化する方法の決定権は各シリアル化ライブラリに委譲されます.

ただし,  *各フィールドの値がシリアライゼーションライブラリに渡される前に*, 
シリアライズされる方法をカスタマイズできます.

フィールドのシリアル化方法をカスタマイズするには, 次の2つの方法があります.

.. _topics-exporters-serializers:

1. フィールド内のシリアライザの宣言
--------------------------------------

:class:`~.Item` を使用する場合,  
:ref:`フィールドメタデータ <topics-items-fields>` でシリアライザを宣言できます. 
シリアライザは, 呼び出し可能な, 値を受け取ってシリアル化された値を返す形式でなければなりません.

例::

    import scrapy

    def serialize_price(value):
        return '$ %s' % str(value)

    class Product(scrapy.Item):
        name = scrapy.Field()
        price = scrapy.Field(serializer=serialize_price)


2. serialize_field() メソッドのオーバーライド
------------------------------------------

:meth:`~BaseItemExporter.serialize_field()` メソッドをオーバーライドして、フィールド値のエクスポート方法をカスタマイズすることもできます.

カスタムコードの後に, 基本クラスの :meth:`~BaseItemExporter.serialize_field()` メソッドを呼び出すようにしてください.

例::

      from scrapy.exporter import XmlItemExporter

      class ProductXmlExporter(XmlItemExporter):

          def serialize_field(self, field, name, value):
              if field == 'price':
                  return '$ %s' % str(value)
              return super(Product, self).serialize_field(field, name, value)

.. _topics-exporters-reference:

ビルトインアイテムエクスポーターリファレンス
=================================

以下に, Scrapy にバンドルされているアイテムエクスポータのリストがあります. 
これらの中には, 2つのアイテムをエクスポートしていると仮定した出力例が含まれています::

    Item(name='Color TV', price='1200')
    Item(name='DVD player', price='200')

BaseItemExporter
----------------

.. class:: BaseItemExporter(fields_to_export=None, export_empty_fields=False, encoding='utf-8')

   これは, すべてのアイテムエクスポータの（抽象）基本クラスです. 
   エクスポートするフィールドの定義, 空のフィールドのエクスポートの指定, 
   使用するエンコーディングなど, すべての（具体的な）アイテムエクスポータで使用される共通の機能をサポートしています.

   これらの機能は,  :attr:`fields_to_export`,
   :attr:`export_empty_fields`, :attr:`encoding` のそれぞれのインスタンス属性を設定するコンストラクタ引数で設定できます.

   .. method:: export_item(item)

      指定された項目をエクスポートします. このメソッドはサブクラスで実装する必要があります.

   .. method:: serialize_field(field, name, value)

      指定されたフィールドの直列化された値を返します. 
      特定のフィールドまたは値のシリアライズ/エクスポートの方法を制御する場合は, 
      カスタムアイテムエクスポータでこのメソッドをオーバーライドできます.

      デフォルトでは、このメソッドは :ref:`フィールドで宣言された <topics-exporters-serializers>` シリアライザを検索し, 
      そのシリアライザをその値に適用した結果を返します. 
      シリアライザが見つからない場合,  :attr:`encoding` 属性で宣言されたエンコーディングを使用して 
      エンコードされた ``str`` の ``unicode`` 値を除いて, 値は変更されません.

      :param field: フィールドはシリアル化されています。
          生の dict がエクスポートされている場合（:class:`~.Item` ではなく）*フィールド* 値は空の dict です.
      :type field: :class:`~scrapy.item.Field` オブジェクトまたは空の dict

      :param name: シリアル化されているフィールドの名前
      :type name: str

      :param value: シリアル化された値
  
  .. method:: start_exporting()

      エクスポートプロセスの開始を知らせます. 
      一部のエクスポーターは, これを使用して必要なヘッダー（たとえば, :class:`XmlItemExporter`）を生成することがあります. 
      アイテムをエクスポートする前に, このメソッドを呼び出す必要があります.

   .. method:: finish_exporting()

      エクスポートプロセスの終了を知らせます. 
      一部のエクスポーターは, これを使用して必要なフッター (たとえば, :class:`XmlItemExporter` のような)を生成することがあります. 
      エクスポートする項目がなくなったら, 必ずこのメソッドを呼び出す必要があります.

   .. attribute:: fields_to_export

      エクスポートされるフィールドの名前を持つリスト, またはすべてのフィールドをエクスポートする場合は None です. 
      デフォルトは None です.
      
      一部のエクスポーター ( :class:`CsvItemExporter` など) は, この属性で定義されたフィールドの順序を尊重します.

      いくつかのエクスポーターは, スパイダーが ( :class:`~Item` インスタンスでない) dictsを返すとき, 
      データを適切にエクスポートするために fields_to_export リストを要求することがあります.

   .. attribute:: export_empty_fields

      エクスポートされたデータに空のフィールドフィールドまたは空になっていないアイテムフィールドを含めるかどうか.
      デフォルトは ``False`` です. 一部のエクスポーター ( :class:`CsvItemExporter` など)
      はこの属性を無視し、空のフィールドを常にエクスポートします. 
      このオプションは dict 項目では無視されます.

   .. attribute:: encoding

      Unicode値をエンコードするために使用されるエンコード. 
      これはUnicode値にのみ影響します（このエンコーディングを使用してstrに常にシリアル化されます）. 
      他の値の型は, 変更されずに特定の直列化ライブラリに渡されます.

.. highlight:: none

XmlItemExporter
---------------

.. class:: XmlItemExporter(file, item_element='item', root_element='items', \**kwargs)

   Exports Items in XML format to the specified file object.

   :param file: the file-like object to use for exporting the data.

   :param root_element: The name of root element in the exported XML.
   :type root_element: str

   :param item_element: The name of each item element in the exported XML.
   :type item_element: str

   The additional keyword arguments of this constructor are passed to the
   :class:`BaseItemExporter` constructor.

   A typical output of this exporter would be::

       <?xml version="1.0" encoding="utf-8"?>
       <items>
         <item>
           <name>Color TV</name>
           <price>1200</price>
        </item>
         <item>
           <name>DVD player</name>
           <price>200</price>
        </item>
       </items>

   Unless overridden in the :meth:`serialize_field` method, multi-valued fields are
   exported by serializing each value inside a ``<value>`` element. This is for
   convenience, as multi-valued fields are very common.

   For example, the item::

        Item(name=['John', 'Doe'], age='23')

   Would be serialized as::

       <?xml version="1.0" encoding="utf-8"?>
       <items>
         <item>
           <name>
             <value>John</value>
             <value>Doe</value>
           </name>
           <age>23</age>
         </item>
       </items>

CsvItemExporter
---------------

.. class:: CsvItemExporter(file, include_headers_line=True, join_multivalued=',', \**kwargs)

   Exports Items in CSV format to the given file-like object. If the
   :attr:`fields_to_export` attribute is set, it will be used to define the
   CSV columns and their order. The :attr:`export_empty_fields` attribute has
   no effect on this exporter.

   :param file: the file-like object to use for exporting the data.

   :param include_headers_line: If enabled, makes the exporter output a header
      line with the field names taken from
      :attr:`BaseItemExporter.fields_to_export` or the first exported item fields.
   :type include_headers_line: boolean

   :param join_multivalued: The char (or chars) that will be used for joining
      multi-valued fields, if found.
   :type include_headers_line: str

   The additional keyword arguments of this constructor are passed to the
   :class:`BaseItemExporter` constructor, and the leftover arguments to the
   `csv.writer`_ constructor, so you can use any `csv.writer` constructor
   argument to customize this exporter.

   A typical output of this exporter would be::

      product,price
      Color TV,1200
      DVD player,200

.. _csv.writer: https://docs.python.org/2/library/csv.html#csv.writer

PickleItemExporter
------------------

.. class:: PickleItemExporter(file, protocol=0, \**kwargs)

   Exports Items in pickle format to the given file-like object.

   :param file: the file-like object to use for exporting the data.

   :param protocol: The pickle protocol to use.
   :type protocol: int

   For more information, refer to the `pickle module documentation`_.

   The additional keyword arguments of this constructor are passed to the
   :class:`BaseItemExporter` constructor.

   Pickle isn't a human readable format, so no output examples are provided.

.. _pickle module documentation: https://docs.python.org/2/library/pickle.html

PprintItemExporter
------------------

.. class:: PprintItemExporter(file, \**kwargs)

   Exports Items in pretty print format to the specified file object.

   :param file: the file-like object to use for exporting the data.

   The additional keyword arguments of this constructor are passed to the
   :class:`BaseItemExporter` constructor.

   A typical output of this exporter would be::

        {'name': 'Color TV', 'price': '1200'}
        {'name': 'DVD player', 'price': '200'}

   Longer lines (when present) are pretty-formatted.

JsonItemExporter
----------------

.. class:: JsonItemExporter(file, \**kwargs)

   Exports Items in JSON format to the specified file-like object, writing all
   objects as a list of objects. The additional constructor arguments are
   passed to the :class:`BaseItemExporter` constructor, and the leftover
   arguments to the `JSONEncoder`_ constructor, so you can use any
   `JSONEncoder`_ constructor argument to customize this exporter.

   :param file: the file-like object to use for exporting the data.

   A typical output of this exporter would be::

        [{"name": "Color TV", "price": "1200"},
        {"name": "DVD player", "price": "200"}]

   .. _json-with-large-data:

   .. warning:: JSON is very simple and flexible serialization format, but it
      doesn't scale well for large amounts of data since incremental (aka.
      stream-mode) parsing is not well supported (if at all) among JSON parsers
      (on any language), and most of them just parse the entire object in
      memory. If you want the power and simplicity of JSON with a more
      stream-friendly format, consider using :class:`JsonLinesItemExporter`
      instead, or splitting the output in multiple chunks.

.. _JSONEncoder: https://docs.python.org/2/library/json.html#json.JSONEncoder

JsonLinesItemExporter
---------------------

.. class:: JsonLinesItemExporter(file, \**kwargs)

   Exports Items in JSON format to the specified file-like object, writing one
   JSON-encoded item per line. The additional constructor arguments are passed
   to the :class:`BaseItemExporter` constructor, and the leftover arguments to
   the `JSONEncoder`_ constructor, so you can use any `JSONEncoder`_
   constructor argument to customize this exporter.

   :param file: the file-like object to use for exporting the data.

   A typical output of this exporter would be::

        {"name": "Color TV", "price": "1200"}
        {"name": "DVD player", "price": "200"}

   Unlike the one produced by :class:`JsonItemExporter`, the format produced by
   this exporter is well suited for serializing large amounts of data.

.. _JSONEncoder: https://docs.python.org/2/library/json.html#json.JSONEncoder
