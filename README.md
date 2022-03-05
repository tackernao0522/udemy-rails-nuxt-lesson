セクション 3: Docker を使った Rails+Nuxt.js 環境構築

※ `$ mkdir udemy_demoapp_v2 && cd $_`<br>

## 07 Rails を動かす Dockerfile を作成する 1/2 〜作成編〜

### ハンズオン

- `$ mkdir {api,front}`(スペースは空けない)を実行<br>

* `api`ディレクトリに移動する<br>

* `$ touch {Dockerfile,Gemfile,Gemfile.lock}`を実行<br>

- `api/Gemfile`ファイルを編集<br>

```
source 'https://rubygems.org'
gem 'rails', '~> 6.0.0' # 6.0.0以上、6.1.0未満で最新 = 6.0.x
```

- `api/Dockerfile`ファイルを編集<br>

```
FROM ruby:2.7.1-alpine

ARG WORKDIR

ENV RUNTIME_PACKAGES="linux-headers libxml2-dev make gcc libc-dev nodejs tzdata postgresql-dev postgresql git" \
  DEV_PACKAGES="build-base curl-dev" \
  HOME=/${WORKDIR} \
  LANG=C.UTF-8 \
  TZ=Asia/Tokyo

# ENV test（このRUN命令は確認のためなので無くても良い）
RUN echo ${HOME}

WORKDIR ${HOME}

COPY Gemfile* ./

RUN apk update && \
  apk upgrade && \
  apk add --no-cache ${RUNTIME_PACKAGES} && \
  apk add --virtual build-dependencies --no-cache ${DEV_PACKAGES} && \
  bundle install -j4 && \
  apk del build-dependencies

COPY . .

CMD ["rails", "server", "-b", "0.0.0.0"]
```

## 08 Rails を動かす Dockerfile を作成する 2/2 〜解説編〜

- ベースイメージの Ruby バージョンを調べる手順(下記 URL を参照)<br>

* 参考: https://blog.cloud-acct.com/posts/u-rails-dockerfile/#%E3%83%99%E3%83%BC%E3%82%B9%E3%82%A4%E3%83%A1%E3%83%BC%E3%82%B8%E3%81%AEruby%E3%83%90%E3%83%BC%E3%82%B8%E3%83%A7%E3%83%B3%E3%82%92%E8%AA%BF%E3%81%B9%E3%82%8B%E6%89%8B%E9%A0%86 <br>

- `api/Dockerfile`を編集<br>

```
# ベースイメージを指定する
# FROM ベースイメージ:タグ(タグはなくてもよいが最新のものが指定されることになる)
FROM ruby:2.7.2-alpine

# Dockerfile内で使用する変数を定義
# appという値が入る
ARG WORKDIR

# 環境変数を定義(Dockerfile, コンテナ参照可能)
# Rails ENV["TZ"] => Asia/Tokyoが出力される
ENV RUNTIME_PACKAGES="linux-headers libxml2-dev make gcc libc-dev nodejs tzdata postgresql-dev postgresql git" \
  DEV_PACKAGES="build-base curl-dev" \
  # /app
  HOME=/${WORKDIR} \
  LANG=C.UTF-8 \
  TZ=Asia/Tokyo

# ベースイメージに対してコマンドを実行する
# ${HOME}, $HOME => /app
RUN echo ${HOME}

# Dockerfime内で指定した命令を実行する・・・RUN, COPY, ADD, ENTORYPOINT, CMD
# 作業ディレクトリを定義
# コンテナ/app/Railsアプリ
WORKDIR ${HOME}

# ホスト側の(PC)のファイルをコンテナにコピー
# COPY コピー元(ホスト) コピー先(コンテナ)
# Gemfile* ... Gemfileから始まるファイルを全指定(Gemfile, Gemfile, Gemfile.lock)
# コピー元(ホスト) ... Dockerfileがあるディレクトリ以下を指定(api) ../ NG
# コピー先(コンテナ) ... 絶対パス or 相対パス(./ ... 今いる(カレント)ディレクトリ)
COPY Gemfile* ./

    # apk ... Alpine Linuxのコマンド
    # apk update = パッケージの最新リストを取得
RUN apk update && \
  # apk upgrade = インストールパッケージを最新のものに
  apk upgrade && \
  # apk add = パッケージのインストールを実行
  # --no-cache = パッケージをキャッシュしない(Dokcerイメージを軽量化)
  apk add --no-cache ${RUNTIME_PACKAGES} && \
  # --virtual 名前(任意) = 仮想パッケージ
  apk add --virtual build-dependencies --no-cache ${DEV_PACKAGES} && \
  # Gemのインストールコマンド
  # -j4(jobs=4) = Gemインストールの高速化
  bundle install -j4 && \
  # パーケージを削除(Dokcerイメージを軽量化)
  apk del build-dependencies

# . ... Dockerfileがあるディレクトリ全てのファイル(サブディレクトリも含む)
COPY . ./

# コンテナ内で実行したいコマンドを定義
# -b ... バインド、プロセスを指定してip(0.0.0.0)アドレスに紐付け(バインド)する
CMD ["rails", "server", "-b", "0.0.0.0"]

# ホスト(PC)       | コンテナ
# ブラウザ(外部)    | Rails
```
