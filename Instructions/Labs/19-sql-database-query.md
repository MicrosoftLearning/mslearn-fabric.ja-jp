---
lab:
  title: Microsoft Fabric で SQL Database を操作する
  module: Get started with SQL Database in Microsoft Fabric
---

# Microsoft Fabric で SQL Database を操作する

Microsoft Fabric の SQL データベースは、Azure SQL Database に基づく開発者向けのトランザクション データベースであり、Fabric で運用データベースを簡単に作成できます。 Fabric の SQL データベースでは、SQL Database エンジンを Azure SQL Database として使用します。

このラボは完了するまで、約 **30** 分かかります。

> **注**:この演習を完了するには、[Microsoft Fabric 試用版](https://learn.microsoft.com/fabric/get-started/fabric-trial)が必要です。

## ワークスペースの作成

Fabric でデータを操作する前に、Fabric 試用版を有効にしてワークスペースを作成してください。

1. `https://app.fabric.microsoft.com/home?experience=fabric` の [Microsoft Fabric ホーム ページ](https://app.fabric.microsoft.com/home?experience=fabric)で、**[Synapse Data Warehouse]** を選択します。
1. 左側のメニュー バーで、 **[ワークスペース]** を選択します (アイコンは &#128455; に似ています)。
1. 任意の名前で新しいワークスペースを作成し、Fabric 容量を含むライセンス モード ("試用版"、*Premium*、または *Fabric*) を選択します。**
1. 開いた新しいワークスペースは空のはずです。

    ![Fabric の空のワークスペースを示すスクリーンショット。](./Images/new-workspace.png)

## サンプル データを使用して SQL データベースを作成する

これでワークスペースが作成されたので、次に SQL データベースを作成します。

1. 左のパネルで **+ 作成**を選択します。
1. **[データベース]** セクションに移動し、**[SQL Database]** を選択します。
1. データベース名として「**AdventureWorksLT**」と入力し、**[作成]** を選択します。
1. データベースを作成したら、**[サンプル データ]** カードからサンプル データをデータベースに読み込むことができます。

    1 分ほど経過すると、シナリオ用のサンプル データがデータベースに入力されます。

    ![サンプル データが読み込まれた新しいデータベースのスクリーンショット。](./Images/sql-database-sample.png)

## SQL データベースのクエリを実行する

SQL クエリ エディターでは、IntelliSense、コード補完、構文の強調表示、クライアント側の解析と検証がサポートされます。 データ定義言語 (DDL)、データ操作言語 (DML)、およびデータ制御言語 (DCL) ステートメントを実行できます。

1. **AdventureWorksLT** データベース ページで、**[ホーム]** に移動し、**[新しいクエリ]** を選択します。

1. 新しい空のクエリ ペインに、次の T-SQL コードを入力して実行します。

    ```sql
    SELECT 
        p.Name AS ProductName,
        pc.Name AS CategoryName,
        p.ListPrice
    FROM 
        SalesLT.Product p
    INNER JOIN 
        SalesLT.ProductCategory pc ON p.ProductCategoryID = pc.ProductCategoryID
    ORDER BY 
    p.ListPrice DESC;
    ```
    
    このクエリは、`Product` テーブルと `ProductCategory` テーブルを結合して、製品名、カテゴリ、および表示価格を価格の降順に並べ替えて表示します。

1. 新しいクエリ エディターに、次の T-SQL コードを入力して実行します。

    ```sql
   SELECT 
        c.FirstName,
        c.LastName,
        soh.OrderDate,
        soh.SubTotal
    FROM 
        SalesLT.Customer c
    INNER JOIN 
        SalesLT.SalesOrderHeader soh ON c.CustomerID = soh.CustomerID
    ORDER BY 
        soh.OrderDate DESC;
    ```

    このクエリでは、注文日と小計と共に顧客の一覧を取得し、注文日で降順に並べ替えます。 

1. すべてのクエリ タブを閉じます。

## データを外部データ ソースと統合する

祝日に関する外部データを販売注文と統合します。 次に、祝日と一致する販売注文を特定し、休日が販売活動に与える影響に関する分析情報を提供します。

1. **[ホーム]** に移動し、**[新しいクエリ]** を選択します。

1. 新しい空のクエリ ペインに、次の T-SQL コードを入力して実行します。

    ```sql
    CREATE TABLE SalesLT.PublicHolidays (
        CountryOrRegion NVARCHAR(50),
        HolidayName NVARCHAR(100),
        Date DATE,
        IsPaidTimeOff BIT
    );
    ```

    このクエリは、次の手順の準備として `SalesLT.PublicHolidays` テーブルを作成します。

1. 新しいクエリ エディターに、次の T-SQL コードを入力して実行します。

    ```sql
    INSERT INTO SalesLT.PublicHolidays (CountryOrRegion, HolidayName, Date, IsPaidTimeOff)
    SELECT CountryOrRegion, HolidayName, Date, IsPaidTimeOff
    FROM OPENROWSET 
    (BULK 'abs://holidaydatacontainer@azureopendatastorage.blob.core.windows.net/Processed/*.parquet'
    , FORMAT = 'PARQUET') AS [PublicHolidays]
    WHERE countryorRegion in ('Canada', 'United Kingdom', 'United States')
        AND YEAR([date]) = 2024
    ```
    
    このクエリは、Azure Blob Storage の Parquet ファイルから休日データを読み取り、2024 年のカナダ、英国、および米国の休日のみを含むようにフィルター処理し、このフィルター処理されたデータを `SalesLT.PublicHolidays` テーブルに挿入します。    

1. 新規または既存のクエリ エディターで、次の T-SQL コードを入力して実行します。

    ```sql
    -- Insert new addresses into SalesLT.Address
    INSERT INTO SalesLT.Address (AddressLine1, City, StateProvince, CountryRegion, PostalCode, rowguid, ModifiedDate)
    VALUES
        ('123 Main St', 'Seattle', 'WA', 'United States', '98101', NEWID(), GETDATE()),
        ('456 Maple Ave', 'Toronto', 'ON', 'Canada', 'M5H 2N2', NEWID(), GETDATE()),
        ('789 Oak St', 'London', 'England', 'United Kingdom', 'EC1A 1BB', NEWID(), GETDATE());
    
    -- Insert new orders into SalesOrderHeader
    INSERT INTO SalesLT.SalesOrderHeader (
        SalesOrderID, RevisionNumber, OrderDate, DueDate, ShipDate, Status, OnlineOrderFlag, 
        PurchaseOrderNumber, AccountNumber, CustomerID, ShipToAddressID, BillToAddressID, 
        ShipMethod, CreditCardApprovalCode, SubTotal, TaxAmt, Freight, Comment, rowguid, ModifiedDate
    )
    VALUES
        (1001, 1, '2024-12-25', '2024-12-30', '2024-12-26', 1, 1, 'PO12345', 'AN123', 1, (SELECT TOP 1 AddressID FROM SalesLT.Address WHERE AddressLine1 = '789 Oak St'), (SELECT TOP 1 AddressID FROM SalesLT.Address WHERE AddressLine1 = '123 Main St'), 'Ground', '12345', 100.00, 10.00, 5.00, 'New Order 1', NEWID(), GETDATE()),
        (1002, 1, '2024-11-28', '2024-12-03', '2024-11-29', 1, 1, 'PO67890', 'AN456', 2, (SELECT TOP 1 AddressID FROM SalesLT.Address WHERE AddressLine1 = '123 Main St'), (SELECT TOP 1 AddressID FROM SalesLT.Address WHERE AddressLine1 = '456 Maple Ave'), 'Air', '67890', 200.00, 20.00, 10.00, 'New Order 2', NEWID(), GETDATE()),
        (1003, 1, '2024-02-19', '2024-02-24', '2024-02-20', 1, 1, 'PO54321', 'AN789', 3, (SELECT TOP 1 AddressID FROM SalesLT.Address WHERE AddressLine1 = '456 Maple Ave'), (SELECT TOP 1 AddressID FROM SalesLT.Address WHERE AddressLine1 = '789 Oak St'), 'Sea', '54321', 300.00, 30.00, 15.00, 'New Order 3', NEWID(), GETDATE()),
        (1004, 1, '2024-05-27', '2024-06-01', '2024-05-28', 1, 1, 'PO98765', 'AN321', 4, (SELECT TOP 1 AddressID FROM SalesLT.Address WHERE AddressLine1 = '789 Oak St'), (SELECT TOP 1 AddressID FROM SalesLT.Address WHERE AddressLine1 = '789 Oak St'), 'Ground', '98765', 400.00, 40.00, 20.00, 'New Order 4', NEWID(), GETDATE());
    ```

    このコードは、新しい住所と注文をデータベースに追加し、さまざまな国からの架空の注文をシミュレートします。

1. 新規または既存のクエリ エディターで、次の T-SQL コードを入力して実行します。

    ```sql
    SELECT DISTINCT soh.SalesOrderID, soh.OrderDate, ph.HolidayName, ph.CountryOrRegion
    FROM SalesLT.SalesOrderHeader AS soh
    INNER JOIN SalesLT.Address a
        ON a.AddressID = soh.ShipToAddressID
    INNER JOIN SalesLT.PublicHolidays AS ph
        ON soh.OrderDate = ph.Date AND a.CountryRegion = ph.CountryOrRegion
    ```

    少し時間を取って、それぞれの国の祝日と一致する販売注文をクエリで識別する方法に注意して、結果を確認します。 これにより、注文パターンに関する貴重な分析情報と、休日が販売活動に及ぼす潜在的な影響を把握できます。

1. すべてのクエリ タブを閉じます。

## データをセキュリティで保護する

レポートを生成するために、特定のユーザー グループが米国のデータにのみアクセスできるようにする必要があるとします。

先ほど使用したクエリに基づいてビューを作成し、それにフィルターを追加してみましょう。

1. 新しい空のクエリ ペインに、次の T-SQL コードを入力して実行します。

    ```sql
    CREATE VIEW SalesLT.vw_SalesOrderHoliday AS
    SELECT DISTINCT soh.SalesOrderID, soh.OrderDate, ph.HolidayName, ph.CountryOrRegion
    FROM SalesLT.SalesOrderHeader AS soh
    INNER JOIN SalesLT.Address a
        ON a.AddressID = soh.ShipToAddressID
    INNER JOIN SalesLT.PublicHolidays AS ph
        ON soh.OrderDate = ph.Date AND a.CountryRegion = ph.CountryOrRegion
    WHERE a.CountryRegion = 'United Kingdom';
    ```

1. 新規または既存のクエリ エディターで、次の T-SQL コードを入力して実行します。

    ```sql
    -- Create the role
    CREATE ROLE SalesOrderRole;
    
    -- Grant select permission on the view to the role
    GRANT SELECT ON SalesLT.vw_SalesOrderHoliday TO SalesOrderRole;
    ```

    `SalesOrderRole` ロールにメンバーとして追加されたユーザーは、フィルター処理されたビューにのみアクセスできます。 このロールのユーザーが他のユーザー オブジェクトにアクセスしようとすると、次のようなエラー メッセージが表示されます。

    ```
    Msg 229, Level 14, State 5, Line 1
    The SELECT permission was denied on the object 'ObjectName', database 'DatabaseName', schema 'SchemaName'.
    ```

> **詳細情報**: プラットフォームで使用できる他のコンポーネントの詳細については、Microsoft Fabric ドキュメントの「[Microsoft Fabric とは](https://learn.microsoft.com/fabric/get-started/microsoft-fabric-overview)」をご覧ください。

この演習では、Microsoft Fabric の SQL データベースで外部データの作成とインポートを行い、データのクエリとセキュリティ保護を行いました。

## リソースをクリーンアップする

データベースの探索が完了したら、この演習用に作成したワークスペースを削除できます。

1. 左側のバーで、ワークスペースのアイコンを選択して、それに含まれるすべての項目を表示します。
2. ツール バーの **[...]** メニューで、 **[ワークスペースの設定]** を選択してください。
3. **[全般]** セクションで、**[このワークスペースの削除]** を選択します。
