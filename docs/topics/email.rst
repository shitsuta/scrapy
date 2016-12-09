.. _topics-email:

==============
Eメールを送る
==============

.. module:: scrapy.mail
   :synopsis: Email sending facility

Although Python makes sending e-mails relatively easy via the `smtplib`_
library, Scrapy provides its own facility for sending e-mails which is very
easy to use and it's implemented using `Twisted non-blocking IO`_, to avoid
interfering with the non-blocking IO of the crawler. It also provides a
simple API for sending attachments and it's very easy to configure, with a few
:ref:`settings <topics-email-settings>`.

.. _smtplib: https://docs.python.org/2/library/smtplib.html
.. _Twisted non-blocking IO: https://twistedmatrix.com/documents/current/core/howto/defer-intro.html

簡単な例
=============

MailSenderをインスタンス化するには2つの方法があります. 標準コンストラクタを使用してインスタンス化できます::

    from scrapy.mail import MailSender
    mailer = MailSender()

または, :ref:`settings <topics-email-settings>` を尊重する, Scrapy設定オブジェクトを渡してインスタンス化することもできます::

    mailer = MailSender.from_settings(settings)

そして、MailSenderを使って電子メールを送信する方法（添付ファイルなし）::

    mailer.send(to=["someone@example.com"], subject="Some subject", body="Some body", cc=["another@example.com"])

MailSender クラスリファレンス
==========================

MailSenderは, フレームワークの他の部分と同様に, `Twisted non-blocking IO`_, 
を使用するため, Scrapyから電子メールを送信するために使用するのに望ましいクラスです.

.. class:: MailSender(smtphost=None, mailfrom=None, smtpuser=None, smtppass=None, smtpport=None)

    :param smtphost: the SMTP host to use for sending the emails. If omitted, the
      :setting:`MAIL_HOST` setting will be used.
    :type smtphost: str

    :param mailfrom: the address used to send emails (in the ``From:`` header).
      If omitted, the :setting:`MAIL_FROM` setting will be used.
    :type mailfrom: str

    :param smtpuser: the SMTP user. If omitted, the :setting:`MAIL_USER`
      setting will be used. If not given, no SMTP authentication will be
      performed.
    :type smtphost: str

    :param smtppass: the SMTP pass for authentication.
    :type smtppass: str

    :param smtpport: the SMTP port to connect to
    :type smtpport: int

    :param smtptls: enforce using SMTP STARTTLS
    :type smtptls: boolean

    :param smtpssl: enforce using a secure SSL connection
    :type smtpssl: boolean

    .. classmethod:: from_settings(settings)

        Instantiate using a Scrapy settings object, which will respect
        :ref:`these Scrapy settings <topics-email-settings>`.

        :param settings: the e-mail recipients
        :type settings: :class:`scrapy.settings.Settings` object

    .. method:: send(to, subject, body, cc=None, attachs=(), mimetype='text/plain', charset=None)

        Send email to the given recipients.

        :param to: the e-mail recipients
        :type to: str or list of str

        :param subject: the subject of the e-mail
        :type subject: str

        :param cc: the e-mails to CC
        :type cc: str or list of str

        :param body: the e-mail body
        :type body: str

        :param attachs: an iterable of tuples ``(attach_name, mimetype,
          file_object)`` where  ``attach_name`` is a string with the name that will
          appear on the e-mail's attachment, ``mimetype`` is the mimetype of the
          attachment and ``file_object`` is a readable file object with the
          contents of the attachment
        :type attachs: iterable

        :param mimetype: the MIME type of the e-mail
        :type mimetype: str

        :param charset: the character encoding to use for the e-mail contents
        :type charset: str


.. _topics-email-settings:

メール設定
=============

これらの設定は, :class:`MailSender` クラスのデフォルトのコンストラクタ値を定義し, 
コードを記述することなくプロジェクト内の電子メール通知を構成するために使用できます（これらの拡張子と :class:`MailSender` を使用するコード用）.

.. setting:: MAIL_FROM

MAIL_FROM
---------

初期値: ``'scrapy@localhost'``

Eメールの送信に使用する送信者Eメール (``From:`` ヘッダー).

.. setting:: MAIL_HOST

MAIL_HOST
---------

初期値: ``'localhost'``

Eメールの送信に使用するSMTPホスト.

.. setting:: MAIL_PORT

MAIL_PORT
---------

初期値: ``25``

Eメールの送信に使用するSMTPポート.

.. setting:: MAIL_USER

MAIL_USER
---------

初期値: ``None``

SMTP認証に使用するユーザー. 無効にすると、SMTP認証は実行されません.

.. setting:: MAIL_PASS

MAIL_PASS
---------

初期値: ``None``

:setting:`MAIL_USER` とともにSMTP認証に使用するパスワード.

.. setting:: MAIL_TLS

MAIL_TLS
--------

初期値: ``False``

STARTTLSを使用して強制する. STARTTLSは、既存の安全でない接続を取得し, SSL / TLSを使用して安全な接続にアップグレードする方法です.

.. setting:: MAIL_SSL

MAIL_SSL
--------

初期値: ``False``

SSL暗号化接続を使用して接続を強制する.
