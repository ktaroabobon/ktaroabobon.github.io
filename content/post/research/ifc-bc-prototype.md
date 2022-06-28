+++
author = "ktaroabobon"
title = "python-ifcopenshellのインストール方法"
date = "2022-06-18"
description = "ifcopenshellをpythonで触る際のインストール方法のメモ"
featured = true
tags = [
"ifcopenshell",
"ifc.js"
]
categories = [
"ifc",
"ifcopenshell",
"python"
]
thumbnail = "images/building.png"
draft = false

+++

# 背景

- 以前、「IFCの属性情報を用いた建築確認自動化の可能性に関する研究」という題で論文を執筆した。
  - [slide share](https://www.slideshare.net/KeitaroWatanabe2/20220225-251744026)
- IFC viewerの機能を中心としたフレームワークである[ifc.js](https://ifcjs.github.io/info/ja/)を知り、建築確認機能を持つIFC viewerを作成できると考えた。

# Webサービス

## 初期構成

まず最初に考えた構成図は次図

![ifc bc first architecture](/static/images/ifc-bc-first.png)

viewerにifc.jsを使用し、建築確認機能をAPI化させる。APIはAWS Lambdaを使用することを想定する。

しかし、ここで課題となるのは**viewerとAPIの間でIFCのデータをどのような形式で送受信させるか**ということである。

そこでIFCをJSON形式に変換する方法などを視野に入れ、検討を行った。

## IFCデータのやり取りの方法

前述した通り、IFCデータをviewerとAPIの間においてどのような形式でやりとりを行うのかという課題がある。今回はifc-spfデータ（以下、ifcspf）またはIFCをjsonに変換したデータ（以下、ifcjson）を対象とした。なお、jsonに変換する際は以前記事にした[ifcJSON](https://ktaroabobon.github.io/post/architecture/ifcjson-about/)を用いて行った。

1. ifcspfを用いる方法：ifcspfを用いる場合は[ifcopenshell](http://ifcopenshell.org/)を用いてifcデータを読み込み解析を行う
   1. 利点
      1. ifcのデータ構造をあまり意識せずに処理できる
   2. 欠点
      1. 処理の環境がifcopenshellに依存する
2. ifcjsonを用いる方法
   1. 利点
      1. 処理の環境がifcopenshellなどifcのデータに依存せずに処理できる
      2. 処理内においてifcをデータベースとして扱うことができる
   2. 欠点
      1. データ構造を一つ一つ確認してプログラムの設計やAPIの設計を行わなければならない

それぞれについては以上のような利点と欠点が考えられた。

また、データのサイズについても考慮した。それぞれのデータのサイズとしては次表の通りとなっている。

   種類 | サイズ（MB）
--------|------
ifcspf | 約7.8MB
ifcjson| 約65MB

以上より、ifcspfとifcjsonでは主にサイズの観点からifcspfを使用した方が良いと判断した。

## プロトタイプの作成

### 概要

プロトタイプに関しては、ifc.jsはまだチュートリアルを終えた程度であったので代替としてjupyter notebookを使用した。プロトタイプの構成図としては次図である。

![ifc bc prototype architecture](/static/images/ifc-bc-prototype.png)

処理概要としては、まずifcspfをtextと読み込み圧縮バイナリ化した後、APIに送信を行う。この時、ifcデータの特定の部材のみなどではなく全てのデータを送信する。解析API側では受信したデータを解凍し解析を行い結果を返す。

### IFCデータの圧縮

前項の内容から、ifcデータに関してはifcspfを用いて行う。しかし、データサイズはまだ大きいため圧縮が必要である。処理についてはpythonを使用して作成したため、標準パッケージを比較した結果lzmaを用いて行った。圧縮後のデータとしては約1.2MBまで落とすことができた。

### APIインターフェース

### input(body)

```json
{
	'ifc': xxxx, // ifcのデータを圧縮しバイナリ化したもの
	'metadata': { 
		// レスポンスにも保持される内容
		'target': 'test.ifc',
	}
}
```

### output(body)

#### success

```json
{
	'message': 'Success', // メッセージ
	'result': {
		'conformityElements': [ // 適合部材
			{
				'globalId': 'yyyy',
				'type': 'IfcWall',
				'name': '標準壁',
				'propertySetNum': 5, // このオブジェクトに含まれるプロパティセットの数
				'propertySet': [
					{
						'name': 'FireRating',
						'value': 60,
					}, ...
				],
			}, ...
		],
		'notConformityElements': [ // 不適合部材
		...
		],
		'exceptionElements': [ // 例外部材
		...
		]
	},
	'metadata': { 
		'target': 'test.ifc',
	}
}
```

#### failed

```json
{
	'message': 'Failed', // メッセージ
	'error': 'ValidationError', // エラーメッセージ
	'metadata': { 
		'target': 'test.ifc',
	}
}
```

### 考察

プロトタイプを開発をしたことでわかったことを最後にまとめる。

#### データベースを介した構成にするべき

プロトタイプではIFCデータを圧縮し解析処理に送信した。しかしこの方法は、ifcのデータに依存するため