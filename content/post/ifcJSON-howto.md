+++
author = "ktaroabobon"
title = "ifcJSONの使い方"
date = "2022-06-18"
description = "IFCファイルをjsonに変換するコンバータの一つであるifcJSONの使い方の解説"
featured = true
tags = [
"ifcJSON",
]
categories = [
"ifc",
"json",
]
series = ["ifc2json"]
thumbnail = "images/building.png"
draft = false

+++

本記事ではifcJSONについての使い方を解説しています。

## 概要

- [github](https://github.com/buildingSMART/ifcJSON)
- ifcファイルをjson形式に変換する
- ifcを保守運用しているBuildingSMARTが作成している
- pythonで書かれている
- ifcの対応バージョンはifc2x3, ifc4.0, ifc4.3
- このコンバータの目的や開発戦略についての解説は[こちら](https://ktaroabobon.github.io/post/ifcjson-about/)から

## 実行環境・事前準備

本記事の内容はDockerを用いて環境構築を行いました。  
Dockerのインストールについては[こちら](https://www.docker.com/products/docker-desktop/)から

## 使い方

### ダウンロード

まずは、手元のpcにgithubのリポジトリからダウンロードしましょう。

```bash
git clone https://github.com/ktaroabobon/ifcJSON.git
```

構成は以下のようになっています

```bash
.
├── Documentation
│   └── README.md
├── LICENSE
├── README.md
├── Samples
├── Schema
├── file_converters
│   ├── README.md
│   ├── dataset
│   │   ├── input
│   │   └── output
│   ├── ifc2json.py
│   ├── ifcjson
│   ├── json2ifc.py
│   └── samples.py
└── schema_converters
```

それぞれのディレクトリの内容は以下の通りです。

- Documentation: ifcJSONについてのドキュメント
- LICENSE: ライセンスについて
- README.md: README
- Samples: jsonに変換したサンプルデータ等が格納されている
- Schema: ifcJSONのスキーマについて書かれている
- file_converters: 変換する際の実行ファイルが格納されている
- schema_converters: スキーマを変換する実行ファイル

今回主に使用するのはfile_converters内のファイルです。

### 環境構築

本記事ではDockerを用いて環境構築を行います。  
実行ファイルはfile_converters内のものを用いて行うため、Dockerfile等のファイルをfile_converters内で作成します。

はじめに、依存パッケージの管理を行います。今回はpoetryを用いて行っています。
詳しい方法は[公式ドキュメント](https://cocoatomo.github.io/poetry-ja/)等を参照してください。

また、使用するパッケージとしては以下の通りです。実行においてifcopenshellが必要であるためそのパッケージを追加するのを忘れないようにしてください。

> pandas  
> python-ifcopenshell

poetryを実行後のファイル構成

```bash
.
├── ⚫️poetry.lock
├── ⚫️pyproject.toml
├── README.md
├── dataset
│   ├── input
│   └── output
├── ifc2json.py
├── ifcjson
│   ├── __init__.py
│   ├── common.py
│   ├── ifc2json4.py
│   ├── ifc2json5a.py
│   ├── mesh.py
│   ├── reader.py
│   └── to_ifcopenshell.py
├── json2ifc.py
└── samples.py
```

続いて、file_converters内にDockerfileとdocker-compose.ymlファイルを作成します。

```bash
.
├── ⚫️Dockerfile
├── ⚫️docker-compose.yml
├── poetry.lock
├── pyproject.toml├── README.md
├── dataset
│   ├── input
│   └── output
├── ifc2json.py
├── ifcjson
│   ├── __init__.py
│   ├── common.py
│   ├── ifc2json4.py
│   ├── ifc2json5a.py
│   ├── mesh.py
│   ├── reader.py
│   └── to_ifcopenshell.py
├── json2ifc.py
└── samples.py
```

docker-compose.ymlには以下の内容を記述します。

```yaml
version: '3.8'
services:
  ifcjson:
    container_name: ifcjson
    build:
      context: .
      dockerfile: ./Dockerfile
    ports:
      - "8000:8000"
    tty: true
    volumes:
      - .:/code/
      - ${PIP_CACHE_DIR_DETECTION:-cache-detection}:/root/.cache

volumes:
  cache-detection:
```

Dockerfileには以下の内容を記述します。

```dockerfile
FROM python:3.9

ENV APP_PATH=/code \
    PYTHONPATH=.
#　開発物のソースコードはcodeデイレクトリ下に配置する

WORKDIR $APP_PATH

RUN apt-get update && \
    apt-get upgrade -y && \
    pip install poetry
# コンテナのセットアップ

COPY . .

RUN poetry install
#　必要なパッケージ等をインストールする

EXPOSE 8000
```

全て完了したら、環境構築をしていきます。

```bash
cd file_converters

docker-compose up -d
```

これで、Dockerを用いた環境構築は終了です。

### 実行

先ほど立ち上げた環境内でifcファイルを変換していきます。
手元にifcのサンプルファイルがない方は、[こちら](https://github.com/IFCjs/hello-world/tree/main/IFC)等からサンプルモデルをダンロードして使用してみましょう。

まず、file_converters/dataset/inputにifcファイルを格納します。

続いて、Docker環境においてifc2json.pyファイルを実行します。下記の実行コマンド内のパラメータは環境に合わせたを設定してください。

```bash
docker exec ifcjson poetry run python ifc2json.py -i dataset/input/test.ifc -o dataset/output/test.json --compact
```

> ifc2json
> > -h, --help: このパッケージのヘルプを表示します。
> > -i, I: inputファイルのパスを指定します。  
> > -o, O: outputファイルのパスを指定します。  
> > -v, V: ifcJSONのバージョンを指定します。デフォルトでは4が指定されています。  
> > -c, --compact: jsonに変換される際にインデントをなくし、type情報を格納せずに出力する。  
> > -n, --no_inverse: オブジェクトの逆参照を記載します。デフォルトではオンです。  
> > -e, --empty_properties: 値がない属性情報も書き出します。デフォルトではオフです。  
> > -w, --no_ownerhistory:   
> > ifcJSONのバージョン4を指定している際IfcOwnerHistoryの情報を消去します。デフォルトではオフです。【警告】この設定をオンにするとIFC
> > Schemaに則った変換ではなくなります。  
> > -g GEOMETRY, --geometry GEOMETRY:   
> > ジオメトリの書き出し設定を"none", "tessellate", "unchanged"
> > から指定できます。デフォルトではunchangedが指定されています。【警告】この設定をnoneにするとIFC Schemaに則った変換ではなくなる可能性があります。
>
> json2ifc
> > -i, I: inputファイルのパスを指定します。  
> > -o, O: outputファイルのパスを指定します。
>

実行が成功すると指定した場所にjsonファイルが格納され、処理時間が出力されます。

```bash
Conversion took  65.31001250000008  seconds
```

## その他

その他の詳細な内容等については、[公式ドキュメント](https://github.com/buildingSMART/ifcJSON/tree/master/Documentation)
及び[公式github](https://github.com/buildingSMART/ifcJSON)でのpull requestなどを用いて閲覧・検索してみてください。



