---
date: 2025-01-29T19:29:31+09:00
draft: false
title: Misskeyサーバーに藍(bot)を導入する
categories: Develop
tags: [Misskey, 藍]
---
Misskey用の日本語botである藍をフォークしてオリジナルキャラクターのbotをお一人様鯖に導入しました。  
https://lotuskey.net/@rikki 
※試験運用中のためサイレンスロールを付与しています

Rikkiと書いてリッキーと読みます。メイドのﾈｺﾁｬﾝです。  
オリジナルの藍からセリフ等のキャラクター設定のみ改変して運用しています。具体的には`serif.ts` と`index.ts`に含まれるセリフデータ、 `config.json`のaichat用のキャラクター設定プロンプトを変更しています。  

今回は`npm`を使用して導入しました。以下導入手順メモです。

## 事前に準備した方が良いもの

- botとして動かすMisskeyアカウントのアクセストークン
- botに付与するロール
	- レートリミットやパブリック投稿の制限などお好みで

### aichat(Gemini)を使用する場合

- Gemini APIキー
	- 2025年1月21日現在はGoogle AI Studioから無料で取得可能
	  https://aistudio.google.com/app/apikey?hl=ja
- aichat用プロンプト
	- キャラクターの性格をデフォルトから変更したい場合、あらかじめ用意しておくと楽
	- Web版Geminiを使用してプロンプトが想定通りに動くことを検証しておくと◎


## Node.jsとnpmのインストール

既にMisskeyが稼働しているサーバーならインストールされていそう？  
ない場合はインストールする

## MeCabのインストール

```
$ sudo apt install mecab
$ sudo apt install libmecab-dev
$ sudo apt install mecab-ipadic-utf8
$ mecab
特急はくたか
特急    名詞,一般,*,*,*,*,特急,トッキュウ,トッキュー
は      助詞,係助詞,*,*,*,*,は,ハ,ワ
く      動詞,自立,*,*,カ変・クル,体言接続特殊２,くる,ク,ク
た      助動詞,*,*,*,特殊・タ,基本形,た,タ,タ
か      助詞,副助詞／並立助詞／終助詞,*,*,*,*,か,カ,カ
EOS
```
### mecab-ipadic-neologdのインストール
```
$ git clone https://github.com/neologd/mecab-ipadic-neologd.git
$ cd mecab-ipadic-neologd
$ sudo bin/install-mecab-ipadic-neologd

# 失敗したら再インストールする
$ sudo apt install --reinstall build-essential
$ sudo bin/install-mecab-ipadic-neologd

$ sudo mv /usr/lib/x86_64-linux-gnu/mecab/dic/mecab-ipadic-neologd /var/lib/mecab/dic

$ sudo vim /etc/mecabrc
```

```/etc/mecabrc
# dicdirを変更する
;
; Configuration file of MeCab
;
; $Id: mecabrc.in,v 1.3 2006/05/29 15:36:08 taku-ku Exp $;
;
; dicdir = /var/lib/mecab/dic/debian # 変更その１
dicdir = /var/lib/mecab/dic/mecab-ipadic-neologd # 変更その２
; userdic = /home/foo/bar/user.dic
; output-format-type = wakati
; input-buffer-size = 8192
; node-format = %m\n
; bos-format = %S\n
; eos-format = EOS\n
```

```
$ mecab
特急はくたか
特急はくたか
特急    名詞,一般,*,*,*,*,特急,トッキュウ,トッキュー
はくたか        名詞,固有名詞,一般,*,*,*,はくたか,ハクタカ,ハクタカ
EOS
```

## フォントインストール

好きなフォントを入れる
```
$ wget https://yourfont.ttf
$ mv yourfont.ttf font.ttf
```


## 藍のインストール

```
$ git clone https://github.com/syuilo/ai # フォークを使用する場合はリポジトリを変更する
$ cd ai
$ nano config.json # リポジトリのREADME.mdを参照して編集

$ npm install
$ npm run build
$ npm start
```
`npm run build`後にエラーが出ていましたが内部的にはビルド出来ているらしく（？）、そのまま`npm start`したら動きました（？？？）

## 動作確認

`@ai ping`する　※ダイレクトメッセージではなく@でメンションする  
pongが返って来たら一旦停止してSystemdに登録する

```
$ sudo nano /etc/systemd/system/ai.service
```

```/etc/systemd/system/misskey-ai.service
[Unit]
Description=Misskey bot daemon

[Service]
Type=simple
User=user
Environment="NODE_ENV=production"
ExecStart=/usr/bin/npm start
WorkingDirectory=/home/dir/ai
TimeoutSec=60
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=user
Restart=always

[Install]
WantedBy=multi-user.target
```

```
$ sudo systemctl daemon-reload
$ sudo systemctl start misskey-ai
$ sudo systemctl enable misskey-ai
```

## アップデート

ソースコードをpullしてビルドして立ち上げるだけです。
```
$ git fetch
$ git pull
$ npm run build
$ sudo systemctl start misskey-ai
```

## 参考

以下の記事等を参考にさせて頂きました！

Misskeyサーバー設立備忘録  
https://note.com/beta_1673/n/n3c4eb6545396#7843661b-1c3b-4c34-a82a-45f83ff18da1

Misskey入れてみる備忘録(+α)【~v10】  
https://qiita.com/YuzuRyo61/items/7105d16ac75c78899f1c#%CE%B1--%E8%97%8D%E3%81%A1%E3%82%83%E3%82%93%E3%82%92%E5%85%A5%E3%82%8C%E3%81%A6%E3%81%BF%E3%82%88%E3%81%86

Ubuntu 20.04 LTSにMeCab(mecab-ipadic-neologd)/CaboChaのインストール  
https://qiita.com/kado_u/items/e736600f8d295afb8bd9

Error building in node v21 #133  
https://github.com/syuilo/ai/issues/133
