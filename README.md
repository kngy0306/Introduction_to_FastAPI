# FastAPI入門
https://zenn.dev/sh0nk/books/537bb028709ab9

### 高速Webフレームワーク、FastAPIを利用したWebAPI作成

### 特徴
- リクエストとレスポンスのスキーマ定義に合わせて自動的にSwagger UIのドキュメントが生成される
- 上記のスキーマを明示的に定義することにより、型安全な開発が可能
- ASGIに対応しているので、非同期処理を行うことができ、高速

> FastAPIはリクエストとレスポンスのスキーマを定義することになります。これによって、フロントエンドエンジニアが実装の際に利用するドキュメントを簡単に自動生成でき、さらに実際にリクエストパラメータを変更してAPIの呼び出しを試すこともできるのです。
>
> スキーマを先に定義することによってフロントエンドとバックエンド間のインターフェイスを取り決め、それぞれの開発を同時にスタートする方式を、一般に 「スキーマ駆動開発（Schema-Driven Development）」 と呼びます。  

Flaskの影響を受けている。Flaskにはない特徴として、**Swagger UIのドキュメント自動生成**。**型安全**。**高速**。である。

### 環境構築
docker-compose.yml
```yml
version: '3'
services:
  demo-app:
    build: .
    volumes:
      - .dockervenv:/src/.venv
      - .:/src
    ports:
      - 8000:8000
```

Dockerfile
```Dockerfile
FROM python:3.9-buster
ENV PYTHONUNBUFFERED=1

WORKDIR /src

# pipを使ってpoetryをインストール
RUN pip install poetry

# poetryの定義ファイルをコピー (存在する場合)
COPY pyproject.toml* poetry.lock* ./

# poetryでライブラリをインストール (pyproject.tomlが既にある場合)
RUN poetry config virtualenvs.in-project true
RUN if [ -f pyproject.toml ]; then poetry install; fi

# uvicornのサーバーを立ち上げる
ENTRYPOINT ["poetry", "run", "uvicorn", "api.main:app", "--host", "0.0.0.0", "--reload"]
```

imageの作成
```sh
docker-compose build
```

### FastAPIのインストール
#### poetryによるPython環境のセットアップ
poetry ... パッケージ同士の依存関係を解決してくれるツール。  

最初は`poetry`に置いて依存関係を管理する`pyproject.toml`が存在しないため、`poetry`でFastAPIをインストールし、`pyproject.toml`を作成する。  

Authorのみ`n`を入力。それ以外Enter  
```sh
docker-compose run \
  --entrypoint "poetry init \
    --name demo-app \
    --dependency fastapi \
    --dependency uvicorn[standard]" \
  demo-app
```
Dockerコンテナ（`demo-app`）の中で、 `poetry init` コマンドを実行。引数として、 `fastapi` と、ASGIサーバーである `uvicorn` をインストールする依存パッケージとして指定。  

#### FastAPIのインストール
上記工程でFastAPIを依存パッケージに含んでいる、poetryの定義ファイルを作成した。
```sh
docker-compose run --entrypoint "poetry install" demo-app
```  

上記工程により、Dockerイメージを1から作った際は、FastAPIを含んだPython環境をイメージ内に含めることができる。  
新たなパッケージの追加時には、イメージの再ビルドだけでpyproject.tomlを含む全てのパッケージをインストール可。  
```sh
docker-compose build --no-cache
```  

### Hello, World!
#### `api`ディレクトリ、ファイルの作成。
api/__init__.py
```

```  
api/main.py
```py
from fastapi import FastAPI

app = FastAPI()


@app.get("/hello")
async def hello():
    return {"message": "hello world!"}
```   
`__init__.py`は、`api`ディレクトリがPythonモジュールであることを示す**空ファイル**。`main.py`にはコードを記載。

#### ファイル構成図
```sh
❯ tree
.
├── Dockerfile
├── README.md
├── app
│   ├── __init__.py
│   └── main.py
├── docker-compose.yml
├── poetry.lock
└── pyproject.toml

1 directory, 7 files
```  

#### API立ち上げ
```sh
docker-compose up
```  

アクセスする。  
http://localhost:8000/docs  

![スクリーンショット 2021-07-18 10 03 34](https://user-images.githubusercontent.com/57553474/126052547-43bca99b-8920-4cf6-af1d-d59954976df6.png)  
↑ **Swagger UI**  
Swagger UIは、OpneAPIという形式で定義される、RESTful APIを定義するフォーマットで記述された、APIドキュメント。  
実際にAPI動作を検証することができる。（対話的ドキュメント）  

#### コードの中身
```py
app = FastAPI()
```
↑ `app`はFastAPIのインスタンス。`main.py`内に`if __name__ == "__main__":`が無いが、実際にはunicornを通して`app`インスタンスが参照される。  
```py
@app.get("/hello")
```  
`@`で始まる↑を**デコレータ**という。関数を修飾し、新たな機能を追加する。  
FastAPIインスタンスに対する、デコレータで修飾された関数を、FastAPIでは**パスオペレーション関数**という。  
パスオペレーション関数の構成は以下2つ。  
- パス
- オペレーション
`/hello`のエンドポイントのことを**パス**という。  
 `get`の部分を**オペレーション**という。RESTにおけるHTTPメソッドを定義する。

### アプリケーションの概要
TODOアプリを作成する。  

#### REST API
REST APIでは、HTTPでやり取りする際に、URLですべての「リソース」を定義する。リソースを表す**エンドポイント**と、**HTTPメソッド**(GET/POST/PUT/DELETE)を組み合わせてAPI全体を構成する。  

- TODOリストを表示する
- TODOにタスクを追加する
- TODOのタスクの説明文を変更する
- TODOのタスク自体を削除する
- TODOタスクに「完了」フラグを立てる
- TODOタスクから「完了」フラグを外す

という機能を実装するために、REST APIでは、  
`HTTPメソッド` `エンドポイント`  
  
- GET /tasks
- POST /tasks
- PUT /tasks/{task_id}
- DELETE /tasks/{task_id}
- PUT /tasks/{task_id}/done
- DELETE /tasks/{task_id}/done

#### ディレクトリ構成
FastAPIではディレクトリ構成、ファイル分割を厳密に決めてはいない。このチュートリアルでは比較的細かくディレクトリを分割する。  

```sh
❯ tree
.
├── __init__.py
├── __pycache__
│   ├── __init__.cpython-39.pyc
│   └── main.cpython-39.pyc
├── cruds
│   └── __init__.py
├── main.py
├── models
│   └── __init__.py
├── routers
│   └── __init__.py
└── schemas
    └── __init__.py

5 directories, 8 files
```



