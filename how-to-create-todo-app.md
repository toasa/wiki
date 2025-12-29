## 概要

以下の環境で ユーザ認証付きの ToDo ウェブアプリを作る

- Linux
- Nginx
- PostgreSQL
- TypeScript

ステップバイステップでインクリメンタルに開発を進める。

## Step 1: 環境構築

1. Docker で Node.js 環境を構築
2. TypeScript環境: package.json の作成と、tsconfig.json の設定
3. Express (Webフレームワーク) のインストール
4. 「Hello World」を返すサーバーを書き、ホスト側のコンソールからアクセス

### プロジェクトディレクトリの作成

まず、Ubuntu 側で作業用ディレクトリを作る：

```
$ mkdir todo-app
$ cd todo-app
```

### Docker 環境の定義

Node.js の実行環境を Docker コンテナ内に作成する。
Docker イメージは `node:24-alpine` を使用する。

以下の `compose.yaml` を作成する：

```
services:
  app:
    image: node:24-alpine
    working_dir: /app
    ports:
      - "3000:3000"
    volumes:
      - ./:/app
    command: sh -c "npm install && npm run dev"
```

### 初期化とライブラリのインストール

今の時点では `package.json` がないため、一度コンテナの中に入って初期化する：

```
$ sudo docker compose run --rm --entrypoint sh app
```

コンテナ内のシェル (`/app`) に入ったら、以下を実行する：

```
# 1. package.json の生成
npm init -y

# 2. express (Webフレームワーク) のインストール
npm install express

# 3. 開発用ライブラリのインストール
# -D ：開発時のみ使うという意味
# typescript: コンパイラ本体
# @types/...: 型定義ファイル
# ts-node: TSをコンパイルせずに直接実行するツール (開発用)
# nodemon: ファイル変更を検知してサーバーを自動再起動するツール
npm install -D typescript @types/node @types/express ts-node nodemon
```

### TypeScript の設定

コンテナ内で、TypeScript の設定ファイル `tsconfig.json` を生成する：

```
npx tsc --init
```

`tsconfig.json` を以下のように修正：

```
{
  "compilerOptions": {
    "target": "es2020",
    "module": "commonjs",
    "rootDir": "src",
    "outDir": "dist",
    "strict": true,
    "esModuleInterop": true
  }
}
```

### 実行スクリプトの定義

`npm run dev` というコマンドでサーバーが立ち上げるように、
`package.json` の `scripts` の部分を書き換える：

```
  "scripts": {
    "dev": "nodemon src/index.ts"                        
  },   
```

### ソースコードの作成

ソースコードの置き場所を作成する：

```
$ mkdir src
```

`src/index.ts` を作成する：

```ts
import express, { Request, Response } from 'express';

const app = express();
const port = 3000;

app.get('/', (req: Request, res: Response) => {
  res.send('Hello Embedded Engineer! Linux + Docker + TS is working.');
});

app.listen(port, () => {
  console.log(`Server is running at http://localhost:${port}`);
});
```

### 起動確認

開いたコンテナ内のシェルは exit で抜け、ホスト上のターミナルから Docker Compose でサーバーを起動する：

```
$ docker compose up
```

ターミナルに `Server is running at http://localhost:3000` と表示されれば成功。

別ターミナルを開いて、アクセスを確認する：

```
$ curl http://localhost:3000
```

`Hello Embedded Engineer! Linux + Docker + TS is working.` が返ってくれば OK。

## Step2: データベース接続

1. compose.yaml に PostgreSQL のコンテナを追加する。
2. Node.js に DBドライバ (pg) をインストールする。
3. アプリから DB に接続し、現在時刻を取得して表示する。
4. ToDo を保存するテーブルを作成する。

### compose.yaml の修正

DBサーバーを立ち上げる定義を追加する。
ホスト側の compose.yaml を以下のように修正する：

```yaml
services:
  app:
    image: node:24-alpine
    working_dir: /app
    ports:
      - "3000:3000"
    volumes:
      - ./:/app
    # 環境変数を追加（アプリがDB接続情報を知るため）
    environment:
      - DB_HOST=db
      - DB_USER=myuser
      - DB_PASS=mypassword
      - DB_NAME=todo_db
    command: sh -c "npm install && npm run dev"
    depends_on:
      - db

  # ここを追加
  db:
    image: postgres:15-alpine
    # データの永続化（コンテナを消してもデータが残るように）
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_USER=myuser
      - POSTGRES_PASSWORD=mypassword
      - POSTGRES_DB=todo_db
    ports:
      - "5432:5432" # ホストからもDBを見れるように公開

volumes:
  postgres_data:
```

### コンテナの再起動とライブラリ追加

設定を変えたため、一度再起動して、DB用のライブラリを入れる：

```
# 設定反映のため再構築して起動（-d はバックグラウンド実行）
$ sudo docker compose up -d

# pg: PostgreSQL用クライアントライブラリ
$ sudo docker compose exec app npm install pg
$ sudo docker compose exec app npm install -D @types/pg
```

### ソースコードの修正

`src/index.ts` を以下のように修正する：

```ts
import express, { Request, Response } from 'express';
import { Pool } from 'pg'; // DBクライアントのクラス

const app = express();
const port = 3000;

// DB接続プールの設定
const pool = new Pool({
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASS,
  database: process.env.DB_NAME,
  port: 5432,
});

// JSONボディをパースするための設定（POSTリクエスト用）
app.use(express.json());

app.get('/', (req: Request, res: Response) => {
  res.send('Hello Embedded Engineer! Step 2: DB Connection.');
});

// DB接続テスト用API
app.get('/db-test', async (req: Request, res: Response) => {
  try {
    // クライアントを1つ借りて SQL を実行
    // 'SELECT NOW()' はDBサーバーの現在時刻を返すSQL
    const result = await pool.query('SELECT NOW()');
    
    // 結果をJSONで返す
    res.json({ 
      message: 'Database Connected!', 
      time: result.rows[0].now 
    });
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Database connection failed' });
  }
});

app.listen(port, () => {
  console.log(`Server is running at http://localhost:${port}`);
});
```

### 接続確認

ソースコードの保存後、nodemon が自動で再起動しているはず。以下のコマンドでテストする：

```
$ curl http://localhost:3000/db-test
```

以下のような JSON が返ってくれば、アプリとDBが繋がっている：

```
{"message":"Database Connected!","time":"2024-01-01T12:00:00.000Z"}
```

### テーブルの作成

ToDo データを保存するための DB テーブルを作る。
ここでは、DBコンテナに直接入って SQL コマンドを手打ちする。

```
$ sudo docker compose exec db psql -U myuser -d todo_db
```

プロンプトが `todo_db=#` に変わったら、以下のSQLを入力する：

```
-- テーブル作成
CREATE TABLE todos (
    id SERIAL PRIMARY KEY,      -- 自動連番ID
    title TEXT NOT NULL,        -- ToDoの内容
    is_completed BOOLEAN DEFAULT false, -- 完了フラグ
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP -- 作成日時
);

-- 確認用：テーブル一覧を表示
\dt

-- 終了して抜ける
\q
```

`\dt` で `todos` テーブルが表示されれば作成完了。