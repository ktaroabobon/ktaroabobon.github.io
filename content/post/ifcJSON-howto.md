+++
author = "ktaroabobon"
title = "ifcJSONの使い方"
date = "2022-06-16"
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
draft = true

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

まず、file_converters内にDockerfileとdocker-compose.ymlファイルを作成します。

```bash
.
├── ⚫️Dockerfile
├── ⚫️docker-compose.yml
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

docker-compose.ymlには以下の内容を記述します。

```yaml
version: '3.8'
services:
  ifc2json:
    container_name: ifc2json
    build:
      context: .
      dockerfile: ./Dockerfile
    volumes:
      - ./:/code
    ports:
      - "8000:8000"
    tty: true
```

Dockerfileには以下の内容を記述します。
実行においてifcopenshellが必要であるためその環境を構築します。

```dockerfile
FROM continuumio/miniconda3

ENV APP_PATH=/code \
    PYTHONPATH=.
#　開発物のソースコードはcodeデイレクトリ下に配置する

RUN conda create -n ifc2json python==3.8

SHELL ["conda", "run", "-n", "ifc2json", "/bin/bash", "-c"]
RUN conda install -c conda-forge -c oce -c dlr-sc -c ifcopenshell ifcopenshell
RUN conda install -c conda-forge poetry
RUN conda install -c conda-forge poetry-core

WORKDIR $APP_PATH

RUN apt-get update && \
    apt-get upgrade -y

EXPOSE 8000
```

ifcopenshellをDockerを用いて環境構築を行う詳しい内容は[こちらの記事](https://qiita.com/ktaroabobon/items/c276c76e4dc3442a041b)を参照してください。

続いて、依存パッケージの管理を行います。今回はpoetryを用いて行っています。
詳しい方法は[公式ドキュメント](https://cocoatomo.github.io/poetry-ja/)等を参照してください。

また、使用するパッケージとしては以下の通りです。

> pandas

全て完了したら、環境構築を実行していきます。

```bash
cd file_converters

docker-compose up -d

{poetryのコマンド}
```

これで、Dockerを用いた環境構築は終了です。

### 実行

先ほど立ち上げた環境内でifcファイルを変換していきます。
手元にifcのサンプルファイルがない方は、[こちら](https://github.com/IFCjs/hello-world/tree/main/IFC)等からサンプルモデルをダンロードして使用してみましょう。

まず、file_converters/dataset/inputにifcファイルを格納します。

続いて、Docker環境においてifc2json.pyファイルを実行します。

```bash
{実行コマンド}
```

また、実行する際のパラメータは以下の通りです。

> ifc2json
>> -i: inputファイルのパスを指定します。  
>> -o: outputファイルのパスを指定します。  
>> -v: ifcJSONのバージョンを指定します。デフォルトでは4が指定されています。  
>> -c: jsonに変換される際にインデントをなくし、type情報を格納せずに出力する。  
>> 


## その他

その他の詳細な内容等については、[公式ドキュメント](https://github.com/buildingSMART/ifcJSON/tree/master/Documentation)
及び[公式github](https://github.com/buildingSMART/ifcJSON)でのpull requestなどを用いて閲覧・検索してみてください。



