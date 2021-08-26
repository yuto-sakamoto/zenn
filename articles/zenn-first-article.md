---
title: "たった10分でdocker + docker-compose + railsの環境構築!!"
emoji: "✌️"
type: "tech"
topics: ["docker", "docker-compose", "rails"]
published: true
---

# 前提条件

- [docker desktopがインストールされていること](https://www.docker.com/products/docker-desktop)  
**※ インストールの手順はこの記事では解説しません。**
- Railsプロジェクトが存在すること
- DB(mysql)

# 事前確認

- version確認

  ターミナルで以下のコマンドを入力する。  
  1. `docker version`  
![](https://storage.googleapis.com/zenn-user-upload/088c67aa2dcbc90e88314663.png)

  2. `docker-compose version`  
![](https://storage.googleapis.com/zenn-user-upload/8b089d1899b95f51e3230736.png)  
バージョンは違っていてもエラーが出ていなければ大丈夫です!!  

  3. `docker run --rm hello-world`を実行後、下図のように実行できればdockerの準備はOKです!!  
![](https://storage.googleapis.com/zenn-user-upload/d478dbc78390b08bb581be8b.png)

# docker環境の構築

まずは、Railsプロジェクトがルートフォルダに移動する。(ご自身の環境に合わせてください)  
`cd hoge/fuga`

## Dockerfileの作成

1. `mkdir -p docker/rails`
2. `cd docker/rails`
3. `touch Dockerfile`

## Dockerfileの編集

```Dockerfile:Dockerfile
# ご自身のRubyのバージョンに合わせてください
FROM ruby:2.6.5
RUN apt-get update -qq && \
  apt-get install -y build-essential \
  libpq-dev \
  nodejs

# ご自身のアプリ名にすると良い(任意)
ENV APP_ROOT /example

USER example_user

# Set working directory as APP_ROOT
WORKDIR $APP_ROOT

# Add Gemfile
COPY Gemfile* ./

# Install Gemfile's bundle
RUN bundle install
COPY . ./
```

# docker-compose環境の構築

Railsプロジェクトのルートフォルダに移動する。(ご自身の環境に合わせてください)  
`cd hoge/fuga`

## docker-compose.ymlの作成

`touch docker-compose.yml`


## docker-compose.ymlファイルの編集

```yml:docker-compose.yml
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
      - tmp-data:/example/tmp
    ports:
      - "3001:3000"
  db:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: admin
    volumes:
      - mysql-data:/var/lib/mysql
    ports:
      - "3306:3306"

volumes:
  mysql-data:
  tmp-data:
```

## database.yml修正

`host`、`password`は`docker-compose.yml`でに記載しているサービス名`db`と  
環境変数に設定している`MYSQL_ROOT_PASSWORD`の値を設定する

```yml:database.yml
default: &default
  adapter: mysql2
  encoding: utf8
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  username: root
  password: admin
  host: db
```

# 開発環境の構築 

1. DB作成  
`docker-compose run --rm app rails db:setup`

2. docker imageの作成&コンテナ起動  
`docker-compose up`

3. コンテナの停止  
`docker-compose down`

4. 確認  
ブラウザに`localhost:3001`を入力しRailsアプリの画面が開けばOKです。


ざっくり10分で開発環境の構築をしたので細かい話は、需要がありそうなら書いていこうと思います (^^;)
