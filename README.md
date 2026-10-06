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
