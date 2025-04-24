---
lab:
  title: Microsoft Fabric 内のデータ ウェアハウスを監視する
  module: Monitor a data warehouse in Microsoft Fabric
---

# Microsoft Fabric 内のデータ ウェアハウスを監視する

Microsoft Fabric では、データ ウェアハウスによって大規模な分析用のリレーショナル データベースが提供されます。 Microsoft Fabric 内のデータ ウェアハウスには、アクティビティとクエリの監視に使用できる動的管理ビューが含まれています。

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

## 動的管理ビューを探索する

Microsoft Fabric データ ウェアハウスには動的管理ビュー (DMV) が含まれています。これを使用して、データ ウェアハウス インスタンス内の現在のアクティビティを識別できます。

1. **[sample-dw]** データ ウェアハウス ページの **[新しい SQL クエリ]** ドロップダウン リストで、**[新しい SQL クエリ]** を選択します。
1. 新しい空のクエリ ペインに次の Transact-SQL コードを入力して、**sys.dm_exec_connections** DMV にクエリを実行します。

    ```sql
   SELECT * FROM sys.dm_exec_connections;
    ```

1. **[&#9655; 実行]** ボタンを使用して SQL スクリプトを実行し、結果を確認します。結果には、データ ウェアハウスへの接続の詳細が含まれています。
1. 次のように、**sys.dm_exec_sessions** DMV にクエリを実行するように SQL コードを変更します。

    ```sql
   SELECT * FROM sys.dm_exec_sessions;
    ```

1. 変更したクエリを実行し、結果を確認します。結果には、すべての認証済みセッションの詳細が表示されます。
1. 次のように、**sys.dm_exec_requests** DMV にクエリを実行するように SQL コードを変更します。

    ```sql
   SELECT * FROM sys.dm_exec_requests;
    ```

1. 変更したクエリを実行し、結果を確認します。結果には、データ ウェアハウスで実行中のすべての要求の詳細が表示されます。
1. 次のように、DMV を結合して同じデータベースで現在実行中の要求リに関する情報を返すよう SQL コードを変更します。

    ```sql
   SELECT connections.connection_id,
    sessions.session_id, sessions.login_name, sessions.login_time,
    requests.command, requests.start_time, requests.total_elapsed_time
   FROM sys.dm_exec_connections AS connections
   INNER JOIN sys.dm_exec_sessions AS sessions
       ON connections.session_id=sessions.session_id
   INNER JOIN sys.dm_exec_requests AS requests
       ON requests.session_id = sessions.session_id
   WHERE requests.status = 'running'
       AND requests.database_id = DB_ID()
   ORDER BY requests.total_elapsed_time DESC;
    ```

1. 変更したクエリを実行し、結果を確認します。結果には、データベースで実行中のすべてのクエリ (このクエリを含む) の詳細が表示されます。
1. **[新しい SQL クエリ]** ドロップダウン リストで、**[新しい SQL クエリ]** を選択して、2 つ目のクエリ タブを追加します。次に、新しい空のクエリ タブで、次のコードを実行します。

    ```sql
   WHILE 1 = 1
       SELECT * FROM Trip;
    ```

1. クエリを実行したまま、DMV にクエリを実行するコードが含まれているタブに戻り、それを再実行します。 今回は、他のタブで実行中の 2 番目のクエリが結果に含まれているはずです。そのクエリの経過時間に注目します。
1. 数秒待ってからコードを再実行し、DMV に再度クエリを実行します。 もう一方のタブのクエリの経過時間が増加しているはずです。
1. クエリがまだ実行されている 2 番目のクエリ タブに戻り、**[キャンセル]** を選択して取り消します。
1. DMV にクエリを実行するコードが表示されたタブに戻り、クエリを再実行して、2 番目のクエリが実行されていないことを確認します。
1. すべてのクエリ タブを閉じます。

> **詳細情報**: DMV の使用の詳細については、Microsoft Fabric ドキュメントの「[DMV を使用した接続、セッション、要求を監視する](https://learn.microsoft.com/fabric/data-warehouse/monitor-using-dmv)」を参照してください。

## クエリの分析情報を調べる

Microsoft Fabric データ ウェアハウスは、"クエリの分析情報" を提供します。これは、データ ウェアハウスで実行中のクエリの詳細を示す特別なビューのセットです。**

1. **[sample-dw]** データ ウェアハウス ページの **[新しい SQL クエリ]** ドロップダウン リストで、**[新しい SQL クエリ]** を選択します。
1. 新しい空のクエリ ペインに次の Transact-SQL コードを入力して、**exec_requests_history** ビューにクエリを実行します。

    ```sql
   SELECT * FROM queryinsights.exec_requests_history;
    ```

1. **[&#9655; 実行]** ボタンを使用して SQL スクリプトを実行し、結果を確認します。結果には、以前に実行したクエリの詳細が含まれています。
1. 次のように、**frequently_run_queries** ビューに対してクエリを実行するように SQL コードを変更します。

    ```sql
   SELECT * FROM queryinsights.frequently_run_queries;
    ```

1. 変更したクエリを実行し、結果を確認します。結果には、頻繁に実行されるクエリの詳細が表示されます。
1. 次のように、**long_running_queries** ビューに対してクエリを実行するように SQL コードを変更します。

    ```sql
   SELECT * FROM queryinsights.long_running_queries;
    ```

1. 変更したクエリを実行し、結果を確認します。結果には、すべてのクエリとそれらの継続時間の詳細が表示されます。

> **詳細情報**: クエリ分析情報の使用の詳細については、Microsoft Fabric ドキュメントの「[Fabric データ ウェアハウスのクエリ分析情報](https://learn.microsoft.com/fabric/data-warehouse/query-insights)」を参照してください。


## リソースをクリーンアップする

この演習では、動的管理ビューとクエリ分析情報を使用して、Microsoft Fabric データ ウェアハウスのアクティビティを監視しました。

データ ウェアハウスの探索が完了したら、この演習用に作成したワークスペースを削除できます。

1. 左側のバーで、ワークスペースのアイコンを選択して、それに含まれるすべての項目を表示してください。
1. **[ワークスペースの設定]** を選択し、**[全般]** セクションで下にスクロールし、**[このワークスペースを削除する]** を選択します。
1. **[削除]** を選択して、ワークスペースを削除します。
