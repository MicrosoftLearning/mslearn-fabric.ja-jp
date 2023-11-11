
## ***作業ドラフト**
---
lab:
  title: リアルタイム ダッシュボード
  module: Query data from a Kusto Query database in Microsoft Fabric
---
# Microsoft Fabric での Kusto データベースのクエリの概要
リアルタイム ダッシュボードでは、Kusto 照会言語 (KQL) を使って構造化データと非構造化データの両方を取得して Microsoft Fabric 内から分析情報を収集し、Power BI 内のスライサーと同様にリンクできるグラフ、散布図、テーブルなどをパネル内にレンダリングできます。 

このラボの所要時間は約 **25** 分です。

## ワークスペースの作成

Fabric でデータを操作する前に、Fabric 試用版を有効にしてワークスペースを作成してください。

1. `https://app.fabric.microsoft.com` で [Microsoft Fabric](https://app.fabric.microsoft.com) にサインインし、 **[Power BI]** を選択してください。
2. 左側のメニュー バーで、 **[ワークスペース]** を選択します (アイコンは &#128455; に似ています)。
3. 任意の名前で新しいワークスペースを作成し、Fabric 容量を含むライセンス モード ("試用版"、*Premium*、または *Fabric*) を選択してください。**
4. 新しいワークスペースを開くと次に示すように空のはずです。

    ![Power BI の空のワークスペースのスクリーンショット。](./Images/new-workspace.png)

このラボでは、Fabric のリアルタイム分析 (RTA) を使って、サンプルのイベントストリームから KQL データベースを作成します。 Real-Time Analytics には、RTA の機能を探索するために使用できる便利なサンプル データセットが用意されています。 このサンプル データを使い、リアルタイム データを分析する KQL と SQL のクエリとクエリセットを作成し、ダウンストリームのプロセスで他の目的に使用できるようにします。

## KQL データベースを作成する

1. **Real-Time Analytics** で、 **[KQL データベース]** ボックスを選択します。

   ![KQL データベースの選択の画像](./Images/select-kqldatabase.png)

2. KQL データベースに**名前を付ける**よう求められます

   ![KQL データベースの名前付けの画像](./Images/name-kqldatabase.png)

3. KQL データベースに覚えやすい名前 (**MyStockData** など) を指定して、 **[作成]** を選びます。

4. **[データベースの詳細]** パネルで、鉛筆アイコンを選択して OneLake で可用性を有効にします。

   ![OneLake の有効化の画像](./Images/enable-onelake-availability.png)

5. ***[データの取得から開始]*** のオプションから **[サンプル データ]** ボックスを選択します。
 
   ![サンプル データが強調表示されている選択オプションの画像](./Images/load-sample-data.png)

6. サンプル データのオプションから **[自動車のメトリック分析]** ボックスを選択します。

   ![ラボの分析データを選ぶ画像](./Images/create-sample-data.png)

7. データの読み込みが完了したら、KQL データベースが設定されていることを確認できます。

   ![データを KQL データベースに読み込み中](./Images/choose-automotive-operations-analytics.png)

7. データが読み込まれたら、データが KQL データベースに読み込まれたことを確認します。 この操作を行うには、テーブルの右側にある省略記号を選び、 **[クエリ テーブル]** に移動して、 **[Show any 100 records] (100 件のレコードを表示する)** を選びます。

    ![RawServerMetrics テーブルから上位 100 個のファイルを選択している画像](./Images/rawservermetrics-top-100.png)

   > **注**: これを初めて実行するときは、コンピューティング リソースの割り当てに数秒かかる場合があります。

    ![データからの 100 件のレコードの画像](./Images/explore-with-kql-take-100.png)


## シナリオ
このシナリオでは、Microsoft Fabric によって提供されるサンプル データに基づいて、さまざまな方法でデータを表示できるリアルタイム ダッシュボードを作成し、変数を作成し、この変数を使ってダッシュボードのパネルをリンクして、ソース システム内で起きていることについてのより詳細な分析情報を得ます。 このモジュールでは、ニューヨークのタクシー データセットを使って、区ごとの現在の交通の詳細などを表示します。

1. Fabric のメイン ページで **[リアルタイム分析]** に移動してから **[リアルタイム ダッシュボード]** を選びます。

    ![[リアルタイム ダッシュボード] を選びます。](./Images/select-real-time-dashboard.png)

1. **[新しいタイルの追加]** ボタンを選びます。

```kusto

Trips
| summarize ["Total Trip Distance"] = sum(trip_distance) by pickup_boroname
| project Borough = case(isempty(pickup_boroname) or isnull(pickup_boroname), "Unidentified", pickup_boroname), ["Total Trip Distance"]
| sort by Borough asc 

```
3. [実行] ボタンを選んで、クエリにエラーがないことを確認します。
4. パネルの右側にある **[ビジュアルの書式設定]** タブを選び、***[タイル名]*** と ***[視覚化タイプ]*** を指定します。

   ![ビジュアルの書式設定タイルの画像。](./Images/visual-formatting-tile.png)

