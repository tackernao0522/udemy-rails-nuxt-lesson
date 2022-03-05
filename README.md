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
