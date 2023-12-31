---
lab:
  title: Microsoft Fabric で Data Wrangler を使用してデータを前処理する
  module: Preprocess data with Data Wrangler in Microsoft Fabric
---

# ノートブックを使用して Microsoft Fabric でモデルをトレーニングする

このラボでは、Microsoft Fabric で Data Wrangler を使用してデータを前処理し、一般的なデータ サイエンス操作のライブラリを使用してコードを生成する方法について説明します。

このラボは完了するまで、約 **30** 分かかります。

> **注**: この演習を完了するには、Microsoft Fabric ライセンスが必要です。 無料の Fabric 試用版ライセンスを有効にする方法の詳細については、[Fabric の概要](https://learn.microsoft.com/fabric/get-started/fabric-trial)に関するページを参照してください。 これを行うには、Microsoft の "*学校*" または "*職場*" アカウントが必要です。 お持ちでない場合は、[Microsoft Office 365 E3 以降の試用版にサインアップ](https://www.microsoft.com/microsoft-365/business/compare-more-office-365-for-business-plans)できます。

## ワークスペースの作成

Fabric でデータを操作する前に、Fabric 試用版を有効にしてワークスペースを作成してください。

1. `https://app.fabric.microsoft.com` で [Microsoft Fabric](https://app.fabric.microsoft.com) にサインインし、 **[Power BI]** を選択してください。
2. 左側のメニュー バーで、 **[ワークスペース]** を選択します (アイコンは &#128455; に似ています)。
3. 任意の名前で新しいワークスペースを作成し、Fabric 容量を含むライセンス モード ("試用版"、*Premium*、または *Fabric*) を選択してください。**
4. 新しいワークスペースを開くと次に示すように空のはずです。

    ![Power BI の空のワークスペースのスクリーンショット。](./Images/new-workspace.png)

## レイクハウスを作成してファイルをアップロードする

ワークスペースが作成されたので、次はポータルの *[Data Science]* エクスペリエンスに切り替えて、分析するデータ ファイルのデータ レイクハウスを作成します。

1. Power BI ポータルの左下にある **[Power BI]** アイコンを選択し、 **[Data Engineering]** エクスペリエンスに切り替えます。
1. **[Data Engineering]** ホーム ページで、任意の名前で新しい**レイクハウス**を作成します。

    1 分ほどすると、**Tables** や **Files** のない新しいレイクハウスが作成されます。 分析のために、データ レイクハウスにいくつかのデータを取り込む必要があります。 これを行う複数の方法がありますが、この演習では、ローカル コンピューター (該当する場合はラボ VM) にテキスト ファイルのフォルダーをダウンロードして抽出し、レイクハウスにアップロードします。

1. すること: [https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/XXXXX.csv](https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/XXXXX.csv) からこの演習用の CSV ファイル `dominicks_OJ.csv` をダウンロードして保存します。


1. レイクハウスを含む Web ブラウザー タブに戻り、 **[レイク ビュー]** ペインの **Files** ノードの **[...]** メニューで **[アップロード]** と **[ファイルのアップロード]** を選択し、ローカル コンピューター (または該当する場合はラボ VM) からレイクハウスに **dominicks_OJ.csv** ファイルをアップロードします。
6. ファイルがアップロードされた後、**Files** を展開し、CSV ファイルがアップロードされたことを確認します。

## ノートブックを作成する

モデルをトレーニングするために、''*ノートブック*'' を作成できます。 ノートブックでは、''実験'' として (複数の言語で) コードを記述して実行できる対話型環境が提供されます。**

1. Power BI ポータルの左下にある **[Data Engineering]** アイコンを選択し、 **[Data Science]** エクスペリエンスに切り替えます。

1. **[Data Science]** ホーム ページで、新しい**ノートブック**を作成します。

    数秒後に、1 つの ''セル'' を含む新しいノートブックが開きます。** ノートブックは、''コード'' または ''マークダウン'' (書式設定されたテキスト) を含むことができる 1 つまたは複数のセルで構成されます。** **

1. 最初のセル (現在は ''コード'' セル) を選択し、右上の動的ツール バーで **[M&#8595;]** ボタンを使用してセルを ''マークダウン'' セルに変換します。** **

    セルがマークダウン セルに変わると、それに含まれるテキストがレンダリングされます。

1. **[&#128393;]** (編集) ボタンを使用してセルを編集モードに切り替え、その内容を削除して次のテキストを入力します。

    ```text
   # Train a machine learning model and track with MLflow

   Use the code in this notebook to train and track models.
    ``` 

## データフレームにデータを読み込む

これで、データを準備してモデルをトレーニングするためのコードを実行する準備ができました。 データを操作するには、''データフレーム'' を使用します。** Spark のデータフレームは Python の Pandas データフレームに似ており、行と列のデータを操作するための共通の構造が提供されます。

1. **[レイクハウスの追加]** ペインで、 **[追加]** を選択してレイクハウスを追加します。
1. **[既存のレイクハウス]** を選び、 **[追加]** を選択します。
1. 前のセクションで作成したレイクハウスを選択します。
1. **Files** フォルダーを展開して、ノートブック エディターの横に CSV ファイルが表示されるようにします。
1. **churn.csv** の **[...]** メニューで、 **[データの読み込み]**  >  **[Pandas]** の順に選択します。 次のコードを含む新しいコード セルがノートブックに追加されるはずです。

    ```python
    import pandas as pd
    df = pd.read_csv("/lakehouse/default/" + "Files/dominicks_OJ.csv") 
    display(df.head(5))
    ```

    > **ヒント**: 左側のファイルを含むペインは、その **[<<]**  アイコンを使用して非表示にすることができます。 そうすると、ノートブックに集中するのに役立ちます。

1. セルの左側にある **[&#9655;] (セルの実行)** ボタンを使用して実行します。

    > **注**: このセッション内で Spark コードを実行したのはこれが最初であるため、Spark プールを起動する必要があります。 これは、セッション内での最初の実行が完了するまで 1 分ほどかかる場合があることを意味します。 それ以降は、短時間で実行できます。

## 概要の統計情報を表示する

Data Wrangler が起動すると、データフレームの説明的な概要が [概要] パネルに生成されます。 

1. 上部のメニューで **[データ]** を選択し、 **[Data Wrangler]** ドロップダウンで `df` データセットを参照します。

    ![Data Wrangler の起動オプションのスクリーンショット。](./Images/launch-data-wrangler.png)

1. **[Large HH]** 列を選択し、この特徴のデータ分布を簡単に判断できることを確認します。

    ![特定の列のデータ分布を示す Data Wrangler ページのスクリーンショット。](./Images/data-wrangler-distribution.png)

    この特徴は正規分布に従っている点に注意してください。

1. [概要] サイド パネルを確認します。パーセンタイル範囲に注意してください。 

    ![概要パネルの詳細を示す Data Wrangler ページのスクリーンショット。](./Images/data-wrangler-summary.png)

    ほとんどのデータが **0.098** から **0.132** の間にあり、データ値の 50% がその範囲内にあることがわかります。

## テキスト データの書式設定

次に、いくつかの変換を **Brand** という特徴に適用してみましょう。

1. **[Data Wrangler]** ページで、`Brand` という特徴を選択します。

1. **[操作]** パネルに移動し、 **[検索と置換]** を展開して、 **[検索と置換]** を選択します。

1. **[検索と置換]** パネルで、次のプロパティを変更します。
    
    - **元の値:** "."
    - **新しい値:** " " (スペース文字)

    ![[検索と置換] パネルを示す [Data Wrangler] ページのスクリーンショット。](./Images/data-wrangler-find.png)

    操作の結果は、表示グリッドに自動的にプレビュー表示されます。

1. **[適用]** を選択します。

1. **[操作]** パネルに戻り、 **[書式設定]** を展開します。

1. **[Convert text to capital case] (テキストを大文字に変換する)** を選択します。

1. **[Convert text to capital case] (テキストを大文字に変換する)** パネルで、 **[適用]** を選択します。

1. **[Add code to notebook] (ノートブックにコードを追加する)** を選択します。 さらに、変換したデータセットを .csv ファイルとして保存することもできます。

    コードがノートブックのセルに自動的にコピーされ、使用できる状態になっていることに注意してください。

1. コードを実行します。

> **重要:** 生成されたコードは、元のデータフレームを上書きしません。 

コードを簡単に生成し、Data Wrangler 操作を使用してテキスト データを処理する方法について説明しました。 

## ワンホット エンコーダー変換を適用する

次に、前処理ステップとしてワンホット エンコーダー変換を適用するコードを生成してみましょう。

1. 上部のメニューで **[データ]** を選択し、 **[Data Wrangler]** ドロップダウンで `df` データセットを参照します。

1. **[操作]** パネルで、 **[数式]** を展開します。

1. **[One-hot encode] (ワンホット エンコード)** を選択します。

1. **[One-hot encode] (ワンホット エンコード)** パネルで、 **[適用]** を選択します。

    Data Wrangler 表示グリッドの末尾に移動します。 3 つの新しい特徴が追加され、`Brand` という特徴が削除されていることに注意してください。

1. **[Add code to notebook] (ノートブックにコードを追加する)** を選択します。

1. コードを実行します。

## 並べ替えとフィルター処理の操作

1. 上部のメニューで **[データ]** を選択し、 **[Data Wrangler]** ドロップダウンで `df` データセットを参照します。

1. **[操作]** パネルで、 **[並べ替えとフィルター]** を展開します。

1. **[フィルター]** を選択します。

1. **[フィルター]** パネルで、次の条件を追加します。
    
    - **ターゲット列:** Store
    - **操作:** 次の値と等しい
    - **値:** 2

1. **[適用]** を選択します。

    Data Wrangler 表示グリッドでの変更に注意してください。

1. **[操作]** パネルに戻り、 **[並べ替えとフィルター]** を展開します。

1. **[値の並べ替え]** を選択します

1. **[Price]** パネルで、次の条件を追加します。
    
    - **列名:** Price
    - **並べ替え順序:** 降順

1. **[適用]** を選択します。

    Data Wrangler 表示グリッドでの変更に注意してください。

## データの集計

1. **[操作]** パネルに戻り、 **[Group by and aggregate] (グループ化と集計)** を選択します。

1. **[Columns to group by] (グループ化する列)** プロパティで、`Store` という特徴を選択します。

1. **[集計の追加]** を選択します。

1. **[集計する列]** プロパティで、`Quantity` という特徴を選択します。

1. **[集計の種類]** プロパティの **[カウント]** を選択します。

1. **[適用]** を選択します。 

    Data Wrangler 表示グリッドでの変更に注意してください。

## ステップを参照して削除する

間違いがあったために前のステップで作成した集計を削除する必要があるとします。 削除するには、次の手順に従います。

1. **[Cleaning steps] (クリーニング ステップ)** パネルを展開します。

1. **[Group by and aggregate (default)] (グループ化と集計 (既定))** ステップを選択します。

1. それを、削除アイコンを選択して削除します。

    ![[検索と置換] パネルを示す [Data Wrangler] ページのスクリーンショット。](./Images/data-wrangler-delete.png)

    > **重要:** グリッド ビューと概要は、現在のステップに限定されています。

    変更が前のステップ ( **[値の並べ替え]** ステップ) に戻されていることに注意してください。

1. **[Add code to notebook] (ノートブックにコードを追加する)** を選択します。

1. コードを実行します。

一部の前処理操作のコードを生成し、関数としてノートブックに保存し直しました。それを再利用または必要に応じて変更できます。

## ノートブックを保存して Spark セッションを終了する

モデリングのためのデータの前処理が済んだので、わかりやすい名前でノートブックを保存し、Spark セッションを終了できます。

1. ノートブックのメニュー バーで、[⚙️] (**設定**) アイコンを使用してノートブックの設定を表示します。
2. ノートブックの **[名前]** を「**Data Wrangler でデータを前処理する**」に設定して、設定ペインを閉じます。
3. ノートブック メニューで、 **[セッションの停止]** を選択して Spark セッションを終了します。

## リソースをクリーンアップする

この演習では、ノートブックを作成し、Data Wrangler を使用して機械学習モデルのデータを探索し、前処理しました。

前処理ステップの探索が終了したら、この演習用に作成したワークスペースを削除できます。

1. 左側のバーで、ワークスペースのアイコンを選択して、それに含まれるすべての項目を表示します。
2. ツール バーの **[...]** メニューで、 **[ワークスペースの設定]** を選択します。
3. **[その他]** セクションで、 **[このワークスペースの削除]** を選択します。
