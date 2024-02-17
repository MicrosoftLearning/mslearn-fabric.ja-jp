---
lab:
  title: Microsoft Fabric のノートブックを使用してデータ サイエンスのためにデータを探索する
  module: Explore data for data science with notebooks in Microsoft Fabric
---

# Microsoft Fabric のノートブックを使用してデータ サイエンスのためにデータを探索する

このラボでは、データ探索のためにノートブックを使います。 ノートブックは、対話形式でデータを探索および分析するための強力なツールです。 この演習では、ノートブックを作成し、それを使用して、データセットの探索、概要統計情報の生成、データをより詳細に理解するための視覚化の作成を行う方法について説明します。 このラボを終了すると、ノートブックを使ってデータの探索と分析を行う方法を確実に理解できます。

このラボの所要時間は約 **30** 分です。

> **注**:この演習を完了するには、Microsoft の "学校" または "職場" アカウントが必要です。**** お持ちでない場合は、[Microsoft Office 365 E3 以降の試用版にサインアップ](https://www.microsoft.com/microsoft-365/business/compare-more-office-365-for-business-plans)できます。

## ワークスペースの作成

Fabric でデータを操作する前に、Fabric 試用版を有効にしてワークスペースを作成してください。

1. ブラウザーで Microsoft Fabric ホーム ページ `https://app.fabric.microsoft.com` に移動し、必要に応じて Fabric 資格情報でサインインします。
1. Fabric ホーム ページで、**[Synapse Data Science]** を選択します。
1. 左側のメニュー バーで、 **[ワークスペース]** を選択します (アイコンは &#128455; に似ています)。
1. 任意の名前で新しいワークスペースを作成し、Fabric 容量を含むライセンス モード ("試用版"、*Premium*、または *Fabric*) を選択します。**
1. 開いた新しいワークスペースは空のはずです。

    ![Fabric の空のワークスペースを示すスクリーンショット。](./Images/new-workspace.png)

## ノートブックを作成する

モデルをトレーニングするために、''*ノートブック*'' を作成できます。 ノートブックでは、''実験'' として (複数の言語で) コードを記述して実行できる対話型環境が提供されます。**

1. **Synapse Data Science** ホーム ページで、新しい**ノートブック**を作成します。

    数秒後に、1 つの ''セル'' を含む新しいノートブックが開きます。** ノートブックは、''コード'' または ''マークダウン'' (書式設定されたテキスト) を含むことができる 1 つまたは複数のセルで構成されます。** **

1. 最初のセル (現在は ''コード'' セル) を選択し、右上の動的ツール バーで **[M&#8595;]** ボタンを使用してセルを ''マークダウン'' セルに変換します。** **

    セルがマークダウン セルに変わると、それに含まれるテキストがレンダリングされます。

1. 必要に応じて、**[&#128393;]** (編集) ボタンを使用してセルを編集モードに切り替えた後、その内容を削除して次のテキストを入力します。

    ```text
   # Perform data exploration for data science

   Use the code in this notebook to perform data exploration for data science.
    ```

## データフレームにデータを読み込む

これで、データを取得するコードを実行する準備ができました。 Azure Open Datasets から [**Diabetes データセット**](https://learn.microsoft.com/azure/open-datasets/dataset-diabetes?tabs=azureml-opendatasets?azure-portal=true)を操作します。 データを読み込んだ後、データを Pandas データフレームに変換します。これは、行と列でデータを操作するための一般的な構造です。

1. ノートブックで、最新のセルの下にある **[+ コード]** アイコンを使用して、新しいコード セルをノートブックに追加します。

    > **ヒント**: **[+ コード]** アイコンを表示するには、マウスを現在のセルの出力のすぐ下の左側に移動します。 別の方法として、メニュー バーの **[編集]** タブで、**[+ コード セルの追加]** を選択します。

1. データセットをデータフレームに読み込むには、次のコードを入力します。

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

    出力には、Diabetes データセットの行と列が表示されます。 このデータは、糖尿病患者に関する 10 個のベースライン変数 (年齢、性別、BMI、平均血圧、6 つの血清測定値) と、目的反応 (ベースラインから1年後の病気の進行度の定量的尺度) で構成され、これは **Y** とラベル付けされています。

1. データは Spark データフレームとして読み込まれます。 Scikit-learn では、入力データセットが Pandas データフレームであることが想定されます。 データセットを Pandas データフレームに変換するには、次のコードを実行します。

    ```python
   df = df.toPandas()
   df.head()
    ```

## データの形式を確認する

データを読み込んだので、行と列の数、データ型、不足している値など、データセットの構造を確認できます。

1. セル出力の下にある **[+ コード]** アイコンを使用して、ノートブックに新しいコード セルを追加し、次のコードを入力します。

    ```python
   # Display the number of rows and columns in the dataset
   print("Number of rows:", df.shape[0])
   print("Number of columns:", df.shape[1])

   # Display the data types of each column
   print("\nData types of columns:")
   print(df.dtypes)
    ```

    データセットには **442 個の行**と **11 個の列**が含まれています。 つまり、データセットには 442 個のサンプルと 11 個の特徴量または変数があります。 **SEX** 変数には、カテゴリ データまたは文字列データが含まれている可能性があります。

## 不足している値がないか確認する

1. セル出力の下にある **[+ コード]** アイコンを使用して、ノートブックに新しいコード セルを追加し、次のコードを入力します。

    ```python
   missing_values = df.isnull().sum()
   print("\nMissing values per column:")
   print(missing_values)
    ```

    不足している値の有無をコードで確認します。 データセットに不足しているデータがないことを確認します。

## 数値変数の記述統計を生成する

次に、数値変数の分布を理解するために記述統計を生成してみましょう。

1. セル出力の下にある **[+ コード]** アイコンを使用して、ノートブックに新しいコード セルを追加し、次のコードを入力します。

    ```python
   df.describe()
    ```

    平均年齢は約 48.5 歳であり、標準偏差は 13.1 歳です。 最年少の個人は 19 歳で、最高齢者は 79 歳です。 平均 BMI は約 26.4 で、[WHO 基準](https://www.who.int/health-topics/obesity#tab=tab_1)に従って**過体重**のカテゴリに分類されます。 最小 BMI は 18 で、最大は 42.2 です。

## データ分布をプロットする

BMI の特徴量を検証し、分布をプロットして、その特性をより深く理解しましょう。

1. ノートブックに別のコード セルを追加します。 次に、このセルに次のコードを入力して実行します。

    ```python
   import matplotlib.pyplot as plt
   import seaborn as sns
   import numpy as np
    
   # Calculate the mean, median of the BMI variable
   mean = df['BMI'].mean()
   median = df['BMI'].median()
   
   # Histogram of the BMI variable
   plt.figure(figsize=(8, 6))
   plt.hist(df['BMI'], bins=20, color='skyblue', edgecolor='black')
   plt.title('BMI Distribution')
   plt.xlabel('BMI')
   plt.ylabel('Frequency')
    
   # Add lines for the mean and median
   plt.axvline(mean, color='red', linestyle='dashed', linewidth=2, label='Mean')
   plt.axvline(median, color='green', linestyle='dashed', linewidth=2, label='Median')
    
   # Add a legend
   plt.legend()
   plt.show()
    ```

    このグラフから、データセット内の BMI の範囲と分布を確認できます。 たとえば、BMI の大部分は 23.2 と 29.2 の範囲内にあり、データは右に歪んでいます。

## 多変量分析を実行する

散布図やボックス プロットなどの視覚化を生成して、データ内のパターンと関係を明らかにしましょう。

1. セル出力の下にある **[+ コード]** アイコンを使用して、ノートブックに新しいコード セルを追加し、次のコードを入力します。

    ```python
   import matplotlib.pyplot as plt
   import seaborn as sns

   # Scatter plot of Quantity vs. Price
   plt.figure(figsize=(8, 6))
   sns.scatterplot(x='BMI', y='Y', data=df)
   plt.title('BMI vs. Target variable')
   plt.xlabel('BMI')
   plt.ylabel('Target')
   plt.show()
    ```

    BMI が高くなると、ターゲット変数も大きくなることがわかります。これは、これら 2 つの変数間の正の線形関係を示しています。

1. ノートブックに別のコード セルを追加します。 次に、このセルに次のコードを入力して実行します。

    ```python
   import seaborn as sns
   import matplotlib.pyplot as plt
    
   fig, ax = plt.subplots(figsize=(7, 5))
    
   # Replace numeric values with labels
   df['SEX'] = df['SEX'].replace({1: 'Male', 2: 'Female'})
    
   sns.boxplot(x='SEX', y='BP', data=df, ax=ax)
   ax.set_title('Blood pressure across Gender')
   plt.tight_layout()
   plt.show()
    ```

    これらの観察は、男性と女性の患者の血圧プロファイルに違いがあることを示唆しています。 平均して、女性患者は男性患者よりも血圧が高くなっています。

1. データを集計すると、視覚化と分析を管理しやすくなります。 ノートブックに別のコード セルを追加します。 次に、このセルに次のコードを入力して実行します。

    ```python
   import matplotlib.pyplot as plt
   import seaborn as sns
    
   # Calculate average BP and BMI by SEX
   avg_values = df.groupby('SEX')[['BP', 'BMI']].mean()
    
   # Bar chart of the average BP and BMI by SEX
   ax = avg_values.plot(kind='bar', figsize=(15, 6), edgecolor='black')
    
   # Add title and labels
   plt.title('Avg. Blood Pressure and BMI by Gender')
   plt.xlabel('Gender')
   plt.ylabel('Average')
    
   # Display actual numbers on the bar chart
   for p in ax.patches:
       ax.annotate(format(p.get_height(), '.2f'), 
                   (p.get_x() + p.get_width() / 2., p.get_height()), 
                   ha = 'center', va = 'center', 
                   xytext = (0, 10), 
                   textcoords = 'offset points')
    
   plt.show()
    ```

    このグラフは、男性患者と比較して女性患者の平均血圧が高いことを示しています。 さらに、平均ボディマス指数 (BMI) が男性よりも女性の方がわずかに高いことを示しています。

1. ノートブックに別のコード セルを追加します。 次に、このセルに次のコードを入力して実行します。

    ```python
   import matplotlib.pyplot as plt
   import seaborn as sns
    
   plt.figure(figsize=(10, 6))
   sns.lineplot(x='AGE', y='BMI', data=df, errorbar=None)
   plt.title('BMI over Age')
   plt.xlabel('Age')
   plt.ylabel('BMI')
   plt.show()
    ```

    19 歳から 30 歳までの年齢グループは平均 BMI 値が最も低く、平均 BMI が最も高いのは 65 歳から 79 歳までの年齢グループです。 さらに、ほとんどの年齢グループの平均 BMI が過体重の範囲にあることがわかります。

## 相関関係分析

さまざまな特徴量の間の相関関係を計算して、それらの関係と依存関係を理解しましょう。

1. セル出力の下にある **[+ コード]** アイコンを使用して、ノートブックに新しいコード セルを追加し、次のコードを入力します。

    ```python
   df.corr(numeric_only=True)
    ```

1. ヒートマップは、変数ペア間の関係の強さと方向をすばやく視覚化するための便利なツールです。 強い正または負の相関関係を強調表示するだけでなく、相関関係がないペアを識別することもできます。 ヒートマップを作成するには、ノートブックに別のコード セルを追加し、次のコードを入力します。

    ```python
   plt.figure(figsize=(15, 7))
   sns.heatmap(df.corr(numeric_only=True), annot=True, vmin=-1, vmax=1, cmap="Blues")
    ```

    S1 および S2 変数と正の相関関係は **0.89** と高く、同じ方向に移動することを示します。 S1 が増加すると S2 も増加する傾向があり、その逆も同様です。 さらに、S3 と S4 の間には、**-0.73** という強い負の相関関係があります。 これは、S3 が増加するにつれて S4 が減少する傾向があることを意味します。

## ノートブックを保存して Spark セッションを終了する

データの探索が済んだので、わかりやすい名前でノートブックを保存し、Spark セッションを終了できます。

1. ノートブックのメニュー バーで、[⚙️] (**設定**) アイコンを使用してノートブックの設定を表示します。
2. ノートブックの **[名前]** を「**データ サイエンスのためにデータを探索する**」に設定して、設定ペインを閉じます。
3. ノートブック メニューで、 **[セッションの停止]** を選択して Spark セッションを終了します。

## リソースをクリーンアップする

この演習では、データ探索のためにノートブックを作成して使用しました。 また、概要統計情報を計算し、データ内のパターンと関係をより深く理解するために視覚化を作成するコードも実行しました。

モデルと実験の探索が完了したら、この演習用に作成したワークスペースを削除できます。

1. 左側のバーで、ワークスペースのアイコンを選択して、それに含まれるすべての項目を表示します。
2. ツール バーの **[...]** メニューで、 **[ワークスペースの設定]** を選択してください。
3. **[その他]** セクションで、 **[このワークスペースの削除]** を選択してください。
