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
