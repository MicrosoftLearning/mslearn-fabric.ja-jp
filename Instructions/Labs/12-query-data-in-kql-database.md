---
lab:
  title: KQL データベースでデータにクエリを実行する
  module: Query data from a Kusto Query database in Microsoft Fabric
---
# Microsoft Fabric での Kusto データベースのクエリの概要
KQL クエリセットは、KQL データベースからのクエリの実行、変更、クエリ結果の表示を可能にするツールです。 KQL クエリセットの各タブを別の KQL データベースにリンクし、将来使用するためにクエリを保存したり、データ分析のために他のユーザーと共有したりできます。 任意のタブの KQL データベースを切り替えることもできるため、さまざまなデータ ソースからのクエリ結果を比較できます。

KQL クエリセットは、多くの SQL 関数と互換性のある Kusto 照会言語を使用してクエリを作成します。 [Kusto 照会言語 (KQL)](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/?context=%2Ffabric%2Fcontext%2Fcontext) の詳細については、 

このラボは完了するまで、約 **25** 分かかります。

## ワークスペースの作成

Fabric でデータを操作する前に、Fabric 試用版を有効にしてワークスペースを作成してください。

1. `https://app.fabric.microsoft.com` で [Microsoft Fabric](https://app.fabric.microsoft.com) にサインインし、 **[Power BI]** を選択してください。
2. 左側のメニュー バーで、 **[ワークスペース]** を選択します (アイコンは &#128455; に似ています)。
3. 任意の名前で新しいワークスペースを作成し、Fabric 容量を含むライセンス モード ("試用版"、*Premium*、または *Fabric*) を選択してください。**
4. 新しいワークスペースを開くと次に示すように空のはずです。

    ![Power BI の空のワークスペースのスクリーンショット。](./Images/new-workspace.png)

このラボでは、Fabric の Real-Time Analytics (RTA) を使用して、サンプル イベントストリームから KQL データベースを作成します。 Real-Time Analytics には、RTA の機能を探索するために使用できる便利なサンプル データセットが用意されています。 このサンプル データを使用して、一部のリアルタイム データを分析し、ダウンストリーム プロセスでの追加利用を可能にする KQL | SQL クエリとクエリセットを作成します。

## KQL データベースを作成する

1. **Real-Time Analytics** で、 **[KQL データベース]** ボックスを選択します。

   ![KQL データベースの選択の画像](./Images/select-kqldatabase.png)

2. KQL データベースに**名前を付ける**ダイアログが表示されます

   ![KQL データベースの名前付けの画像](./Images/name-kqldatabase.png)

3. KQL データベースに、覚えやすい名前 (**MyStockData** など) を指定し、 **[作成]** を押します。

4. **[データベースの詳細]** パネルで、鉛筆アイコンを選択して OneLake で可用性を有効にします。

   ![OneLake の有効化の画像](./Images/enable-onelake-availability.png)

5. ***[データの取得から開始]*** のオプションから **[サンプル データ]** ボックスを選択します。
 
   ![サンプル データが強調表示されている選択オプションの画像](./Images/load-sample-data.png)

6. サンプル データのオプションから **[メトリック分析]** ボックスを選びます。

   ![ラボの分析データを選ぶ画像](./Images/create-sample-data.png)

7. データが読み込まれると、データが KQL データベースに読み込まれたことを確認します。 これを実現するには、テーブルの右側にある省略記号を選択し、 **[クエリ テーブル]** に移動し、 **[100 件のレコードを表示する]** を選択します。

    ![RawServerMetrics テーブルから上位 100 個のファイルを選択している画像](./Images/rawservermetrics-top-100.png)

> **注**: これを初めて実行するときは、コンピューティング リソースの割り当てに数秒かかる場合があります。

## シナリオ
このシナリオにおいて、あなたは、Fabric 環境から実装する予定である架空の SQL Server の未加工メトリックのサンプル データセットのクエリを実行するタスクを与えられたアナリストです。 データに関する有用な分析情報を得るために KQL と T-SQL を使用してこのデータのクエリを実行して情報を収集します。

