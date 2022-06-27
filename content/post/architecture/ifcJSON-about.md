+++
author = "ktaroabobon"
title = "ifcJSONについて"
date = "2022-05-08"
description = "IFCファイルをjsonに変換するコンバータの一つであるifcJSONの解説"
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

本記事ではifcJSONについての概要や開発戦略を記載おり、[公式ドキュメント](https://github.com/buildingSMART/ifcJSON/tree/master/Documentation)
を和訳したものが中心となっています。

## 概要

- [github](https://github.com/buildingSMART/ifcJSON)
- ifcを保守運用しているBuildingSMARTが作成している
- pythonで書かれている
- ifcの対応バージョンはifc2x3, ifc4.0, ifc4.3

## ifcJSONによって解決する問題

- 開発者の中にはEXPRESSやSTPのファイルを閲覧・使用した経験がほとんどない人もいるため、IFC-stpファイル等を用いて必要なデータを抽出するには労力が増加してしまう。
- IFCのデータは通常IFC-stpファイル等を介して交換されるが、これはほとんどの設計・建設プロジェクトや製品に当てはまる、膨大な量の製品が異なった定義や属性情報を持ち、それらが互いに依存して成立しているデータとは相性が悪い。

1つ目に関しては、IFCファイルを直接閲覧・編集を行うのは難しいということである。  
2つ目に関しては、ifcは建築物のデータベースの形式の1つであるのでそれをファイル形式でデータの保存・交換を行うことは適していないということである。

## 目的

- ifcのために推奨される単一のjson仕様を定義することを目的としている

## 開発戦略

ifcをjsonに変換する場合、考慮すべきいくつかの基準がある。

- 後方互換性
- ifcファイルとjson間での相互変換の容易性
- 可読性、直感性
- 柔軟性、拡張性
- コードとの親和性
- XML形式との並列性
- EXPRESSスキーマとの並列性
- 明確な参照構造（フラット構造やネスト構造）

一方で上記の内容全てを満たすことは不可能であるため、以下の内容を満たすことをifcJSONでは開発戦略としている。

- 後方互換性
- ifcファイルとjson間での相互変換の容易性
- EXPRESSスキーマとの並列性

ifcJSONにおいて、一番重要であるのはオリジナルのIFC EXPRESSスキーマに存在する全ての定義を含むことである。

## 仕様

1. IFC EXPRESS スキーマで必須である多くのオブジェクト、キー、エンティティは変換する際においては必須ではない。つまり、必要最低限の情報のみでjsonに変換することができる。
2. 値としてnullやuserdefined、notdefinedを持つ属性の変換は非推奨である。
3. ヘッダーはSPFファイルのヘッダー内容を受け継いでいる
4. インスタンスのユニーク性はglobalIdで保証される
5. ifcJSONの構造の変更は柔軟であるため木構造にすることも可能である。
6. パラメータの名前はcamelCaseで記述する
7. IFCtypeはtypeという属性に、object typeはobjectTypeに格納される
8. ジオメトリについてはSTEPファイルの内容を変換する
9. 属性情報についてはifc EXPRESSの内容を変換する

## その他

その他の詳細な内容等については、[公式ドキュメント](https://github.com/buildingSMART/ifcJSON/tree/master/Documentation)
及び[公式github](https://github.com/buildingSMART/ifcJSON)でのpull requestなどを用いて閲覧・検索してみてください。



