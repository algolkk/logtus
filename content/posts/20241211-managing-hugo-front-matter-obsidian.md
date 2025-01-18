---
date: 2024-12-11T18:46:41+09:00
draft: false
title: ObsidianでHugoのFront matterを作成・更新する
categories: Develop
tags: [Obsidian, Hugo]
---

Hugo の記事作成に Obsidian のコミュニティプラグインを使えばわ～いらくち～んになれる気がしたので色々試してみました。  
とりあえず思い付いたものは Front matter の入力自動化くらいですが、個人的にこれだけでもかなり便利になったと思うので設定メモを残しておきます。

## 環境情報

- Hugo + Blowfish テーマ
- `content`配下を Obsidian の保管庫とする
  - `/content/.obsidian`が存在している状態

## 使用したコミュニティプラグイン

- Templater (v2.9.1)
- Linter (v1.27.1)

## プラグインの設定

### Templater

Hugo では markdown ファイルの Front matter(prefix のようなもの)にメタデータを持っています。
今回は Front matter の内容をテンプレート化して Obsidian からファイルを作成したタイミングで自動生成するように設定します。

まずは下記のテンプレートを用意し、`/content/template/template.md`として保存します。  
(フォルダ名やファイル名は何でも良いですが、変更する場合はこの後に記載されているパスを適宜読み替えてください)

```markdown
---
date: <% tp.date.now("YYYY-MM-DD[T]HH:mm:ssZ") %>
draft: true
title: <% tp.file.title %>
categories: ""
tags: []
---
```

次に、プラグインの設定を下記の通りに編集します。

- `Template fodler location`: テンプレートファイルを格納したフォルダを指定
- `Trigger Templater on new file creation`: 有効
- `Forder templates > Enable folder templates`: 有効
  - `folder`: /(root)
  - `Template`: /template/template.md

これで Templater が有効になり、Obsidian でファイル作成時に`hugo new`と同じようにメタデータが自動生成されるようになります。  
しかしこのままでは Hugo 側のビルド時にエラーになるため `/config/_default/hugo.toml`に`ignoreFiles`の設定を追加し、テンプレートを Hugo のビルド対象から除外します。

```toml
ignoreFiles = [
  'content/template/.*'
]
```

これで`hugo server -D`をしてもエラーが出なくなります。

### Linter

メタデータの date をファイル保存日時と合わせるコマンドを設定します。  
`hugo new`時はファイル作成日時が date に自動挿入されますが、事前に作成しておいて後日公開したい！といった場合には日付を直すのが少し面倒です。(手入力出来ますが、日付が誤っていたら悲しい)
そこで、Linter を使用してファイル保存後にコマンドパレットから呼び出すことでメタデータの date を更新出来るようにします。

- `Format YAML Array`: 全て有効
- `YAML Timestamp` : 有効
  - `Date Modified` : 有効
  - `Format` : `YYYY-MM-DD[T]HH:mm:ssZ`
- その他設定は初期値

上記を設定後、コマンドパレットから更新したいファイルを開いた状態で
`Linter: lint the current file`を実行するとメタデータの日付が更新されます。

## あとがき

Obsidian にコミュニティプラグインを導入することで、重いエディタを開かなくても記事を書けるようになりました。  
これでゲームをしながらメモを書く時も安心ですね(ゲーム中に記事を書くなというのはそう)  
ここから更に拡張したり、Hugo 側の設定変更に合わせてプラグインを調整する事も可能なので色々試していきたいです。

本題とは無関係ですが、Misskey へのシェアボタンが標準実装されていなかったので勝手に追加しました。  
リンク先は Mastodon へのシェアと同じ Share2Fedi で、まとめて Fediverse ボタンにしようかとも考えましたが折角 Misskey アイコンを追加したので個別にしてもいいよね。
