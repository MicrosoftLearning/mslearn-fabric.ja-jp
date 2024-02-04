---
lab:
  title: Apache Spark で差分テーブルを使用する
  module: Work with Delta Lake tables in Microsoft Fabric
---

# Apache Spark で差分テーブルを使用する

Microsoft Fabric のレイクハウスは、Apache Spark のオープンソースの *Delta Lake* 形式に基づいています。 Delta Lake では、バッチ データ操作とストリーミング データ操作の両方にリレーショナル セマンティクスのサポートが追加され、Apache Spark を使用して、データ レイク内の基になるファイルに基づくテーブル内のデータを処理しクエリを実行できる Lakehouse アーキテクチャを作成できます。

この演習の所要時間は約 **40** 分です

> **注**:この演習を完了するには、Microsoft の"学校" または "職場" アカウントが必要です。**** お持ちでない場合は、[Microsoft Office 365 E3 以降の試用版にサインアップ](https://www.microsoft.com/microsoft-365/business/compare-more-office-365-for-business-plans)できます。

## ワークスペースの作成

Fabric でデータを操作する前に、Fabric 試用版を有効にしてワークスペースを作成してください。

1. [Microsoft Fabric ホーム ページ](https://app.fabric.microsoft.com) (`https://app.fabric.microsoft.com`) で、**[Synapse Data Engineering]** を選択します。
2. 左側のメニュー バーで、 **[ワークスペース]** を選択します (アイコンは &#128455; に似ています)。
3. 任意の名前で新しいワークスペースを作成し、Fabric 容量を含むライセンス モード ("試用版"、*Premium*、または *Fabric*) を選択します。**
4. 開いた新しいワークスペースは空のはずです。

    ![Fabric の空のワークスペースを示すスクリーンショット。](./Images/new-workspace.png)

## レイクハウスを作成してデータをアップロードする

ワークスペースが作成されたので、次に分析するデータ用のデータ レイクハウスを作成します。

1. **Synapse Data Engineering** ホーム ページで、任意の名前で新しい **Lakehouse** を作成します。

    1 分ほど経つと、新しい空のレイクハウスが表示されます。 分析のために、データ レイクハウスにいくつかのデータを取り込む必要があります。 これを行うには複数の方法がありますが、この演習では、テキスト ファイルをローカル コンピューター (または、該当する場合はラボ VM) にダウンロードし、レイクハウスにアップロードするだけです。

1. `https://github.com/MicrosoftLearning/dp-data/raw/main/products.csv` からこの演習用の[データ ファイル](https://github.com/MicrosoftLearning/dp-data/raw/main/products.csv)をダウンロードし、ローカル コンピューター (または、該当する場合はラボ VM) に **products.csv** として保存します。

1. レイクハウスを含む Web ブラウザー タブに戻り、**エクスプローラー** ペインの **Files** フォルダーの **[...]** メニューで、 **[新しいサブフォルダー]** を選択し、**products** という名前のフォルダーを作成します。

1. **products** フォルダーの **[...]** メニューで、 **[アップロード]** と **[ファイルのアップロード]** を選択し、**products.csv** ファイルをローカル コンピューター (または、該当する場合はラボ VM) からレイクハウスにアップロードします。
1. ファイルがアップロードされた後、**products** フォルダーを選択し、次に示すように **products.csv** ファイルがアップロードされたことを確認します。

    ![レイクハウスにアップロードされた products.csv ファイルのスクリーンショット。](./Images/products-file.png)

## データフレーム内のデータを調べる

1. Datalake の **products** フォルダーの内容を表示したまま、 **[ホーム]** ページの **[ノートブックを開く]** メニューで、 **[新しいノートブック]** を選択します。

    数秒後に、1 つの ''セル'' を含む新しいノートブックが開きます。** ノートブックは、''コード'' または ''マークダウン'' (書式設定されたテキスト) を含むことができる 1 つ以上のセルで構成されます。** **

2. ノートブック内の既存のセルを選択します。これには単純なコードが含まれています。次に、右上にある **&#128465;** ([削除]) アイコンを使用して削除します。このコードは必要ありません。**
3. 左側の **レイクハウス エクスプローラー** ペインで、**Files** を展開し、 **[products]** を選択すると、前にアップロードした **products.csv** ファイルを表示した新しいペインが表示されます。

    ![[Files] ペインを含むノートブックのスクリーンショット。](./Images/notebook-products.png)

4. **products.csv** の **[...]** メニューで、 **[データの読み込み]**  >  **[Spark]** の順に選択します。 次のコードを含む新しいコード セルがノートブックに追加されます。

    ```python
   df = spark.read.format("csv").option("header","true").load("Files/products/products.csv")
   # df now is a Spark DataFrame containing CSV data from "Files/products/products.csv".
   display(df)
    ```

    > **ヒント**: 左側のファイルを含むペインは、その **[<<]**  アイコンを使用して非表示にすることができます。 そうすることにより、ノートブックに集中できます。

5. セルの左側にある **[&#9655;]** ([セルの実行]) ボタンを使用して実行します。**

    > **注**: このノートブック内で Spark コードを実行したのはこれが最初であるため、Spark セッションを起動する必要があります。 これは、最初の実行が完了するまで 1 分ほどかかる場合があることを意味します。 それ以降は、短時間で実行できます。

6. セル コマンドが完了したら、セルの下にある出力を確認します。これは次のようになるはずです。

    | インデックス | ProductID | ProductName | カテゴリ | ListPrice |
    | -- | -- | -- | -- | -- |
    | 1 | 771 | Mountain-100 Silver, 38 | マウンテン バイク | 3399.9900 |
    | 2 | 772 | Mountain-100 Silver, 42 | マウンテン バイク | 3399.9900 |
    | 3 | 773 | Mountain-100 Silver, 44 | マウンテン バイク | 3399.9900 |
    | ... | ... | ... | ... | ... |

## デルタ テーブルを作成する

`saveAsTable` メソッドを使用すると、データフレームを差分テーブルとして保存できます。 Delta Lake では、"マネージド" と "外部" の両方のテーブルの作成がサポートされています。** **

### "マネージド" テーブルを作成する**

"マネージド" テーブルは、スキーマ メタデータとデータ ファイルの両方が Fabric によって管理されるテーブルです。** このテーブルのデータ ファイルは、**Tables** フォルダーに作成されます。

1. 最初のコード セルから返された結果の下で、**[+ コード]** アイコンを使用して、新しいコード セルを追加します (まだない場合)。

    > **ヒント**: **[+ コード]** アイコンを表示するには、マウスを現在のセルの出力のすぐ下の左側に移動します。 または、メニュー バーの **[編集]** タブで、**[+ コード セルの追加]** を選択します。

2. 新しいセルに次のコードを入力して実行します。

    ```python
   df.write.format("delta").saveAsTable("managed_products")
    ```

3. **レイクハウス エクスプローラー** ペインで、**Tables** フォルダーの **[...]** メニューにある **[更新]** を選択します。 次に、**Tables** ノードを展開し、**managed_products** テーブルが作成されていることを確認します。

### "外部" テーブルを作成する**

"外部" テーブルを作成することもできます。このテーブルのスキーマ メタデータはレイクハウスのメタストアで定義されますが、データ ファイルは外部の場所に格納されます。**

1. 別の新しいセルを追加し、次のコードを追加します。

    ```python
   df.write.format("delta").saveAsTable("external_products", path="abfs_path/external_products")
    ```

2. **レイクハウス エクスプローラー** ペインで、**Files** フォルダーの **[...]** メニューにある **[ABFS パスのコピー]** を選択します。

    ABFS パスは、レイクハウスの OneLake ストレージ内にある **Files** フォルダーへの完全修飾パスです。次のようになります。

    *abfss://workspace@tenant-onelake.dfs.fabric.microsoft.com/lakehousename.Lakehouse/Files*

3. コード セルに入力したコードで、**abfs_path** を、クリップボードにコピーしたパスに置き換えます。これにより、コードを実行すると、**Files** フォルダーの場所にある **external_products** という名前のフォルダーに、データ ファイルと共にデータフレームが外部テーブルとして保存されます。 完全なパスは次のようになります。

    *abfss://workspace@tenant-onelake.dfs.fabric.microsoft.com/lakehousename.Lakehouse/Files/external_products*

4. **レイクハウス エクスプローラー** ペインで、**Tables** フォルダーの **[...]** メニューにある **[更新]** を選択します。 次に、**Tables** ノードを展開し、**external_products** テーブルが作成されていることを確認します。

5. **レイクハウス エクスプローラー** ペインで、**Files** フォルダーの **[...]** メニューにある **[更新]** を選択します。 次に、**Files** ノードを展開し、テーブルのデータ ファイル用の **external_products** フォルダーが作成されていることを確認します。

### "マネージド" テーブルと "外部" テーブルを比較する** **

マネージド テーブルと外部テーブルの違いを調べましょう。

1. 別のコード セルを追加し、次のコードを実行します。

    ```sql
   %%sql

   DESCRIBE FORMATTED managed_products;
    ```

    結果で、テーブルの **Location** プロパティを確認します。これは、レイクハウスの OneLake ストレージへのパスであり、 **/Tables/managed_products** で終わります (完全なパスを表示するには、**データ型**列の幅を広げることが必要な場合があります)。

2. `DESCRIBE` コマンドを次のように変更して、**external_products** テーブルの詳細を表示します。

    ```sql
   %%sql

   DESCRIBE FORMATTED external_products;
    ```

    結果で、テーブルの **Location** プロパティを確認します。これは、レイクハウスの OneLake ストレージへのパスであり、 **/Files/external_products** で終わります (完全なパスを表示するには、**データ型**列の幅を広げることが必要な場合があります)。

    マネージド テーブルのファイルは、レイクハウスの OneLake ストレージ内にある **Tables** フォルダーに格納されます。 この場合、**managed_products** という名前のフォルダーが作成され、Parquet ファイルと、作成したテーブルの **delta_log** フォルダーが格納されます。

3. 別のコード セルを追加し、次のコードを実行します。

    ```sql
   %%sql

   DROP TABLE managed_products;
   DROP TABLE external_products;
    ```

4. **レイクハウス エクスプローラー** ペインで、**Tables** フォルダーの **[...]** メニューにある **[更新]** を選択します。 次に、**Tables** ノードを展開し、一覧にテーブルが表示されていないことを確認します。

5. **レイクハウス エクスプローラー** ペインで、**Files** フォルダーを展開し、**external_products** が削除されていないことを確認します。 このフォルダーを選択すると、Parquet データ ファイルと、以前は **external_products** 内にあったデータ用の **_delta_log** フォルダーが表示されます。 外部テーブルのテーブル メタデータは削除されましたが、ファイルには影響ありません。

### SQL を使用してテーブルを作成する

1. 別のコード セルを追加し、次のコードを実行します。

    ```sql
   %%sql

   CREATE TABLE products
   USING DELTA
   LOCATION 'Files/external_products';
    ```

2. **レイクハウス エクスプローラー** ペインで、**Tables** フォルダーの **[...]** メニューにある **[更新]** を選択します。 次に、**Tables** ノードを展開し、**products** という名前の新しいテーブルが一覧表示されていることを確認します。 その後、テーブルを展開して、そのスキーマが、**external_products** フォルダーに保存された元のデータフレームと一致することを確認します。

3. 別のコード セルを追加し、次のコードを実行します。

    ```sql
   %%sql

   SELECT * FROM products;
   ```

## テーブルのバージョン管理を調べる

差分テーブルのトランザクション履歴は、**delta_log** フォルダー内の JSON ファイルに格納されます。 このトランザクション ログを使用して、データのバージョンを管理できます。

1. ノートブックに新しいコード セルを追加し、次のコードを実行します。

    ```sql
   %%sql

   UPDATE products
   SET ListPrice = ListPrice * 0.9
   WHERE Category = 'Mountain Bikes';
    ```

    このコードは、マウンテン バイクの価格を 10% 値下げします。

2. 別のコード セルを追加し、次のコードを実行します。

    ```sql
   %%sql

   DESCRIBE HISTORY products;
    ```

    結果には、テーブルに関して記録されたトランザクションの履歴が表示されます。

3. 別のコード セルを追加し、次のコードを実行します。

    ```python
   delta_table_path = 'Files/external_products'

   # Get the current data
   current_data = spark.read.format("delta").load(delta_table_path)
   display(current_data)

   # Get the version 0 data
   original_data = spark.read.format("delta").option("versionAsOf", 0).load(delta_table_path)
   display(original_data)
    ```

    結果には 2 つのデータフレームが表示されます。1 つは値下げ後のデータが含まれたもので、もう 1 つはデータの元のバージョンを示します。

## ストリーミング データにデルタ テーブルを使用する

Delta Lake では、"ストリーミング" データがサポートされています。 デルタ テーブルは、Spark 構造化ストリーミング API を使用して作成されたデータ ストリームの "シンク" または "ソース" に指定できます。** ** この例では、モノのインターネット (IoT) のシミュレーション シナリオで、一部のストリーミング データのシンクにデルタ テーブルを使用します。

1. ノートブックに新しいコード セルを追加します。 この新しいセルに次のコードを追加して実行します。

    ```python
   from notebookutils import mssparkutils
   from pyspark.sql.types import *
   from pyspark.sql.functions import *

   # Create a folder
   inputPath = 'Files/data/'
   mssparkutils.fs.mkdirs(inputPath)

   # Create a stream that reads data from the folder, using a JSON schema
   jsonSchema = StructType([
   StructField("device", StringType(), False),
   StructField("status", StringType(), False)
   ])
   iotstream = spark.readStream.schema(jsonSchema).option("maxFilesPerTrigger", 1).json(inputPath)

   # Write some event data to the folder
   device_data = '''{"device":"Dev1","status":"ok"}
   {"device":"Dev1","status":"ok"}
   {"device":"Dev1","status":"ok"}
   {"device":"Dev2","status":"error"}
   {"device":"Dev1","status":"ok"}
   {"device":"Dev1","status":"error"}
   {"device":"Dev2","status":"ok"}
   {"device":"Dev2","status":"error"}
   {"device":"Dev1","status":"ok"}'''
   mssparkutils.fs.put(inputPath + "data.txt", device_data, True)
   print("Source stream created...")
    ```

    メッセージ「*Source stream created...* 」が確実に出力されるようにします。 先ほど実行したコードによって、一部のデータが保存されているフォルダーに基づいてストリーミング データ ソースが作成されました。これは、架空の IoT デバイスからの読み取り値を表しています。

2. 新しいコード セルに、次のコードを追加して実行します。

    ```python
   # Write the stream to a delta table
   delta_stream_table_path = 'Tables/iotdevicedata'
   checkpointpath = 'Files/delta/checkpoint'
   deltastream = iotstream.writeStream.format("delta").option("checkpointLocation", checkpointpath).start(delta_stream_table_path)
   print("Streaming to delta sink...")
    ```

    このコードは、**iotdevicedata** という名前のフォルダーに、ストリーミング デバイス データを差分形式で書き込みます。 このフォルダーの場所を示すパスは **Tables** フォルダー内であるため、テーブルは自動的に作成されます。

3. 新しいコード セルに、次のコードを追加して実行します。

    ```sql
   %%sql

   SELECT * FROM IotDeviceData;
    ```

    このコードを実行して、ストリーミング ソースのデバイス データが含まれる **IotDeviceData** テーブルに対してクエリを実行します。

4. 新しいコード セルに、次のコードを追加して実行します。

    ```python
   # Add more data to the source stream
   more_data = '''{"device":"Dev1","status":"ok"}
   {"device":"Dev1","status":"ok"}
   {"device":"Dev1","status":"ok"}
   {"device":"Dev1","status":"ok"}
   {"device":"Dev1","status":"error"}
   {"device":"Dev2","status":"error"}
   {"device":"Dev1","status":"ok"}'''

   mssparkutils.fs.put(inputPath + "more-data.txt", more_data, True)
    ```

    このコードを実行して、さらに架空のデバイス データをストリーミング ソースに書き込みます。

5. 次のコードを含むセルを再実行します。

    ```sql
   %%sql

   SELECT * FROM IotDeviceData;
    ```

    このコードを実行して、**IotDeviceData** テーブルに対してもう一度クエリを実行します。今度は、ストリーミング ソースに追加された追加データが含まれるはずです。

6. 新しいコード セルに、次のコードを追加して実行します。

    ```python
   deltastream.stop()
    ```

    このコードを実行して、ストリームを停止します。

## リソースをクリーンアップする

この演習では、Microsoft Fabric で差分テーブルを操作する方法について学習しました。

レイクハウスの探索が完了したら、この演習用に作成したワークスペースを削除できます。

1. 左側のバーで、ワークスペースのアイコンを選択して、それに含まれるすべての項目を表示します。
2. ツール バーの **[...]** メニューで、 **[ワークスペースの設定]** を選択してください。
3. **[その他]** セクションで、 **[このワークスペースの削除]** を選択してください。
