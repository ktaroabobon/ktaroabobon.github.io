+++
author = "ktaroabobon"
title = "ifcJSONについて"
date = "2022-05-08"
description = "IFCファイルをjsonに変換するコンバータの一つであるifc2jsonの解説"
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

本記事ではifcJSONについての概要や開発戦略を記載しています。また、その内容については[公式ドキュメント](https://github.com/buildingSMART/ifcJSON/tree/master/Documentation)
を和訳した内容が中心となっています。

## 概要

- [github](https://github.com/buildingSMART/ifcJSON)
- ifcを保守運用しているBuildingSMARTが作成している
- pythonで書かれている
- ifcの対応バージョンはifc2x3, ifc4.0, ifc4.3

## ifcJSONが解決する問題

- 多くの開発者はEXPRESSや[STP](https://www.adobe.com/jp/creativecloud/file-types/image/vector/step-file.html)
  のインスタンスファイルを閲覧したり使用したりしたことがほとんどない。そのため、純正のIFCファイルは必要なデータを抽出するのに必要な労力が増加してしまう。
- IFCのデータは通常IFCファイルを介して交換されるが、これはほとんどの設計・建設プロジェクトや製品に見られる、リンクされ分散し変化するデータとは相性が悪い。

1つ目に関しては、IFCファイルをそのまま閲覧したり編集したりするのは難しいということである。  
2つ目に関しては、ifcは建築物のデータベースであるのでifcファイルは変化を伴うデータベースとして適した形式ではないということである。

## 目的

- ifcのために推奨される単一のjson仕様を指定することを目的としている

## 開発戦略

ifcをjsonとする場合、考慮すべきいくつかの基準がある。

- 後方互換性
- 相互変換の容易性
- 可読性、直感性
- 柔軟性、拡張性
- コードとの統合性
- XML形式との並列性
- EXPRESSスキーマとの並列性
- 明確な参照構造（フラットやネスト）

一方で上記の内容全てを満たすことは不可能であるため、以下の内容を満たすことをifc2jsonでは開発戦略としている。

- 後方互換性
- 相互変換の容易性
- EXPRESSスキーマとの並列性

ifc2jsonにおいて、一番重要であるのはオリジナルのIFC EXPRESSスキーマに存在するもの全てを含むことである。

## 仕様

1. IFC EXPRESS Schemaで必須である多くのオブジェクト、キー、エンティティは変換する際においては必須ではない。つまり、必要最低限の情報のみでjsonに変換することができる。
2. 値としてnullやuserdefined、notdefinedを持つ属性の変換は非推奨である。
3. ヘッダーはSPFファイルのヘッダー内容を受け継いでいる
4. インスタンスのユニーク性はglobalIdで判別される
5. ifcJsonの構造は柔軟であるため木構造にすることも可能である。
6. パラメータの名前はcamelCaseで記述する
7. IFCtypeはtypeという属性に、object typeはobjectTypeに格納される
8. ジオメトリについてはSTEPファイルの内容を変換する
9. 属性情報についてはifc EXPRESSの内容を変換する

## その他

その他の詳細な内容等については、[公式ドキュメント](https://github.com/buildingSMART/ifcJSON/tree/master/Documentation)
及び[公式github](https://github.com/buildingSMART/ifcJSON)でのpull requestなどを用いて閲覧・検索してみてください。



