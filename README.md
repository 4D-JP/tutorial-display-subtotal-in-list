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

* エクスプローラー上で右クリックし，**コンテキストメニュー**からメソッドを実行することもできます。

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

アプリケーションモードに移行し，メニューから《LISTBOX》または《LISTFORM》を選択すると，それぞれ新規プロセスでリストボックスまたはリストフォームを利用したサンプル画面が表示されます。

Searchエリアに番号を入力すると，```[Invoice]```のセレクションが絞り込まれます。左側のリストの各行には，```[Invoice]```のフィールド，およびリンクしている```[InvoiceDetail]```のセレクションから集計されたSubtotalが表示されており，リストの一番下には，合計が表示されています。左側のリストで行を選択すると，その行がカレントレコードになり，右側のリストに```[InvoiceDetail]```のセレクションが表示されます。

リストボックス
---
通常，セレクションや配列をリスト形式で表示するには，フォームオブジェクトの**リストボックス**を使用します。昔のバージョンでは，レコードの一覧をリストフォームで表示することが一般的でしたが，現在，推奨されているのは，リストボックスです。

リストボックスは，テーブルフォームはもちろん，プロジェクトフォームにも自由に配置することができます。

* リストフォームも，プロジェクトフォームに配置することができますが，デザインモード（ユーザーモード）のリスト画面に同一テーブルのセレクションが表示されている場合，プロジェクトフォームに配置されたほうのリスト画面（サブフォーム）は表示されません。

リストボックスには，代表的なメカニズムが最初から揃っており，セットアップがとても簡単です。交互に使用する背景色・罫線・列幅のリサイズ・行および列のドラッグ＆ドロップ・スクロールしない列（左側に固定され横スクロールしない）・ヘッダークリックによる並び替え・フッター表示といったことは，**プログラミングをせずに**プロパティ設定だけで実現できます。

サンプルの《Form1》は，リストボックスを使用しています。
 

リストボックスのデータソースには，配列・カレントセレクション・命名セレクションのどれかを設定することができます。データソースをカレントセレクションに設定する場合，マスターテーブルを選びます。

リストボックス各列には，マスターテーブルのフィールドを表示させるのが一般的ですが，実際のデータソースプロパティは《式》となっており，メソッド・関数・結合した文字列など，何であれ，値を返す式，つまりフォーミュラを記述することができます。

Form1の場合，1番目の列は```[Invoice]code```つまりフィールドがデータソースですが，2番目の列は```INVOICE_Subtotal```というメソッドが設定されていることに注目してください。このメソッドは，行を表示するたびに，つまり```On Display Detail```と同じタイミングで評価されます。

* リストボックスや列のオブジェクトメソッドで```On Display Detail```することもできます。

メソッドは，集計値を算定するために，リレーションコマンドを実行しています。

```
C_LONGINT($0)

CUT NAMED SELECTION([InvoiceDetail];"$temp")

LOAD RECORD([Invoice])
RELATE MANY([Invoice]ID)
$0:=Sum([InvoiceDetail]amount)

C_POINTER($1)
If (Count parameters#0)
 If (Not(Nil($1)))
  $1->:=Sum([InvoiceDetail]count)
 End if 
End if 

USE NAMED SELECTION("$temp")
```

 
 
 
リストフォーム
---

サンプルの《Form2》は，リストボックスを使用しています。
