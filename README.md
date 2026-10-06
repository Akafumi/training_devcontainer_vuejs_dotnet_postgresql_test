# training_devcontainer_vuejs_dotnet_postgresql_test

- `db`(PostgreSQL)
- `backend`(C#.NET)
- `frontend`（Vue.js）

この3つの独立したサービス(コンテナ)に分けた構成．

---

## ディレクトリ構成

プロジェクトのルートに `.devcontainer`，`backend`，`frontend` を配置．

```text
my-project/
 ├── .devcontainer/
 │    ├── devcontainer.json
 │    └── docker-compose.yaml
 ├── backend/
 │    └── Dockerfile
 └── frontend/
      └── Dockerfile
```

---

## 設定ファイルの準備

### 1. 各フォルダ用の Dockerfile

それぞれのコンテナで必要なランタイムが異なるので専用の Dockerfile を用意する．

#### `backend/Dockerfile` (C#.NET用)

Microsoft公式の .NET 開発用コンテナイメージをベースにする．

```dockerfile
FROM mcr.microsoft.com/devcontainers/dotnet:8.0
```

#### `frontend/Dockerfile` (Vue.js用)

Microsoft公式の Node.js 開発用コンテナイメージをベースにする．

```dockerfile
FROM mcr.microsoft.com/devcontainers/javascript-node:20
```

---

### 2. `.devcontainer/docker-compose.yaml`

`db`，`backend`，`frontend` の3つのサービスを定義する．

```yaml
version: '3.8'

services:
  # 1. データベース (PostgreSQL)
  db:
    image: postgres:16-alpine
    restart: unless-stopped
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data

  # 2. バックエンド (.NET)
  backend:
    build:
      context: ../backend
      dockerfile: Dockerfile
    command: sleep infinity
    volumes:
      - ../backend:/workspaces/backend:cached
    ports:
      - "5000:5000"
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ConnectionStrings__DefaultConnection=Host=db;Database=mydb;Username=postgres;Password=password
    depends_on:
      - db

  # 3. フロントエンド (Vue.js)
  frontend:
    build:
      context: ../frontend
      dockerfile: Dockerfile
    command: sleep infinity
    volumes:
      - ../frontend:/workspaces/frontend:cached
    ports:
      - "5173:5173"
    stdin_open: true
    tty: true
    depends_on:
      - backend

volumes:
  pgdata:
```

バックエンドからのデータベース接続先(ホスト名)は，`localhost` ではなくサービス名の `db` になる．

---

### 3. `.devcontainer/devcontainer.json`

VS Codeでどのコンテナにアタッチ（接続）するかを指定する．
一般的にコードをガッツリ書く **`backend`** または **`frontend`** のどちらかを指定する．
今回はメインとなりやすい `backend` を指定する．

```json
{
  "name": "Fullstack DevContainer (Backend Main)",
  "dockerComposeFile": "docker-compose.yaml",
  "service": "backend",
  "workspaceFolder": "/workspaces/backend",

  "customizations": {
    "vscode": {
      "extensions": [
        "ms-dotnettools.csharp",
        "Vue.volar",
        "dbaeumer.vscode-eslint",
        "esbenp.prettier-vscode",
        "cweijan.vscode-postgresql-client2"
      ]
    }
  },

  "remoteUser": "vscode"
}
```

---

## 開発の始め方

1. **プロジェクトフォルダと設定ファイルの作成:**

プロジェクトのルートディレクトリに `backend`，`frontend`，`.devcontainer` フォルダを作成し，上記で紹介した5つのファイル（各 Dockerfile，`docker-compose.yaml`，`devcontainer.json`）をそれぞれ正しい位置に配置する（※最初は `backend` や `frontend` のフォルダ内が空っぽでも問題ない）．

* **検証方法**: ルートフォルダ内に `.devcontainer`，`backend`，`frontend` の各ディレクトリとファイルが正しく存在していることをファイルエクスプローラーで確認する．

2. **VS Codeでのプロジェクトオープン:**

VS Codeを起動し，メニューの **「ファイル」>「フォルダーを開く...」** から，ルートとなる `my-project` フォルダを選択して開く．

* **検証方法**: VS Codeのエクスプローラーに `backend`，`frontend`，`.devcontainer` のツリーが表示されていることを確認する．

3. **コンテナのビルドと起動（Reopen in Container）:**

VS Codeのコマンドパレットを開き（ショートカット: `F1` または `Ctrl+Shift+P` / `Cmd+Shift+P`），**「Dev Containers: Reopen in Container」** と入力して実行する．初回はDockerイメージのダウンロードやビルドが行われるため，数分程度かかる．

* **検証方法**: 画面右下のステータスバーに緑色のインジケーター（`Dev Container: Fullstack DevContainer (Backend Main)` など）が表示され，エラーなくコンテナが起動したことを確認する．

4. **コンテナ内環境の動作確認:**

VS Code内の統合ターミナル（ショートカット: `Ctrl + ``）を開き，`.NET` やファイルが正しく認識されているか確認するためのコマンドを実行する．

```bash
dotnet --version

```

* **検証方法**: ターミナルに `.NET` のバージョン番号（例: `8.0.x` など）が出力され，コマンドが正常に応答することを確認する．
