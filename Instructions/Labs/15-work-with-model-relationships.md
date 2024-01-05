---
lab:
  title: モデル リレーションシップを操作する
  module: Design and build tabular models
---

# モデル リレーションシップを操作する

## 概要

**このラボの推定所要時間: 45 分**

このラボでは、特に多様ディメンションの必要性に対処するために、モデル リレーションシップを使用します。 アクティブなリレーションシップと非アクティブなリレーションシップの操作、およびリレーションシップの動作を変更するデータ分析式 (DAX) 関数も含まれます。

このラボでは、次の作業を行う方法について説明します。

- モデル図のリレーションシップ プロパティを解釈します。

- リレーションシップのプロパティを設定します。

- リレーションシップの動作を変更する DAX 関数を使用します。

## モデル リレーションシップを探索する

この演習では、事前に開発された Power BI Desktop ソリューションを開き、データ モデルについて学習します。 その後、アクティブなモデル リレーションシップの動作を調べていきます。

## はじめに
### このコースのリポジトリを複製する

1. スタート メニューで、コマンド プロンプトを開きます
   
    ![](../images/command-prompt.png)

2. コマンド プロンプト ウィンドウで、次のように入力して D ドライブに移動します。

    `d:` 

   Enter キーを押します。
   
    ![](../images/command-prompt-2.png)


3. コマンド プロンプト ウィンドウで、次のコマンドを入力して、コース ファイルをダウンロードし、DP500 という名前のフォルダーに保存します。
    
    `git clone https://github.com/MicrosoftLearning/DP-500-Azure-Data-Analyst DP500`
   

1. リポジトリが複製されたら、コマンド プロンプト ウィンドウを閉じます。 
   
1. エクスプローラーで D ドライブを開き、ファイルがダウンロードされていることを確認します。

### Power BI Desktop を設定する

このタスクでは、事前に開発された Power BI Desktop ソリューションを開きます。

1. エクスプローラーを開くには、タスク バーで**エクスプローラー**のショートカットを選択します。

2. エクスプローラーで、**D:\DP500\Allfiles\06\Starter** フォルダーに移動します。

3. 開発済みの Power BI Desktop ファイルを開くには、**Sales Analysis - Work with model relationships.pbix** ファイルをダブルクリックします。

4. ファイルを保存するには、 **[ファイル]** リボン タブで **[名前を付けて保存]** を選択します。

5. **[名前を付けて保存]** ウィンドウで、**D:\DP500\Allfiles\06\MySolution** フォルダーに移動します。

6. **[保存]** を選択します。

### データ モデルを確認する

このタスクでは、データ モデルを確認します。

1. Power BI Desktop の左側で、 **[モデル]** ビューに切り替えます。

    ![](../images/dp500-work-with-model-relationships-image2.png)

2. モデル図を使って、モデルのデザインを確認します。

    ![](../images/dp500-work-with-model-relationships-image3.png)

    モデルは、6 つのディメンション テーブルと 1 つのファクト テーブルで構成されます。**Sales** ファクト テーブルには販売注文の詳細が格納されます。これは、クラシック スター スキーマ設計です。**
 
3. **Date** と **Sales** のテーブルの間には 3 つのリレーションシップがあることがわかります。

    ![](../images/dp500-work-with-model-relationships-image4.png)

    **Date** テーブルの **DateKey** 列は、リレーションシップの "一" の側を表す一意の列です。**Date** テーブルの任意の列に適用されたフィルターは、いずれかのリレーションシップを使用して **Sales** テーブルに反映されます。**

4. 3 つのリレーションシップのそれぞれにカーソルを合わせると、**Sales** テーブルの "多" 側の列が強調表示されます。

5. **OrderDateKey** 列とのリレーションシップは実線であり、他のリレーションシップは点線で表されていることに注意してください。

    実線は、アクティブなリレーションシップを表します。2 つのモデル テーブル間にアクティブなリレーションシップ パスは 1 つだけ存在でき、そのパスが既定でテーブル間でのフィルター反映のために使用されます。逆に、点線は非アクティブなリレーションシップを表します。非アクティブなリレーションシップは、DAX 数式によって明示的に呼び出された場合にのみ使用されます。**

    現在のモデル デザインは、**Date** テーブルが多様ディメンションであることを示しています。このディメンションは、受注日、納品期日、または出荷日としての役割を果たすことができます。どの役割になるかは、レポートの分析要件によって決まります。**

    このラボでは、多様ディメンションをサポートするモデルを設計する方法について説明します。**

### 日付データを視覚化する

このタスクでは、日付別に販売データを視覚化し、リレーションシップのアクティブな状態を切り替えます。

1. **レポート** ビューに切り替えます。

    ![](../images/dp500-work-with-model-relationships-image5.png)

2. テーブル ビジュアルを追加するには、 **[視覚化]** ペインで**テーブル** ビジュアル アイコンを選択します。

    ![](../images/dp500-work-with-model-relationships-image6.png)

3. テーブル ビジュアルに列を追加するには、 **[データ]** ペイン (右側にあります) で、まず **Date** テーブルを展開します。

    ![](../images/dp500-work-with-model-relationships-image7.png)

4. **Fiscal Year** 列をドラッグして、テーブル ビジュアルにドロップします。

    ![](../images/dp500-work-with-model-relationships-image8.png)

5. **Sales** テーブルを展開して開き、**Sales Amount** 列をドラッグしてテーブル ビジュアルにドロップします。

    ![](../images/dp500-work-with-model-relationships-image9.png)

6. テーブル ビジュアルを確認します。

    ![](../images/dp500-work-with-model-relationships-image10.png)

    テーブル ビジュアルには、**Sales Amount** 列が年別にグループ化されて表示されています。しかし、**Fiscal Year** は何を意味するのでしょうか。テーブル **Date** と **Sales** の間には **OrderDateKey** 列とのアクティブなリレーションシップがあるため、**Fiscal Year** は、注文を受けた会計年度を意味します。**

    どの会計年度かを明確にするために、ビジュアル フィールドの名前を変更 (またはビジュアルにタイトルを追加) しましょう。**

7. テーブル ビジュアルの **[視覚化]** ペインで、 **[値]** ウェル内から下矢印を選択し、 **[この視覚エフェクトの名前変更]** を選択します。

    ![](../images/dp500-work-with-model-relationships-image11.png)

8. テキストを「**Order Year**」に置き換えて、**Enter** キーを押します。

    ![](../images/dp500-work-with-model-relationships-image12.png)

    ヒント: ビジュアル フィールドの名前を変更するには、ビジュアル フィールドの名前をダブルクリックすると簡単です。**

9. テーブル ビジュアルの列ヘッダーが新しい名前に更新されていることに注意してください。

    ![](../images/dp500-work-with-model-relationships-image13.png)

### リレーションシップのアクティブな状態を変更する

このタスクでは、2 つのリレーションシップのアクティブな状態を変更します。

1. **[モデリング]** リボンの **[リレーションシップの管理]** を選択します。

    ![](../images/dp500-work-with-model-relationships-image14.png)

2. **[リレーションシップの管理]** ウィンドウで、**Sales** と **Date** のテーブルの間のリレーションシップについて、**OrderDateKey** 列 (リストの 3 番目) の **[アクティブ]** チェック ボックスをオフにします。

    ![](../images/dp500-work-with-model-relationships-image15.png)

3. **Sales** と **Date** のテーブルの間のリレーションシップについて、**ShipDateKey** 列 (リストの最後) の **[アクティブ]** チェック ボックスをオンにします。

    ![](../images/dp500-work-with-model-relationships-image16.png)

4. **[閉じる]** を選択します。

    ![](../images/dp500-work-with-model-relationships-image17.png)

    これらの構成により、**Date** と **Sales** のテーブルの間のアクティブなリレーションシップが **ShipDateKey** 列に切り替わりました。**

5. テーブル ビジュアルに、出荷年別にグループ化された販売額が表示されるようになったことを確認します。

    ![](../images/dp500-work-with-model-relationships-image18.png)

6. 最初の列の名前を **Ship Year** に変更します。

    ![](../images/dp500-work-with-model-relationships-image19.png)

    最初の行は空白のグループを表します。これは、一部の注文がまだ出荷されていないためです。言い換えると、**Sales** テーブルの **ShipDateKey** 列に空白があります。**

7. **[リレーションシップの管理]** ウィンドウで、次の手順に従って **OrderDateKey** リレーションシップをアクティブに戻します。

    - **[リレーションシップの管理]** ウィンドウを開きます

    - **ShipDateKey** リレーションシップ (リストの最後) の **[アクティブ]** チェック ボックスをオフにします

    - **OrderDateKey** リレーションシップ (リストの 3 番目) の **[アクティブ]** チェック ボックスをオンにします

    - **[リレーションシップの管理]** ウィンドウを閉じます

    - テーブル ビジュアルの最初のビジュアル フィールドの名前を **Order Year** に変更します

    ![](../images/dp500-work-with-model-relationships-image20.png)

    次の演習では、DAX 数式でリレーションシップをアクティブにする方法を学習します。**

## 非アクティブなリレーションシップを使用する

この演習では、DAX 数式でリレーションシップをアクティブにする方法を学習します。

### 非アクティブなリレーションシップを使用する

このタスクでは、USERELATIONSHIP 関数を使用して非アクティブなリレーションシップをアクティブにします。

1. **[データ]** ペインで、**Sales** テーブルを右クリックし、 **[新しいメジャー]** を選択します。

    ![](../images/dp500-work-with-model-relationships-image21.png)

2. 数式バー (リボンの下にあります) で、テキストを次のメジャー定義に置き換え、**Enter** キーを押します。

    ヒント: すべての数式は、**D:\DP500\Allfiles\06\Assets\Snippets.txt** からコピーして貼り付けることができます。**

    ```
    Sales Shipped =
    CALCULATE (
    SUM ( 'Sales'[Sales Amount] ),
    USERELATIONSHIP ( 'Date'[DateKey], 'Sales'[ShipDateKey] )
    )
    ``` 
    この数式で、CALCULATE 関数を使用してフィルター コンテキストを変更します。この計算では、これが **ShipDateKey** リレーションシップをアクティブにする USERELATIONSHIP 関数です。**

3. **[メジャー ツール]** コンテキスト リボンの **[書式設定]** グループ内で、小数点以下の桁数を **2** に設定します。

    ![](../images/dp500-work-with-model-relationships-image22.png)

4. **Sales Shipped** メジャーをテーブル ビジュアルに追加します。

    ![](../images/dp500-work-with-model-relationships-image23.png)

5. テーブル ビジュアルの幅を広げて、すべての列が表示されるようにします。

    ![](../images/dp500-work-with-model-relationships-image24.png)

    リレーションシップをアクティブとして一時的に設定するメジャーを作成することは、多様ディメンションを操作する 1 つの方法です。ただし、多くのメジャー用に多様バージョンを作成する必要がある場合は、面倒になる可能性があります。たとえば、販売関連のメジャーが 10 個あり、多様ディメンションである日付が 3 個ある場合は、30 個のメジャーを作成することを意味します。それらを計算グループを使用して作成すると、プロセスが簡単になる可能性があります。**

    もう 1 つの方法は、多様ディメンションごとに異なるモデル テーブルを作成することです。これは、次の演習で行います。**

6. テーブル ビジュアルからメジャーを削除するには、 **[視覚化]** ペインの **[値]** ウェル内で、**Sales Shipped** フィールドの **[X]** を押します。

    ![](../images/dp500-work-with-model-relationships-image25.png)

## 別の日付テーブルを追加する

この演習では、出荷日の分析をサポートする日付テーブルを追加します。

### 非アクティブなリレーションシップを削除する

このタスクでは、**ShipDateKey** 列への既存のリレーションシップを削除します。

1. **[モデル]** ビューに切り替えます。

    ![](../images/dp500-work-with-model-relationships-image26.png)

2. モデル図で **ShipDateKey** リレーションシップを右クリックし、 **[削除]** を選択します。

    ![](../images/dp500-work-with-model-relationships-image27.png)

3. 削除するかどうかを確認するメッセージが表示されたら、 **[OK]** を選択します。

    ![](../images/dp500-work-with-model-relationships-image28.png)

    リレーションシップを削除すると、**Sales Shipped** メジャーにエラーが発生します。このラボでは、後ほどメジャーの数式を書き換えます。**

### リレーションシップ オプションを無効にする

このタスクでは、2 つのリレーションシップ オプションを無効にします。

1. **[ファイル]** リボン タブで、 **[オプションと設定]** を選択してから、 **[オプション]** を選択します。

    ![](../images/dp500-work-with-model-relationships-image29.png)

2. **[オプション]** ウィンドウの左下にある **[現在のファイル]** グループ内で、 **[データの読み込み]** を選択します。

    ![](../images/dp500-work-with-model-relationships-image30.png)

3. **[リレーションシップ]** セクションで、有効になっている 2 つのオプションをオフにします。

    ![](../images/dp500-work-with-model-relationships-image31.png)

    一般的に、日常の作業では、これらのオプションを有効にしておいてかまいません。ただし、このラボの目的のために、リレーションシップを明示的に作成します。**

4. **[OK]** を選択します。

    ![](../images/dp500-work-with-model-relationships-image32.png)

### 別の日付テーブルを追加する

このタスクでは、別の日付テーブルをモデルに追加するクエリを作成します。

1. **[ホーム]** リボン タブの **[クエリ]** グループ内から、 **[データの変換]** アイコンを選びます。

    ![](../images/dp500-work-with-model-relationships-image33.png) 

    接続方法の指定を求められたら、 **[資格情報を編集]** でサインイン方法を指定します。**

    ![](../images/dp500-work-with-model-relationships-image52.png)

    **[接続]** を選択します**

     ![](../images/dp500-work-with-model-relationships-image53.png)
 
2. **[Power Query エディター]** ウィンドウの **[クエリ]** ペイン (左側にあります) で、**Date** クエリを右クリックして、 **[参照]** を選びます。

    ![](../images/dp500-work-with-model-relationships-image34.png)

    参照元のクエリとは、別のクエリをソースとして使用するクエリです。つまり、この新しいクエリは、**Date** クエリから日付を 取得します。**

3. **[クエリの設定]** ペイン (右側にあります) で、 **[名前]** ボックスのテキストを「**Ship Date**」に置き換えます。

    ![](../images/dp500-work-with-model-relationships-image35.png)

4. **DateKey** 列の名前を変更するには、**DateKey** 列ヘッダーをダブルクリックします。

5. テキストを **ShipDateKey** に置き換えてから、**Enter** キーを押します。

    ![](../images/dp500-work-with-model-relationships-image36.png)

6. また、**Fiscal Year** 列の名前を **Ship Year** に変更します。

    可能であれば、すべての列の名前をそれらの役割がわかりやすいように変更することをお勧めします。このラボでは、簡単にするために 2 つの列の名前だけを変更します。**

7. テーブルをモデルに読み込むには、 **[ホーム]** リボン タブで **[閉じて適用]** アイコンを選びます。

    ![](../images/dp500-work-with-model-relationships-image37.png)

8. テーブルがモデルに追加されたら、リレーションシップを作成するために、**Ship Date** テーブルから **ShipDateKey** 列を **Sales** テーブルの **ShipDateKey** 列にドラッグします。

    ![](../images/dp500-work-with-model-relationships-image38.png)

9. **Ship Date** と **Sales** のテーブルの間にアクティブなリレーションシップが存在するようになったことに注目してください。

### 出荷日データを視覚化する

このタスクでは、新しいテーブル ビジュアル内の出荷日データを視覚化します。

1. **レポート** ビューに切り替えます。

    ![](../images/dp500-work-with-model-relationships-image39.png)

2. テーブル ビジュアルを複製するには、まずビジュアルを選択します。

3. **[ホーム]** リボン タブで、 **[クリップボード]** グループの **[コピー]** を選択します。

    ![](../images/dp500-work-with-model-relationships-image40.png)

4. コピーしたビジュアルを貼り付けるには、 **[ホーム]** リボン タブで、 **[クリップボード]** グループの **[貼り付け]** を選択します。

    ヒント: **Ctrl + C** と **Ctrl + V** のショートカットを使用することもできます。**

    ![](../images/dp500-work-with-model-relationships-image41.png)

5. 新しいテーブル ビジュアルを既存のテーブル ビジュアルの右側に移動します。
 
6. 新しいテーブル ビジュアルを選択し、 **[視覚化]** ペインの **[値]** ウェル内で、**Order Year** フィールドを削除します。

    ![](../images/dp500-work-with-model-relationships-image42.png)

7. **[データ]** ペインで、**Ship Date** テーブルを展開して開きます。

8. 新しいテーブル ビジュアルに新しいフィールドを追加するには、**Ship Date** テーブルから **Ship Year** フィールドを **[値]** ウェルの **Sales Amount** フィールドの上にドラッグします。

    ![](../images/dp500-work-with-model-relationships-image43.png)

9. 新しいテーブル ビジュアルに、出荷年別にグループ化された販売額が表示されていることを確認します。

    ![](../images/dp500-work-with-model-relationships-image44.png)

    これで、モデルに 2 つの日付テーブルが存在するようになり、それぞれに **Sales** テーブルとのアクティブなリレーションシップがあります。この設計アプローチの利点は、柔軟性があることです。すべてのメジャーと集計可能なフィールドをどちらの日付テーブルでも使用できるようになりました。**

    ただし、いくつかの欠点があります。各多様テーブルは、より大きな規模のモデルには有利ですが、ディメンション テーブルは通常、行数に関しては大規模ではありません。多様テーブルごとにモデル構成を複製することも必要になります (日付テーブルのマーキング、階層の作成、その他の設定など)。また、テーブルを追加すると、フィールドの数が圧倒的に多くなる可能性があります。ユーザーが必要なモデル リソースを見つけることが難しくなる場合があります。**

    最後に、1 つのビジュアルでフィルターの組み合わせを実現することはできません。たとえば、メジャーを作成せずに、同じビジュアルで受注済みの販売と出荷済みの販売を組み合わせることはできません。次の演習では、そのメジャーを作成します。**

## 他のリレーションシップ関数を探索する

この演習では、他の DAX リレーションシップ関数を操作します。

### 他のリレーションシップ関数を探索する

このタスクでは、CROSSFILTER および TREATAS 関数を使用して、計算中のリレーションシップの動作を変更します。

1. **[データ]** ペインの **Sales** テーブル内から、**Sales Shipped** メジャーを選択します。

    ![](../images/dp500-work-with-model-relationships-image45.png)

2. 数式バーのテキストを次の定義で置き換えます。

    ```
    Sales Shipped =
    CALCULATE (
    SUM ( 'Sales'[Sales Amount] ),
    CROSSFILTER ( 'Date'[DateKey], 'Sales'[OrderDateKey], NONE ),
    TREATAS (
    VALUES ( 'Date'[DateKey] ),
    'Ship Date'[ShipDateKey]
        )
    )
    ```

    この数式では、CALCULATE 関数を使用し、変更されたリレーションシップの動作を使って **Sales Amount** 列を合計します。CROSSFILTER 関数で、**OrderDateKey** 列へのアクティブなリレーションシップを無効にします (この関数はフィルターの方向を変更することもできます)。TREATAS 関数で、コンテキスト内の **DateKey** 値を **ShipDateKey** 列に適用することにより、仮想リレーションシップを作成します。**

3. 変更した **Sales Shipped** メジャーを最初のテーブル ビジュアルに追加します。

    ![](../images/dp500-work-with-model-relationships-image46.png)

4. 最初のテーブル ビジュアルを確認します。

    ![](../images/dp500-work-with-model-relationships-image47.png)

5. 空白のグループがないことに注意してください。

    **OrderDateKey** 列に空白がないため、空白のグループは生成されませんでした。未出荷の販売を表示するには、別のアプローチが必要です。**

### 未出荷の販売を表示する

このタスクでは、未出荷の販売額を表示するメジャーを作成します。

1. 次の定義を使用して、**Sales Unshipped** という名前のメジャーを作成します。

    ```
    Sales Unshipped =
    CALCULATE (
    SUM ( 'Sales'[Sales Amount] ),
    ISBLANK ( 'Sales'[ShipDateKey] )
    )
    ```
    この数式により、**ShipDateKey** 列が空白である **Sales Amount** 列を合計します。**

2. 小数点以下 2 桁を使用してメジャーの書式を設定します。

3. 新しいビジュアルをページに追加するには、最初にレポート ページの空白の領域を選択します。

4. **[視覚化]** ペインで **[カード]** ビジュアル アイコンを選択します。

    ![](../images/dp500-work-with-model-relationships-image48.png)

5. **Sales Unshipped** メジャーをカード ビジュアルにドラッグします。

    ![](../images/dp500-work-with-model-relationships-image49.png)

6. 最終的なレポート ページ レイアウトが次のようになっていることを確認します。

    ![](../images/dp500-work-with-model-relationships-image50.png)

### 仕上げ

このタスクでは、完了作業を行います。

1. Power BI Desktop ファイルを保存します。

    ![](../images/dp500-work-with-model-relationships-image51.png)

2. Power BI Desktop を閉じます。