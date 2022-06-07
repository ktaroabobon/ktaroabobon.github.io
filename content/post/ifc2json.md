+++
author = "ktaroabobon"
title = "ifc2jsonについて"
date = "2022-05-08"
description = "IFCファイルをjsonに変換するコンバータの一つであるifc2jsonの解説"
featured = true
tags = [
"ifc",
"json",
]
categories = [
"ifc",
]
series = ["IFC JSON"]
thumbnail = "images/building.png"
draft = true

+++

# ifc2json

## 概要

### [github](https://github.com/buildingSMART/ifcJSON)

- ifcを保守運用しているBuildingSMARTが作成している
- pythonで書かれている
- ifcの対応バージョンはifc2x3, ifc4.0, ifc4.3

## ifc2jsonが解決する問題

- 多くの開発者はEXPRESSや[STP](https://www.adobe.com/jp/creativecloud/file-types/image/vector/step-file.html)のインスタンスファイルを閲覧したり使用したりしたことがほとんどない。そのため、純正のIFCファイルは必要なデータを抽出するのに必要な労力が増加してしまう。
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
