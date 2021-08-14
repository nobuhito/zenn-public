---
title: "VS Code の Dev Container で Zenn の執筆環境を \"ミニマム\" に構築"
emoji: "✍"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["VSCode", "Zenn", "Docker"]
published: true
description: "Building Zenn's writing environment with VS Code's Dev Container"
slug: "devcontainer-of-zenn"
---
# モチベーション

Container に Zenn の執筆環境を作って、VS Code さえあればすぐにいつでも同じ環境を用意できるようにしたい。

Google で検索するとそれなりに同じような記事がヒットする。だけど、いきなり `Dockerfile` の解説から始まってたり、多分不要だと思われる `dockker-compose` ありきな記事ばっかり。なので、VS Code に用意された機能を使いながら、必要な設定だけでミニマムにコンテナ環境を作ってみることにする。

だって、内容が理解できない複雑なコンテナなんて、誰も使いたくないだろうから。

ちなみに、Zenn の執筆環境だけだったら `npm` さえ入っていれば良いのでそれほど面倒じゃないが、`textlint` とかも使ったりするとコンテナ環境というのは結構便利。また、ローカル環境が汚れないというのも精神的に良い。

# 前提条件

前提条件としては、PC に `Docker` が、VS Code には `remote-container` がインストールされていること。

![](/images/dev-container-of-zenn/remote-container.png)

あとは、GitHub に Zenn 用のリポジトリがあること。

# Dev Container の構築

## 1. Dockerfile の作成

シンプルにとはいえ、Dev Container の機能を使うので `Dockerfile` は必須になる。
しかし、VS Code というか Microsoft がいろいろ準備してくれているので、それほど Docker に詳しくなくとも大丈夫。

手順としては、まずコマンドプロンプトを開いて `Remote-Containers: Open Folder in Container...` を選択し、ウィザードを起動する。

![](/images/dev-container-of-zenn/open-folder-in-container.png)

その後は、以下の流れで進めると Container のビルドまで自動的に実行される。Docker とかのコマンドがわからなくても大丈夫。

1. Dev Container で開きたいフォルダを選択
  今開いているフォルダが既に開かれてる状態だとおもうので、そのまま `Open` をクリック。
1. コンテナのベースとなるものを選択
  `Show All Definitions > Node.js` と進んで `16` を選択。

Container のビルドが終わると、 `.devcontainer` へ `Dockerfile` が自動的に作られてる。

ちなみに、単に Node.js だけのコンテナだが、Microsoft のカスタマイズが入っているのか、プロンプトが可愛い。

![](/images/dev-container-of-zenn/terminal.png)

現時点でローカルかコンテナかの表示は左下で確認でき、クリックするとサブコマンドの実行もできる。

![](/images/dev-container-of-zenn/dev-container-node.js.png)

VS Code の起動方法によっては、今後もこの `Open Folder in Container...` コマンドの実行は必要になる。

## 2. Node 環境の整備

次に Node.js の環境を整えていくのだが、何はなくとも `zenn-cli` のインストールが必要になる。

作られた Dockerfile を見ると、コメントアウトされてる node モジュールのインストールコマンド行が用意されている。

```Dockerfile
# [Optional] Uncomment if you want to install more global node modules
# RUN su node -c "npm install -g <your-package-list-here>"
```

指示に従って `RUN su node -c "npm install -g zenn-cli@latest"` とし、コマンドパレットから `Rebuild Container` を実行する。

コンテナのリビルドが終わると `zenn` コマンドが使えるようになっているはずなので、ターミナルで `zenn --help` や `zenn --version` を実行してみる。

![](/images/dev-container-of-zenn/installed-zenn.png)

無事にコマンドが実行されれば完了。

:::message
いろいろな記事で Dockerfile に `npm --init` だったり `COPY package.js /ws/package.js` とかの記載がある。
だけど、専用のコンテナなので `package.js` は不要。専用のコンテナだから、全部グローバルにインストールして問題ないはず。
:::

また、Dev Container が自動的にポート転送もしてくれる（この認識であってるだろうか）ので、`Dockerfile` や `docker-compose` でのポート転送の設定も必要ない。

プレビューで確認したいときは `zenn preview` すると下記のようなポップアップが表示される。
この `ブラウザーで開く` をクリックすると、ポート転送の設定がなくてもブラウザで確認できる。

![](/images/dev-container-of-zenn/port-forward.png)

# フォルダ構成

ここで、考えてみたいのがフォルダ構成。
Git での管理方法にも絡むのだが、Zenn と GitHub との連携をどうしてるかによって３通りの方法が考えられる。

## 1. すべてパブリックリポジトリで管理

これが一番楽。
`zenn init` したフォルダに Dev Container を追加し、 `.devcontainer` もまとめて Git で管理するだけ。

```
/public-repository
  - /.git
  - /.devcontainer
    - devcontainer.json
    - Dockerfile
  - /articles
  - /books
  - .gitignore
```

## 2. パブリックリポジトリとプライベートリポジトリを別々に管理

有料記事を書いている人は、プライベートリポジトリを用している人が多いと思われる。
その場合、それぞれ別々の Dockerfile（内容は同じ）で管理すると扱いが楽になる。前項のフォルダ構成を２つ用意して別々に管理するだけ。

```
/public-repository
  - /.git
  - /.devcontainer
    - devcontainer.json
    - Dockerfile
  - /articles
  - /books
  - .gitignore
/private-repository
  - /.git
  - /.devcontainer
    - devcontainer.json
    - Dockerfile
  - /articles
  - /books
  - .gitignore
```

## 3. パブリックリポジトリとプライベートリポジトリをまとめて管理

前述の方法でも良いのだが、一方の `.devcontainer` 以下を編集するともう一方も追従しなければならない。
最低限の環境で良ければそれほど変更する必要もないのだが、textlint などを追加していくと２つの管理というのは結構面倒になると思われる。

なので、現実的に良さそうなのは３つのリポジトリで管理する方法。
ちなみに、VS Code は GitSubmodule にも対応しているので下記コマンドで複数リポジトリを追加できる。

```shell
git submodule add https://github.com/hoge/private.Git
```

こうすると管理が楽。

```
/public-repository
  - /.git
  - /.devcontainer
    - devcontainer.json
    - Dockerfile
  - .gitignore
  - .gitmodules
  - /public-repository
    - /.git
    - /articles
    - /books
    - .gitignore
  - /private-repository
    - /.git
    - /articles
    - /books
    - .gitignore
```

ただし、この場合は Dev Container 環境に Git コマンドが必要になるので、Dockerfile の修正が必要となる。
とはいえ、Dockerfile にすでにコメントアウトされた雛形が準備されている。

```Dockerfile
# [Optional] Uncomment this section to install additional OS packages.
# RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
#     && apt-get -y install --no-install-recommends <your-package-list-here>
```

そこを、下記のように書き換えるだけで OK。

```Dockerfile
# [Optional] Uncomment this section to install additional OS packages.
RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -y install --no-install-recommends git
```

また、他にもコマンドが必要であれば同じようにインストール出来る。

# その他

この項は調整を実施したら都度追加していく。

## 1. textlintの導入

少しでも人様が読むに耐えるまともな文章へとするため、textlint を導入する。

### `Dockerfile`

既存の Dockerfile を以下のように変更し、textlint と評判が良さそうなルールを追加する。

```diff
# [Optional] Uncomment if you want to install more global node modules
-RUN su node -c "npm install -g zenn-cli@latest"
+RUN su node -c "npm install -g zenn-cli@latest\
+                               textlint \
+                               textlint-rule-preset-ja-spacing \
+                               textlint-rule-preset-ja-technical-writing \
+                               @proofdict/textlint-rule-proofdict"
```

### `.textlintrc`

変更した Dockerfile をリビルドすると `textlint` コマンドが使えるようになってるはずなので、 `textlint --init` を実行して.textlintrc を作する。

```json
{
  "filters": {},
  "rules": {
    "preset-ja-spacing": {
      "ja-space-between-half-and-full-width": {
        "space": "always"
      }
    },
    "preset-ja-technical-writing": true,
    "@proofdict/proofdict": {
      "dictURL": "https://azu.github.io/proof-dictionary/"
    }
  }
}
```

:::message
上記は１つの例としてだが、個人的には以下の [現時点の設定](https://github.com/nobuhito/dev-container-of-zenn/blob/main/.textlintrc) のような設定が良い感じ。
ここらへんは各人で最適な設定が違うと思われるので、いろいろ試してみると良い。
**現時点の設定** → https://github.com/nobuhito/dev-container-of-zenn/blob/main/.textlintrc

:::




### `devcontainer.json`

Dev Container への拡張機能は、ローカルの拡張機能と基本的には別になる。
なので、すでにローカル環境へ textlint をインストールしていても、Dev Container 環境へも別途インストールしないと利用できない。

すでに、 `dbaeumer.vscode-eslint"` が登録されている `extensions` へ `taichi.vscode-textlint` を追加。

また、settings も以下のようにしてみた。

```json
	"settings": {
		"editor.formatOnSave": true,
		"textlint.autoFixOnSave": true,
		"textlint.run": "onType",
	},
```

# コンクルージョン

このくらいだったらローカルに構築しても良い感じだが、VS Code の Dev Container の仕組み的なところを理解できたような感じがする。

あと、Dockerfile や docker-compose.yml を探して結構時間を浪費した。
ショートカットで楽しようとせず、自分で一からちゃんとやってみるのが吉。自分自身の勉強にもなるし、最終的に時間的もかからない場合が多い。