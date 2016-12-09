.. _versioning:

============================
VersioningとAPIの安定性
============================

Versioning
==========

Scrapyバージョンには3つの数字があります： *A* . *B* . *C*

* *A* はメジャーバージョンです。これはめったに変わらず、非常に大きな変更を意味します。
* *B* はリリース番号です。これには、後方互換性を損なう可能性のある機能やものを含む多くの変更が含まれますが、
      これらのケースを最小限に抑えるよう努めています。
* *C* はバグ修正リリース番号です。

Backward-incompatibilities are explicitly mentioned in the :ref:`release notes <news>`,
and may require special attention before upgrading.

Development releases do not follow 3-numbers version and are generally
released as ``dev`` suffixed versions, e.g. ``1.3dev``.

.. note::
    With Scrapy 0.* series, Scrapy used `odd-numbered versions for development releases`_.
    This is not the case anymore from Scrapy 1.0 onwards.

    Starting with Scrapy 1.0, all releases should be considered production-ready.

For example:

* *1.1.1* is the first bugfix release of the *1.1* series (safe to use in
  production)


API Stability
=============

API stability was one of the major goals for the *1.0* release.

Methods or functions that start with a single dash (``_``) are private and
should never be relied as stable.

Also, keep in mind that stable doesn't mean complete: stable APIs could grow
new methods or functionality but the existing methods should keep working the
same way.


.. _odd-numbered versions for development releases: https://en.wikipedia.org/wiki/Software_versioning#Odd-numbered_versions_for_development_releases

