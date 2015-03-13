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

* リストボックスや列のオブジェクトメソッドで```On Display Detail```イベントを処理することもできます。

メソッドは，集計値を算定するために，リレーションコマンドを実行しています。

```
C_LONGINT($0)

LOAD RECORD([Invoice])
RELATE MANY([Invoice]ID)
$0:=Sum([InvoiceDetail]amount)
```

```On Dipsplay Detail```は，フォームを表示したときや，セレクションが変わったときだけでなく，縦または横にリストをスクロールした・ウィンドウをリサイズした・ウィンドウがアクティブになった・非アクティブになったなど，さまざまな条件で頻繁に発生するイベントです。ですから，このイベントにコードを記述するときには，下記のことに配慮する必要があります。

* 表示している他テーブルのカレントセレクションやカレントレコードを崩さない
* できるだけ処理を短時間に抑える（特にクライアント/サーバー）

Form1は，同一フォームの別リストボックスに```[InvoiceDetail]```のセレクションが表示されるようになっています。```RELATE MANY```は，```[InvoiceDetail]```のセレクションを変えてしまう操作なので，このままでは，```[Invoice]```で```On Display Detail```イベントが実行されるたびに，隣のリストボックスの内容が変動することになります。最終的に表示されるのは，```On Display Detail```イベントが最後に発生した行，つまりリストの最下行，あるいは縦スクロールのため余分に評価された行です。

Form1では，この問題を避けるために，右側のリストボックスでは，命名セレクションをデータソースに使用しています。命名セレクションは，カレントセレクションのスナップショットのようなものです。```[Invoice]```のカレントレコードが変更されたときには，```CUT NAMED SELECTION```を実行し，```On Display Detail```など，集計の都合で```RELATE MANY```が実行されるたびに内容が変わらないように工夫しています。**カレントセレクションの代わりに命名セレクションを表示できる**，というのもリストボックスのおおきな利点です。
 
集計そのものは，単純な```Sum```ですが，この関数は，v11以降，**インデックスが使用できる**ようになっているので，フィールドにはインデックスが設定されています。 
 
また，クライアント/サーバー版でネットワークトラフィックを最小限に抑えるため，メソッドの《**サーバー上で実行**》プロパティが有効にされています。

```RELATE MANY```は，カレントレコードがロードされていなければ，何もしません。リスト表示中は，さまざまな理由により，カレントレコードがアンロードされている（カレントレコードの位置は保持されている）ことが少なくないので，この例では，```RELATE MANY```の前に```LOAD RECORD```を呼び出すようにしています。


```On Display Detail```で算定された集計値は，リストボックスの列に表示されていますが，この列のデータソースは《式》であり，**特定の変数や配列に代入されているわけではありません**。特定の行に表示されている集計値を```On Display Detail```以外の場所で参照するためには，改めて集計メソッドを実行する必要があります。そのためには，そのレコードをカレントレコードにした上で，列のデータソースに設定されているのと同じメソッドを呼び出します。

カレントセレクション内におけるカレントレコードの位置は，```Selected record number```で調べることができます。カレントレコードの位置を```GOTO SELECTED RECORD```に渡せば，そのレコードがカレントレコードになります。

リストフォーム（選択モード: 単一）とは違い，リストボックスにカレントセレクションが表示されている場合，行を**クリックしただけではそのレコードがカレントレコードになりません**。ハイライトセット（リストフォームの ```"UserSet"```，サブフォームの```GET HIGHLIGHTED RECORDS```に相当）にそのレコードが追加されるだけです。

選択された行のレコードをカレントレコードにするためには，やはり```GOTO SELECTED RECORD```を使用します。選択された位置は，```LISTBOX GET CELL POSITION```で調べることができます。

```
C_LONGINT($column;$row)
LISTBOX GET CELL POSITION(*;OBJECT Get name(Object current);$column;$row)

If ($row#0)

 GOTO SELECTED RECORD([Invoice];$row)
 RELATE MANY([Invoice])
 CUT NAMED SELECTION([InvoiceDetail];"$List2Selection")

Else 
 UNLOAD RECORD([Invoice])
End if 
```
集計メソッドは，カレントレコードに対して実行されるものですが，合計メソッドは，カレントセレクションに対して実行されるものです。ですから```RELATE MANY```ではなく```RELATE MANY SELECTION```を使用します。

```
C_LONGINT($0)

RELATE MANY SELECTION([InvoiceDetail]invoice)
$0:=Sum([InvoiceDetail]amount)
```

この値は，リストボックスのフッターに代入されています。ヘッダーの変数は，ボタン（数値）でなければなりませんが，　フッターの変数は，数値・日付・時間・文字列で宣言することができます。最小・最大・平均・合計など，単純な統計であれば，《フッターの計算》プロパティで設定するだけで済みますが，メソッドの戻り値をフッターに表示したい場合は，フッターの計算を《カスタム》に設定し，**通常の変数と同じように値を代入**します。

```
$Total:=OBJECT Get pointer(Object named;"Total")
$Total->:=INVOICE_Total 
```

フッターがプロセス変数（フォームエディター/プロパティリストに変数名を入力）であれば，そのまま代入することができますが，この例では，変数名を空にすることにより，**フォームローカル変数**を使用しているので，オブジェクト名からポインターを取得し，間接的に値を代入しています。
 
行毎に表示された集計は，再度メソッドを実行しなければいけませんが，フッターに代入された合計は，そのままフッターの変数から読み取ることができます。
 
リストフォーム
---

サンプルの《Form2》は，リストボックスを使用しています。

特別な必要がない限り，一覧画面はリストボックスで作成したほうが簡単ですが，似たような画面は旧来のリストフォームでも作成することができます。

行の背景色を交互に変更するためには，```Displayed line number```を参照し，それが偶数行と奇数行を判別します。

```
$event:=Form event

Case of 
 : ($event=On Display Detail)

  If (Displayed line number%2=0)
   OBJECT SET VISIBLE(*;"BgEven";True)
   OBJECT SET VISIBLE(*;"BgOdd";False)
  Else 
   OBJECT SET VISIBLE(*;"BgEven";False)
   OBJECT SET VISIBLE(*;"BgOdd";True)
  End if 

End case 
```
行毎の集計は，```On Display Detail```で算定します。

```
$event:=Form event

Case of 
 : ($event=On Display Detail)

  Self->:=INVOICE_Subtotal 

End case 
```

* 変数オブジェクトの《変数名》に式を記述することにより，同じタイミングでメソッドなどを実行することもできます。

リストフォームの各行に配置された変数は，```On Display Detail```が実行されるたびに書き換えられ，以前の値は失われることに留意してください。特定の行に表示されている集計値をOn Display Detail以外の場所で参照するためには，改めて集計メソッドを実行する必要があります。そのためには，そのレコードをカレントレコードにした上で，列のデータソースに設定されているのと同じメソッドを呼び出します。つまり，リストボックスの場合と同じ原則が適用されます。

合計は，リストフォームのフッターに配置された変数に代入されています。合計は，リストが再描画されるたびに一度だけ実行すれば良いので，```On Header```で計算しています。

```
$event:=Form event

Case of 
 : ($event=On Header)

  $Total:=OBJECT Get pointer(Object named;"Total")
  $Total->:=INVOICE_Total 

End case 
```

* ```On Load```と```On Header```を混同しないでください。```On Load```は，フォームが表示されたときに1度だけ発生するイベントですが，```On Header```は，リストフォームが再描画されるたびに繰り返し発生するイベントです。これには，フォームを表示したときや，セレクションが変わったときだけでなく，縦または横にリストをスクロールしたとき・ウィンドウをリサイズしたとき・ウィンドウがアクティブになったとき・非アクティブになったとき，などが含まれます。
