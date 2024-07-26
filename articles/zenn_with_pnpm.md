---
title: "pnpmでZennCLIを扱いたい"
emoji: "💻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["pnpm","zenncli","zenn"]
published: true
---

## pnpmを使いたい

フラットなnode_modulesには問題があるからです。
@[card](https://www.kochan.io/nodejs/pnpms-strictness-helps-to-avoid-silly-bugs.html)
`yarn@berry`を使用してもこの問題を回避できます。

## pnpmでのZenn CLI導入手順

基本的に違いがあるところだけ記載しますので、[公式のnpm用ガイド](https://zenn.dev/zenn/articles/install-zenn-cli)も都度参照してください。

### 1. CLIをインストールする

```shell
pnpm init --yes
```

```json:package.json
"packageManager": "pnpm@6.32.3"
```

を追記して使用するパッケージマネージャーを指定しておくと便利です。
`6.32.3`は執筆時点での最新バージョンを示しています。

```shell
pnpm add zenn-cli
```

peer dependenciesにのみ指定されているパッケージは現行のpnpmはインストールしないので、追加します。

```shell
pnpm add @types/markdown-it@"*"
```

### 2. Zenn用のセットアップを行う

```shell
pnpm zenn init
```

`pnpm exec`は[execを省略して実行できます](https://pnpm.io/ja/cli/exec#例)。

### 3. 導入完了🎉

プレビューサーバーをたててみます。

```shell
$ pnpm zenn preview
# 👀 Preview on http://localhost:8000
```

以後も `npx zenn`とあるところは`pnpm zenn`に置き換えましょう。

## CLIのアップデート

```shell
pnpm add zenn-cli@latest
```

で更新できます。
