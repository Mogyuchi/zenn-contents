---
title: "【Node.js】フォーマッター・リンターをコミット時に実行する(パッケージ非依存)"
emoji: "🛠️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nodejs","git","eslint","prettier"]
published: true
---

# はじめに
Node.jsプロジェクトの品質を保つためコミット時にフォーマッターやリンターをかけるのを強制したい時、[husky](https://www.npmjs.com/package/husky)・[lint-staged](https://www.npmjs.com/package/lint-staged)を使用する例が散見されます。ですが、この目的のためだけに依存関係を増やしたくない場合もあるでしょう。
この記事では、これらのパッケージに依存することなくpre-commitフックを共有し、[Prettier](https://prettier.io)とリンターをコミット時に必ず実行させる方法を紹介します。
:::message
npm以外のパッケージマネージャーを使用している場合は適宜読み替えてください。 Plug'n'Playを使用している場合は実行ファイルのパス指定では動きませんのでパッケージマネージャー経由で実行するようにしてください。
:::

# やり方
1. `.githooks/pre-commit`ファイルをつくり次のように記載する
```sh:.githooks/pre-commit
#!/bin/sh
FILES=$(git diff --cached --name-only --diff-filter=ACMR | sed 's| |\\ |g')
[ -z "$FILES" ] && exit 0

npm run lint \
&& echo "$FILES" | xargs ./node_modules/.bin/prettier --ignore-unknown --write \
&& echo "$FILES" | xargs git add
```
2. `package.json`に次の行を追記する
```diff json:package.json
{
  "scripts": {
+   "postinstall": "git config --local core.hooksPath .githooks && chmod -R +x .githooks/"
  }
}
```
3. installを行う
```shell
$ npm install
```

# 説明
`.git`配下のファイルなどはgit管理されません。そのため、Gitのバージョン2.9より追加されたhooksの置き場を任意に指定できる機能を使い、`.githooks`ディレクトリを使用するようにします。
このディレクトリは任意の名前にして問題ありません。ただし、ファイル名は一致させる必要があります。
```sh:.githooks/pre-commit
#!/bin/sh
FILES=$(git diff --cached --name-only --diff-filter=ACMR | sed 's| |\\ |g')
[ -z "$FILES" ] && exit 0

npm run lint \
&& echo "$FILES" | xargs ./node_modules/.bin/prettier --ignore-unknown --write \
&& echo "$FILES" | xargs git add
```
Prettierのドキュメントを参考にしています。
https://prettier.io/docs/en/precommit.html#option-6-shell-script

---
> ```sh
> FILES=$(git diff --cached --name-only --diff-filter=ACMR | sed 's| |\\ |g')
> ```
ステージ上にあるファイルの一覧を取得し、変数に入れています。

---
> ```sh
> npm run lint
> ```
`package.json`に`lint`スクリプトが記載されている前提のコードとなっています。
eslintを使用している場合は次のようにするとよいでしょう。
```sh
./node_modules/.bin/eslint --cache .
```
:::message
`--cache`オプションをつけることで更新があったファイルのみlintが実行されるようになります。
生成される`.eslintcache`は`.gitignore`に記載してください。
:::

> ```sh
> echo "$FILES" | xargs ./node_modules/.bin/prettier --ignore-unknown --write
> ```
前述した一覧を渡すことで、ステージ上にあるファイルのみフォーマットされるようにしています。

> ```sh
> echo "$FILES" | xargs git add
> ```
ここまでの処理による変更をステージ上に反映させます。

これらを`&&`で繋ぐことで、lintエラーが出たりフォーマットが失敗したりした段階でコミットが中止されるようにしています。

---
```json:package.json
"postinstall": "git config --local core.hooksPath .githooks && chmod -R +x .githooks/"
```
`postinstall`スクリプトはインストールが終わった後に実行されます。
このときに、gitのフックのパスを先ほど作成したフォルダにしその配下のスクリプトが使われるように変更しています。
