========
Preamble
========


はじめに
========

この度，日本 Cloud Foundry グループでは，「Cloud Foundry 百日行」と題して，オープンソースの PaaS である Cloud Foundry 上に，オープンソースのアプリケーションをデプロイしてみる営みを，百日間にわたって続けようというプロジェクトを始めることになりました。

このプロジェクトの目的は，大きく次の3つです：

#. Cloud Foundry の特徴を知ってもらう
#. アプリケーションを Cloud Foundry 上で動かす際，どのような点に留意すべきかを知る
#. Cloud Foundry に対して，どのようなニーズ/要望があるかを知る

3番目の項目に関しては，読んでいただいた方からのご意見が頼りです。ページ下部の Twitter ボタンから `#cfgrjp <https://twitter.com/search?q=%23cfgrjp>`_ ハッシュタグを付与してツイートする等の方法でご意見・ご質問等いただければありがたく思います。また「このオープンソース・アプリケーションを試してほしい」等のご要望もお待ちしております。


前提
====

このシリーズでは，各記事共通の前提として，以下を想定しています。

* 環境は，原則，アプリ検証時の最新版に近い版の Cloud Foundry を `bosh-lite <https://github.com/cloudfoundry/bosh-lite>`_ で構築したものを使用します

  * 【参考】bosh-lite を使った Cloud Foundry 構築の解説記事は，以下のようなものがあります

    * | CloudFoundry(v194)をbosh-lite上にデプロイする
      | http://qiita.com/k-kurumi/items/66057620d5d344d07414
    * | CygwinなThinkpad E130にbosh-liteでCloud Foundry V2(v194)を入れてみた
      | http://ipcrm.hatenablog.com/entry/2015/05/17/051820
    * | Cloud Foundry V2 (v172)をBosh-liteで構築する(VirtualBox+Vagrant編)
      | http://qiita.com/jacopen/items/463b5045015de2460d74

  * 【参考】Cloud Foundry のパブリック・サービスとしては，以下のようなものがあります

    * | Anynines
      | http://www.anynines.com/
    * | AppFog
      | http://www.appfog.com/
    * | Bluemix
      | https://console.ng.bluemix.net/
    * | Cloudn PaaS
      | https://www.ntt.com/cloudn/data/paas.html
    * | Pivotal Web Services
      | https://run.pivotal.io/

* | Cloud Foundry には既にログイン済みの状態とします
  | 【参考】ログイン手順は以下の通りです

  ::

      $ cf api api.10.244.0.34.xip.io --skip-ssl-validation
      $ cf login -u ＜ユーザー名＞


注記
====

* | 記事中で使用するツール類(cf, git, hg, curl等)についての解説は，原則として省略します
  | それぞれのツールの提供元のサイト等で情報をお調べください
* デプロイできたアプリについては動作確認を行いますが，それらは簡易なものであり，当該アプリが Cloud Foundry 上で完全に動作することを保証するものではありません

以上，予めご了承ください。
