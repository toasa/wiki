https://gemini.google.com/app/7fc2d4af025c5569

---

## 概要

以下の環境で ユーザ認証付きの ToDo ウェブアプリを作る

- Linux
- Nginx
- PostgreSQL
- TypeScript

ステップバイステップでインクリメンタルに開発を進める。

全体像となるアーキテクチャは以下：

- Client (Browser): HTML/JS (UI)
- Reverse Proxy (Nginx): クライアントからのリクエストを受け取り、バックエンドへ流す
- App Server (Node.js + Express): TypeScriptで記述。ビジネスロジックを処理。
- Database (PostgreSQL): データの永続化。

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

```yaml
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

```json
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

```sql
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

## Step3: ToDo APIの実装 (CRUD)

`src/index.ts` を以下のように修正する：

```ts
import express, { Request, Response } from 'express';
import { Pool } from 'pg';

const app = express();
const port = 3000;

// DB接続設定
const pool = new Pool({
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASS,
  database: process.env.DB_NAME,
  port: 5432,
});

// JSONボディを受け取るためのミドルウェア
app.use(express.json());

// TypeScript用の型定義 (Cのstructに相当)
interface Todo {
  id: number;
  title: string;
  is_completed: boolean;
  created_at: Date;
}

// 1. ToDo一覧の取得 (READ)
app.get('/todos', async (req: Request, res: Response) => {
  try {
    const result = await pool.query<Todo>('SELECT * FROM todos ORDER BY id ASC');
    res.json(result.rows);
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Internal Server Error' });
  }
});

// 2. ToDoの作成 (CREATE)
app.post('/todos', async (req: Request, res: Response) => {
  try {
    const { title } = req.body; // リクエストボディから title を取り出す
    if (!title) {
       res.status(400).json({ error: 'Title is required' });
       return;
    }
    
    // $1 はパラメータプレースホルダ。安全に値を埋め込みます。
    // RETURNING * は、INSERTした結果（自動生成されたid含む）を返すPostgreSQLの機能
    const result = await pool.query<Todo>(
      'INSERT INTO todos (title) VALUES ($1) RETURNING *',
      [title]
    );
    res.status(201).json(result.rows[0]);
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Internal Server Error' });
  }
});

// 3. ToDoの更新 (UPDATE) - 完了状態の切り替えなどに使用
//   :id はURLパラメータ（例: /todos/1）
app.put('/todos/:id', async (req: Request, res: Response) => {
  try {
    const id = parseInt(req.params.id);
    const { title, is_completed } = req.body;

    const result = await pool.query<Todo>(
      'UPDATE todos SET title = $1, is_completed = $2 WHERE id = $3 RETURNING *',
      [title, is_completed, id]
    );

    if (result.rowCount === 0) {
      res.status(404).json({ error: 'Todo not found' });
      return;
    }
    res.json(result.rows[0]);
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Internal Server Error' });
  }
});

// 4. ToDoの削除 (DELETE)
app.delete('/todos/:id', async (req: Request, res: Response) => {
  try {
    const id = parseInt(req.params.id);
    const result = await pool.query('DELETE FROM todos WHERE id = $1', [id]);

    if (result.rowCount === 0) {
      res.status(404).json({ error: 'Todo not found' });
      return;
    }
    res.status(204).send(); // 204 No Content (成功したが返す中身はない)
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Internal Server Error' });
  }
});

app.listen(port, () => {
  console.log(`Server is running at http://localhost:${port}`);
});
```

### 動作確認

コードを保存すると `nodemon` がサーバーを再起動する。
別のターミナルから `curl` コマンドで API を叩く。

#### ToDo を作成 (Post)：

```
# -X POST: メソッド指定
# -H: ヘッダ指定 (JSONを送ると宣言)
# -d: データ本体
$ curl -X POST http://localhost:3000/todos \
     -H "Content-Type: application/json" \
     -d '{"title": "Learn TypeScript"}'

# もう一つ追加
$ curl -X POST http://localhost:3000/todos \
     -H "Content-Type: application/json" \
     -d '{"title": "Build a Web App"}'
```

成功すると {"id":1, "title":"Learn TypeScript", ...} のようなJSONが返る。

#### 一覧を取得 (GET)：

```
$ curl http://localhost:3000/todos
```

配列 [ ... ] に入ったデータが返ってくればOK。

#### ToDo を更新 (PUT)

IDが 1 のタスクを「完了(true)」にし、タイトルも修正する：

```
$ curl -X PUT http://localhost:3000/todos/1 \
       -H "Content-Type: application/json" \
       -d '{"title": "Learn TypeScript & Node.js", "is_completed": true}'
```

#### ToDo を削除 (DELETE)

IDが 1 のタスクを削除する：

```
$ curl -X DELETE http://localhost:3000/todos/1
```

## Step4: ユーザ認証の追加

1. Usersテーブル を作成し、ToDoテーブルと紐付ける（外部キー制約）。
2. パスワードをハッシュ化して保存する。
3. ログイン時に JWT を発行する。
4. APIアクセス時に JWT を検証する ミドルウェア を実装する。
5. 「自分のToDo」だけが見えるようにSQLを修正する。

### データベースの変更

既存のデータがあると生合成が取れなくなるため、学習用の今回は一度データを空にする。
DBコンテナに入る：

```
$ docker compose exec db psql -U myuser -d todo_db
```

以下のSQLを実行する：

```sql
-- 既存データをクリア（依存関係があるため作り直す）
DROP TABLE IF EXISTS todos;
DROP TABLE IF EXISTS users;

-- 1. ユーザーテーブル作成
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username TEXT UNIQUE NOT NULL,
    password_hash TEXT NOT NULL
);

-- 2. ToDoテーブル再作成（user_idカラムを追加）
CREATE TABLE todos (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id), -- 外部キー
    title TEXT NOT NULL,
    is_completed BOOLEAN DEFAULT false,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

\q
```

### ライブラリのインストール

認証に必要な暗号化ライブラリを追加する：

```
# bcrypt: パスワードハッシュ化
# jsonwebtoken: トークン生成・検証
docker compose exec app npm install bcrypt jsonwebtoken
docker compose exec app npm install -D @types/bcrypt @types/jsonwebtoken
```

### ソースコードの実装

`src/index.ts` を以下のように修正する：

```ts
import express, { Request, Response, NextFunction } from 'express';
import { Pool } from 'pg';
import bcrypt from 'bcrypt';
import jwt from 'jsonwebtoken';

const app = express();
const port = 3000;
const JWT_SECRET = 'my-secret-key-change-this-in-production'; // 本来は環境変数から読むべき

// DB接続設定
const pool = new Pool({
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASS,
  database: process.env.DB_NAME,
  port: 5432,
});

app.use(express.json());

// --- 型定義 ---

// ExpressのRequest型を拡張して、user情報を運べるようにする
declare global {
  namespace Express {
    interface Request {
      user?: { id: number; username: string };
    }
  }
}

// --- 認証ミドルウェア ---

// APIを守るための「関所」。トークンがないリクエストはここで弾く。
const authenticateToken = (req: Request, res: Response, next: NextFunction) => {
  const authHeader = req.headers['authorization'];
  // Header形式: "Bearer <token>"
  const token = authHeader && authHeader.split(' ')[1];

  if (!token) {
    return res.status(401).json({ error: 'Access token required' });
  }

  jwt.verify(token, JWT_SECRET, (err, user) => {
    if (err) return res.status(403).json({ error: 'Invalid token' });
    
    // トークンが正しければ、解読したユーザー情報をreqオブジェクトに付加して次へ
    req.user = user as { id: number; username: string };
    next();
  });
};

// --- 認証系 API (公開) ---

// ユーザー登録
app.post('/register', async (req: Request, res: Response) => {
  try {
    const { username, password } = req.body;
    if (!username || !password) {
      return res.status(400).json({ error: 'Username and password required' });
    }

    // パスワードをハッシュ化 (ソルト処理含む)
    const hashedPassword = await bcrypt.hash(password, 10);

    const result = await pool.query(
      'INSERT INTO users (username, password_hash) VALUES ($1, $2) RETURNING id, username',
      [username, hashedPassword]
    );
    res.status(201).json(result.rows[0]);
  } catch (err: any) {
    // 一意制約違反(23505)のチェック
    if (err.code === '23505') {
      return res.status(409).json({ error: 'Username already exists' });
    }
    console.error(err);
    res.status(500).json({ error: 'Internal Server Error' });
  }
});

// ログイン
app.post('/login', async (req: Request, res: Response) => {
  try {
    const { username, password } = req.body;
    
    // ユーザー検索
    const result = await pool.query('SELECT * FROM users WHERE username = $1', [username]);
    if (result.rows.length === 0) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }
    const user = result.rows[0];

    // パスワード照合
    const validPassword = await bcrypt.compare(password, user.password_hash);
    if (!validPassword) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }

    // JWT発行 (署名)
    const token = jwt.sign({ id: user.id, username: user.username }, JWT_SECRET, { expiresIn: '1h' });
    res.json({ token });
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Internal Server Error' });
  }
});

// --- ToDo API (要認証) ---

// authenticateToken を通ったリクエストだけが以下の関数を実行できる

// 1. 一覧取得 (自分のToDoのみ)
app.get('/todos', authenticateToken, async (req: Request, res: Response) => {
  try {
    // req.user.id にはミドルウェアがセットしたIDが入っている
    const userId = req.user?.id;
    const result = await pool.query('SELECT * FROM todos WHERE user_id = $1 ORDER BY id ASC', [userId]);
    res.json(result.rows);
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Internal Server Error' });
  }
});

// 2. 作成 (自分のIDを紐付ける)
app.post('/todos', authenticateToken, async (req: Request, res: Response) => {
  try {
    const { title } = req.body;
    const userId = req.user?.id;

    const result = await pool.query(
      'INSERT INTO todos (title, user_id) VALUES ($1, $2) RETURNING *',
      [title, userId]
    );
    res.status(201).json(result.rows[0]);
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Internal Server Error' });
  }
});

// 3. 更新 (自分のToDoのみ)
app.put('/todos/:id', authenticateToken, async (req: Request, res: Response) => {
  try {
    const id = parseInt(req.params.id);
    const { title, is_completed } = req.body;
    const userId = req.user?.id;

    const result = await pool.query(
      'UPDATE todos SET title = $1, is_completed = $2 WHERE id = $3 AND user_id = $4 RETURNING *',
      [title, is_completed, id, userId]
    );

    if (result.rowCount === 0) {
      res.status(404).json({ error: 'Todo not found or not authorized' });
      return;
    }
    res.json(result.rows[0]);
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Internal Server Error' });
  }
});

// 4. 削除 (自分のToDoのみ)
app.delete('/todos/:id', authenticateToken, async (req: Request, res: Response) => {
  try {
    const id = parseInt(req.params.id);
    const userId = req.user?.id;

    const result = await pool.query('DELETE FROM todos WHERE id = $1 AND user_id = $2', [id, userId]);

    if (result.rowCount === 0) {
      res.status(404).json({ error: 'Todo not found or not authorized' });
      return;
    }
    res.status(204).send();
  } catch (err) {
    console.error(err);
    res.status(500).json({ error: 'Internal Server Error' });
  }
});

app.listen(port, () => {
  console.log(`Server is running at http://localhost:${port}`);
});
```

### 動作確認

`nodemon` による再起動を確認後、`curl` でテストする。

まず、ユーザー登録を行う：

```
$ curl -X POST http://localhost:3000/register \
       -H "Content-Type: application/json" \
       -d '{"username": "dev_user", "password": "secure_password"}'
```

次に、ログインしてトークンの取得を行う：

```
$ curl -X POST http://localhost:3000/login \
       -H "Content-Type: application/json" \
       -d '{"username": "dev_user", "password": "secure_password"}'
```

出力例: `{"token":"eyJhbGciOiJIUzI1NiIsInR5cCI6..."}` このトークン文字列（`eyJ...`の部分）をコピーする。

次に、トークンを使って ToDo を作成する。コピーしたトークンを `<TOKEN>` の部分に貼り付ける：

```
$ curl -X POST http://localhost:3000/todos \
       -H "Content-Type: application/json" \
       -H "Authorization: Bearer <TOKEN>" \
       -d '{"title": "Secret Task"}'
```

最後に、トークンなしでアクセスして拒否されることを確認する：

```
$ curl http://localhost:3000/todos
```

`{"error":"Access token required"}` となれば成功。

## Step 5: Nginx との連携 (デプロイ)

### フロントエンド（静的ファイル）の作成

プロジェクトのルートディレクトリに `public` フォルダを作成：

```
$ mkdir public
```

以下の内容の `public/index.html` 作成：

```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>ToDo App</title>
</head>
<body>
    <h1>My ToDo App</h1>
    
    <div id="login-section">
        <h2>Login</h2>
        <input type="text" id="username" placeholder="Username" value="dev_user">
        <input type="password" id="password" placeholder="Password" value="secure_password">
        <button onclick="login()">Login</button>
    </div>

    <div id="todo-section" style="display:none;">
        <h2>My Tasks</h2>
        <ul id="todo-list"></ul>
        <input type="text" id="new-todo" placeholder="New Task">
        <button onclick="addTodo()">Add</button>
    </div>

    <script>
        let token = '';

        // ログイン処理
        async function login() {
            const username = document.getElementById('username').value;
            const password = document.getElementById('password').value;

            // Nginx経由なので /api/login に投げる
            const res = await fetch('/api/login', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ username, password })
            });

            const data = await res.json();
            if (data.token) {
                token = data.token;
                document.getElementById('login-section').style.display = 'none';
                document.getElementById('todo-section').style.display = 'block';
                fetchTodos();
            } else {
                alert('Login Failed: ' + JSON.stringify(data));
            }
        }

        // 一覧取得
        async function fetchTodos() {
            const res = await fetch('/api/todos', {
                headers: { 'Authorization': `Bearer ${token}` }
            });
            const todos = await res.json();
            const list = document.getElementById('todo-list');
            list.innerHTML = '';
            todos.forEach(todo => {
                const li = document.createElement('li');
                li.textContent = `${todo.id}: ${todo.title} (${todo.is_completed ? 'Done' : 'Doing'})`;
                list.appendChild(li);
            });
        }

        // ToDo追加
        async function addTodo() {
            const title = document.getElementById('new-todo').value;
            await fetch('/api/todos', {
                method: 'POST',
                headers: { 
                    'Content-Type': 'application/json',
                    'Authorization': `Bearer ${token}`
                },
                body: JSON.stringify({ title })
            });
            document.getElementById('new-todo').value = '';
            fetchTodos();
        }
    </script>
</body>
</html>
```

### Nginx 設定ファイル作成

プロジェクトルートに `nginx` フォルダを作成し、`default.conf` を置く：

```
$ mkdir nginx
```

`nginx/default.conf`

```
server {
    listen 80;
    server_name localhost;

    # 1. 静的ファイルの設定
    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    # 2. APIへのリバースプロキシ設定
    # /api/ へのアクセスを Node.js コンテナへ転送する
    location /api/ {
        # proxy_pass の末尾に / を付けると、転送時に /api 部分を削除してくれる
        # 例: ブラウザ /api/todos -> Nginx -> Node.js /todos
        proxy_pass http://app:3000/;
        
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### compose.yaml の最終更新

Nginx コンテナを追加し、Node.js アプリを外部から隠蔽する：

```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"  # ホストの80番をNginxの80番へ
    volumes:
      - ./public:/usr/share/nginx/html  # フロントエンドのファイルをマウント
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf # 設定ファイルをマウント
    depends_on:
      - app

  app:
    image: node:24-alpine
    working_dir: /app
    # ports:        <-- 削除！ アプリの3000番は外部に公開しない
    #   - "3000:3000"
    volumes:
      - ./:/app
    environment:
      - DB_HOST=db
      - DB_USER=myuser
      - DB_PASS=mypassword
      - DB_NAME=todo_db
    command: sh -c "npm install && npm run dev"
    depends_on:
      - db

  db:
    image: postgres:15-alpine
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      - POSTGRES_USER=myuser
      - POSTGRES_PASSWORD=mypassword
      - POSTGRES_DB=todo_db
    # ports:        <-- DBも本番では隠すのが一般的だが、開発中は確認用に開けておいてもOK
    #   - "5432:5432"

volumes:
  postgres_data:
  ```

### 起動と動作確認

構成が変わったので、一度コンテナを作り直す：

```
$ sudo docker compose down
$ sudo docker compose up --build
```

ブラウザで `http://localost/` にアクセスして、タスクを追加。

## Step6 :

バックエンドには既に CRUD 全て（GET, POST, PUT, DELETE）の実装が終わっているため、フロントエンド (`public/index.html`) を修正して、それらを呼び出すように変更する。

`public/index.html` を以下のように修正する：

```html
<!DOCTYPE html>
<html lang="ja">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ToDo App</title>
    <style>
        /* 少しだけ見やすくするためのCSS */
        body {
            font-family: sans-serif;
            max-width: 600px;
            margin: 2rem auto;
            padding: 0 1rem;
        }

        .task-item {
            display: flex;
            align-items: center;
            justify-content: space-between;
            padding: 8px 0;
            border-bottom: 1px solid #eee;
        }

        .task-content {
            display: flex;
            align-items: center;
            gap: 10px;
        }

        .completed {
            text-decoration: line-through;
            color: #888;
        }

        button {
            cursor: pointer;
        }

        .btn-delete {
            color: white;
            background-color: #ff4444;
            border: none;
            padding: 4px 8px;
            border-radius: 4px;
        }
    </style>
</head>

<body>
    <h1>My ToDo App</h1>

    <div id="login-section">
        <h2>Login</h2>
        <input type="text" id="username" placeholder="Username" value="dev_user">
        <input type="password" id="password" placeholder="Password" value="secure_password">
        <button onclick="login()">Login</button>
    </div>

    <div id="todo-section" style="display:none;">
        <h2>My Tasks</h2>

        <div style="margin-bottom: 20px;">
            <input type="text" id="new-todo" placeholder="New Task">
            <button onclick="addTodo()">Add Task</button>
        </div>

        <ul id="todo-list" style="list-style: none; padding: 0;"></ul>

        <div style="margin-top: 20px;">
            <button onclick="logout()">Logout</button>
        </div>
    </div>

    <script>
        let token = localStorage.getItem('todo_token') || '';

        // ページ読み込み時にトークンがあれば自動でログイン状態にする
        if (token) {
            showTodoSection();
            fetchTodos();
        }

        function showTodoSection() {
            document.getElementById('login-section').style.display = 'none';
            document.getElementById('todo-section').style.display = 'block';
        }

        function logout() {
            token = '';
            localStorage.removeItem('todo_token');
            document.getElementById('login-section').style.display = 'block';
            document.getElementById('todo-section').style.display = 'none';
        }

        // --- 認証 (Login) ---
        async function login() {
            const username = document.getElementById('username').value;
            const password = document.getElementById('password').value;

            try {
                const res = await fetch('/api/login', {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ username, password })
                });

                const data = await res.json();
                if (res.ok && data.token) {
                    token = data.token;
                    localStorage.setItem('todo_token', token); // ブラウザに保存
                    showTodoSection();
                    fetchTodos();
                } else {
                    alert('Login Failed: ' + (data.error || 'Unknown error'));
                }
            } catch (e) {
                console.error(e);
                alert('Network Error');
            }
        }

        // --- 一覧取得 (READ: GET) ---
        async function fetchTodos() {
            if (!token) return;
            const res = await fetch('/api/todos', {
                headers: { 'Authorization': `Bearer ${token}` }
            });

            if (res.status === 401 || res.status === 403) {
                logout();
                return;
            }

            const todos = await res.json();
            renderList(todos);
        }

        // --- 新規作成 (CREATE: POST) ---
        async function addTodo() {
            const input = document.getElementById('new-todo');
            const title = input.value;
            if (!title) return;

            await fetch('/api/todos', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': `Bearer ${token}`
                },
                body: JSON.stringify({ title })
            });

            input.value = '';
            fetchTodos(); // リスト再取得
        }

        // --- 更新 (UPDATE: PUT) ---
        // 完了状態(checkbox)を切り替える
        async function toggleTodo(id, currentTitle, currentStatus) {
            // 反転したステータスを送信
            const newStatus = !currentStatus;

            await fetch(`/api/todos/${id}`, {
                method: 'PUT',
                headers: {
                    'Content-Type': 'application/json',
                    'Authorization': `Bearer ${token}`
                },
                // バックエンドの実装上、title も必須なので今の title をそのまま送る
                body: JSON.stringify({
                    title: currentTitle,
                    is_completed: newStatus
                })
            });

            fetchTodos(); // リスト再取得
        }

        // --- 削除 (DELETE: DELETE) ---
        async function deleteTodo(id) {
            if (!confirm('Are you sure?')) return;

            await fetch(`/api/todos/${id}`, {
                method: 'DELETE',
                headers: {
                    'Authorization': `Bearer ${token}`
                }
            });

            fetchTodos(); // リスト再取得
        }

        // リスト描画用関数
        function renderList(todos) {
            const list = document.getElementById('todo-list');
            list.innerHTML = '';

            todos.forEach(todo => {
                const li = document.createElement('li');
                li.className = 'task-item';

                // HTML生成
                // onchange で toggleTodo を呼ぶ (PUT)
                // onclick で deleteTodo を呼ぶ (DELETE)
                li.innerHTML = `
                    <div class="task-content">
                        <input type="checkbox"
                               ${todo.is_completed ? 'checked' : ''}
                               onchange="toggleTodo(${todo.id}, '${todo.title}', ${todo.is_completed})">
                        <span class="${todo.is_completed ? 'completed' : ''}">${todo.title}</span>
                    </div>
                    <button class="btn-delete" onclick="deleteTodo(${todo.id})">Delete</button>
                `;
                list.appendChild(li);
            });
        }
    </script>
</body>

</html>
```