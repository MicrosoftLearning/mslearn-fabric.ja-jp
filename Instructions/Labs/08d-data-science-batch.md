---
lab:
  title: Microsoft Fabric でデプロイされたモデルを使ってバッチ予測を生成する
  module: Generate batch predictions using a deployed model in Microsoft Fabric
---

# Microsoft Fabric でデプロイされたモデルを使ってバッチ予測を生成する

このラボでは、機械学習モデルを使用して、糖尿病の定量的尺度を予測します。

このラボを完了すると、予測の生成と結果の視覚化に関するハンズオン経験が得られます。

このラボは完了するまで、約 **20** 分かかります。

> **注**:この演習を完了するには、[Microsoft Fabric 試用版](https://learn.microsoft.com/fabric/get-started/fabric-trial)が必要です。

## ワークスペースの作成

Fabric でデータを操作する前に、Fabric 試用版を有効にしてワークスペースを作成してください。

1. ブラウザーの `https://app.fabric.microsoft.com/home?experience=fabric` で [Microsoft Fabric ホーム ページ](https://app.fabric.microsoft.com/home?experience=fabric)に移動します。
1. Microsoft Fabric ホーム ページで、**[Synapse Data Science]** を選択します
1. 左側のメニュー バーで、 **[ワークスペース]** を選択します (アイコンは &#128455; に似ています)。
1. 任意の名前で新しいワークスペースを作成し、Fabric 容量を含むライセンス モード ("試用版"、*Premium*、または *Fabric*) を選択します。**
1. 開いた新しいワークスペースは空のはずです。

    ![Fabric の空のワークスペースを示すスクリーンショット。](./Images/new-workspace.png)

## ノートブックを作成する

この演習では、"ノートブック" を使用して、モデルをトレーニングして使用します。**

1. **Synapse Data Science** ホーム ページで、新しい**ノートブック**を作成します。

    数秒後に、1 つの ''セル'' を含む新しいノートブックが開きます。** ノートブックは、''コード'' または ''マークダウン'' (書式設定されたテキスト) を含むことができる 1 つまたは複数のセルで構成されます。** **

1. 最初のセル (現在は ''コード'' セル) を選択し、右上の動的ツール バーで **[M&#8595;]** ボタンを使用してセルを ''マークダウン'' セルに変換します。** **

    セルがマークダウン セルに変わると、それに含まれるテキストがレンダリングされます。

1. 必要に応じて、**[&#128393;]** (編集) ボタンを使用してセルを編集モードに切り替えた後、その内容を削除して次のテキストを入力します。

    ```text
   # Train and use a machine learning model
    ```

## 機械学習モデルのトレーニング

まず、"回帰" アルゴリズムを使用して糖尿病患者の目的反応 (ベースラインから 1 年後の病気の進行度の定量的尺度) を予測する機械学習モデルをトレーニングしましょう**

1. ノートブックで、最新のセルの下にある **[+ コード]** アイコンを使用して、新しいコード セルをノートブックに追加します。

    > **ヒント**: **[+ コード]** アイコンを表示するには、マウスを現在のセルの出力のすぐ下の左側に移動します。 または、メニュー バーの **[編集]** タブで、**[+ コード セルの追加]** を選択します。

1. 次のコードを入力して、データを読み込んで準備し、それを使用してモデルをトレーニングします。

    ```python
   import pandas as pd
   import mlflow
   from sklearn.model_selection import train_test_split
   from sklearn.tree import DecisionTreeRegressor
   from mlflow.models.signature import ModelSignature
   from mlflow.types.schema import Schema, ColSpec

   # Get the data
   blob_account_name = "azureopendatastorage"
   blob_container_name = "mlsamples"
   blob_relative_path = "diabetes"
   blob_sas_token = r""
   wasbs_path = f"wasbs://%s@%s.blob.core.windows.net/%s" % (blob_container_name, blob_account_name, blob_relative_path)
   spark.conf.set("fs.azure.sas.%s.%s.blob.core.windows.net" % (blob_container_name, blob_account_name), blob_sas_token)
   df = spark.read.parquet(wasbs_path).toPandas()

   # Split the features and label for training
   X, y = df[['AGE','SEX','BMI','BP','S1','S2','S3','S4','S5','S6']].values, df['Y'].values
   X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.30, random_state=0)

   # Train the model in an MLflow experiment
   experiment_name = "experiment-diabetes"
   mlflow.set_experiment(experiment_name)
   with mlflow.start_run():
       mlflow.autolog(log_models=False)
       model = DecisionTreeRegressor(max_depth=5)
       model.fit(X_train, y_train)
       
       # Define the model signature
       input_schema = Schema([
           ColSpec("integer", "AGE"),
           ColSpec("integer", "SEX"),\
           ColSpec("double", "BMI"),
           ColSpec("double", "BP"),
           ColSpec("integer", "S1"),
           ColSpec("double", "S2"),
           ColSpec("double", "S3"),
           ColSpec("double", "S4"),
           ColSpec("double", "S5"),
           ColSpec("integer", "S6"),
        ])
       output_schema = Schema([ColSpec("integer")])
       signature = ModelSignature(inputs=input_schema, outputs=output_schema)
   
       # Log the model
       mlflow.sklearn.log_model(model, "model", signature=signature)
    ```

1. セルの左側にある **[&#9655;] (セルの実行)** ボタンを使用して実行します。 代わりに、キーボードで **SHIFT** + **ENTER** キーを押してセルを実行することもできます。

    > **注**: このセッション内で Spark コードを実行したのはこれが最初であるため、Spark プールを起動する必要があります。 これは、セッション内での最初の実行が完了するまで 1 分ほどかかる場合があることを意味します。 それ以降は、短時間で実行できます。

1. セル出力の下にある **[+ コード]** アイコンを使用して新しいコード セルをノートブックに追加し、次のコードを入力して、前のセルの実験によってトレーニングされたモデルを登録します。

    ```python
   # Get the most recent experiement run
   exp = mlflow.get_experiment_by_name(experiment_name)
   last_run = mlflow.search_runs(exp.experiment_id, order_by=["start_time DESC"], max_results=1)
   last_run_id = last_run.iloc[0]["run_id"]

   # Register the model that was trained in that run
   print("Registering the model from run :", last_run_id)
   model_uri = "runs:/{}/model".format(last_run_id)
   mv = mlflow.register_model(model_uri, "diabetes-model")
   print("Name: {}".format(mv.name))
   print("Version: {}".format(mv.version))
    ```

    これで、モデルが **diabetes-model** としてワークスペースに保存されました。 必要に応じて、ワークスペースの参照機能を使用して、ワークスペース内のモデルを検索し、UI を使用してモデルを確認できます。

## レイクハウス内にテスト データセットを作成する

モデルを使用するには、糖尿病診断を予測する必要がある患者の詳細のデータセットが必要になります。 このデータセットを、Microsoft Fabric レイクハウス内のテーブルとして作成します。

1. ノートブック エディターの左側の **[エクスプローラー]** ペインで、**[+ データ ソース]** を選択してレイクハウスを追加します。
1. **[新しいレイクハウス]** を選択して **[追加]** を選択し、任意の有効な名前で新しい**レイクハウス**を作成します。
1. 現在のセッションを停止するように求められたら、**[今すぐ停止]** を選択してノートブックを再起動します。
1. レイクハウスが作成され、ノートブックにアタッチされたら、新しいコード セルを追加して次のコードを実行してデータセットを作成し、それをレイクハウス内のテーブルに保存します。

    ```python
   from pyspark.sql.types import IntegerType, DoubleType

   # Create a new dataframe with patient data
   data = [
       (62, 2, 33.7, 101.0, 157, 93.2, 38.0, 4.0, 4.8598, 87),
       (50, 1, 22.7, 87.0, 183, 103.2, 70.0, 3.0, 3.8918, 69),
       (76, 2, 32.0, 93.0, 156, 93.6, 41.0, 4.0, 4.6728, 85),
       (25, 1, 26.6, 84.0, 198, 131.4, 40.0, 5.0, 4.8903, 89),
       (53, 1, 23.0, 101.0, 192, 125.4, 52.0, 4.0, 4.2905, 80),
       (24, 1, 23.7, 89.0, 139, 64.8, 61.0, 2.0, 4.1897, 68),
       (38, 2, 22.0, 90.0, 160, 99.6, 50.0, 3.0, 3.9512, 82),
       (69, 2, 27.5, 114.0, 255, 185.0, 56.0, 5.0, 4.2485, 92),
       (63, 2, 33.7, 83.0, 179, 119.4, 42.0, 4.0, 4.4773, 94),
       (30, 1, 30.0, 85.0, 180, 93.4, 43.0, 4.0, 5.3845, 88)
   ]
   columns = ['AGE','SEX','BMI','BP','S1','S2','S3','S4','S5','S6']
   df = spark.createDataFrame(data, schema=columns)

   # Convert data types to match the model input schema
   df = df.withColumn("AGE", df["AGE"].cast(IntegerType()))
   df = df.withColumn("SEX", df["SEX"].cast(IntegerType()))
   df = df.withColumn("BMI", df["BMI"].cast(DoubleType()))
   df = df.withColumn("BP", df["BP"].cast(DoubleType()))
   df = df.withColumn("S1", df["S1"].cast(IntegerType()))
   df = df.withColumn("S2", df["S2"].cast(DoubleType()))
   df = df.withColumn("S3", df["S3"].cast(DoubleType()))
   df = df.withColumn("S4", df["S4"].cast(DoubleType()))
   df = df.withColumn("S5", df["S5"].cast(DoubleType()))
   df = df.withColumn("S6", df["S6"].cast(IntegerType()))

   # Save the data in a delta table
   table_name = "diabetes_test"
   df.write.format("delta").mode("overwrite").saveAsTable(table_name)
   print(f"Spark dataframe saved to delta table: {table_name}")
    ```

1. コードが完了したら、**[レイクハウス エクスプローラー]** ペインの **[テーブル]** の横にある **[...]** を選択して、**[更新]** を選択します。 **diabetes_test** テーブルが表示されるはずです。
1. 左側のペインで **diabetes_test** テーブルを展開して、テーブルが含むすべてのフィールドを表示します。

## モデルを適用して予測を生成する

これで、先ほどトレーニングしたモデルを使用して、テーブル内の患者データの行に対する糖尿病の進行の予測を生成できます。

1. 新しいコード セルを追加して次のコードを実行します。

    ```python
   import mlflow
   from synapse.ml.predict import MLFlowTransformer

   ## Read the patient features data 
   df_test = spark.read.format("delta").load(f"Tables/{table_name}")

   # Use the model to generate diabetes predictions for each row
   model = MLFlowTransformer(
       inputCols=["AGE","SEX","BMI","BP","S1","S2","S3","S4","S5","S6"],
       outputCol="predictions",
       modelName="diabetes-model",
       modelVersion=1)
   df_test = model.transform(df)

   # Save the results (the original features PLUS the prediction)
   df_test.write.format('delta').mode("overwrite").option("mergeSchema", "true").saveAsTable(table_name)
    ```

1. コードが完了したら、**[レイクハウス エクスプローラー]** ペインの **[diabetes_test]** テーブルの横にある **[...]** を選択して、**[更新]** を選択します。 新しいフィールド **predictions** が追加されました。
1. ノートブックに新しいコード セルを追加して **diabetes_test** テーブルをそこにドラッグします。 テーブルの内容を表示するために必要なコードが表示されます。 セルを実行してデータを表示します。

## リソースをクリーンアップする

この演習では、モデルを使用してバッチ予測を生成しました。

ノートブックの詳細の確認が終了したら、この演習用に作成したワークスペースを削除して構いません。

1. 左側のバーで、ワークスペースのアイコンを選択して、それに含まれるすべての項目を表示します。
2. ツール バーの **[...]** メニューで、 **[ワークスペースの設定]** を選択してください。
3. **[全般]** セクションで、**[このワークスペースの削除]** を選択します。
