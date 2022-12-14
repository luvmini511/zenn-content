---
title: "SSHにおける公開鍵・秘密鍵を用いた認証方法をちゃんと理解しておく"
emoji: "🔑"
type: "tech"
topics: ["ssh"]
published: true
published_at: "2022-06-13 15:00"
---

# 🌼 はじめに

こないだ新しく開発環境セットアップしてて「サーバーにSSH接続できるようにするので公開鍵お願いします」と言われました。一瞬「SSHとか公開鍵とか何となく知ってたけど正確になんだっけ、、？」となり、しっかり知っておこうと思って記事にします。

# 1. SSH とは

## 1-1. SSHってなに
- 「Secure SHell」の略で、**ネットワークプロトコル**の一種
- パソコンとパソコンがネットワーク上でつながるとき**安全な通信**をするためのプロトコル

+) プロトコルとは
*コンピュータでデータをやりとりするために定められた手順や規約、信号の電気的規則、通信における送受信の手順などを定めた規格*

## 1-2. 代表的な使用例
1. 遠隔操作（ex. AWS）
2. データコピ・転送（ex. github）


# 2. 認証の仕組み

## 2-1. 公開鍵・秘密鍵

SSHはパスワードではなく、公開鍵(Public Key)と秘密鍵（Private Key）と呼ばれるセットの鍵を用いた認証を行ってます。(鍵って言ってますが本当はファイルです)

- **秘密鍵（Private key）**
  - 公開しちゃ絶対ダメ

- **公開鍵（Public key）**
  - 公開しても大丈夫

## 2-2. 認証方法

このセットの鍵は複雑な仕組みで結ばれていて、SSHはそれを用いて接続時に認証を行います。具体的にどうやってるかを今から見ていきましょう。

まず事前準備として公開鍵をサーバー側に置いといて、秘密鍵は自分のPCに保管します。

![](https://storage.googleapis.com/zenn-user-upload/ca1a06886bb9-20220612.jpg)
*間違えても秘密鍵は絶対外部に送らないように！*


~~（本当はもっと複雑のはずですが）この鍵を利用した認証過程ざっくりまとめてみました。~~

~~1. クライアントがサーバーに接続をリクエスト~~
~~2. サーバーは任意の文字列を生成し、クライアントの公開鍵で暗号化してクライアントに送る~~
~~3. クライアントはもらった暗号を自分の秘密鍵で復号~~
~~4. こうやってクライアントとサーバーは同じ文字列を持つ~~
~~5. クライアントとサーバー両方文字列をハッシュ化する~~
~~6. クライアントがサーバーにハッシュ化した文字列を送る~~
~~7. サーバーは送られてきたハッシュ値と自分がハッシュ化したハッシュ値を比較する~~
~~8. ２つのハッシュ値が同じだと判断したら接続許可~~

**+)2022.06.13追記**
はい、打ち消し線のところはデタラメの情報でした、すみません（^_^;)
@angel_p_57さんからご指摘いただき、正しい記事を参考にさせていただきました。

https://qiita.com/angel_p_57/items/19eda15576b3dceb7608#%E5%85%AC%E9%96%8B%E9%8D%B5%E8%AA%8D%E8%A8%BC%E3%81%AE%E3%82%A4%E3%83%A1%E3%83%BC%E3%82%B8

> 電子署名の応用で、**秘密鍵の所有者しか正しい署名が作れないことを利用し、署名の正しさを公開鍵で検証してもらうことで、確かに所有者であることを確認**してもらいます。
> 
> このような使い方ですから、秘密鍵は利用者本人以外に漏らしてはいけませんし、公開鍵は予めサーバに登録しておく必要があります。
これが本人確認になっているということです。

本当は暗号化・復号ではなく電子署名を用いた認証方法でした。ざっくりまとめるとこのような感じだと思います。
1. クライアントは秘密鍵を使って署名を生成し、サーバーに送る
2. サーバーは公開鍵を使って署名が正しいか検証する

詳しい手順はこちらの記事で詳しく説明してくれてます。
https://qiita.com/angel_p_57/items/2e3f3f8661de32a0d432#%E5%A6%A5%E5%BD%93%E3%81%AA%E8%AA%AC%E6%98%8E

## 2-3. 鍵の作成

それでは関心な公開鍵・秘密鍵はどう作るのかでが、これに関しては既に素晴らしい記事がたくさんあるので今回は省略します。

ちなみに私がいつもお世話になってる記事はこちらです。
https://qiita.com/suthio/items/2760e4cff0e185fe2db9

## 2-4. Passphrase

鍵セットを生成するときpassphraseの入力を求められます。これは秘密鍵にアクセスするためのもので、任意なので設定しないこともできます。

passphraseを設定することのメリットは、万が一秘密鍵を盗まれたとしてもpassphraseを入力しないと秘密鍵が使えないのでセキュリティ的にもっと安全ということです。

## 2-3. 鍵たちはどこに

鍵生成もしたし、サーバー側に公開鍵送ろう！となったとき「え、鍵たちどこにあるんだ、、？」となる方のために。

鍵生成のとき特に保存場所を指定してなければルートディレクトリの配下に`.ssh`というディレクトリが作られ、その中に鍵たちが入ってます。パスでいうと`~/.ssh/`の配下になりますね。

Finderで探したい場合はルートディレクトリで `⌘`+`⇧`+`.`（隠しファイル表示ショートカット）を押したら`.ssh`が出てきます

![](https://storage.googleapis.com/zenn-user-upload/bc6848d50eb2-20220612.png)



# 🌷 終わり

これで「サーバーにSSH接続できるようにするので公開鍵お願いします」と言われても完璧に対応できるようになりました。やった