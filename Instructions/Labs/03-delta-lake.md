---
lab:
  title: Apache Spark で差分テーブルを使用する
  module: Work with Delta Lake tables in Microsoft Fabric
---

# Apache Spark で Delta テーブルを使用する

Microsoft Fabric レイクハウスのテーブルは、オープンソースの Delta Lake 形式に基づいています。 Delta Lake では、バッチ データとストリーミング データの両方に対するリレーショナル セマンティクスのサポートが追加されます。 この演習では、Delta テーブルを作成し、SQL クエリを使用してデータを探索します。

この演習の完了に要する時間は約 45 分です

> [!NOTE]
> この演習を完了するには、[Microsoft Fabric](/fabric/get-started/fabric-trial) 試用版が必要です。

## ワークスペースの作成

まず、*Fabric 試用版*を有効にしてワークスペース作成します。

1. Microsoft Fabric ホーム ページ (https://app.fabric.microsoft.com) で、**[Synapse Data Engineering]** エクスペリエンスを選択します。
1. 左側のメニュー バーで、**[ワークスペース]** (🗇) を選択します。
1. 任意の名前で**新しいワークスペース**を作成し、Fabric 容量があるライセンス モード (試用版、Premium、または Fabric) を選択します。
1. 開いた新しいワークスペースは空のはずです。

![空の Fabric ワークスペースの画面画像。](Images/workspace-empty.jpg)

## レイクハウスを作成してデータをアップロードする

ワークスペースが作成されたので、次にレイクハウスを作成し、一部のデータをアップロードします。

1. **Synapse Data Engineering** ホーム ページで、任意の名前で新しい **Lakehouse** を作成します。 
1. データを取り込む方法はいろいろありますが、この演習では、ローカル コンピューター (または該当する場合はラボ VM) にテキスト ファイルをダウンロードし、レイクハウスにアップロードします。 https://github.com/MicrosoftLearning/dp-data/raw/main/products.csv から[データ ファイル](https://github.com/MicrosoftLearning/dp-data/raw/main/products.csv)をダウンロードし、*products.csv* として保存します。
1.  レイクハウスを含む Web ブラウザー タブに戻り、[エクスプローラー] ペインで、**Files** フォルダーの横にある [...]  メニューを選択します。  *products* という**新しいサブフォルダー**を作成します。
1.  の products フォルダーの [...] メニューで、ローカル コンピューター (または該当する場合はラボ VM) から *products.csv* ファイルを**アップロード**します。
1.  ファイルがアップロードされたら、**products** フォルダーを選択し、次に示すようにファイルがアップロードされていることを確認します。

![レイクハウスにアップロードされた products.csv の画面画像。](Images/upload-products.jpg)
  
## DataFrame 内のデータを探索する

1.  **新しいノートブック**を作成します。 数秒後に、1 つの ''セル'' を含む新しいノートブックが開きます。 ノートブックは、''コード'' または ''マークダウン'' (書式設定されたテキスト) を含むことができる 1 つ以上のセルで構成されます。
2.  最初のセル (今はコード セル) を選択し、右上のツール バーで **[M↓]** ボタンを使用して Markdown セルに変換します。 セルに含まれるテキストは、書式設定されたテキストで表示されます。 マークダウン セルを使用して、コードに関する説明情報を記載します。
3.  🖉 ([編集]) ボタンを使用してセルを編集モードに切り替え、マークダウンを次のように変更します。

```markdown
# Delta Lake tables 
Use this notebook to explore Delta Lake functionality 
```

4. セルの外側のノートブック内の任意の場所をクリックして編集を停止し、レンダリングされたマークダウンを確認します。
5. 新しいコード セルを追加し、次のコードを追加して、定義したスキーマにより製品データを DataFrame に読み取ります。

```python
from pyspark.sql.types import StructType, IntegerType, StringType, DoubleType

# define the schema
schema = StructType() \
.add("ProductID", IntegerType(), True) \
.add("ProductName", StringType(), True) \
.add("Category", StringType(), True) \
.add("ListPrice", DoubleType(), True)

df = spark.read.format("csv").option("header","true").schema(schema).load("Files/products/products.csv")
# df now is a Spark DataFrame containing CSV data from "Files/products/products.csv".
display(df)
```

> [!TIP]
> シェブロン [«] アイコンを使用して、エクスプローラー ペインを非表示または表示にできます。 こうすることで、ノートブックまたはファイルに集中できます。

7. セルの左側にある **[セルの実行]** (▷) ボタンを使用して実行します。

> [!NOTE]
> このノートブックで Spark コードを実行したのはこれが最初であるため、Spark セッションを開始する必要があります。 これは、最初の実行が完了するまで 1 分ほどかかる場合があることを意味します。 それ以降は、短時間で実行できます。

8. セル コードが完了したら、セルの下にある出力を確認します。これは次のようになるはずです。

![products.csv データの画面画像。](Images/products-schema.jpg)
 
## Delta テーブルを作成する

*saveAsTable* メソッドを使用すると、DataFrame を Delta テーブルとして保存できます。 Delta Lake では、マネージド テーブルと外部テーブルの両方の作成がサポートされています。

* **マネージド** Delta テーブルでは、スキーマ メタデータとデータ ファイルの両方が Fabric によって管理されるため、パフォーマンスが向上します。
* **外部**テーブルを使用すると、Fabric によって管理されるメタデータを使用して、データを外部に格納できます。

### マネージド テーブルを作成する

データ ファイルは、**Tables** フォルダーに作成されます。

1. 最初のコード セルから返された結果の下で、[+ コード] アイコンを使用して新しいコード セルを追加します。

> [!TIP]
> [+ コード] アイコンを表示するには、マウスを現在のセルの出力のすぐ下の左側に移動します。 または、メニュー バーの [編集] タブで、**[+ コード セルの追加]** を選択します。

2. マネージド Delta テーブルを作成するには、新しいセルを追加し、次のコードを入力して、セルを実行します。

```python
df.write.format("delta").saveAsTable("managed_products")
```

3.  [レイクハウス エクスプローラー] ペインで、Tables フォルダーを**更新**し、Tables ノードを展開して、**managed_products** テーブルが作成されていることを確認します。

>[!NOTE]
> ファイル名の横にある三角形アイコンは Delta テーブルを示します。

マネージド テーブルのファイルは、レイクハウス内の **Tables** フォルダーに格納されます。 Parquet ファイルと、テーブルの delta_log フォルダーが格納される、*managed_products* というフォルダーが作成されています。

### 外部テーブルを作成する

レイクハウスに格納されているスキーマ メタデータを使用して、レイクハウス以外の場所に格納できる外部テーブルを作成することもできます。

1.  [レイクハウス エクスプローラー] ペインで、 **Files** フォルダーの [...] メニューにある **[ABFS パスのコピー]** を選択します。 ABFS パスは、レイクハウスの Files フォルダーへの完全修飾パスです。

2.  新しいコード セルに、ABFS パスを貼り付けます。 切り取りと貼り付けを使用して、コード内の正しい場所に abfs_path を挿入して、次のコードを追加します。

```python
df.write.format("delta").saveAsTable("external_products", path="abfs_path/external_products")
```

完全なパスは次のようになります。

```python
abfss://workspace@tenant-onelake.dfs.fabric.microsoft.com/lakehousename.Lakehouse/Files/external_products
```

4. セルを**実行**し、DataFrame を外部テーブルとして Files/external_products フォルダーに保存します。

5.  [レイクハウス エクスプローラー] ペインで、Tables フォルダーを**更新**し、Tables ノードを展開して、スキーマ メタデータが入った external_products テーブルが作成されていることを確認します。

6.  [レイクハウス エクスプローラー] ペインで、 Files フォルダーの [...] メニューにある **[更新]** を選択します。 次に、Files ノードを展開し、テーブルのデータ ファイル用に external_products フォルダーが作成されていることを確認します。

### "マネージド" テーブルと "外部" テーブルを比較する 

%%sql マジック コマンドを使用して、マネージド テーブルと外部テーブルの違いを調べましょう。

1. 新しいコード セルで、次のコードを実行します。

```pthon
%%sql
DESCRIBE FORMATTED managed_products;
```

2. 結果で、テーブルの "場所" プロパティを確認します。 [データ型] 列の [場所] 値をクリックすると、完全なパスが表示されます。 OneLake ストレージの場所が /Tables/managed_products で終わることに注意してください。

3. 次のように、external_products テーブルの詳細を表示するように DESCRIBE コマンドを変更します。

```python
%%sql
DESCRIBE FORMATTED external_products;
```

4. セルを実行し、結果でテーブルの "場所" プロパティを確認します。 [データ型] 列の幅を広げて完全なパスを表示し、OneLake ストレージの場所が /Files/external_products で終わることに注意してください。

5. 新しいコード セルで、次のコードを実行します。

```python
%%sql
DROP TABLE managed_products;
DROP TABLE external_products;
```

6. [レイクハウス エクスプローラー] ペインで、Tables フォルダーを**更新**し、Tables ノードにテーブルが表示されていないことを確認します。
7.  [レイクハウス エクスプローラー] ペインで、Files フォルダーを**更新**し、external_products ファイルが削除されて*いない*ことを確認します。 このフォルダーを選択して、Parquet データ ファイルと _delta_log フォルダーを表示します。 

外部テーブルのメタデータは削除されましたが、データ ファイルは削除されていません。

## SQL を使用して Delta テーブルを作成する

今度は、%%sql マジック コマンドを使用して Delta テーブルを作成します。 

1. 別のコード セルを追加し、次のコードを実行します。

```python
%%sql
CREATE TABLE products
USING DELTA
LOCATION 'Files/external_products';
```

2. [レイクハウス エクスプローラー] ペインで、 **Tables** フォルダーの [...] メニューにある **[更新]** を選択します。 次に、Tables ノードを展開し、*products* という名前の新しいテーブルが表示されていることを確認します。 そして、テーブルを展開してスキーマを表示します。

3. 別のコード セルを追加し、次のコードを実行します。

```python
%%sql
SELECT * FROM products;
```

## テーブルのバージョン管理を調べる

Delta テーブルのトランザクション履歴は、delta_log フォルダー内の JSON ファイルに格納されます。 このトランザクション ログを使用して、データのバージョンを管理できます。

1.  ノートブックに新しいコード セルを追加し、マウンテン バイクの価格を 10% 値下げする次のコードを実行します。

```python
%%sql
UPDATE products
SET ListPrice = ListPrice * 0.9
WHERE Category = 'Mountain Bikes';
```

2. 別のコード セルを追加し、次のコードを実行します。

```python
%%sql
DESCRIBE HISTORY products;
```

結果には、テーブルに関して記録されたトランザクションの履歴が表示されます。

3.  別のコード セルを追加し、次のコードを実行します。

```python
delta_table_path = 'Files/external_products'
# Get the current data
current_data = spark.read.format("delta").load(delta_table_path)
display(current_data)

# Get the version 0 data
original_data = spark.read.format("delta").option("versionAsOf", 0).load(delta_table_path)
display(original_data)
```

2 つの結果セットが返されます。1 つは値下げ後のデータが入っており、もう 1 つはデータの元のバージョンが表示されます。

## SQL クエリを使用してデルタ テーブルのデータを分析する

SQL マジック コマンドを使用すると、Pyspark の代わりに SQL 構文を使用できます。 ここでは、`SELECT` ステートメントを使用して、製品テーブルから一時ビューを作成します。

1. 新しいコード セルを追加し、次のコードを実行して、一時ビューを作成および表示します。

```python
%%sql
-- Create a temporary view
CREATE OR REPLACE TEMPORARY VIEW products_view
AS
    SELECT Category, COUNT(*) AS NumProducts, MIN(ListPrice) AS MinPrice, MAX(ListPrice) AS MaxPrice, AVG(ListPrice) AS AvgPrice
        FROM products
        GROUP BY Category;

SELECT *
    FROM products_view
    ORDER BY Category;
        
```

2. 新しいコード セルを追加し、次のコードを実行して、製品の数で上位 10 のカテゴリを返します。

```python
%%sql
SELECT Category, NumProducts
    FROM products_view
    ORDER BY NumProducts DESC
    LIMIT 10;
```

データが返されたら、 **[グラフ]** ビューを選択して棒グラフを表示します。

![SQL select ステートメントと結果の画面画像。](Images/sql-select.jpg)

または、PySpark を使用して SQL クエリを実行することもできます。

1. 新しいコード セルを追加し、次のコードを実行します。

```python
from pyspark.sql.functions import col, desc

df_products = spark.sql("SELECT Category, MinPrice, MaxPrice, AvgPrice FROM products_view").orderBy(col("AvgPrice").desc())
display(df_products.limit(6))
```

## ストリーミング データに Delta テーブルを使用する

Delta Lake ではストリーミング データがサポートされています。 デルタ テーブルは、Spark 構造化ストリーミング API を使用して作成されたデータ ストリームの "シンク" または "ソース" に指定できます。 この例では、モノのインターネット (IoT) のシミュレーション シナリオで、一部のストリーミング データのシンクに Delta テーブルを使用します。

1.  新しいコード セルを追加し、次のコードを追加して、実行します。

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

メッセージ「*Source stream created...*」が表示されることを 確認します。 先ほど実行したコードによって、一部のデータが保存されているフォルダーに基づいてストリーミング データ ソースが作成されました。これは、架空の IoT デバイスからの読み取り値を表しています。

2. 新しいコード セルに、次のコードを追加して実行します。

```python
# Write the stream to a delta table
delta_stream_table_path = 'Tables/iotdevicedata'
checkpointpath = 'Files/delta/checkpoint'
deltastream = iotstream.writeStream.format("delta").option("checkpointLocation", checkpointpath).start(delta_stream_table_path)
print("Streaming to delta sink...")
```

このコードは、iotdevicedata という名前のフォルダーに、ストリーミング デバイス データを差分形式で書き込みます。 このフォルダーの場所を示すパスは Tables フォルダー内であるため、テーブルは自動的に作成されます。

3. 新しいコード セルに、次のコードを追加して実行します。

```python
%%sql
SELECT * FROM IotDeviceData;
```

このコードを実行して、ストリーミング ソースのデバイス データが含まれる IotDeviceData テーブルに対してクエリを実行します。

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

```python
%%sql
SELECT * FROM IotDeviceData;
```

このコードを実行して、IotDeviceData テーブルに対してもう一度クエリを実行します。今度は、ストリーミング ソースに追加された追加データが含まれるはずです。

6. 新しいコード セルに、ストリームを停止するコードを追加して、セルを実行します。

```python
deltastream.stop()
```

## リソースをクリーンアップする

この演習では、Microsoft Fabric で Delta テーブルを操作する方法を学習しました。

レイクハウスの探索が終了したら、この演習用に作成したワークスペースを削除できます。

1. 左側のバーで、ワークスペースのアイコンを選択して、それに含まれるすべての項目を表示します。
2. の ツール バーの [...] メニューで、**[ワークスペース設定]** を選択します。
3. [全般] セクションで、**[このワークスペースを削除する]** を選択します。
