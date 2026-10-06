# training_devcontainer_html

## ファイル構成

```text
training_devcontainer_html/
├── .devcontainer/
│   ├── devcontainer.json
│   ├── docker-compose.yml
│   └── Dockerfile
└── app/
      └── index.html

```

## 構築手順

### 1. フォルダの作成

```bash
mkdir -p .devcontainer app
```

### 2. `Dockerfile` の作成

* **役割**: 
コンテナの **中身** (OSやインストールするソフトウェア) の土台を作る設計図

```bash
vi .devcontainer/Dockerfile
```

```Dockerfile
FROM mcr.microsoft.com/devcontainers/javascript-node:18
```

* **解説**: 
Microsoft公式の Node.js 18 が入った開発用イメージ（`[mcr.microsoft.com/devcontainers/javascript-node:18](https://mcr.microsoft.com/devcontainers/javascript-node:18)`）をベースに指定している．
将来的に追加のパッケージを入れたい場合は，ここに `RUN` 命令などを書き足していく．

### 3. `docker-compose.yml` の作成

* **役割**: 
コンテナの **起動方法** や **ホストとの連携ルール** をまとめる司令ファイル．

```bash
vi .devcontainer/docker-compose.yml
```

```yml
version: '3.8'

services:
  web:
    build:
      context: .
      dockerfile: Dockerfile
    volumes:
      - ../..:/workspace:cached
    command: sleep infinity
```

* **解説**:
  - `services.web`: 
    `web` という名前のコンテナを定義．

    - `build`: 
      同じ階層にある `Dockerfile` を使ってコンテナをビルドする指定．

    - `volumes`: 
      ホストのプロジェクト全体を，コンテナ内の `/workspace` という場所にマウントする．
      これでコンテナ外でファイルを編集しても即座にコンテナ内に反映される．

    - `command: sleep infinity`: 
      コンテナ起動後，何もせずに即終了してしまうのを防ぐため，無限に起動状態を維持させるコマンド．

### 4. `devcontainer.json` の作成

* **役割**: 
VSCodeに "このプロジェクトをどのコンテナ環境でどう開くか" を伝える設定ファイル

```bash
vi .devcontainer/devcontainer.json
```

```json
{
  "name": "Docker Compose HTML Dev",
  "dockerComposeFile": "docker-compose.yml",
  "service": "web",
  "workspaceFolder": "/workspace",
  "customizations": {
    "vscode": {
      "extensions": [
        "ritwickdey.LiveServer"
      ]
    }
  },
  "forwardPorts": [8080]
}
```

* **解説**:
  - `"dockerComposeFile": "docker-compose.yml"`: 
    どの Docker Compose ファイルを使うかを指定．

  - `"service": "web"`: 
    複数のコンテナがある中で，どれをVSCodeのメイン作業場として接続するかを指定．

  - `"workspaceFolder": "/workspace"`: 
    コンテナに接続したとき，最初に開くフォルダの場所を指定．

  - `"customizations.vscode.extensions"`: 
    コンテナ起動時に自動でインストールしてほしいVSCodeの拡張機能(今回は Live Server)を指定．

  - `"forwardPorts": [8080]`: 
    コンテナ内の `8080` 番ポートを手元のパソコンからアクセスできるように転送する設定．

5. `app/index.html` の作成

```bash
vi app/index.html
```

```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>Hello Devcontainer</title>
</head>
<body>
    <h1>Hello World!</h1>
    <p>docker-compose構成からのこんにちは！</p>
</body>
</html>
```

---

## 起動と動作確認

### 1. VSCodeを起動

```bash
code .
```

### 2. webサーバの起動

初回はDockerのビルドが走るため少し待つ．
コンテナが立ち上がり，VSCode内のターミナルが開いたら，以下のコマンドを実行してWebサーバーを起動する．

```bash
http-server app -p 8080
```

* **確認方法**: 
  ターミナルに `Available on: [http://127.0.0.1:8080](http://127.0.0.1:8080)` のようなログが表示される．

### 3. アクセス確認

ブラウザで `http://localhost:8080` にアクセスし，「Hello World!」と表示されることを確認する．

* **確認方法**: 
  画面に「Hello World!」が表示される．

---

# devcontainer の一般解説

Docker Composeを用いたマルチコンテナ環境における `devcontainer` の構築について、特定の技術や名前に依存しない**極めて一般的な汎用構造**と、各パラメータや設計思想についての詳細な内訳（各行・各プロパティの持つ意味）を解説します。

---

## 📁 全体設計のアーキテクチャ（概念モデル）

複数コンテナ（A〜E）を扱うDevcontainerは、「VS Codeがアタッチ（接続）して開発作業を行うメインのコンテナ」**と、**「それを裏から支える周辺コンテナ（DBやキャッシュなど）」をDocker Composeで束ね、VS Codeにその関係性を教えるという構造をとります。

```text
[プロジェクトルート]
 ├── .devcontainer/
 │    ├── devcontainer.json   # どのCompose構成を使い、どこに接続するかを定義
 │    ├── docker-compose.yml  # すべてのコンテナ（A〜E）のネットワークや依存関係を定義
 │    └── Dockerfile          # メインコンテナ用の追加ビルド設定
 └── (ソースコードやファイル群)

```

---

## 1. `devcontainer.json` の詳細な解説（網羅的仕様）

このファイルは、VS Codeに対して「どのComposeファイルを読み込み、どのコンテナに飛び込むか」を指示するための設計図です。

```json
{
  // 1. 識別名（VS CodeのUI上で表示される名前）
  "name": "General Multi-Container Architecture",

  // 2. 連携するDocker Composeファイルのパス（このjsonからの相対パス）
  "dockerComposeFile": "docker-compose.yml",

  // 3. アタッチ先指定（docker-compose.yml内のサービス名のうち、どれを開発の主戦場にするか）
  "service": "main_service",

  // 4. コンテナ内の作業ディレクトリ（VS Codeを開いたときに最初に開かれるパス）
  "workspaceFolder": "/path/to/workspace",

  // 5. ライフサイクル・フック（コンテナが完全に立ち上がった直後に1度だけ自動実行されるコマンド）
  "postCreateCommand": "echo 'Setup scripts can be placed here'",

  // 6. ポートフォワーディング（コンテナ内のポートをホストマシン側へ公開・転送する設定）
  "forwardPorts": [8080, 5432],

  // 7. VS Codeのカスタム設定（コンテナ内で自動的に有効化したい拡張機能や設定）
  "customizations": {
    "vscode": {
      "extensions": [
        "publisher.extension-name"
      ],
      "settings": {
        "editor.formatOnSave": true
      }
    }
  },

  // 8. リモートユーザーの指定（コンテナ内でどのユーザー権限として作業するか）
  "remoteUser": "root"
}

```

### 💡 重要な設計ポイント

* **`service` の選択**: 複数のサービス（A〜E）がある中で、**自分がコードを書き、ターミナルを叩いて作業する「母艦」となるコンテナ**を必ず1つここに指定します。

---

## 2. `docker-compose.yml` の詳細な解説（網羅的仕様）

A〜Eにあたるすべてのコンテナを定義し、それらのネットワーク結合やボリューム（データの永続化）、起動順序を管理します。

```yaml
version: '3.8'

services:
  # ==========================================
  # メインサービス（devcontainer.jsonの "service" と一致させる）
  # ==========================================
  main_service:
    build:
      context: .           # Dockerfileが存在するディレクトリ
      dockerfile: Dockerfile
    
    # データの同期（ホスト側のソースコードをコンテナ内の workspace にマウントする）
    volumes:
      - ../..:/path/to/workspace:cached
    
    # 【必須の概念】プロセス常駐化
    # Dockerコンテナはメインプロセスが終了すると停止します。
    # Devcontainerがアタッチする前にコンテナが死なないよう、常時生存するコマンドを指定します。
    command: /bin/sh -c "while sleep 1000; do :; done"
    
    # ネットワークとポート
    ports:
      - "8080:8080"
    
    # 環境変数
    environment:
      - TZ=Asia/Tokyo
    
    # 起動順序の制御（裏側のサービスが立ち上がってからメインが動くようにする）
    depends_on:
      - auxiliary_service_1

  # ==========================================
  # 周辺サービス（データベース、キャッシュ、別サーバー等）
  # ==========================================
  auxiliary_service_1:
    image: some-official-image:latest
    restart: always
    environment:
      - ENV_VAR_NAME=value
    ports:
      - "5432:5432"
    volumes:
      - volume_name:/data

# データの永続化ボリューム定義
volumes:
  volume_name:

```

### 💡 重要な設計ポイント

1. **マウントの向き (`volumes`)**: ホスト側（PC上）のプロジェクトルートを、コンテナ内の作業ディレクトリに双方向で共有（バインドマウント）させます。これにより、VS Codeで編集した内容がリアルタイムでコンテナ側に反映されます。
2. **コンテナ間通信**: Docker Composeはデフォルトで同一ネットワークに属するため、コンテナ同士はサービス名（例: `auxiliary_service_1`）をホスト名として直接通信（名前解決）できます。

---

## 3. `Dockerfile` の詳細な解説（網羅的仕様）

メインコンテナ（`main_service`）の土台となるベースイメージを選定し、開発に必要なツールや環境を構築するための手順書です。

```dockerfile
# 1. ベースイメージの指定
# （マイクロソフト公式が提供するdevcontainer用ベースイメージを利用すると、VS Codeの裏方サーバーが自動導入されるため確実です）
FROM mcr.microsoft.com/devcontainers/base:ubuntu

# 2. 環境変数の非対話モード設定（パッケージインストールの際に入力を求められて止まるのを防ぎます）
ENV DEBIAN_FRONTEND=noninteractive

# 3. システムパッケージのアップデートと追加ツールのインストール
RUN apt-get update && apt-get -y install --no-install-recommends \
    curl \
    git \
    build-essential \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# 4. 作業ディレクトリの作成と指定
WORKDIR /path/to/workspace

# 5. （必要に応じて）追加のユーザー作成や権限変更などをここに記述

```

---

## 全体の動作フロー（何が起きているか）

1. **ビルドフェーズ**: Docker Composeが `Dockerfile` を読み込み、メインコンテナのイメージを作り上げる。
2. **起動フェーズ**: `docker-compose.yml` に定義されたすべてのコンテナ（メイン＋周辺サービスA〜Eの一部）が一斉に立ち上がる。このとき `command` によってメインコンテナが生存し続ける。
3. **アタッチフェーズ**: VS Codeが起動し、`devcontainer.json` の `service`（メインコンテナ）の内部へリモート接続（SSHのような仕組み）する。
4. **開発開始**: ユーザーはコンテナ内部の環境を自分のPCのように使いながら、裏側の他のコンテナとも連携したシステム開発を行える。