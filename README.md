レッスン：リスト画面に集計値と合計値を表示する
---

リスト画面の行毎に集計値を表示したり，フッターに合計値を表示したりする方法を学習することができます。(v13+)

* リストボックス
 
![](https://github.com/4D-JP/tutorial-display-subtotal-in-list/blob/master/images/1.png)

* リストフォーム
 
![](https://github.com/4D-JP/tutorial-display-subtotal-in-list/blob/master/images/2.png)

データベース
---

サンプルにデータは含まれていません。初めにデザインモードの《実行》メニューから```INIT```を実行してください。1000件分の乱数データが作成されます。件数を増減したい場合，```$invoiceCount:=1000```を書き換えてください。

* エクスプローラー上でプロジェクトメソッドを右クリックし，**コンテキストメニュー**から実行することもできます。

```
TRUNCATE TABLE([Invoice])
TRUNCATE TABLE([InvoiceDetail])

SET DATABASE PARAMETER([Invoice];Table Sequence Number;0)
SET DATABASE PARAMETER([InvoiceDetail];Table Sequence Number;0)

$invoiceCount:=1000

$format:="0"*(Length(String($invoiceCount))+2)

C_LONGINT($i;$j)

For ($i;1;$invoiceCount)

CREATE RECORD([Invoice])

[Invoice]code:=Change string($format;String($i);Length($format)-Length(String($i))+1)

$detailCount:=(Random%16)+1

For ($j;1;$detailCount)

CREATE RECORD([InvoiceDetail])
[InvoiceDetail]invoice:=[Invoice]ID
[InvoiceDetail]price:=((Random%100)+1)*100
[InvoiceDetail]count:=(Random%9)+1
[InvoiceDetail]amount:=[InvoiceDetail]price*[InvoiceDetail]count
SAVE RECORD([InvoiceDetail])

End for 

SAVE RECORD([Invoice])

End for 

UNLOAD RECORD([Invoice])
UNLOAD RECORD([InvoiceDetail])
```

**POINT**

* [TRUNCATE TABLE](http://doc.4d.com/4Dv14/4D/14.3/TRUNCATE-TABLE.301-1697352.en.html)で**テーブルのレコードをすべて削除**することができます。

* [SET DATABASE PARAMETER](http://doc.4d.com/4Dv14/4D/14.3/SET-DATABASE-PARAMETER.301-1696621.ja.html)でテーブルの**自動インクリメント番号**，つまり[Sequence number](http://doc.4d.com/4Dv14/4D/14.3/Sequence-number.301-1697199.ja.html)をリセットすることができます。


