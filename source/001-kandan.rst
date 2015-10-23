================================
Cloud Foundry 百日行 #001 Kandan
================================

今日から始まる「Cloud Foundry 百日行」，第1回目に取り上げるアプリは， `Kandan <http://kandanapp.com/>`_ です。


基本情報
========

* | 公式サイト
  | http://kandanapp.com/
  | (2015-06-04T07:10:+0900 時点では接続不能)

* | ソースコード
  | https://github.com/kandanapp/kandan

* | 参考
  | http://alternativeto.net/software/kandan/

Kandan は，GitHub の README によると，「オープンソースの HipChat クローン」即ち，モダンなチャット／インスタント・メッセージングのためのソフトウェアです。


デプロイ
========

ではいよいよ，本題である Cloud Foundry (以下CF)へのデプロイに入ります。

実はこのアプリには，CF向けの `デプロイ手順 <https://github.com/kandanapp/kandan/blob/4a7d91216133d2260ce2dc0660e41f0003d2ca98/DEPLOY.md#cloud-foundry>`_ が用意されています。ただしこれは Cloud Foundry v1 向けの手順なので，そのまま使うことはできません。ただ，以下の手順では，上記及び `Stand Alone Server の起動手順 <https://github.com/kandanapp/kandan/blob/4a7d91216133d2260ce2dc0660e41f0003d2ca98/DEPLOY.md#standalone-server>`_ を参考にしました。

以下手順の概要です：

* 0) ソースコードの取得
* 1) アプリのアップロード
* 2) サービスの作成
* 3) アプリとサービスの結合
* 4) アプリの起動

では順次見ていきます。

ソースコードの取得
------------------

アプリのソースコードをGitHubからcloneします。 ::

    $ git clone https://github.com/kandanapp/kandan.git
    Cloning into 'kandan'...
    remote: Counting objects: 7350, done.
    remote: Total 7350 (delta 0), reused 0 (delta 0), pack-reused 7350
    Receiving objects: 100% (7350/7350), 8.94 MiB | 3.15 MiB/s, done.
    Resolving deltas: 100% (2918/2918), done.
    Checking connectivity... done.

::

    $ cd kandan/

これで，デプロイするソースコードが用意できました。

アプリの push
-------------

まず，CF環境にアプリを登録し，アップロードします。この行為をCFではしばしば "push" するといいます。

通常，CFでは，アプリはpushされると自動的に起動されます。しかし，今回のアプリはデータベース・サービスを必要とするので，この段階ではまだ起動しないことにします。そのため，以下のコマンドには， ``--no-start`` というオプションが付いています。 ::

    $ cf push kandan --no-start
    Creating app kandan in org nota-ja / space 100 as nota-ja...
    OK
    
    Creating route kandan.10.244.0.34.xip.io...
    OK
    
    Binding kandan.10.244.0.34.xip.io to kandan...
    OK
    
    Uploading kandan...
    Uploading app files from: /home/nota-ja/workspace/100/kandan
    Uploading 6M, 1259 files
    Done uploading
    OK

アプリのアップロードまで終わればOKです。

サービスの作成
--------------

次に，このアプリで使用するデータベース・サービスを作成します。

このアプリは，データベースとしてPostgreSQLを使います。今回は，CFのコミュニティで提供されているサービス・ブローカー `postgresql-cf-service-broker <https://github.com/cloudfoundry-community/postgresql-cf-service-broker>`_ を使ってデータベース・サービスを作成します。

* 提供されているサービス種別の一覧 ::

    $ cf marketplace
    Getting services from marketplace in org nota-ja / space 100 as nota-ja...
    OK
    
    service      plans                    description
    PostgreSQL   Basic PostgreSQL Plan*   PostgreSQL on shared instance.

* サービスの作成 ::

    $ cf create-service PostgreSQL "Basic PostgreSQL Plan" pg4kandan
    Creating service instance pg4kandan in org nota-ja / space 100 as nota-ja...
    OK

  1番目の引数はサービスの種類，2番目の引数はプラン，3番目の引数はこれから作るサービスの名前です。

* 確認 ::

    $ cf services
    Getting services in org nota-ja / space 100 as nota-ja...
    OK
    
    name        service      plan                    bound apps   last operation
    pg4kandan   PostgreSQL   Basic PostgreSQL Plan

  ``PostgreSQL`` タイプの ``pg4kandan`` が表示されていることを確認します。

これでPostgreSQLサービスが利用可能になりました。

CFにおけるサービスとは，「利用の開始/終了等に関する統一的な手順が定義された何か」です。使い始めた後は，対象サービスと直接やりとりをするので，その部分についてのオーバーヘッドはありません。

PostgreSQLサービスでは，1サービスごとに1つのデータベース(テーブルの集合)と，その管理権限を持つユーザーが作成されます。

アプリとサービスの結合
----------------------

次に，アプリとサービスを結合します。これは，具体的にはアプリにサービスにアクセスするための情報を渡すための操作です。この場合は，PostgreSQLが待ち受けているホスト/ポート,アクセス先のデータベース名,アクセス権を持つユーザー名とパスワードが，アプリに伝えられます。CFではこの操作を「バインド」と呼びます。

* バインド ::

    $ cf bind-service kandan pg4kandan
    Binding service pg4kandan to app kandan in org nota-ja / space 100 as nota-ja...
    OK

1番目の引数がアプリ名，2番目の引数がサービス名です。

* 確認 ::

    $ cf services
    Getting services in org nota-ja / space 100 as nota-ja...
    OK
    
    name        service      plan                    bound apps   last operation
    pg4kandan   PostgreSQL   Basic PostgreSQL Plan   kandan

``kandan-pg`` の ``bound apps`` のカラムに， ``kandan`` が表示されていればOKです。

実際にアプリに渡される情報は，VCAP_SERVICES という環境変数の中に入っています。 ::

    $ cf env kandan
    Getting env variables for app kandan in org nota-ja / space 100 as nota-ja...
    OK
    
    System-Provided:
    {
     "VCAP_SERVICES": {
      "PostgreSQL": [
       {
        "credentials": {
         "uri": "postgres://username:password@host:port/dbname"
        },
        "label": "PostgreSQL",
        "name": "pg4kandan",
        "plan": "Basic PostgreSQL Plan",
        "tags": [
         "PostgreSQL",
         "Database storage"
        ]
       }
      ]
     }
    }
    (以下略)


アプリの起動
------------

データベースとの接続準備もできたので，先ほど停止状態でpushしたアプリを起動します。
具体的には，アプリの起動コマンドを指定して，アプリを再度pushします。

* アプリの再push ::

    $ cf push kandan -c 'bundle exec rake db:create db:migrate kandan:bootstrap && bundle exec thin --port $PORT start'
    Updating app kandan in org nota-ja / space 100 as nota-ja...
    (中略)
    requested state: started
    instances: 1/1
    usage: 256M x 1 instances
    urls: kandan.10.244.0.34.xip.io
    last uploaded: Mon Jun 1 02:34:28 +0000 2015
    stack: lucid64
    
         state     since                    cpu    memory           disk      details
    #0   running   2015-06-01 11:36:28 AM   0.0%   134.3M of 256M   0 of 1G

アプリの ``state`` が ``running`` になっていればOKです。

起動については，注意点(というより，今回記事を書くにあたって動作検証をした際に引っかかった点)が2点あります。

* | 1) 起動コマンドで，データベースの構築を行う
  | 通常は，起動コマンドはデフォルトで設定されるため，特に指定する必要はありません。しかし，今回はPostgreSQLサービスを利用していて，作成したばかりのPostgreSQLサービスのデータベースは空っぽで，スキーマもデータも何も入っていません。従って，アプリを正常に動作させるためにはデータベースの"migration"を行う必要があります。今回は，アプリの `README <https://github.com/kandanapp/kandan/blob/master/DEPLOY.md#stand-alone-server>`_ の記述を参考に，起動コマンド内で ``bundle exec rake db:create db:migrate kandan:bootstrap`` を実行するようにしました。
* | 2) 起動コマンドで，listenするポートを環境変数から取得する
  | デフォルトの起動コマンドでは，環境変数 ``VCAP_APP_PORT`` (もしくは ``PORT``) に設定された値のポートをlistenするようになっています。しかし，独自に起動コマンドを設定した場合，デフォルトの起動コマンドが上書きされてしまうため，その点についても考慮して起動コマンドを設定する必要があります。今回は，起動コマンド最後の部分で， ``$PORT`` を参照して起動するようにしました。


動作検証
========

アプリが起動できたので，ブラウザーからアクセスしてみます。

初期画面
--------

.. image:: /imgs/001/001-initial.png
   :width: 50%

ユーザー登録
------------

.. image:: /imgs/001/001-user-registration.png
   :width: 50%

ログイン
--------

.. image:: /imgs/001/001-logging-in.png
   :width: 50%

↓

.. image:: /imgs/001/001-first-screen-after-login.png
   :width: 50%

動画再生
--------

.. image:: /imgs/001/001-playing-movie.png
    :width: 50%

書き込み
--------

.. image:: /imgs/001/001-typing-message.png
   :width: 50%

↓

.. image:: /imgs/001/001-sent-message.png
   :width: 50%


まとめ
======

以上，CF上にアプリをデプロイし，一通り動作することを確認しました。

今回のアプリは，もともとHerokuやCF等のPaaSでの動作を想定したものだったので，比較的容易にデプロイできました。一方で，既存のアプリをCF上で動かす際に起きる際の代表的な課題のうちの2つ(サービス関連の設定, listenするポートの指定)が含まれていて，最初に取り上げるアプリとしては適当だったのではないかと考えています。

今後は，こんな感じで原則1日1アプリ(土日祝は休み)のペースで記事更新を行っていく予定です。100アプリだと約5か月かかる計算になりますが，ゆるゆるとお付き合いいただければと思います。


今回使用した環境
================

* | cf-release (v194)
  | https://github.com/cloudfoundry/cf-release/tree/v194
  | ( https://github.com/cloudfoundry/cf-release/tree/345a8b3e1ea0005a3e9fced13f0bf6fa6f7ad981 )
* | bosh-lite
  | https://github.com/cloudfoundry/bosh-lite/tree/01db9da7b4122f7d02858d92e0fe938e91256649
* | CF CLI (v6.11.3-cebadc9-2015-05-20T19:00:58+00:00)
  | https://github.com/cloudfoundry/cli/releases/tag/v6.11.3
* | postgresql-cf-service-broker
  | https://github.com/cloudfoundry-community/postgresql-cf-service-broker/tree/2e550a065374ffab1a999657c3dabdaa312aa61b
* | Docker Image
  | REPOSITORY=postgres, TAG=9.4.2, ID=1636d90f0662
