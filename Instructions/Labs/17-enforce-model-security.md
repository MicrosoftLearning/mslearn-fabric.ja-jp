---
lab:
  title: セマンティック モデルのセキュリティを適用する
  module: Design and build tabular models
---

# モデル セキュリティを適用する

この演習では、事前に開発されているデータ モデルを更新してセキュリティを適用します。 具体的には、Adventure Works 社の営業担当者は、割り当てられた営業地域に関連する売上データのみを表示できる必要があります。

この演習では、次の方法を学習します。

- 静的ロールを作成します。
- 動的ロールを作成します。
- ロールを検証します。
- セキュリティ プリンシパルをセマンティック モデルのロールにマップします。

このラボの所要時間は約 **45** 分です。

> **注**:この演習を完了するには、[Microsoft Fabric 試用版](https://learn.microsoft.com/fabric/get-started/fabric-trial)が必要です。

## 作業の開始

この演習では、環境を準備します。

### Power BI スターター ファイルをダウンロードする

1. `https://aka.ms/fabric-security-starter` から [Sales Analysis スターター ファイル](https://aka.ms/fabric-security-starter)をダウンロードし、それをローカル コンピューター (任意のフォルダー) に保存します。

1. ダウンロードしたファイルに移動し、Power BI Desktop で開きます。

1. メッセージが表示されたら、職場または学校のアカウントでサインインします。

### Power BI サービスにサインインする

このタスクでは、Power BI サービスにサインインし、試用版ライセンスを開始して、ワークスペースを作成します。

1. Web ブラウザーで、`https://app.powerbi.com/` に移動します。

2. Power BI Desktop で使用されているのと同じアカウントでサインイン プロセスを完了します。

    重要: Power BI Desktop からのサインインに使ったのと場合と同じ資格情報を使う必要があります。**

    "ヒント: Power BI の Web ブラウザー エクスペリエンスは、**Power BI サービス**と呼ばれます。"**

### ワークスペースの作成

このタスクでは、ワークスペースを作成します。

1. Power BI サービスでワークスペースを作成するには、**ナビゲーション** ウィンドウ (左側にあります) で **[ワークスペース]**、**[+ 新しいワークスペース]** の順に選びます。

2. **[ワークスペースの作成]** ペイン (右側にあります) で、**[名前]** ボックスにワークスペースの名前を入力します。

    "ワークスペース名はテナント内で一意である必要があります。"**

3. **適用**を選択します。

    "作成したら、ワークスペースが開きます。*後の演習では、セマンティック モデルをこのワークスペースに発行します。"*

### データ モデルを確認する

このタスクでは、データ モデルを確認します。

1. Power BI Desktop の左側で、 **[モデル]** ビューに切り替えます。

    ![](Images/enforce-model-security-image8.png)

2. モデル図を使って、モデルの設計を確認します。

    ![](Images/enforce-model-security-image9.png)

    モデルは、6 つのディメンション テーブルと 1 つのファクト テーブルで構成されます。**Sales** ファクト テーブルには販売注文の詳細が格納されます。これは、クラシック スター スキーマ設計です。**

3. **Sales Territory** テーブルを展開して開きます。

    ![](Images/enforce-model-security-image10.png)

4. テーブルに **Region** 列が含まれていることがわかります。

    **Region** 列には、Adventure Works の営業地域が格納されます。この組織では、営業担当者は、割り当てられた営業地域に関連するデータのみを表示できます。このラボでは、データのアクセス許可を適用するために、2 つの異なる行レベル セキュリティ手法を実装します。**

## 静的ロールを作成する

この演習では、静的ロールを作成して検証した後、セキュリティ プリンシパルをセマンティック モデルのロールにマップする方法を説明します。

### 静的ロールを作成する

このタスクでは、2 つの静的ロールを作成します。

1. **レポート** ビューに切り替えます。

    ![](Images/enforce-model-security-image11.png)

2. 積み上げ縦棒グラフ ビジュアルの凡例で、(現在は) 多くの地域を表示できることに注意してください。

    ![](Images/enforce-model-security-image12.png)

    現状のグラフは、必要以上に繁雑に見えます。これは、すべての地域を表示できるためです。ソリューションで行レベル セキュリティを適用すると、レポートのユーザーには 1 つの地域のみが表示されるようになります。**

3. セキュリティ ロールを追加するには、 **[モデリング]** リボン タブで、 **[セキュリティ]** グループ内から **[ロールの管理]** を選びます。

    ![](Images/enforce-model-security-image13.png)

4. **[ロールの管理]** ウィンドウで、 **[作成]** を選びます。

    ![](Images/enforce-model-security-image14.png)

5. ロールに名前を付けるには、選んだテキストを **Australia** に置き換えて、**Enter** キーを押します。

    ![](Images/enforce-model-security-image15.png)

6. **[テーブル]** の一覧で、**Sales Territory** テーブルの省略記号を選び、 **[フィルターの追加]**  >  **[[Region]]** を選びます。

    ![](Images/enforce-model-security-image16.png)

7. **[テーブル フィルターの DAX 式]** ボックスで、**Value** を **Australia** に置き換えます。

    ![](Images/enforce-model-security-image17.png)

    この式は、**Region** 列を値 **Australia** でフィルター処理します。**

8. 別のロールを作成するには、 **[作成]** を選びます。

    ![](Images/enforce-model-security-image18.png)

9. このタスクの手順を繰り返して、**Canada** で **Region** 列をフィルター処理する **Canada** という名前のロールを作成します。

    ![](Images/enforce-model-security-image19.png)

    このラボでは、ロールを 2 つだけ作ります。ただし、実際のソリューションでは、Adventure Works の 11 の地域ごとにロールを作成する必要があることを検討してください。**

10. **[保存]** を選択します。

    ![](Images/enforce-model-security-image20.png)

### 静的ロールを検証する

このタスクでは、静的ロールの 1 つを検証します。

1. **[モデリング]** リボン タブで、 **[セキュリティ]** グループ内から **[表示方法]** を選びます。

    ![](Images/enforce-model-security-image21.png)

2. **[ロールとして表示]** ウィンドウで、**Australia** ロールを選びます。

    ![](Images/enforce-model-security-image22.png)

3. **[OK]** を選択します。

    ![](Images/enforce-model-security-image23.png)

4. レポート ページで、積み上げ縦棒グラフ ビジュアルに表示されるデータが Australia のみであることに注意してください。

    ![](Images/enforce-model-security-image24.png)

5. レポートの上部に、適用されているロールを示す黄色のバナーが表示されることに注意してください。

    ![](Images/enforce-model-security-image25.png)

6. ロールを使った表示を止めるには、黄色いバナーの右側にある **[表示の停止]** を選びます。

    ![](Images/enforce-model-security-image26.png)

### レポートを発行する

このタスクでは、レポートを発行します。

1. Power BI Desktop ファイルを保存します。 保留中の変更を適用するように求められたら、**[後で適用]** を選びます。

    ![](Images/enforce-model-security-image27.png)

2. レポートを発行するには、 **[ホーム]** リボン タブの **[発行]** を選びます。

    ![](Images/enforce-model-security-image28.png)

3. **[Power BI へ発行]** ウィンドウで、自分のワークスペースを選んで **[選択]** を選びます。

    ![](Images/enforce-model-security-image29.png)

4. 発行が成功したら、 **[了解]** を選びます。

    ![](Images/enforce-model-security-image30.png)

### 行レベルのセキュリティを構成する ("省略可能")**

このタスクでは、Power BI サービスで行レベル セキュリティを構成する方法について説明します。

このタスクを行うには、作業中のテナントに **Salespeople_Australia** セキュリティ グループが存在している必要があります。 このセキュリティ グループは、テナントに自動的には存在しません。 テナントに対するアクセス許可がある場合は、次の手順に従うことができます。 トレーニングで提供されたテナントを使用している場合、セキュリティ グループを作成するための適切なアクセス許可はありません。 タスクをお読みください。ただし、セキュリティ グループが存在しない場合はタスクを完了できないことに注意してください。 **読んだ後、クリーンアップ タスクに進みます。**

1. Power BI サービス (Web ブラウザー) に切り替えます。

2. ワークスペースのランディング ページで、**Sales Analysis - Enforce model security** セマンティック モデルに注目してください。

    ![](Images/enforce-model-security-image31.png)

3. セマンティック モデルをカーソルでポイントし、省略記号が表示されたら、省略記号を選んで、**[セキュリティ]** を選びます。

    ![](Images/enforce-model-security-image32.png)

    **[セキュリティ]** オプションは、セキュリティ グループとユーザーを含む Microsoft Azure Active Directory (Azure AD) セキュリティ プリンシパルのマッピングをサポートします。**

4. 左側のロールの一覧で **Australia** が選択されていることに注意してください。

    ![](Images/enforce-model-security-image33.png)

5. **[メンバー]** ボックスに「**Salespeople_Australia**」と入力します。

    ステップ 5 から 8 は、Salespeople_Australia セキュリティ グループの作成または存在に依存するため、デモンストレーションのみを目的としています。セキュリティ グループを作成するためのアクセス許可と知識がある場合は、そのまま続けてください。それ以外の場合は、クリーンアップ タスクに進んでください。**

    ![](Images/enforce-model-security-image34.png)

6. **[追加]** を選択します。

    ![](Images/enforce-model-security-image35.png)

7. ロールのマッピングを完了するには、 **[保存]** を選びます。

    ![](Images/enforce-model-security-image36.png)

    これで、**Salespeople_Australia** セキュリティ グループのすべてのメンバーが **Australia** ロールにマップされ、オーストラリアの売上のみが表示されるようにデータ アクセスが制限されます。**

    実際のソリューションでは、各ロールをセキュリティ グループにマップする必要があります。**

    "*この設計アプローチはシンプルで、地域ごとにセキュリティ グループが存在する場合に効果的です。ただし欠点があります。作成と設定にはより多くの労力が必要です。また、新しい地域がオンボードされたときは、セマンティック モデルを更新して再発行する必要もあります。"*

    次の演習では、データドリブンの動的ロールを作成します。この設計アプローチは、これらの欠点に対処するのに役立ちます。**

8. ワークスペースのランディング ページに戻るには、**ナビゲーション** ウィンドウでワークスペースを選びます。

### ソリューションをクリーンアップする

このタスクでは、セマンティック モデルとモデルのロールを削除してソリューションをクリーンアップします。

1. セマンティック モデルを削除するには、セマンティック モデルをカーソルでポイントし、省略記号が表示されたら、省略記号を選んで、**[削除]** を選びます。

    ![](Images/enforce-model-security-image37.png)

    "次の演習では、変更されたセマンティック モデルを再発行します。"**

2. 削除するかどうかを確認するメッセージが表示されたら、**[削除]** を選択します。

    ![](Images/enforce-model-security-image38.png)

3. Power BI Desktop に切り替えます。

4. セキュリティ ロールを削除するには、 **[モデリング]** リボン タブで、 **[セキュリティ]** グループ内から **[ロールの管理]** を選びます。

    ![](Images/enforce-model-security-image39.png)

5. **[ロールの管理]** ウィンドウで最初のロールを削除するには、 **[削除]** を選びます。

    ![](Images/enforce-model-security-image40.png)

6. 削除の確認を求められたら、 **[はい、削除します]** を選びます。

    ![](Images/enforce-model-security-image41.png)

7. 2 番目のロールも削除します。

8. **[保存]** を選択します。

    ![](Images/enforce-model-security-image42.png)

## 動的ロールを作成する

この演習では、モデルにテーブルを追加し、動的ロールを作成して検証してから、セキュリティ プリンシパルをセマンティック モデルのロールにマップします。

### Salesperson テーブルを追加する

このタスクでは、**Salesperson** テーブルをモデルに追加します。

1. **[モデル]** ビューに切り替えます。

    ![](Images/enforce-model-security-image43.png)

2. **[ホーム]** リボン タブの **[クエリ]** グループ内から、 **[データの変換]** アイコンを選びます。

    ![](Images/enforce-model-security-image44.png)

    接続方法の指定を求められたら、 **[資格情報を編集]** でサインイン方法を指定します。**

    ![](Images/work-with-model-relationships-image52.png)

    **[接続]** を選択します**

     ![](Images/work-with-model-relationships-image53.png)

    ***[暗号化のサポート]** ページで、**[OK]** を選びます。*

3. **[Power Query エディター]** ウィンドウの **[クエリ]** ペイン (左側) で、**Customer** クエリを右クリックして、 **[複製]** を選びます。

    ![](Images/enforce-model-security-image45.png)

    **Customer** クエリにはデータ ウェアハウスを接続する手順が既に含まれているため、新しいクエリの作成を始めるには、それを複製するのが効率的な方法です。**

4. **[クエリの設定]** ペイン (右側) で、 **[名前]** ボックスのテキストを「**Salesperson**」に置き換えます。

    ![](Images/enforce-model-security-image46.png)

5. **[適用したステップ]** の一覧で、 **[削除された他の列]** ステップ (3 番目のステップ) を右クリックして、 **[最後まで削除]** を選びます。

    ![](Images/enforce-model-security-image47.png)

6. ステップの削除を確認するメッセージが表示されたら、 **[削除]** を選びます。

    ![](Images/enforce-model-security-image48.png)

7. 別のデータ ウェアハウスのテーブルからデータを取得するには、 **[適用したステップ]** の一覧の **[ナビゲーション]** ステップ (2 番目のステップ) で、歯車アイコン (右側) を選びます。

    ![](Images/enforce-model-security-image49.png)

8. **[ナビゲーション]** ウィンドウで、**DimEmployee** テーブルを選びます。

    ![](Images/enforce-model-security-image50.png)

9. **[OK]** を選択します。

    ![](Images/enforce-model-security-image51.png)

10. 必要のない列を削除するには、 **[ホーム]** リボン タブの **[列の管理]** グループ内から、 **[列の選択]** アイコンを選びます。

    ![](Images/enforce-model-security-image52.png)

11. **[列の選択]** ウィンドウで、すべての列のチェック ボックスをオフにするには、 **[(すべての列を選択)]** 項目のチェック ボックスをオフにします。

    ![](Images/enforce-model-security-image53.png)

12. 次の 3 つの列をオンにします。

    - EmployeeKey

    - SalesTerritoryKey

    - EmailAddress

13. **[OK]** を選択します。

    ![](Images/enforce-model-security-image54.png)

14. **EmailAddress** 列の名前を変更するには、**EmailAddress** 列のヘッダーをダブルクリックします。

15. テキストを「**UPN**」に置き換えてから、**Enter** キーを押します。

    UPN は、ユーザー プリンシパル名の頭字語です。この列の値は、Azure AD アカウント名と一致します。**

    ![](Images/enforce-model-security-image55.png)

16. テーブルをモデルに読み込むには、 **[ホーム]** リボン タブで **[閉じて適用]** アイコンを選びます。

    ![](Images/enforce-model-security-image56.png)

17. テーブルがモデルに追加されると、**Sales Territory** テーブルへのリレーションシップが自動的に作成されることに注意してください。

### リレーションシップを構成する

このタスクでは、新しいリレーションシップのプロパティを構成します。

1. **Salesperson** と **Sales Territory** テーブルの間のリレーションシップを右クリックして、 **[プロパティ]** を選びます。

    ![](Images/enforce-model-security-image57.png)

2. **[リレーションシップの編集]** ウィンドウの **[クロス フィルター方向]** ドロップダウン リストで、 **[両方]** を選びます。

3. **[両方向にセキュリティ フィルターを適用する]** チェック ボックスをオンにします。

    ![](Images/enforce-model-security-image58.png)

    **Sales Territory** テーブルから **Salesperson** テーブルには 1 対多のリレーションシップがあるため、フィルターは **Sales Territory** テーブルから **Salesperson** テーブルに対してのみ伝達されます。他方の方向に伝達を強制するには、クロス フィルターの方向を両方に設定する必要があります。**

4. **[OK]** を選択します。

    ![](Images/enforce-model-security-image59.png)

5. テーブルを非表示にするには、**Salesperson** テーブルの右上にある目のアイコンを選びます。

    ![](Images/enforce-model-security-image60.png)

    **Salesperson** テーブルの目的は、データのアクセス許可を適用することです。非表示にすると、レポート作成者と Q&A エクスペリエンスにテーブルまたはそのフィールドは表示されません。**

### 動的ロールを作成する

このタスクでは、モデル内のデータに基づいてアクセス許可を適用する動的ロールを作成します。

1. **レポート** ビューに切り替えます。

    ![](Images/enforce-model-security-image61.png)

2. セキュリティ ロールを追加するには、 **[モデリング]** リボン タブで、 **[セキュリティ]** グループ内から **[ロールの管理]** を選びます。

    ![](Images/enforce-model-security-image62.png)

3. **[ロールの管理]** ウィンドウで、 **[作成]** を選びます。

    ![](Images/enforce-model-security-image63.png)

4. ロールの名前を指定するには、選んだテキストを **Salespeople** に置き換えます。

    ![](Images/enforce-model-security-image64.png)

    今回は、作成する必要があるロールは 1 つだけです。**

5. **Salesperson** テーブルの **UPN** 列にフィルターを追加します。

    ![](Images/enforce-model-security-image65.png)

6. **[テーブル フィルターの DAX 式]** ボックスで、**"Value"** を `USERPRINCIPALNAME()` に置き換えます。

    ![](Images/enforce-model-security-image66.png)

    この式は、USERPRINCIPALNAME 関数で UPN 列をフィルター処理します。これにより、認証されたユーザーのユーザー プリンシパル名 (**UPN**) が返されます。**

    UPN で **Salesperson** テーブルをフィルター処理すると、**Sales Territory** テーブルがフィルター処理され、さらに **Sales** テーブルがフィルター処理されます。これにより、認証されたユーザーに、割り当てられた地域の売上データのみが表示されます。**

7. **[保存]** を選択します。

    ![](Images/enforce-model-security-image67.png)

### 動的ロールを検証する

このタスクでは、動的ロールを検証します。

1. **[モデリング]** リボン タブで、 **[セキュリティ]** グループ内から **[表示方法]** を選びます。

    ![](Images/enforce-model-security-image68.png)

2. **[ロールとして表示]** ウィンドウで **[その他のユーザー]** をオンにしてから、対応するボックスに「`michael9@adventure-works.com`」と入力します。

    ![](Images/enforce-model-security-image69.png)

    テストのため、 **[その他のユーザー]** を USERPRINCIPALNAME 関数によって返される値にします。この営業担当者は **Northeast** 地域に割り当てられていることに注意してください。**

3. **[営業担当者]** ロールを確認します。

    ![](Images/enforce-model-security-image70.png)

4. **[OK]** を選択します。

    ![](Images/enforce-model-security-image71.png)

5. レポート ページで、積み上げ縦棒グラフ ビジュアルに表示されるデータが Northeast のみであることに注意してください。

    ![](Images/enforce-model-security-image72.png)

6. レポートの上部に、適用されているロールを示す黄色のバナーが表示されることに注意してください。

    ![](Images/enforce-model-security-image73.png)

7. ロールを使った表示を止めるには、黄色いバナーの右側にある **[表示の停止]** を選びます。

    ![](Images/enforce-model-security-image74.png)

### 設計を完成させる

このタスクでは、レポートを発行し、セキュリティ グループをロールにマッピングすることで、設計を完成させます。

このタスクの手順は、意図的に簡単にしてあります。手順について詳しくは、前の演習のタスクの手順をご覧ください。**

1. Power BI Desktop ファイルを保存します。

2. ラボの最初に作成したワークスペースにレポートを発行します。

3. Power BI Desktop を閉じます。

4. Power BI サービス (Web ブラウザー) に切り替えます。

5. **Sales Analysis - Enforce model security** セマンティック モデルのセキュリティ設定に移動します。

6. **Salespeople** セキュリティ グループを **Salespeople** ロールにマップします。

    ![](Images/enforce-model-security-image76.png)

    これで、**Salespeople** セキュリティ グループのすべてのメンバーが **Salespeople** ロールにマップされます。認証されたユーザーが **Salesperson** テーブル内の行で表されていると、その割り当てられた営業区域が販売テーブルのフィルター処理に使われます。**

    この設計アプローチは、データ モデルにユーザー プリンシパル名の値が格納されている場合に、簡単で効果的です。営業担当者が追加または削除されたり、異なる営業区域に割り当てられたりしたとき、この設計アプローチは簡単に機能します。**
