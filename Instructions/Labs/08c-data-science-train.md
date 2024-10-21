---
lab:
  title: Microsoft Fabric で MLflow を使用して機械学習モデルをトレーニングおよび追跡する
  module: Train and track machine learning models with MLflow in Microsoft Fabric
---

# Microsoft Fabric で MLflow を使用して機械学習モデルをトレーニングおよび追跡する

このラボでは、機械学習モデルをトレーニングして、糖尿病の定量的尺度を予測します。 scikit-learn を使用して回帰モデルをトレーニングし、モデルを MLflow で追跡および比較します。

このラボを完了することで、機械学習とモデル追跡に関するハンズオンの経験を積み、Microsoft Fabric で*ノートブック*、*実験*、*モデル*を操作する方法を学習できます。

このラボは完了するまで、約 **25** 分かかります。

> **注**:この演習を完了するには、[Microsoft Fabric 試用版](https://learn.microsoft.com/fabric/get-started/fabric-trial)が必要です。

## ワークスペースの作成

Fabric でデータを操作する前に、Fabric 試用版を有効にしてワークスペースを作成してください。

1. ブラウザーの `https://app.fabric.microsoft.com/home?experience=fabric` で [Microsoft Fabric ホーム ページ](https://app.fabric.microsoft.com/home?experience=fabric)に移動し、必要に応じて Fabric 資格情報でサインインします。
1. Fabric ホーム ページで、**[Synapse Data Science]** を選択します。
1. 左側のメニュー バーで、 **[ワークスペース]** を選択します (アイコンは &#128455; に似ています)。
1. 任意の名前で新しいワークスペースを作成し、Fabric 容量を含むライセンス モード ("試用版"、*Premium*、または *Fabric*) を選択します。**
1. 開いた新しいワークスペースは空のはずです。

    ![Fabric の空のワークスペースを示すスクリーンショット。](./Images/new-workspace.png)

## ノートブックを作成する

モデルをトレーニングするために、''*ノートブック*'' を作成できます。 ノートブックは、(複数の言語で) コードを記述して実行できる対話型環境を提供します。

1. **Synapse Data Science** ホーム ページで、新しい**ノートブック**を作成します。

    数秒後に、1 つの ''セル'' を含む新しいノートブックが開きます。** ノートブックは、''コード'' または ''マークダウン'' (書式設定されたテキスト) を含むことができる 1 つまたは複数のセルで構成されます。** **

1. 最初のセル (現在は ''コード'' セル) を選択し、右上の動的ツール バーで **[M&#8595;]** ボタンを使用してセルを ''マークダウン'' セルに変換します。** **

    セルがマークダウン セルに変わると、それに含まれるテキストがレンダリングされます。

1. 必要に応じて、**[&#128393;]** (編集) ボタンを使用してセルを編集モードに切り替えた後、その内容を削除して次のテキストを入力します。

    ```text
   # Train a machine learning model and track with MLflow
    ```

## データフレームにデータを読み込む

これで、データを取得してモデルをトレーニングするためのコードを実行する準備ができました。 Azure Open Datasets から [Diabetes データセット](https://learn.microsoft.com/azure/open-datasets/dataset-diabetes?tabs=azureml-opendatasets?azure-portal=true)を操作します。 データを読み込んだ後、データを Pandas データフレームに変換します。これは、行と列でデータを操作するための一般的な構造です。

1. ノートブックで、最新のセル出力の下にある **[+ コード]** アイコンを使用して、新しいコード セルをノートブックに追加します。

    > **ヒント**: **[+ コード]** アイコンを表示するには、マウスを現在のセルの出力のすぐ下かつ左に移動します。 別の方法として、メニュー バーの **[編集]** タブで、**[+ コード セルの追加]** を選択します。

1. それに次のコードを入力します。

    ```python
   # Azure storage access info for open dataset diabetes
   blob_account_name = "azureopendatastorage"
   blob_container_name = "mlsamples"
   blob_relative_path = "diabetes"
   blob_sas_token = r"" # Blank since container is Anonymous access
    
   # Set Spark config to access  blob storage
   wasbs_path = f"wasbs://%s@%s.blob.core.windows.net/%s" % (blob_container_name, blob_account_name, blob_relative_path)
   spark.conf.set("fs.azure.sas.%s.%s.blob.core.windows.net" % (blob_container_name, blob_account_name), blob_sas_token)
   print("Remote blob path: " + wasbs_path)
    
   # Spark read parquet, note that it won't load any data yet by now
   df = spark.read.parquet(wasbs_path)
    ```

1. セルの左側にある **[&#9655;] (セルの実行)** ボタンを使用して実行します。 代わりに、キーボードで **SHIFT** + **ENTER** キーを押してセルを実行することもできます。

    > **注**: このセッション内で Spark コードを実行したのはこれが最初であるため、Spark プールを起動する必要があります。 これは、セッション内での最初の実行が完了するまで 1 分ほどかかる場合があることを意味します。 それ以降は、短時間で実行できます。

1. セル出力の下にある **[+ コード]** アイコンを使用して、ノートブックに新しいコード セルを追加し、次のコードを入力します。

    ```python
   display(df)
    ```

1. セル コマンドが完了したら、セルの下にある出力を確認します。これは次のようになるはずです。

    |AGE|SEX|BMI|BP|S1|S2|S3|S4|S5|S6|Y|
    |---|---|---|--|--|--|--|--|--|--|--|
    |59|2|32.1|101.0|157|93.2|38.0|4.0|4.8598|87|151|
    |48|1|21.6|87.0|183|103.2|70.0|3.0|3.8918|69|75|
    |72|2|30.5|93.0|156|93.6|41.0|4.0|4.6728|85|141|
    |24|1|25.3|84.0|198|131.4|40.0|5.0|4.8903|89|206|
    |50|1|23.0|101.0|192|125.4|52.0|4.0|4.2905|80|135|
    | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |

    出力には、Diabetes データセットの行と列が表示されます。

1. データは Spark データフレームとして読み込まれます。 Scikit-learn では、入力データセットが Pandas データフレームであることが想定されます。 データセットを Pandas データフレームに変換するには、次のコードを実行します。

    ```python
   import pandas as pd
   df = df.toPandas()
   df.head()
    ```

## 機械学習モデルのトレーニング

データを読み込んだので、それを使用して機械学習モデルをトレーニングし、糖尿病の定量的尺度を予測できます。 scikit-learn ライブラリを使用して回帰モデルをトレーニングし、MLflow でモデルを追跡します。

1. 次のコードを実行して、データをトレーニング データセットとテスト データセットに分割し、予測するラベルから特徴量を分離します。

    ```python
   from sklearn.model_selection import train_test_split
    
   X, y = df[['AGE','SEX','BMI','BP','S1','S2','S3','S4','S5','S6']].values, df['Y'].values
    
   X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.30, random_state=0)
    ```

1. ノートブックに新しいコード セルをもう 1 つ追加し、そこに次のコードを入力して実行します。

    ```python
   import mlflow
   experiment_name = "experiment-diabetes"
   mlflow.set_experiment(experiment_name)
    ```

    このコードでは、**experiment-diabetes** という名前の MLflow 実験を作成します。 この実験でモデルが追跡されます。

1. ノートブックに新しいコード セルをもう 1 つ追加し、そこに次のコードを入力して実行します。

    ```python
   from sklearn.linear_model import LinearRegression
    
   with mlflow.start_run():
      mlflow.autolog()
    
      model = LinearRegression()
      model.fit(X_train, y_train)
    
      mlflow.log_param("estimator", "LinearRegression")
    ```

    このコードは、線形回帰を使用して回帰モデルをトレーニングします。 パラメーター、メトリック、および成果物は、MLflow で自動的にログに記録されます。 さらに、**estimator** という名前のパラメーターを値 *LinearRegression* でログしています。

1. ノートブックに新しいコード セルをもう 1 つ追加し、そこに次のコードを入力して実行します。

    ```python
   from sklearn.tree import DecisionTreeRegressor
    
   with mlflow.start_run():
      mlflow.autolog()
    
      model = DecisionTreeRegressor(max_depth=5) 
      model.fit(X_train, y_train)
    
      mlflow.log_param("estimator", "DecisionTreeRegressor")
    ```

    このコードは、デシジョン ツリー リグレッサーを使用して回帰モデルをトレーニングします。 パラメーター、メトリック、および成果物は、MLflow で自動的にログに記録されます。 さらに、**estimator** という名前のパラメーターを値 *DecisionTreeRegressor* でログしています。

## MLflow を使用して実験を検索および表示する

MLflow でモデルをトレーニングして追跡したら、MLflow ライブラリを使用して実験とその詳細を取得できます。

1. すべての実験を一覧表示するには、次のコードを使用します。

    ```python
   import mlflow
   experiments = mlflow.search_experiments()
   for exp in experiments:
       print(exp.name)
    ```

1. 特定の実験を取得するには、その名前で取得できます。

    ```python
   experiment_name = "experiment-diabetes"
   exp = mlflow.get_experiment_by_name(experiment_name)
   print(exp)
    ```

1. 実験名を使用すると、その実験のすべてのジョブを取得できます。

    ```python
   mlflow.search_runs(exp.experiment_id)
    ```

1. ジョブの実行と出力をより簡単に比較するために、結果を並べ替える検索を構成できます。 たとえば、次のセルは結果を *start_time* の順序で並べ替え、最大で 2 個の結果しか表示しません。

    ```python
   mlflow.search_runs(exp.experiment_id, order_by=["start_time DESC"], max_results=2)
    ```

1. 最後に、複数のモデルの評価メトリックを並べてプロットして、モデルを簡単に比較できます。

    ```python
   import matplotlib.pyplot as plt
   
   df_results = mlflow.search_runs(exp.experiment_id, order_by=["start_time DESC"], max_results=2)[["metrics.training_r2_score", "params.estimator"]]
   
   fig, ax = plt.subplots()
   ax.bar(df_results["params.estimator"], df_results["metrics.training_r2_score"])
   ax.set_xlabel("Estimator")
   ax.set_ylabel("R2 score")
   ax.set_title("R2 score by Estimator")
   for i, v in enumerate(df_results["metrics.training_r2_score"]):
       ax.text(i, v, str(round(v, 2)), ha='center', va='bottom', fontweight='bold')
   plt.show()
    ```

    出力は次の画像のようになるはずです。

    ![プロットされた評価メトリックのスクリーンショット。](./Images/data-science-metrics.png)

## 実験を調べる

Microsoft Fabric では、すべての実験を追跡し、それらを視覚的に調べることができます。

1. 左側のメニュー バーからワークスペースに移動します。
1. **[experiment-diabetes]** 実験を選択してこれを開きます。

    > **ヒント:** ログに記録された実験の実行が表示されない場合は、ページを最新の情報に更新します。

1. **[表示]** タブを選択します。
1. **[実行リスト]** を選択します。
1. 各ボックスをオンにして、最新の 2 つの実行を選択します。

    その結果、最後の 2 つの実行が **[メトリック比較]** ペインで相互に比較されます。 既定では、メトリックは実行名でプロットされます。

1. 各実行の平均絶対誤差を視覚化するグラフの **[&#128393;]** (編集) ボタンを選択します。
1. **[視覚化の種類]** を **[バー]** に変更します。
1. **[X 軸]** を **[estimator]** に変更します。
1. **[置換]** を選択し、新しいグラフを調べます。
1. 必要に応じて、 **[メトリック比較]** ペインの他のグラフに対してこれらの手順を繰り返すことができます。

ログに記録された推定器ごとのパフォーマンス メトリックをプロットすることで、どのアルゴリズムがより優れたモデルにつながったかを確認できます。

## モデルを保存する

実験の実行でトレーニングした機械学習モデルを比較した後、最適なパフォーマンス モデルを選ぶことができます。 最適なパフォーマンス モデルを使用するには、モデルを保存し、それを使用して予測を生成します。

1. 実験の概要で、 **[表示]** タブが選択されていることを確かめます。
1. **[実行の詳細]** を選択します。
1. トレーニング R2 スコアが最も高い実行を選択します。
1. **[実行をモデルとして保存]** ボックスで **[保存]** を選択します (これを表示するには、右にスクロールする必要があるかもしれません)。
1. 新しく開いたポップアップ ウィンドウで **[新しいモデルの作成]** を選択します。
1. **model** フォルダーを選択します。
1. モデルに `model-diabetes` という名前を付け、 **[保存]** を選択します。
1. モデルが作成されると画面の右上に表示される通知内の **[ML モデルの表示]** を選択します。 ウィンドウを最新の情報に更新することもできます。 保存されたモデルは、 **[モデル バージョン]** の下にリンクされています。

モデル、実験、実験の実行がリンクされていることに注意してください。これにより、モデルのトレーニング方法を確認できます。

## ノートブックを保存して Spark セッションを終了する

モデルのトレーニングと評価が完了したので、わかりやすい名前でノートブックを保存し、Spark セッションを終了できます。

1. ノートブックに戻り、ノートブックのメニュー バーで、⚙️ **[設定]** アイコンを使用してノートブックの設定を表示します。
2. ノートブックの **[名前]** を **[モデルのトレーニングと比較]** に設定し、設定ペインを閉じます。
3. ノートブック メニューで、 **[セッションの停止]** を選択して Spark セッションを終了します。

## リソースをクリーンアップする

この演習では、ノートブックを作成し、機械学習モデルをトレーニングしました。 scikit-learn を使用してモデルをトレーニングし、MLflow でそのパフォーマンスを追跡しました。

モデルと実験の確認が完了したら、この演習用に作成したワークスペースを削除して構いません。

1. 左側のバーで、ワークスペースのアイコンを選択して、それに含まれるすべての項目を表示します。
2. ツール バーの **[...]** メニューで、 **[ワークスペースの設定]** を選択してください。
3. **[全般]** セクションで、**[このワークスペースの削除]** を選択します。
