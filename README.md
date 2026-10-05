# training_devcontainer_html

修正後の正しいファイル構成（`docker-compose.yml` を使用する形）に合わせて、最初から環境を構築する手順と、HTML以外の設定ファイルについての解説を説明するよ。

### 対象のファイル構成

```text
training_devcontainer_html/
├── .devcontainer/
│   ├── devcontainer.json
│   ├── docker-compose.yml
│   └── Dockerfile
└── app/
    └── index.html

```

---

### ステップ1：ファイルとフォルダの作成（CUIコマンド）

ターミナルを開き、既存のリポジトリディレクトリ（`training_devcontainer_html`）に移動した状態で、以下のコマンドを順番に実行する。

1. **フォルダの作成**
```bash
mkdir -p .devcontainer app

```


2. **`Dockerfile` の作成**
```bash
cat << 'EOF' > .devcontainer/Dockerfile
FROM mcr.microsoft.com/devcontainers/javascript-node:18
EOF

```


* **検証方法**: `cat .devcontainer/Dockerfile` を実行し、内容が正しく表示されること。


3. **`docker-compose.yml` の作成**
```bash
cat << 'EOF' > .devcontainer/docker-compose.yml
version: '3.8'

services:
  web:
    build:
      context: .
      dockerfile: Dockerfile
    volumes:
      - ../..:/workspace:cached
    command: sleep infinity
EOF

```


* **検証方法**: `cat .devcontainer/docker-compose.yml` を実行し、内容が正しく表示されること。


4. **`devcontainer.json` の作成**
```bash
cat << 'EOF' > .devcontainer/devcontainer.json
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
EOF

```


* **検証方法**: `cat .devcontainer/devcontainer.json` を実行し、内容が正しく表示されること。


5. **`app/index.html` の作成**
```bash
cat << 'EOF' > app/index.html
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
EOF

```


* **検証方法**: ブラウザ等で確認する前に `cat app/index.html` で中身を確認できること。



---

### ステップ2：HTML以外の設定ファイルの解説

今回作成した devcontainer 周りの3つのファイルが、それぞれどのような役割を持っているのかを解説するよ。

#### 1. `Dockerfile`

* **役割**: コンテナの「中身（OSやインストールするソフトウェア）」の土台を作る設計図。
* **解説**: Microsoft公式の Node.js 18 が入った開発用イメージ（`[mcr.microsoft.com/devcontainers/javascript-node:18](https://mcr.microsoft.com/devcontainers/javascript-node:18)`）をベースに指定している。将来的に追加のパッケージを入れたい場合は、ここに `RUN` 命令などを書き足していく。

#### 2. `docker-compose.yml`

* **役割**: コンテナの「起動方法」や「パソコン（ホスト）との連携ルール」をまとめる司令塔。
* **解説**:
* `services.web`: `web` という名前のコンテナを定義する。
* `build`: 同じフォルダ内にある `Dockerfile` を使ってコンテナをビルドする指定。
* `volumes`: 手元のパソコンのプロジェクト全体を、コンテナ内の `/workspace` という場所に共有（マウント）する。これにより、コンテナの外でファイルを編集しても即座にコンテナ内に反映される。
* `command: sleep infinity`: コンテナが起動したあと、何もせずに即終了してしまうのを防ぐため、無限に起動状態を維持させるコマンド。



#### 3. `devcontainer.json`

* **役割**: VSCodeに「このプロジェクトをどのコンテナ環境でどう開くか」を伝える設定ファイル。
* **解説**:
* `"dockerComposeFile": "docker-compose.yml"`: どの Docker Compose ファイルを使うかを指定。
* `"service": "web"`: 複数のコンテナがある中で、どれをVSCodeのメイン作業場としてアタッチ（接続）するかを指定。
* `"workspaceFolder": "/workspace"`: コンテナに接続したとき、最初に開くフォルダの場所を指定。
* `"customizations.vscode.extensions"`: コンテナ起動時に自動でインストールしてほしいVSCodeの拡張機能（今回は Live Server）を指定。
* `"forwardPorts": [8080]`: コンテナ内の `8080` 番ポートを手元のパソコンからアクセスできるように転送する設定。



---

### ステップ3：起動と動作確認

1. 以下のコマンドを実行してVSCodeを起動する：
```bash
code .

```


2. 初回はDockerのビルドが走るため少し待つ。コンテナが立ち上がり、VSCode内のターミナルが開いたら、以下のコマンドを実行してWebサーバーを起動する：
```bash
http-server app -p 8080

```


* **確認方法**: ターミナルに `Available on: [http://127.0.0.1:8080](http://127.0.0.1:8080)` のようなログが表示されること。


3. ブラウザで `http://localhost:8080` にアクセスし、「Hello World!」と表示されることを確認する。
* **確認方法**: 画面に「Hello World!」が表示されること。



ここまで進めてみて、エラーや躓くポイントがなかったか教えてね！