---
lab:
  title: Microsoft Fabric で GraphQL 用 API を使用する
  module: Get started with GraphQL in Microsoft Fabric
---

# Microsoft Fabric で GraphQL 用 API を使用する

GraphQL 用 Microsoft Fabric APIは、広く導入されている使い慣れた API テクノロジを使用して、複数のデータ ソースの迅速かつ効率的なクエリを可能にするデータ アクセス レイヤです。 この API を使用すると、バックエンド データ ソースの詳細を抽象化できるため、アプリケーション ロジックに集中し、クライアントが必要とするすべてのデータを 1 回の呼び出しで提供できます。 GraphQL では、単純な照会言語と簡単に操作できる結果セットを使用します。これにより、アプリケーションが Fabric のデータにアクセスするのにかかる時間が最小限に抑えられます。

このラボは完了するまで、約 **30** 分かかります。

> **注**:この演習を完了するには、[Microsoft Fabric 試用版](https://learn.microsoft.com/fabric/get-started/fabric-trial)が必要です。

## ワークスペースの作成

Fabric でデータを操作する前に、Fabric 試用版を有効にしてワークスペースを作成してください。

1. ブラウザーで [Microsoft Fabric ホーム ページ](https://app.fabric.microsoft.com/home?experience=fabric) (`https://app.fabric.microsoft.com/home?experience=fabric`) に移動し、Fabric 資格情報でサインインします。
1. 左側のメニュー バーで、**[新しいワークスペース]** を選択します。
1. 任意の名前で新しいワークスペースを作成し、Fabric 容量を含むライセンス モード ("試用版"、*Premium*、または *Fabric*) を選択します。**
1. 開いた新しいワークスペースは空のはずです。

    ![Fabric の空のワークスペースを示すスクリーンショット。](./Images/new-workspace.png)

## サンプル データを使用して SQL データベースを作成する

これでワークスペースが作成されたので、次に SQL データベースを作成します。

1. 左側のメニュー バーで、**[作成]** を選択します。 *[新規]* ページの *[データベース]* セクションで、**[SQL データベース]** を選択します。

    >**注**: **[作成]** オプションがサイド バーにピン留めされていない場合は、最初に省略記号 (**...**) オプションを選択する必要があります。

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

1. すべてのクエリ タブを閉じます。

## GraphQL 用 API を作成する

まず、GraphQL エンドポイントを設定して販売注文データを公開します。 このエンドポイントを使用すると、日付、顧客、製品などのさまざまなパラメーターに基づいて販売注文を照会できます。

1. Fabric ポータルで、ワークスペースに移動し、**[+ 新しい項目]** を選択します。
1. **[データの開発]** セクションに移動し、**[GraphQL 用 API]** を選択します。
1. 名前を指定し、**[作成]** を選択します。
1. GraphQL 用 API のメイン ページで、**[データ ソースの選択]** を選択します。
1. 接続オプションの選択を求められたら、**[シングル サインオン (SSO) 認証を使用して Fabric データ ソースに接続]** を選択します。
1. **[接続するデータの選択]** ページで、前に作成した `AdventureWorksLT` データベースを選択します。
1. **[接続]** を選択します。
1. **[データの選択]** ページで、`SalesLT.Product` テーブルを選択します。 
1. データをプレビューし、**[読み込み]** を選択します。
1. **[エンドポイントのコピー]** を選択し、パブリック URL リンクを書き留めます。 これは必要ありませんが、ここで API アドレスをコピーします。

## 変更を無効にする

API が作成されたので、このシナリオでは、読み取り操作用の売上データのみを公開します。

1. GraphQL 用 API の**スキーマ エクスプローラー**で、**[変更]** を展開します。
1. 各変更の横にある **...** (省略記号) を選択し、**[無効化]** を選択します。

これにより、API を使用したデータの変更または更新ができなくなります。 つまり、データは読み取り専用になり、ユーザーはデータの表示またはクエリのみを実行できますが、変更することはできません。

## GraphQL を使用してデータのクエリを実行する

次に、GraphQL を使用してデータのクエリを実行し、名前が *"HL Road Frame"* で始まるすべての製品を見つけましょう。

1. GraphQL クエリ エディターで、次のクエリを入力して実行します。

```json
query {
  products(filter: { Name: { startsWith: "HL Road Frame" } }) {
    items {
      ProductModelID
      Name
      ListPrice
      Color
      Size
      ModifiedDate
    }
  }
}
```

このクエリでは、製品がメインの種類であり、そこには `ProductModelID`、`Name`、`ListPrice`、`Color`、`Size`、および `ModifiedDate` のフィールドがあります。 このクエリは、名前が *"HL Road Frame"* で始まる製品のリストを返します。

> **詳細情報**: このプラットフォームで使用できる他のコンポーネントの詳細については、Microsoft Fabric ドキュメントの「[GraphQL 用の Microsoft Fabric API とは](https://learn.microsoft.com/fabric/data-engineering/api-graphql-overview)」をご覧ください。

この演習では、Microsoft Fabric の GraphQL を使用して、SQL データベースからのデータの作成、クエリ実行、および公開を行いました。

## リソースをクリーンアップする

データベースの探索が完了したら、この演習用に作成したワークスペースを削除できます。

1. 左側のバーで、ワークスペースのアイコンを選択して、それに含まれるすべての項目を表示します。
2. ツール バーの **[...]** メニューで、 **[ワークスペースの設定]** を選択してください。
3. **[全般]** セクションで、**[このワークスペースの削除]** を選択します。

