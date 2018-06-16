#Markdown記法のまとめ

## リスト
* テキスト
    * テキスト
    * テキスト

## 番号付きリスト
1. テキスト
2. テキスト
    3. テキスト

## 空行
1行目(＋半角スペース2つ)  
2行目  
3行目<br>
4行目

## 引用
インラインで`引用`する場合
> 他の人の引用
>> さらに他の人の引用
注釈が必要な文章[^1]  
[^1]: 注釈の内容。文末に表示される。

## コード引用
サンプルここから
```php
$param = 12;
$ary = array();
```
サンプルここまで

## 文字装飾
normal **bold** normal
normal __bold__ normal 日本語は __うまく__ でないことがある。
normal ***bold*** normal
normal ___bold___ normal 日本語は ___うまく___ でないことがある。

## Table
|Title|Type|Description|
|:---|:---|:---|
|table|string|テーブルを表示したい|


## リンク
[GitHub repository](http://github.com/ogusu/)

![sample.png](https://github.com/ "sample")


## 水平線
start
アスタリスクかハイフンを３つ以上で罫線。
- - -
headerに近いと日本語の前後がおかしくなることがある。