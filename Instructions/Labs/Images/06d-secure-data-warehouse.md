---
lab:
  title: データ ウェアハウス内のデータをセキュリティで保護する
  module: Get started with data warehouses in Microsoft Fabric
---

# データ ウェアハウス内のデータをセキュリティで保護する

Microsoft Fabric のアクセス許可と詳細な SQL アクセス許可は連携して機能することで、ウェアハウスのアクセスとユーザーのアクセス許可を管理します。 この演習では、詳細なアクセス許可、列レベル セキュリティ、行レベル セキュリティ、動的データ マスクを使用してデータをセキュリティで保護します。

このラボは完了するまで、約 **45** 分かかります。

> **注**:この演習を完了するには、[Microsoft Fabric 試用版](https://learn.microsoft.com/fabric/get-started/fabric-trial)が必要です。

## ワークスペースの作成

Fabric でデータを操作する前に、Fabric 試用版を有効にしてワークスペースを作成してください。

1. [Microsoft Fabric ホーム ページ](https://app.fabric.microsoft.com)で、**Synapse Data Warehouse** を選択します。
1. 左側のメニュー バーで、 **[ワークスペース]** を選択します (アイコンは &#128455; に似ています)。
1. 任意の名前で新しいワークスペースを作成し、Fabric 容量を含むライセンス モード ("試用版"、*Premium*、または *Fabric*) を選択します。**
1. 開いた新しいワークスペースは空のはずです。

    ![Fabric の空のワークスペースを示すスクリーンショット。](./Images/new-empty-workspace.png)

## データ ウェアハウスの作成

次に、作成したばかりのワークスペース内にデータ ウェアハウスを作成します。 Synapse Data Warehouse のホーム ページには、新しいウェアハウスを作成するためのショートカットがあります。

1. **Synapse Data Warehouse** ホーム ページで、新しい**ウェアハウス**を任意の名前で作成します。

    1 分ほどで、新しいレイクハウスが作成されます。

    ![新しいウェアハウスのスクリーンショット。](./Images/new-empty-data-warehouse.png)

## テーブル内の列に動的データ マスクを適用する

動的データ マスク ルールは、テーブル レベルで個々の列に適用されるため、すべてのクエリがマスクの影響を受けます。 機密データを閲覧する明示的なアクセス許可を持たないユーザーにはクエリ結果においてマスクされた値が表示され、データを閲覧する明示的なアクセス許可を持つユーザーには伏せられていない状態で値が表示されます。 マスクには、既定、メール、ランダム、カスタム文字列という 4 つの種類があります。 この演習では、既定のマスク、メール マスク、カスタム文字列マスクを適用します。

1. ウェアハウスで **[T-SQL]** タイルを選択し、既定の SQL コードを次の T-SQL ステートメントに置き換えてテーブルを作成し、データを挿入して表示します。  `CREATE TABLE` ステートメントで適用されるマスクは、以下の処理を行います。

    ```sql
    CREATE TABLE dbo.Customer
    (   
        CustomerID INT NOT NULL,   
        FirstName varchar(50) MASKED WITH (FUNCTION = 'partial(1,"XXXXXXX",0)') NULL,     
        LastName varchar(50) NOT NULL,     
        Phone varchar(20) MASKED WITH (FUNCTION = 'default()') NULL,     
        Email varchar(50) MASKED WITH (FUNCTION = 'email()') NULL   
    );
    GO
    --Users restricted from seeing masked data will see the following when they query the table
    --The FirstName column shows the first letter of the string with XXXXXXX and none of the last characters.
    --The Phone column shows xxxx
    --The Email column shows the first letter of the email address followed by XXX@XXX.com.
    
    INSERT dbo.Customer (CustomerID, FirstName, LastName, Phone, Email) VALUES
    (29485,'Catherine','Abel','555-555-5555','catherine0@adventure-works.com'),
    (29486,'Kim','Abercrombie','444-444-4444','kim2@adventure-works.com'),
    (29489,'Frances','Adams','333-333-3333','frances0@adventure-works.com');
    GO

    SELECT * FROM dbo.Customer;
    GO
    ```

2. **[&#9655; 実行]** ボタンを使用して SQL スクリプトを実行します。これにより、データ ウェアハウスの **dbo** スキーマに **Customer** という名前の新しいテーブルが作成されます。

3. 次に、**[エクスプローラー]** ペインで **[スキーマ]** > **[dbo]** > **[テーブル]** の順に展開し、**Customer** テーブルが作成されていることを確認します。 あなたはマスクされていないデータを表示できるワークスペース管理者として接続されているため、SELECT ステートメントはマスクされていないデータを返します。

4. **閲覧者** ワークスペース ロールのメンバーであるテスト ユーザーとして接続し、次の T-SQL ステートメントを実行します。

    ```sql
    SELECT * FROM dbo.Customer;
    GO
    ```

    このユーザーには UNMASK アクセス許可が付与されていないため、返されるデータの FirstName、Phone、Email 列はマスクされます。これらの列は `CREATE TABLE` ステートメント内でマスクを使用して定義されたためです。

5. ワークスペース管理者として再接続し、次の T-SQL を実行して、テスト ユーザーに対してデータのマスクを解除します。

    ```sql
    GRANT UNMASK ON dbo.Customer TO [testUser@testdomain.com];
    GO
    ```

6. もう一度テスト ユーザーとして接続し、次の T-SQL ステートメントを実行します。

    ```sql
    SELECT * FROM dbo.Customer;
    GO
    ```

    テスト ユーザーに `UNMASK` アクセス許可が付与されたため、データはマスクされていない状態で返されます。

## 行レベル セキュリティを適用する

行レベル セキュリティ (RLS) を使用すると、ID またはクエリを実行するユーザーのロールに基づいて行へのアクセスを制限できます。  この演習では、セキュリティ ポリシーとインライン テーブル値関数として定義されたセキュリティ述語を作成して、行へのアクセスを制限します。

1. 前の演習で作成したウェアハウスで、**[新しい SQL クエリ]** ドロップダウンを選択します。  **[空白]** ヘッダーの下のドロップダウンで、**[新しい SQL クエリ]** を選択します。

2. テーブルを作成してデータを挿入します。 後の手順で行レベル セキュリティをテストできるように、'testuser1@mydomain.com' を自分の環境のユーザー名に、'testuser2@mydomain.com' を自分のユーザー名に置き換えます。
    ```sql
    CREATE TABLE dbo.Sales  
    (  
        OrderID INT,  
        SalesRep VARCHAR(60),  
        Product VARCHAR(10),  
        Quantity INT  
    );
    GO
     
    --Populate the table with 6 rows of data, showing 3 orders for each test user. 
    INSERT dbo.Sales (OrderID, SalesRep, Product, Quantity) VALUES
    (1, 'testuser1@mydomain.com', 'Valve', 5),   
    (2, 'testuser1@mydomain.com', 'Wheel', 2),   
    (3, 'testuser1@mydomain.com', 'Valve', 4),  
    (4, 'testuser2@mydomain.com', 'Bracket', 2),   
    (5, 'testuser2@mydomain.com', 'Wheel', 5),   
    (6, 'testuser2@mydomain.com', 'Seat', 5);  
    GO
   
    SELECT * FROM dbo.Sales;  
    GO
    ```

3. **[&#9655; 実行]** ボタンを使用して SQL スクリプトを実行します。これにより、データ ウェアハウスの **dbo** スキーマに **Sales** という名前の新しいテーブルが作成されます。

4. 次に、**[エクスプローラー]** ペインで **[スキーマ]** > **[dbo]** > **[テーブル]** の順に展開し、**Sales** テーブルが作成されていることを確認します。
5. 新しいスキーマ、関数として定義されたセキュリティ述語、セキュリティ ポリシーを作成します。  

    ```sql
    --Create a separate schema to hold the row-level security objects (the predicate function and the security policy)
    CREATE SCHEMA rls;
    GO
    
    --Create the security predicate defined as an inline table-valued function. A predicate evalutes to true (1) or false (0). This security predicate returns 1, meaning a row is accessible, when a row in the SalesRep column is the same as the user executing the query.

    --Create a function to evaluate which SalesRep is querying the table
    CREATE FUNCTION rls.fn_securitypredicate(@SalesRep AS VARCHAR(60)) 
        RETURNS TABLE  
    WITH SCHEMABINDING  
    AS  
        RETURN SELECT 1 AS fn_securitypredicate_result   
    WHERE @SalesRep = USER_NAME();
    GO 
    
    --Create a security policy to invoke and enforce the function each time a query is run on the Sales table. The security policy has a Filter predicate that silently filters the rows available to read operations (SELECT, UPDATE, and DELETE). 
    CREATE SECURITY POLICY SalesFilter  
    ADD FILTER PREDICATE rls.fn_securitypredicate(SalesRep)   
    ON dbo.Sales  
    WITH (STATE = ON);
    GO
6. Use the **&#9655; Run** button to run the SQL script
7. Then, in the **Explorer** pane, expand **Schemas** > **rls** > **Functions**, and verify that the function has been created.
7. Confirm that you're logged as another user by running the following T-SQL.

    ```sql
    SELECT USER_NAME();
    GO
5. Query the sales table to confirm that row-level security works as expected. You should only see data that meets the conditions in the security predicate defined for the user you're logged in as.

    ```sql
    SELECT * FROM dbo.Sales;
    GO

## Implement column-level security

Column-level security allows you to designate which users can access specific columns in a table. It is implemented by issuing a GRANT statement on a table specifying a list of columns and the user or role that can read them. To streamline access management, assign permissions to roles in lieu of individual users. In this exercise, you will create a table, grant access to a subset of columns on the table, and test that restricted columns are not viewable by a user other than yourself.

1. In the warehouse you created in the earlier exercise, select the **New SQL Query** dropdown.  Under the dropdown under the header **Blank**, select **New SQL Query**.  

2. Create a table and insert data into the table.

 ```sql
    CREATE TABLE dbo.Orders
    (   
        OrderID INT,   
        CustomerID INT,  
        CreditCard VARCHAR(20)      
        );
    GO

    INSERT dbo.Orders (OrderID, CustomerID, CreditCard) VALUES
    (1234, 5678, '111111111111111'),
    (2341, 6785, '222222222222222'),
    (3412, 7856, '333333333333333');
    GO

    SELECT * FROM dbo.Orders;
    GO
 ```

3. テーブル内の列を表示するためのアクセス許可を拒否します。 以下の Transact SQL は、'<testuser@mydomain.com>' が Orders テーブル内の CreditCard 列を表示することを防ぎます。 以下の `DENY` ステートメントで、testuser@mydomain.com をワークスペースに対する閲覧者アクセス許可を持つシステム内のユーザー名に置き換えます。

 ```sql
DENY SELECT ON dbo.Orders (CreditCard) TO [testuser@mydomain.com];
 ```

4. SELECT アクセス許可を禁止したユーザーとして Fabric にログインすることで、列レベル セキュリティをテストします。

5. Orders テーブルに対してクエリを実行し、列レベル セキュリティが想定通りに機能していることを確認します。 次のクエリが返すのは OrderID 列と CustomerID 列だけであり、CrediteCard 列は返しません。  

    ```sql
    SELECT * FROM dbo.Orders;
    GO

    --You'll receive an error because access to the CreditCard column has been restricted.  Try selecting only the OrderID and CustomerID fields and the query will succeed.

    SELECT OrderID, CustomerID from dbo.Orders
    ```

## T-SQL を使用して SQL の詳細なアクセス許可を構成する

Fabric ウェアハウスには、ワークスペース レベルおよびアイテム レベルでデータへのアクセスを制御できるアクセス許可モデルがあります。 Fabric ウェアハウスのセキュリティ保護可能なリソースに関してユーザーが何を実行できるかをより細かく制御する必要がある場合は、標準の SQL データ制御言語 (DCL) コマンドである `GRANT`、`DENY`、`REVOKE` を使用できます。 この演習では、オブジェクトを作成し、それらを `GRANT` と `DENY` を使用してセキュリティで保護し、その後クエリを実行して、詳細なアクセス許可の適用の効果を確認します。

1. 前の演習で作成したウェアハウスで、**[新しい SQL クエリ]** ドロップダウンを選択します。  **[空白]** ヘッダーで、**[新しい SQL クエリ]** を選択します。  

2. ストアド プロシージャとテーブルを作成します。

 ```
    CREATE PROCEDURE dbo.sp_PrintMessage
    AS
    PRINT 'Hello World.';
    GO
  
    CREATE TABLE dbo.Parts
    (
        PartID INT,
        PartName VARCHAR(25)
    );
    GO
    
    INSERT dbo.Parts (PartID, PartName) VALUES
    (1234, 'Wheel'),
    (5678, 'Seat');
    GO  
    
    --Execute the stored procedure and select from the table and note the results you get because you're a member of the Workspace Admin. Look for output from the stored procedure on the 'Messages' tab.
      EXEC dbo.sp_PrintMessage;
    GO
    
    SELECT * FROM dbo.Parts
    GO
  ```

3. 次にワークスペース閲覧者ロールのメンバーであるユーザーに対してテーブルに対するアクセス許可の `DENY SELECT` を実行し、同じユーザーに対してプロシージャに対する `GRANT EXECUTE` を実行します。

 ```sql
    DENY SELECT on dbo.Parts to [testuser@mydomain.com];
    GO

    GRANT EXECUTE on dbo.sp_PrintMessage to [testuser@mydomain.com];
    GO

 ```

4. [testuser@mydomain.com] の代わりに、上の DENY および GRANT ステートメントで指定したユーザーとして Fabric にログインします。 次に、ストアド プロシージャを実行し、テーブルに対してクエリを実行することで、先ほど適用した詳細なアクセス許可をテストします。  

 ```sql
    EXEC dbo.sp_PrintMessage;
    GO
    
    SELECT * FROM dbo.Parts
 ```

## リソースをクリーンアップする

この演習では、テーブル内の列に対して動的データ マスクを適用し、行レベル セキュリティを適用し、列レベル セキュリティを実装し、T-SQL を使用して SQL の詳細なアクセス許可を構成しました。

1. 左側のバーで、ワークスペースのアイコンを選択して、それに含まれるすべての項目を表示します。
2. ツール バーの **[...]** メニューで、 **[ワークスペースの設定]** を選択してください。
3. **[全般]** セクションで、**[このワークスペースの削除]** を選択します。
