---
lab:
  title: T-SQL を使用してウェアハウスにデータを読み込む
  module: Load data into a warehouse in Microsoft Fabric
---

# T-SQL を使用してウェアハウスにデータを読み込む

Microsoft Fabric では、データ ウェアハウスによって大規模な分析用のリレーショナル データベースが提供されます。 レイクハウスで定義されているテーブルの既定の読み取り専用 SQL エンドポイントとは異なり、データ ウェアハウスは完全な SQL セマンティクスを提供します。これには、テーブル内のデータを挿入、更新、削除する機能が含まれます。

このラボは完了するまで、約 **30** 分かかります。

> **注**:この演習を完了するには、[Microsoft Fabric 試用版](https://learn.microsoft.com/fabric/get-started/fabric-trial)が必要です。

## ワークスペースの作成

Fabric でデータを操作する前に、Fabric 試用版を有効にしてワークスペースを作成してください。

1. ブラウザーの `https://app.fabric.microsoft.com/home?experience=fabric` で [Microsoft Fabric ホーム ページ](https://app.fabric.microsoft.com/home?experience=fabric)に移動し、Fabric 資格情報でサインインします。
1. 左側のメニュー バーで、 **[ワークスペース]** を選択します (アイコンは &#128455; に似ています)。
1. 任意の名前で新しいワークスペースを作成し、Fabric 容量を含むライセンス モード ("試用版"、*Premium*、または *Fabric*) を選択します。**
1. 開いた新しいワークスペースは空のはずです。

    ![Fabric の空のワークスペースを示すスクリーンショット。](./Images/new-workspace.png)

## レイクハウスを作成してファイルをアップロードする

このシナリオでは、使用できるデータがないため、ウェアハウスの読み込みに使用するデータを取り込む必要があります。 ウェアハウスの読み込みに使用するデータ ファイル用のデータ レイクハウスを作成します。

1. **[+ 新しい項目]** を選択し、任意の名前で新しい**レイクハウス**を作成します。

    1 分ほどすると、新しい空のレイクハウスが作成されます。 分析のために、データ レイクハウスにいくつかのデータを取り込む必要があります。 これを行うには複数の方法がありますが、この演習では、CSV ファイルをローカル コンピューター (または、該当する場合はラボ VM) にダウンロードし、レイクハウスにアップロードします。

1. この演習用のファイルを `https://github.com/MicrosoftLearning/dp-data/raw/main/sales.csv` からダウンロードします。

1. レイクハウスを含む Web ブラウザー タブに戻り、**[エクスプローラー]** ペインの **Files** フォルダーの **[...]** メニューで **[アップロード]**、**[ファイルのアップロード]** の順に選び、ローカル コンピューター (または該当する場合はラボ VM) からレイクハウスに **sales.csv** ファイルをアップロードします。

1. ファイルをアップロードしたら、**[ファイル]** を選択します。 次に示すように、CSV ファイルがアップロードされたことを確認します。

    ![レイクハウスにアップロードされたファイルのスクリーンショット。](./Images/sales-file-upload.png)

## レイクハウスにテーブルを作成する

1. **[エクスプローラー]** ペインの **sales.csv** ファイルの **[...]** で、**[テーブルに読み込む]**、**[新しいテーブル]** の順に選択します。

1. **[新しいテーブルにファイルを読み込む]** ダイアログで、次の情報を入力します。
    - **新しいテーブル名:** staging_sales
    - **列名にヘッダーを使用する:** オン
    - **区切り:** ,

1. **[読み込み]** を選択します。

## ウェアハウスの作成

ワークスペース、レイクハウス、必要なデータを含む sales テーブルが作成されたので、次はデータ ウェアハウスを作成します。

1. 左側のメニュー バーで、**[作成]** を選択します。 *[新規]* ページの *[データ ウェアハウス]* セクションで、**[ウェアハウス]** を選択します。 任意の一意の名前を設定します。

    >**注**: **[作成]** オプションがサイド バーにピン留めされていない場合は、最初に省略記号 (**...**) オプションを選択する必要があります。

    1 分ほどで、新しいレイクハウスが作成されます。

    ![新しいウェアハウスのスクリーンショット。](./Images/new-empty-data-warehouse.png)

## ファクト テーブル、ディメンション、ビューを作成する

Sales データのファクト テーブルとディメンションを作成しましょう。 また、レイクハウスを指すビューも作成します。これにより、読み込みに使用するストアド プロシージャのコードが簡略化されます。

1. ワークスペースで、作成したウェアハウスを選択します。

1. ウェアハウス ツール バーで、**[新しい SQL クエリ]** を選択してから、次のクエリをコピーして実行します。

    ```sql
    CREATE SCHEMA [Sales]
    GO
        
    IF NOT EXISTS (SELECT * FROM sys.tables WHERE name='Fact_Sales' AND SCHEMA_NAME(schema_id)='Sales')
        CREATE TABLE Sales.Fact_Sales (
            CustomerID VARCHAR(255) NOT NULL,
            ItemID VARCHAR(255) NOT NULL,
            SalesOrderNumber VARCHAR(30),
            SalesOrderLineNumber INT,
            OrderDate DATE,
            Quantity INT,
            TaxAmount FLOAT,
            UnitPrice FLOAT
        );
    
    IF NOT EXISTS (SELECT * FROM sys.tables WHERE name='Dim_Customer' AND SCHEMA_NAME(schema_id)='Sales')
        CREATE TABLE Sales.Dim_Customer (
            CustomerID VARCHAR(255) NOT NULL,
            CustomerName VARCHAR(255) NOT NULL,
            EmailAddress VARCHAR(255) NOT NULL
        );
        
    ALTER TABLE Sales.Dim_Customer add CONSTRAINT PK_Dim_Customer PRIMARY KEY NONCLUSTERED (CustomerID) NOT ENFORCED
    GO
    
    IF NOT EXISTS (SELECT * FROM sys.tables WHERE name='Dim_Item' AND SCHEMA_NAME(schema_id)='Sales')
        CREATE TABLE Sales.Dim_Item (
            ItemID VARCHAR(255) NOT NULL,
            ItemName VARCHAR(255) NOT NULL
        );
        
    ALTER TABLE Sales.Dim_Item add CONSTRAINT PK_Dim_Item PRIMARY KEY NONCLUSTERED (ItemID) NOT ENFORCED
    GO
    ```

    > **重要:** データ ウェアハウスでは、必ずしもテーブル レベルで外部キー制約が必要とは限りません。 外部キー制約はデータ整合性を確保するのに役立ちますが、ETL (抽出、変換、ロード) プロセスにオーバーヘッドが追加され、データの読み込み速度が低下するおそれもあります。 データ ウェアハウスで外部キー制約を使用するかどうかは、データ整合性とパフォーマンスの間のトレードオフを慎重に検討して決定する必要があります。

1. **[エクスプローラー]** で、**[スキーマ] >> [Sales] >> [テーブル]** の順に移動します。 作成したばかりの *Fact_Sales*、*Dim_Customer*、*Dim_Item* テーブルに注目してください。

    > **注**: 新しいスキーマが表示されない場合は、**[エクスプローラー]** ペインの **[テーブル]** で **[...]** メニューを開いてから、**[更新]** を選択します。

1. 新しい **[新しい SQL クエリ]** エディターを開き、次のクエリをコピーして実行します。 作成したレイクハウスで *<your lakehouse name>* を更新します。

    ```sql
    CREATE VIEW Sales.Staging_Sales
    AS
    SELECT * FROM [<your lakehouse name>].[dbo].[staging_sales];
    ```

1. **[エクスプローラー]** で、**[スキーマ] >> [Sales] >> [ビュー]** の順に移動します。 作成した *Staging_Sales* ビューに注目してください。

## ウェアハウスにデータを読み込む

ファクト テーブルとディメンション テーブルが作成されたので、データをレイクハウスからウェアハウスに読み込むストアド プロシージャを作成しましょう。 レイクハウスの作成時に SQL エンドポイントが自動的に作成されているので、T-SQL とデータベース間クエリを使用してウェアハウスからレイクハウス内のデータに直接アクセスできます。

このケース スタディでは、わかりやすくするために、主キーとして顧客名と品目名を使用します。

1. 新しい **[新しい SQL クエリ]** エディターを作成し、次のクエリをコピーして実行します。

    ```sql
    CREATE OR ALTER PROCEDURE Sales.LoadDataFromStaging (@OrderYear INT)
    AS
    BEGIN
        -- Load data into the Customer dimension table
        INSERT INTO Sales.Dim_Customer (CustomerID, CustomerName, EmailAddress)
        SELECT DISTINCT CustomerName, CustomerName, EmailAddress
        FROM [Sales].[Staging_Sales]
        WHERE YEAR(OrderDate) = @OrderYear
        AND NOT EXISTS (
            SELECT 1
            FROM Sales.Dim_Customer
            WHERE Sales.Dim_Customer.CustomerName = Sales.Staging_Sales.CustomerName
            AND Sales.Dim_Customer.EmailAddress = Sales.Staging_Sales.EmailAddress
        );
        
        -- Load data into the Item dimension table
        INSERT INTO Sales.Dim_Item (ItemID, ItemName)
        SELECT DISTINCT Item, Item
        FROM [Sales].[Staging_Sales]
        WHERE YEAR(OrderDate) = @OrderYear
        AND NOT EXISTS (
            SELECT 1
            FROM Sales.Dim_Item
            WHERE Sales.Dim_Item.ItemName = Sales.Staging_Sales.Item
        );
        
        -- Load data into the Sales fact table
        INSERT INTO Sales.Fact_Sales (CustomerID, ItemID, SalesOrderNumber, SalesOrderLineNumber, OrderDate, Quantity, TaxAmount, UnitPrice)
        SELECT CustomerName, Item, SalesOrderNumber, CAST(SalesOrderLineNumber AS INT), CAST(OrderDate AS DATE), CAST(Quantity AS INT), CAST(TaxAmount AS FLOAT), CAST(UnitPrice AS FLOAT)
        FROM [Sales].[Staging_Sales]
        WHERE YEAR(OrderDate) = @OrderYear;
    END
    ```
1. 新しい **[新しい SQL クエリ]** エディターを作成し、次のクエリをコピーして実行します。

    ```sql
    EXEC Sales.LoadDataFromStaging 2021
    ```

    > **注:**  この場合は、2021 年のデータのみを読み込みます。 ただし、以前の年のデータを読み込むように変更することもできます。

## 分析クエリを実行する

いくつかの分析クエリを実行して、ウェアハウス内のデータを検証してみましょう。

1. 上部のメニューで、**[新しい SQL クエリ]** を選択し、次のクエリをコピーして実行します。

    ```sql
    SELECT c.CustomerName, SUM(s.UnitPrice * s.Quantity) AS TotalSales
    FROM Sales.Fact_Sales s
    JOIN Sales.Dim_Customer c
    ON s.CustomerID = c.CustomerID
    WHERE YEAR(s.OrderDate) = 2021
    GROUP BY c.CustomerName
    ORDER BY TotalSales DESC;
    ```

    > **注:**  このクエリは、顧客を 2021 年の売上合計別に表示します。 指定された年の売上合計が最も多い顧客は **Jordan Turner** で、売上合計は **14686.69** です。 

1. 上部のメニューで、**[新しい SQL クエリ]** を選択するか、同じエディターを再利用して、次のクエリをコピーして実行します。

    ```sql
    SELECT i.ItemName, SUM(s.UnitPrice * s.Quantity) AS TotalSales
    FROM Sales.Fact_Sales s
    JOIN Sales.Dim_Item i
    ON s.ItemID = i.ItemID
    WHERE YEAR(s.OrderDate) = 2021
    GROUP BY i.ItemName
    ORDER BY TotalSales DESC;

    ```

    > **注:**  このクエリは、売上が上位の品目を 2021 年の売上合計別に表示します。 これらの結果は、2021 年に顧客の間で最も人気があった品目は、*Mountain-200 bike* モデル (黒とシルバーの両方の色) であることを示唆しています。

1. 上部のメニューで、**[新しい SQL クエリ]** を選択するか、同じエディターを再利用して、次のクエリをコピーして実行します。

    ```sql
    WITH CategorizedSales AS (
    SELECT
        CASE
            WHEN i.ItemName LIKE '%Helmet%' THEN 'Helmet'
            WHEN i.ItemName LIKE '%Bike%' THEN 'Bike'
            WHEN i.ItemName LIKE '%Gloves%' THEN 'Gloves'
            ELSE 'Other'
        END AS Category,
        c.CustomerName,
        s.UnitPrice * s.Quantity AS Sales
    FROM Sales.Fact_Sales s
    JOIN Sales.Dim_Customer c
    ON s.CustomerID = c.CustomerID
    JOIN Sales.Dim_Item i
    ON s.ItemID = i.ItemID
    WHERE YEAR(s.OrderDate) = 2021
    ),
    RankedSales AS (
        SELECT
            Category,
            CustomerName,
            SUM(Sales) AS TotalSales,
            ROW_NUMBER() OVER (PARTITION BY Category ORDER BY SUM(Sales) DESC) AS SalesRank
        FROM CategorizedSales
        WHERE Category IN ('Helmet', 'Bike', 'Gloves')
        GROUP BY Category, CustomerName
    )
    SELECT Category, CustomerName, TotalSales
    FROM RankedSales
    WHERE SalesRank = 1
    ORDER BY TotalSales DESC;
    ```

    > **注:**  このクエリの結果には、売上合計に基づいて、Bike、Helmet、Gloves の各カテゴリの最上位顧客が表示されます。 たとえば、**Joan Coleman** は、**グローブ**カテゴリのトップの顧客です。
    >
    > ディメンション テーブルには、個別のカテゴリ列がないため、カテゴリ情報は、文字列操作を使用して `ItemName` 列から抽出されました。 このアプローチでは、品目名が一貫した名前付け規則に従っていることを前提としています。 品目名が一貫した名前付け規則に従っていない場合、各品目の真のカテゴリが結果に正確に反映されない可能性があります。

この演習では、レイクハウスと、複数のテーブルを含むデータ ウェアハウスを作成しました。 データを取り込み、データベース間クエリを使用してレイクハウスからウェアハウスにデータを読み込みました。 さらに、クエリ ツールを使用して分析クエリを実行しました。

## リソースをクリーンアップする

データ ウェアハウスの探索が完了したら、この演習用に作成したワークスペースを削除できます。

1. 左側のバーで、ワークスペースのアイコンを選択して、それに含まれるすべての項目を表示してください。
1. **[ワークスペースの設定]** を選択し、**[全般]** セクションで下にスクロールし、**[このワークスペースを削除する]** を選択します。
1. **[削除]** を選択して、ワークスペースを削除します。
