+++
author = "ktaroabobon"
title = "ifcをjsonに変換する"
date = "2022-05-08"
description = "IFCファイルをjsonに変換するコンバータを紹介する"
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

# IFCをjsonに変換する

BIMの国際標準であるIFCファイルはそのままだと少々扱いにくい。そこでIFCをjsonに変換するいくつかの方法を紹介する。

# コンバータ一覧

## ifc2json

### [github](https://github.com/buildingSMART/ifcJSON)

- ifcを保守運用しているBuildingSMARTが作成している
- pythonで書かれている
- ifcの対応バージョンはifc2x3, ifc4.0, ifc4.3
