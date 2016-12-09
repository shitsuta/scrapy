:orphan: Ubuntu packages are obsolete

.. _topics-ubuntu:

===============
Ubuntu パッケージ
===============

.. versionadded:: 0.10

`Scrapinghub`_ は一般的にUbuntuのものよりも新鮮なapt-gettableパッケージを公開しています.
`GitHub repo`_ リポジトリ（マスター＆安定したブランチ）から継続的に構築されているので安定しているので, 最新のバグ修正が含まれています.

.. caution:: これらのパッケージは現在更新されておらず, Ubuntu 16.04以上で動作しない可能性があります. ( :issue:`2076` と :issue:`2137` を参照してください.)

パッケージを使用するには:

1. Scrapyパッケージに署名するために使用したGPGキーをAPTキーリングにインポートする::

    sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 627220E7

2. 以下のコマンドで `/etc/apt/sources.list.d/scrapy.list` を作成する::

    echo 'deb http://archive.scrapy.org/ubuntu scrapy main' | sudo tee /etc/apt/sources.list.d/scrapy.list

3. パッケージ一覧を更新し、パッケージをインストールする:

   .. parsed-literal::

      sudo apt-get update && sudo apt-get install scrapy

.. note:: Scrapyをアップグレードしようとしている場合は、手順3を繰り返します.

.. warning:: `python-scrapy` は公式のdebianリポジトリによって提供される別のパッケージです. これは古く、Scrapyチームではサポートされていません.

.. _Scrapinghub: http://scrapinghub.com/
.. _GitHub repo: https://github.com/scrapy/scrapy
