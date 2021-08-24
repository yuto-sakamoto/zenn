---
title: "docker + docker-compose + railsでさくっと環境構築できるようになる"
emoji: "✌️"
type: "tech"
topics: ["docker", "docker-compose", "rails"]
published: true
---

# 前提条件

- [docker desktopがインストールされていること](https://www.docker.com/products/docker-desktop)  
**※ インストールの手順はこの記事では解説しません。**

- Railsプロジェクトが存在すること

# 事前確認
- version確認

  ターミナルで以下のコマンドを入力。  
  1. `docker version`  
![](https://storage.googleapis.com/zenn-user-upload/088c67aa2dcbc90e88314663.png)

  2. `docker-compose version`  
![](https://storage.googleapis.com/zenn-user-upload/8b089d1899b95f51e3230736.png)  
バージョンは違っていてもエラーが出ていなければ大丈夫です!!  

  3. `docker run --rm hello-world`を実行後、下図のように実行できればdockerの順位は完了です^^  

# docker環境の構築

まずは、Railsプロジェクトがあるディレクトリに移動します。
`cd xxx/xxx`

## Dockerfileの作成
`mkdir -p docker/rails`

`cd docker/rails`

`touch Dockerfile`

## Dockerfileの編集

```Dockerfile
FROM ruby:2.6.5 ← ご自身のRubyのバージョンに合わせてください
RUN apt-get update -qq && \
  apt-get install -y build-essential \
  libpq-dev \
  nodejs

ENV APP_ROOT /example ← ご自身のアプリ名にすると良い(任意)

USER example_user

# Set working directory as APP_ROOT
WORKDIR $APP_ROOT

# Add Gemfile
COPY Gemfile* .

# Install Gemfile's bundle
RUN bundle install
COPY . .
```

# docker-compose環境の構築

Railsプロジェクトのルートフォルダに移動
`cd xxx/xxx`

## docker-compose.ymlの作成
`touch docker-compose.yml`


## docker-compose.ymlファイルの編集

```docker-compose.yml

version: "3.8"
services:
  app:
    build:
      context: .
      dockerfile: ./docker/rails/Dockerfile
    command: rails s -b 0.0.0.0
    tty: true
    stdin_open: true
    depends_on:
      - db
    volumes:
      - .:/example
    ports:
      - "3001:3000"
  db:
    build:
      context: ./docker/db
      dockerfile: Dockerfile_db
    environment:
      MYSQL_ROOT_PASSWORD: admin
    ports:
      - "3306:3306"
```

- docker imageの作成&コンテナ起動  
`docker-compose up`

- コンテナの停止
`docker-compose down`



こちらでローカルで修正したファイルもコンテナと同期が取れているので、
じゃんじゃん開発していってください^^

ざっくり10分で開発環境の構築をしたので細かい話は、需要がありそうなら書いていこうと思います (^^;)