## 概要

以下の環境で ユーザ認証付きの ToDo ウェブアプリを作る

- Linux
- Nginx
- PostgreSQL
- TypeScript

ステップバイステップでインクリメンタルに開発を進める。

## 環境構築

Node.js の実行環境を Docker コンテナ内に作成する。
Docker イメージは `node:24-alpine` を使用する。

`compose.yaml` を作成：

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

今の時点では `package.json` がないため、一度コンテナの中に入って初期化する：

```
$ sudo docker compose run --rm --entrypoint sh app
```

コンテナ内のシェル (`/app`) で、以下を実行する：

```
# 1. package.json の生成
npm init -y

# 2. express (Webフレームワーク) のインストール
npm install express

# 3. 開発用ライブラリのインストール
# -D ：開発時のみ使うという意味
# typescript: コンパイラ本体
# @types/...: 型定義ファイル (Cのヘッダファイル .h に相当)
# ts-node: TSをコンパイルせずに直接実行するツール (開発用)
# nodemon: ファイル変更を検知してサーバーを自動再起動するツール
npm install -D typescript @types/node @types/express ts-node nodemon
```

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

`npm run dev` というコマンドでサーバーが立ち上げるように、
`package.json` の `scripts` の部分を書き換える：

```
  "scripts": {
    "dev": "nodemon src/index.ts"                        
  },   
```

ソースコードの置き場所を作成する：

```
$ mkdir src
```

`src/index.ts` を作成する：

```
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

開いたコンテナ内のシェルは exit で抜け、ホスト上のターミナルから Docker Compose でサーバーを起動する：

```
$ docker compose up
```

ターミナルに `Server is running at http://localhost:3000` と表示されれば成功。
別ターミナルを開いて、アクセスを確認する：

```
$ curl http://localhost:3000
```

