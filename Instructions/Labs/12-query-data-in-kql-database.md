---
lab:
  title: KQL データベースでデータにクエリを実行する
  module: Query data from a Kusto Query database in Microsoft Fabric
---

# Microsoft Fabric での Kusto データベースのクエリの概要

KQL クエリセットは、KQL データベースからのクエリの実行、変更、クエリ結果の表示を可能にするツールです。 KQL クエリセットの各タブを別の KQL データベースにリンクし、将来使用するためにクエリを保存したり、データ分析のために他のユーザーと共有したりできます。 任意のタブの KQL データベースを切り替えることもできるため、さまざまなデータ ソースからのクエリ結果を比較できます。

この演習では、あなたは、NYC タクシーの乗車に関する生メトリックのサンプル データセットのクエリを実行する任務を負うアナリストの役割を果たし、Fabric 環境からデータの概要統計 (プロファイリング) をプルします。 データに関する有用な分析情報を得るために、KQL を使ってこのデータのクエリを実行し、情報を収集します。

KQL クエリセットは、多くの SQL 関数と互換性のある Kusto 照会言語を使用してクエリを作成します。 [Kusto 照会言語 (KQL)](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/?context=%2Ffabric%2Fcontext%2Fcontext) の詳細を確認します。

このラボの所要時間は約 **25** 分です。

> **注**:この演習を完了するには、[Microsoft Fabric 試用版](https://learn.microsoft.com/fabric/get-started/fabric-trial)が必要です。

## ワークスペースの作成

Fabric でデータを操作する前に、Fabric 試用版を有効にしてワークスペースを作成してください。

1. [Microsoft Fabric ホーム ページ](https://app.fabric.microsoft.com)で、**Real-Time Analytics** を選択します。
1. 左側のメニュー バーで、 **[ワークスペース]** を選択します (アイコンは &#128455; に似ています)。
1. 任意の名前で新しいワークスペースを作成し、Fabric 容量を含むライセンス モード ("試用版"、*Premium*、または *Fabric*) を選択します。**
1. 開いた新しいワークスペースは空のはずです。

    ![Fabric の空のワークスペースを示すスクリーンショット。](./Images/new-workspace.png)

このラボでは、Fabric のリアルタイム分析 (RTA) を使って、サンプルのイベントストリームから KQL データベースを作成します。 Real-Time Analytics には、RTA の機能を探索するために使用できる便利なサンプル データセットが用意されています。 このサンプル データを使い、リアルタイム データを分析する KQL または SQL のクエリとクエリセットを作成し、ダウンストリームのプロセスで他の目的に使用できるようにします。

## KQL データベースを作成する

1. **Real-Time Analytics** で、 **[KQL データベース]** ボックスを選択します。

   ![KQL データベースの選択の画像](./Images/select-kqldatabase.png)

1. KQL データベースに**名前**を付けるよう求められます

   ![KQL データベースの名前付けの画像](./Images/name-kqldatabase.png)

1. KQL データベースに覚えやすい名前 (**TaxiData** など) を付けて、**[作成]** を選択します。

1. **[データベースの詳細]** パネルで、鉛筆アイコンを選択して OneLake で可用性を有効にします。

   ![OneLake の有効化の画像](./Images/enable-onelake-availability.png)

   次に、スライダーを使って可用性をオンにします。

   ![Data Lake でスライダーを選択している画像](./Images/data-availability-data-lake.png)
   
1. ***[データの取得から開始]*** のオプションから **[サンプル データ]** ボックスを選択します。

   ![サンプル データが強調表示されている選択オプションの画像](./Images/load-sample-data.png)

   サンプル データのオプションから **[Automotive operations analytics] (自動車の動作分析)** ボックスを選択します。

   ![ラボの分析データを選ぶ画像](./Images/create-sample-data.png)

1. データの読み込みが完了したら、KQL データベースが設定されていることを確認できます。

   ![データを KQL データベースに読み込み中](./Images/choose-automotive-operations-analytics.png)

1. データが読み込まれたら、データが KQL データベースに読み込まれたことを確認します。 この操作を行うには、テーブルの右側にある省略記号を選び、**[クエリ テーブル]** に移動して、**[100 件のレコードを表示する]** を選びます。

    ![RawServerMetrics テーブルから上位 100 個のファイルを選択している画像](./Images/rawservermetrics-top-100.png)

   > **注**:これを初めて実行する場合、コンピューティング リソースの割り当てに数秒かかる場合があります。


    ![データからの 100 件のレコードの画像](./Images/explore-with-kql-take-100.png)

## Kusto 照会言語 (KQL) とその構文の概要

Kusto 照会言語 (KQL) は、Azure Fabric の一部である Microsoft Azure Data Explorer でデータを分析するために使用されるクエリ言語です。 KQL はシンプルかつ直感的に設計されているため、初心者でも簡単に学習して使用できます。 同時に、柔軟性が高くカスタマイズ可能であるため、上級ユーザーは複雑なクエリや分析を実行することができます。

KQL は SQL に似た構文に基づいていますが、いくつかの重要な違いがあります。 たとえば、KQL ではコマンドを区切るためにセミコロン (;) ではなくパイプ演算子 (|) を使用します。また、データのフィルター処理と操作に使用する関数と演算子のセットも異なります。

KQL の重要な機能の 1 つは、大量のデータを迅速かつ効率的に処理できることです。 この機能により、KQL は、ログ、利用統計情報、その他の種類のビッグ データの分析に最適です。 KQL では、構造化データや非構造化データなど、幅広いデータ ソースもサポートされているため、データ分析のための多用途ツールとなっています。

Microsoft Fabric のコンテキストでは、KQL を使用して、アプリケーション ログ、パフォーマンス メトリック、システム イベントなどのさまざまなソースからのデータのクエリと分析を行うことができます。 これは、アプリケーションとインフラストラクチャの正常性とパフォーマンスに関する分析情報を取得し、問題と最適化の機会を特定するのに役立ちます。

総じて、KQL は強力で柔軟性の高いクエリ言語であり、Microsoft Fabric やその他のデータ ソースを使用しているかどうかに関係なく、データの分析情報を迅速かつ簡単に得るのに役立ちます。 直感的な構文と強力な機能を備えた KQL は、さらに検討する価値があります。

このモジュールでは、最初に KQL を使用し、次に T-SQL を使用して、KQL データベースに対するクエリの基本に焦点を当てます。 ここでは、クエリに使われる T-SQL 構文の次のような要素に焦点を当てます。

**SELECT** クエリ。1 つ以上のテーブルからデータを取得するために使用されます。 たとえば、SELECT クエリを使用して、社内のすべての従業員の指名と給与を取得できます。

**WHERE** クエリ。特定の条件に基づいてデータをフィルター処理するために使用されます。 たとえば、WHERE クエリを使用して、特定の部署で働く従業員や、給与が特定の金額を超える従業員の名前を取得できます。

**GROUP BY** クエリ。データを 1 つ以上の列でグループ化し、それらの列に対して集計関数を実行するために使用されます。 たとえば、GROUP BY クエリを使用して、部署別または国別の従業員の平均給与を取得できます。

**ORDER BY** クエリ。データを 1 つ以上の列で昇順または降順に並べ替えるために使用されます。 たとえば、ORDER BY クエリを使用して、給与順または姓順に並べ替えられた従業員の名前を取得できます。

   > **警告:** Power BI では T-SQL がデータ ソースとしてサポートされていないため、**T-SQL** を使用してクエリセットから Power BI レポートを作成することはできません。 **Power BI では、クエリセットのネイティブ クエリ言語として KQL のみがサポートされています**。 T-SQL を使用して Microsoft Fabric 内のデータに対してクエリを実行する場合は、Microsoft SQL Server をエミュレートし、データに対して T-SQL クエリを実行できるようにする T-SQL エンドポイントを使用する必要があります。 ただし、T-SQL エンドポイントにはいくつかの制限と、ネイティブ SQL Server との違いがあり、レポートの作成または Power BI への発行はサポートされていません。

> **注**:先ほど示したクエリ ウィンドウを表示する方法のほかに、メインの [KQL データベース] パネルにある **[データの探索]** ボタンをいつでも押すことができます。

   ![[データの探索] ボタンの画像](./Images/explore-your-data.png)

## KQL を使用したサンプル データセットのデータの `SELECT`

1. このクエリでは、Trips テーブルから 100 件のレコードをプルします。 `take` キーワードを使用して、エンジンに対して 100 件のレコードを返すように要求します。

    ```kusto
    
    Trips
    | take 100
    ```

    > **注:** KQL では、パイプ文字 `|` は 2 つの目的で使われます。1 つは、表形式の式ステートメントでクエリ演算子を区切るためです。 また、パイプ文字で区切られた項目の 1 つを指定できることを示すために、角かっこまたは丸かっこ内の論理 OR 演算子としても使用されます。

1. `project` キーワードを使用して、クエリを実行する特定の属性を追加し、`take` キーワードを使用して、返すレコード件数をエンジンに指示すると、精度を高めることができます。

    > **注:** `//` の使用は、Microsoft Fabric の ***[データの探索]*** クエリ ツール内で使用されるコメントを示します。

    ```kusto
    
    // Use 'project' and 'take' to view a sample number of records in the table and check the data.
    Trips 
    | project vendor_id, trip_distance
    | take 10
    ```

1. 分析で一般的に使われるもう 1 つの方法は、クエリセット内の列の名前をよりわかりやすい名前に変更することです。 これを行うには、新しい列名の後に等号と、名前を変更する列を使用します。

    ```kusto
    
    Trips 
    | project vendor_id, ["Trip Distance"] = trip_distance
    | take 10
    ```

1. また、走行距離を集計して、移動したマイル数を確認することもできます。

    ```kusto
    
    Trips
    | summarize ["Total Trip Distance"] = sum(trip_distance)
    ```

## KQL を使用したサンプル データセットのデータの `GROUP BY`

1. 次に、`summarize` 演算子の処理対象である乗車場所の `group by` を行うこともできます。 また、`project` 演算子を使用することもできます。これを使用すると、出力に含める列を選択して名前を変更することができます。 この場合、NY タクシー システム内の区別にグループ化して、各区から移動した合計距離をユーザーに提供します。

```kusto

Trips
| summarize ["Total Trip Distance"] = sum(trip_distance) by pickup_boroname
| project Borough = pickup_boroname, ["Total Trip Distance"]
```

1. この場合、分析に適さない空白の値があります。`case` 関数を `isempty` および `isnull` 関数と共に使用して、フォローアップのために、これらの値を ***[未確認]*** カテゴリに分類することができます。

```kusto

Trips
| summarize ["Total Trip Distance"] = sum(trip_distance) by pickup_boroname
| project Borough = case(isempty(pickup_boroname) or isnull(pickup_boroname), "Unidentified", pickup_boroname), ["Total Trip Distance"]
```

## KQL を使用したサンプル データセットのデータの `ORDER BY`

データの意味をよりわかりやすくするために、通常はデータを列で並べ替えます。KQL では、このプロセスは `sort by` 演算子または `order by` 演算子を使用して行われますが、どちらも同じように動作します。
 
```kusto

// using the sort by operators
Trips
| summarize ["Total Trip Distance"] = sum(trip_distance) by pickup_boroname
| project Borough = case(isempty(pickup_boroname) or isnull(pickup_boroname), "Unidentified", pickup_boroname), ["Total Trip Distance"]
| sort by Borough asc 

// order by operator has the same result as sort by
Trips
| summarize ["Total Trip Distance"] = sum(trip_distance) by pickup_boroname
| project Borough = case(isempty(pickup_boroname) or isnull(pickup_boroname), "Unidentified", pickup_boroname), ["Total Trip Distance"]
| sort by Borough asc 
```

## サンプル KQL クエリでデータをフィルター処理する `WHERE` 句

SQL とは異なり、この KQL クエリでは、`WHERE` 句はすぐに呼び出されます。 where 句内でも、`and` と `or` の論理演算子を使用できます。この場合、テーブルに対して true または false と評価されます。これは、単純な式または複雑な式 (複数の列、演算子、関数を含むもの) にすることができます。

```kusto

// let's filter our dataset immediately from the source by applying a filter directly after the table.
Trips
| where pickup_boroname == "Manhattan"
| summarize ["Total Trip Distance"] = sum(trip_distance) by pickup_boroname
| project Borough = case(isempty(pickup_boroname) or isnull(pickup_boroname), "Unidentified", pickup_boroname), ["Total Trip Distance"]
| sort by Borough asc

```

## T-SQL をして集計情報のクエリを実行する

KQL データベースでは、T-SQL はネイティブにサポートされませんが、Microsoft SQL Server をエミュレートし、データに対して T-SQL クエリを実行できるようにする T-SQL エンドポイントが提供されます。 ただし、T-SQL エンドポイントには、いくつかの制限と、ネイティブの SQL Server との違いがあります。 たとえば、T-SQL エンドポイントでは、テーブルの作成、変更、または削除や、データの挿入、更新、または削除はサポートされません。 また、KQL と互換性のない一部の T-SQL 関数と構文もサポートされません。 T-SQL エンドポイントは、KQL をサポートしていないシステムで、T-SQL を使用して KQL データベース内のデータに対してクエリを実行できるようにするために作成されました。 したがって、KQL の方が T-SQL よりも多くの機能を提供し、高いパフォーマンスを発揮するため、KQL データベースのプライマリ クエリとして KQL を使用することをお勧めします。 また、count、sum、avg、min、max など、一部の SQL 関数は KQL でもサポートされており、これらを使用することもできます。 

## T-SQL を使用したサンプル データセットからのデータの `SELECT`

1. このクエリでは、`Trips` テーブルから `TOP` 句を使って最初の 100 件のレコードをプルします。 

    ```sql
    // We can use the TOP clause to limit the number of records returned
    
    SELECT TOP 100 * from Trips
    ```

1. KQL データベース内での ***[データの探索]*** ツールのコメントである `//` を使う場合、T-SQL クエリを実行するときに、これを強調表示することはできません。代わりに、標準の SQL コメント表記 `--` を使う必要があります。 このダブルハイフンも、KQL エンジンに対して、Azure Data Explorer で T-SQL を想定するように指示するものです。

    ```sql
    -- instead of using the 'project' and 'take' keywords we simply use a standard SQL Query
    SELECT TOP 10 vendor_id, trip_distance
    FROM Trips
    ```

1. この場合も、標準の T-SQL 機能が、trip_distance という名前をよりわかりやすい名前に変更するクエリで正常に動作することがわかります。

    ```sql
    
    -- No need to use the 'project' or 'take' operators as standard T-SQL Works
    SELECT TOP 10 vendor_id, trip_distance as [Trip Distance]
    from Trips
    ```

1. また、走行距離を集計して、移動したマイル数を確認することもできます。

    ```sql
    Select sum(trip_distance) as [Total Trip Distance]
    from Trips
    ```
     >**注:** T-SQL では、KQL クエリとは異なり引用符を使用する必要はありません。また、T-SQL では `summarize` コマンドと `sort by` コマンドは使用できないことに注意してください。

## T-SQL を使用したサンプル データセットからのデータの `GROUP BY`

1. 次に、`GROUP BY` 演算子の処理対象である乗車場所の `group by` を行うこともできます。 また、`AS` 演算子を使用することもできます。これを使用すると、出力に含める列を選択して名前を変更することができます。 この場合、NY タクシー システム内の区別にグループ化して、各区から移動した合計距離をユーザーに提供します。

    ```sql
    SELECT pickup_boroname AS Borough, Sum(trip_distance) AS [Total Trip Distance]
    FROM Trips
    GROUP BY pickup_boroname
    ```

1. この場合、分析に適さない空白の値があります。`CASE` 関数を、`IS NULL` 関数および `''` 空の値と共に使用して、フォローアップのために、これらの値を ***[未確認]*** カテゴリに分類することができます。 

    ```sql
    
    SELECT CASE
             WHEN pickup_boroname IS NULL OR pickup_boroname = '' THEN 'Unidentified'
             ELSE pickup_boroname
           END AS Borough,
           SUM(trip_distance) AS [Total Trip Distance]
    FROM Trips
    GROUP BY CASE
               WHEN pickup_boroname IS NULL OR pickup_boroname = '' THEN 'Unidentified'
               ELSE pickup_boroname
             END;
    ```

## T-SQL を使用したサンプル データセットからのデータの `ORDER BY`

1. データの意味をよりわかりやすくするために、通常はデータを列で並べ替えます。T-SQL では、このプロセスは `ORDER BY` 演演算子を使用して行われます。 T-SQL に ***SORT BY*** 演算子はありません
 
    ```sql
    -- Group by pickup_boroname and calculate the summary statistics of trip_distance
    SELECT CASE
             WHEN pickup_boroname IS NULL OR pickup_boroname = '' THEN 'unidentified'
             ELSE pickup_boroname
           END AS Borough,
           SUM(trip_distance) AS [Total Trip Distance]
    FROM Trips
    GROUP BY CASE
               WHEN pickup_boroname IS NULL OR pickup_boroname = '' THEN 'unidentified'
               ELSE pickup_boroname
             END
    -- Add an ORDER BY clause to sort by Borough in ascending order
    ORDER BY Borough ASC;
    ```
    ## サンプル T-SQL クエリでデータをフィルター処理する `WHERE` 句
    
1. KQL とは異なり、`WHERE` 句は、T-SQL ステートメントの末尾に記述されます。この場合は、`GROUP BY` 句があるため、`HAVING` ステートメントを使う必要があり、列の新しい名前 (この場合は **Borough**) をフィルター対象の列名として使います。

    ```sql
    -- Group by pickup_boroname and calculate the summary statistics of trip_distance
    SELECT CASE
             WHEN pickup_boroname IS NULL OR pickup_boroname = '' THEN 'unidentified'
             ELSE pickup_boroname
           END AS Borough,
           SUM(trip_distance) AS [Total Trip Distance]
    FROM Trips
    GROUP BY CASE
               WHEN pickup_boroname IS NULL OR pickup_boroname = '' THEN 'unidentified'
               ELSE pickup_boroname
             END
    -- Add a having clause due to the GROUP BY statement
    HAVING Borough = 'Manhattan'
    -- Add an ORDER BY clause to sort by Borough in ascending order
    ORDER BY Borough ASC;
    
    ```

## リソースをクリーンアップする

この演習では、KQL データベースを作成し、クエリ用のサンプル データセットを設定しました。 その後、KQL と SQL を使ってデータにクエリを実行しました。 KQL データベースの探索が完了したら、この演習用に作成したワークスペースを削除できます。
1. 左側のバーで、ワークスペースの**アイコン**を選択します。
2. ツール バーの [...] メニューで、**[ワークスペース設定]** を選択します。
3. **[全般]** セクションで、**[このワークスペースの削除]** を選択します。
