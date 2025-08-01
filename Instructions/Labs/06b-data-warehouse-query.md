---
lab:
  title: Microsoft Fabric のデータ ウェアハウスにクエリを実行する
  module: Query a data warehouse in Microsoft Fabric
---

# Microsoft Fabric のデータ ウェアハウスにクエリを実行する

Microsoft Fabric では、データ ウェアハウスによって大規模な分析用のリレーショナル データベースが提供されます。 Microsoft Fabric ワークスペースに組み込まれている豊富なエクスペリエンス セットを使用すると、DirectLake モードで Power BI と統合された、簡単に使用可能で常に接続されたセマンティック モデルを使用できるため、分析情報への時間を短縮できます。 

このラボは完了するまで、約 **30** 分かかります。

> **注**:この演習を完了するには、[Microsoft Fabric 試用版](https://learn.microsoft.com/fabric/get-started/fabric-trial)が必要です。

## ワークスペースの作成

Fabric でデータを操作する前に、Fabric 試用版を有効にしてワークスペースを作成してください。

1. ブラウザーの `https://app.fabric.microsoft.com/home?experience=fabric` で [Microsoft Fabric ホーム ページ](https://app.fabric.microsoft.com/home?experience=fabric)に移動し、Fabric 資格情報でサインインします。
1. 左側のメニュー バーで、 **[ワークスペース]** を選択します (アイコンは &#128455; に似ています)。
1. 任意の名前で新しいワークスペースを作成し、Fabric 容量を含むライセンス モード ("試用版"、*Premium*、または *Fabric*) を選択します。**
1. 開いた新しいワークスペースは空のはずです。

    ![Fabric の空のワークスペースを示すスクリーンショット。](./Images/new-workspace.png)

## データ ウェアハウスのサンプルを作成する

これでワークスペースが作成されたので、次にデータ ウェアハウスを作成します。

1. 左側のメニュー バーで、**[作成]** を選択します。 *[新規]* ページの *[データ ウェアハウス]* セクションで、**[サンプル ウェアハウス]** を選択し、**sample-dw** という名前の新しいデータ ウェアハウスを作成します。

    >**注**: **[作成]** オプションがサイド バーにピン留めされていない場合は、最初に省略記号 (**...**) オプションを選択する必要があります。

    1 分程度で、新しいウェアハウスが作成され、タクシー乗車分析シナリオ用のサンプル データが設定されます。

    ![新しいウェアハウスのスクリーンショット。](./Images/sample-data-warehouse.png)

## データ ウェアハウスに対してクエリを実行する

SQL クエリ エディターでは、IntelliSense、コード補完、構文の強調表示、クライアント側の解析と検証がサポートされます。 データ定義言語 (DDL)、データ操作言語 (DML)、およびデータ制御言語 (DCL) ステートメントを実行できます。

1. **[sample-dw]** データ ウェアハウス ページの **[新しい SQL クエリ]** ドロップダウン リストで、**[新しい SQL クエリ]** を選択します。

1. 新しい空白のクエリ ペインに、次の Transact-SQL コードを入力します。

    ```sql
    SELECT 
    D.MonthName, 
    COUNT(*) AS TotalTrips, 
    SUM(T.TotalAmount) AS TotalRevenue 
    FROM dbo.Trip AS T
    JOIN dbo.[Date] AS D
        ON T.[DateID]=D.[DateID]
    GROUP BY D.MonthName;
    ```

1. **[&#9655; 実行]** ボタンを使用して SQL スクリプトを実行し、結果を確認します。結果には、月ごとの乗車数の合計と総収益が表示されます。

1. 次の Transact-SQL コードを入力します。

    ```sql
   SELECT 
    D.DayName, 
    AVG(T.TripDurationSeconds) AS AvgDuration, 
    AVG(T.TripDistanceMiles) AS AvgDistance 
    FROM dbo.Trip AS T
    JOIN dbo.[Date] AS D
        ON T.[DateID]=D.[DateID]
    GROUP BY D.DayName;
    ```

1. 変更したクエリを実行し、結果を確認します。結果には、曜日ごとの平均の乗車時間と距離が表示されます。

1. 次の Transact-SQL コードを入力します。

    ```sql
    SELECT TOP 10 
        G.City, 
        COUNT(*) AS TotalTrips 
    FROM dbo.Trip AS T
    JOIN dbo.Geography AS G
        ON T.DropoffGeographyID=G.GeographyID
    GROUP BY G.City
    ORDER BY TotalTrips DESC;
    ```

1. 変更されたクエリを実行し、結果を確認します。結果には、人気のある乗車場所と降車場所の上位 10 個の場所が表示されます。

1. すべてのクエリ タブを閉じます。

## データの整合性を検証する

分析と意思決定を行うにあたってデータが正確で信頼できることを確認するには、データの整合性の検証が重要となります。 整合性がないデータは、正しくない分析と誤った結果へと導く可能性があります。 

データ ウェアハウスにクエリを実行して整合性を確認しましょう。

1. **[新しい SQL クエリ]** ドロップダウン リストで、**[新しい SQL クエリ]** を選択します。

1. 新しい空白のクエリ ペインに、次の Transact-SQL コードを入力します。

    ```sql
    -- Check for trips with unusually long duration
    SELECT COUNT(*) FROM dbo.Trip WHERE TripDurationSeconds > 86400; -- 24 hours
    ```

1. 変更されたクエリを実行し、結果を確認します。結果には、時間が異常に長いすべての乗車の詳細が表示されます。

1. **[新しい SQL クエリ]** ドロップダウン リストで、**[新しい SQL クエリ]** を選択して、2 つ目のクエリ タブを追加します。次に、新しい空のクエリ タブで、次のコードを実行します。

    ```sql
    -- Check for trips with negative trip duration
    SELECT COUNT(*) FROM dbo.Trip WHERE TripDurationSeconds < 0;
    ```

1. 新しい空のクエリ ペインに、次の Transact-SQL コードを入力して実行します。

    ```sql
    -- Remove trips with negative trip duration
    DELETE FROM dbo.Trip WHERE TripDurationSeconds < 0;
    ```

    > **注:**  整合性がないデータを処理する方法はいくつかあります。 これを削除するのではなく、平均値や中央値などの別の値に置き換えるという選択肢があります。

1. すべてのクエリ タブを閉じます。

## ビューとして保存

該当データを使用してレポートを生成するユーザーのグループのために、特定の乗車をフィルター処理する必要があるとします。

先ほど使用したクエリに基づいてビューを作成し、それにフィルターを追加してみましょう。

1. **[新しい SQL クエリ]** ドロップダウン リストで、**[新しい SQL クエリ]** を選択します。

1. 新しい空のクエリ ペインに、次の Transact-SQL コードを再入力して実行します。

    ```sql
    SELECT 
        D.DayName, 
        AVG(T.TripDurationSeconds) AS AvgDuration, 
        AVG(T.TripDistanceMiles) AS AvgDistance 
    FROM dbo.Trip AS T
    JOIN dbo.[Date] AS D
        ON T.[DateID]=D.[DateID]
    GROUP BY D.DayName;
    ```

1. クエリを変更して `WHERE D.Month = 1` を追加します。 これにより、1 月のレコードのみを含むようにデータがフィルター処理されます。 最終的なクエリは次のようになるはずです。

    ```sql
    SELECT 
        D.DayName, 
        AVG(T.TripDurationSeconds) AS AvgDuration, 
        AVG(T.TripDistanceMiles) AS AvgDistance 
    FROM dbo.Trip AS T
    JOIN dbo.[Date] AS D
        ON T.[DateID]=D.[DateID]
    WHERE D.Month = 1
    GROUP BY D.DayName
    ```

1. クエリ内の SELECT ステートメントのテキストを選択します。 次に **[&#9655; 実行]** ボタンの横の **[ビューとして保存]** を選択します。

1. **vw_JanTrip** という名前の新しいビューを作成します。

1. **[エクスプローラー]** で、**[スキーマ] >> [dbo] >> [ビュー]** に移動します。 先ほど作成した *vw_JanTrip* ビューに注意してください。

1. すべてのクエリ タブを閉じます。

> **詳細情報**:データ ウェアハウスに対するクエリの詳細については、Microsoft Fabric ドキュメントの「[SQL クエリ エディターを使用したクエリ](https://learn.microsoft.com/fabric/data-warehouse/sql-query-editor)」を参照してください。

## リソースをクリーンアップする

この演習では、クエリを使用して、Microsoft Fabric データ ウェアハウス内のデータの分析情報を取得しました。

データ ウェアハウスの探索が完了したら、この演習用に作成したワークスペースを削除できます。

1. 左側のバーで、ワークスペースのアイコンを選択して、それに含まれるすべての項目を表示してください。
1. **[ワークスペースの設定]** を選択し、**[全般]** セクションで下にスクロールし、**[このワークスペースを削除する]** を選択します。
1. **[削除]** を選択して、ワークスペースを削除します。
