.. Symfony Advent Calendar 2012 - Day 17 documentation master file, created by
   sphinx-quickstart on Sat Dec 15 14:06:21 2012.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

========================================
Symfony Advent Calendar JP 2012 - Day 17
========================================

:Author: Shogo Kawahara <kawahara@bucyou.net> Twitter: `@ooharabucyou`_
:Date: 2012-12-17
:License: `Creative Commons Attribution 3.0 Unported License <http://creativecommons.org/licenses/by/3.0/>`_

.. _`@ooharabucyou`: http://twitter.com/ooharabucyou

こんにちは。 `@ooharabucyou`_ こと川原です。

`Symfony Advent Calendar JP 2012 <http://www.adventar.org/calendars/24>`_ の17日目です。

`<-15日目`_ **17日目(今ここ)** `18日目->`_

.. _`<-15日目`: http://phpmentors.jp/post/37960503952/symfony-2013-hidenorigoto
.. _`18日目->`: http://example.com/


.. note::

    昨日はおやすみの日でした

私は、まだまだ 2.0 をちまちまマイナーバージョンアップしながら利用していますが、幾つかの理由により 2.1 に上げたいと
思っています。
その理由のひとつが、強化されたロギングに関わる設定です。Symfony2.0 + MonologBundle(2.0) + Monolog(1.0) の組み合わせから
今 (Symfony2.1 + MonologBundle2.1 + Monolog1.2) を比べると、どのように変化しているかを見ていきます。

追加されたHandler
=================

Monolog 1.0 時代の Hander は、メールやファイル出力、標準出力についてが用意されていましたが、
最新の安定版 1.2.x では、MongoDB、AMQP、GrayLog2 のための Hander が標準装備されています。

.. note::

    (MongoDB、AMQP の活用には、それぞれの Extension、また GrayLog2 の利用には、
    mlehner/gelf-php を導入する必要があります。)

Channel
=======

MonologBundle(2.0) は、設定ファイルを書くだけで、基本的なログ出力に対応させることができますが。
しかし、実際には特定の操作が行われた時、通常のログファイル出力だけでなく、別のログファイルに出力
したり、特定のアクションを行なって欲しいという場合があるかと思います。

実際に、Symfony2.0 で、データベースの書き出し・アップデートに関わる処理を INFO レベルで書き出し
標準のログとは、別のファイルで書き出したいという場面がありました。

この場合、新たな Logger を定義し service として DI コンテナに追加する必要がありました。

- `How to log messages to different files with Monolog in Symfony2.0 <http://www.ricardclau.com/2012/08/how-to-log-messages-to-different-files-with-monolog-in-symfony2-0/>`_

さて、Symfony2.1 に同梱されている、MonologBundle 2.1 では、 `channel`_  という機能がこの問題を
解決するのに役立ちます。

.. _`channel`:  http://symfony.com/doc/master/cookbook/logging/channels_handlers.html

まずは、当該のチャネルを処理するための Logger を新しく用意します。

.. code-block:: yaml

    services:
        monolog.logger.myservice
            class: Monolog\Logger
            arguments: [myservice]

このとき、サービスID は、 ``monolog.logger.チャネル名`` となるようにします。

Service に、Logger を注入する際に、タグとして channel を指定します。:

.. code-block:: yaml

    services:
      my_service:
          class: %my_service.class%
          arguments: [@logger]
          tags:
              - { name: monolog.logger, channel: myservice }

そして、ログの設定で、それぞれの Hander でその channel を扱うかどうかを記します。:

.. code-block:: yaml

    monolog:
        handlers:
            main:
                type: stream
                path: /var/log/log.log
                channels: !myservice
            myservice_log:
                type: stream
                path: /var/log/myservice.log
                channels: myservice

channels は配列で複数指定することもできて、指定しない場合、全ての channels を
扱います。また、 channel名の前に、 ``!`` を付加することにより、特定の channel を
除外することができます。

上の例では、 main で、 myservice 以外の channel を扱い、 myservice_log で、 myservice
の channel を扱うように設定しています。
これにより、ログファイルを分けることが簡単にできるようになりました。

ちなみに、コンテナには、 ``monolog.logger.チャネル名`` でサービスが登録してあるので、
これを直接呼ぶことで、そのチャネルのログを残すことも可能でしょう。

まとめと、しばらくちゃんとバージョンをあげよう的な話
====================================================

僕は、この機能だけでも Symfony2.0 から 2.1 に上げたいと思いました。 `JsonResponse`_ あたりも魅力的です。

.. _`JsonResponse`: https://github.com/symfony/symfony/blob/master/src/Symfony/Component/HttpFoundation/JsonResponse.php

一方で、来年の5月には Symfony 2.3 が最初の Long Team Support バージョン (サポート期間が3年間と長い)
として出されるそうです。こういう話が出ると、2.3 が出てから、それを使いたいものですが、

-  `The Release Process <http://symfony.com/doc/current/contributing/community/releases.html>`_

上の図を見て分かる通り、Symfony2.0 のメンテ期間は間もなく終了します。
Security Fix が含まれる場合もあるので、ユーザや自分たちを守るためにも、ちゃんとサポートされている
バージョンに追従していきたいわけです。

というわけで、2.1 -> 2.2 -> 2.3 といった感じに、段階的に上げざるおえないと考えています。

.. note::

    Security Fix 周りの問題については、後日 @co3k が取り扱うようです。
