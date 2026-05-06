---
lab:
  title: Microsoft Fabric でセマンティック モデルを AI 用に準備する
  module: Prepare the semantic layer for AI in Microsoft Fabric
  description: Power BI Desktop の "AI 用に準備" の機能を構成し、言語モデリングを介して同意語を追加し、Fabric ワークスペースに発行し、Copilot と HCAAT 診断を使用してテストし、モデルを "Copilot に対して承認済み" と設定します。
  duration: 30 minutes
  level: 300
  islab: true
  primarytopics:
    - Power BI
    - Semantic models
    - Copilot
    - Prep for AI
  categories:
    - Semantic models
  courses:
    - DP-600
---

# セマンティック モデルを AI 用に準備する

Copilot in Power BI によって正確な応答が生成されるかどうかは、セマンティック モデル メタデータに依存します。 準備されていない状態では、Copilot がビジネス用語を誤って解釈する、誤ったメジャーを使用する、返される回答の一貫性がなくなるなどが考えられます。 セマンティック レイヤーを準備しておくと、データをビジネス ユーザーが理解するのと同じ方法で Copilot が理解できるようになります。

学習内容は次のとおりです。

- ビジネスに関連するフィールドだけを Copilot で扱えるように AI データ スキーマを単純化します。
- AI にビジネス コンテキストと用語を教えるための指示を書きます。
- 一般的な質問に対して事前定義済みのビジュアルを返す、確認済みの回答を作成します。
- 自然言語解釈を向上させるために Q&A 言語モデリングを介して同意語を追加します。
- これまでの AI 用の準備を Copilot ペインと HCAAT 診断を使用してテストします。
- このセマンティック モデルを Power BI サービスで "Copilot 用に承認済み" に設定します。

このラボの所要時間は約 **30** 分です。

> **ヒント:** 関連するトレーニング コンテンツについては、「[ Microsoft Fabric で AI のセマンティック レイヤーを準備する](https://learn.microsoft.com/training/modules/fabric-prepare-semantic-layer/)」を参照してください。

## 環境を設定する

> **注**: この演習を完了するには、有料の Fabric 容量が必要です。 Fabric 試用版では、Copilot 機能はサポートされていません。 Fabric ライセンスの詳細については、[Microsoft Fabric ライセンス](https://learn.microsoft.com/fabric/enterprise/licenses)に関するページを参照してください。

この演習を完了するには、[Power BI Desktop](https://www.microsoft.com/download/details.aspx?id=58494) (2025 年 11 月以降) がインストールされている必要があります。 注: UI 要素は、お使いのバージョンによって多少異なる場合があります。**

1. Fabric ホーム ページで、**[Power BI]** を選択します。

1. 左側のナビゲーション バーで **[ワークスペース]** を選択し、**[新しいワークスペース]** を選択します。

1. 新しいワークスペースの名前を入力し (たとえば **dp_fabric**)、Fabric 容量が含まれているライセンス モード (有料 F2 以上) を選択して **[適用]** を選択します。

1. Web ブラウザーを開き、次の URL を入力して [30-prepare-model-ai zip フォルダー](https://github.com/MicrosoftLearning/mslearn-fabric/raw/refs/heads/main/Allfiles/Labs/30/30-prepare-model-ai.zip)をダウンロードします。

    `https://github.com/MicrosoftLearning/mslearn-fabric/raw/refs/heads/main/Allfiles/Labs/30/30-prepare-model-ai.zip`

1. このファイルを **[ダウンロード]** に保存し、zip ファイルの内容を **30-prepare-model-ai** フォルダーに抽出します。

1. 抽出したフォルダーにある **30-Starter-Sales Analysis.pbix** ファイルを開きます。

    > 変更を適用するかどうかを尋ねる警告が表示された場合は、無視して閉じてください。 [変更を破棄する] を選択しないでください。**

1. **[ホーム]** リボンの **[Copilot]** ボタンを選択します。 ワークスペースを選択する画面が表示されたら、自身で作成したワークスペース (たとえば **dp_fabric**) を選択して **[接続]** を選択します。 これで Power BI Desktop が Fabric 容量にリンクされて Copilot 機能を使えるようになります。

1. **[ホーム]** リボンにある **[データを AI 用に準備]** ボタンを見つけます。 見つからない場合は、最新バージョンの Power BI Desktop を使用していることを確認してください。

    > **注**: "データを AI 用に準備" はプレビュー機能です。[データを AI 用に準備] ダイアログのタブが無効化されている場合は、**[モデリング]** > **[Q&A の設定]** に移動してモデルに対して Q&A を有効にします。ダイアログを閉じて、もう一度お試しください。**

> **注**: ラボ VM を使っていてテキストの入力に問題がある場合は、[30-snippets.txt](https://github.com/MicrosoftLearning/mslearn-fabric/raw/refs/heads/main/Allfiles/Labs/30/30-snippets.txt) ファイルを `https://github.com/MicrosoftLearning/mslearn-fabric/raw/refs/heads/main/Allfiles/Labs/30/30-snippets.txt` からダウンロードして VM に保存してください。 このファイルには、このラボで使用されるすべてのテキスト スニペットとトリガー フレーズが含まれています。

## データ スキーマを単純化する

技術的フィールドを隠すと、Copilot がビジネスに関連するデータに焦点を絞ってより正確な応答を生成できるようになります。 このセクションでは、[AI 用に準備] ダイアログを使用して AI データ スキーマでの代理キー、並べ替え列、ETL メタデータの選択を解除します。

1. **[ホーム]** リボンの **[データを AI 用に準備]** を選択します。

1. 開いたダイアログで、**[データ スキーマを単純化する]** を左側のナビゲーションから選択するか、このカードを選択します。

1. 各テーブルを展開し、ビジネス ユーザーに関係のないフィールドの選択を解除します。 既定では、すべてのフィールドが AI から認識可能です。 各テーブルについて、**次のフィールドだけを残します**。

   - **`Customer`**: `City`、`CustomerName`
   - **`Date`**: `Date`、`Day`、`Month`、`Year`
   - **`Product`**: `Category`、`ListPrice`、`ProductName`、`StandardCost`、`Subcategory`
   - **`Sales`**: `LineTotal`、`OrderDate`、`OrderQty`、`Profit`、`Profit Margin`、`SalesOrderID`、`Total Sales`、`TotalCost`、`UnitPrice`

1. **適用**を選択します。

   選択が解除されるフィールドとしては、代理キー (`CustomerKey`、`ProductKey`)、並べ替えヘルパー列 (`Day (Sort Order)`、`Month (Sort Order)`)、ETL メタデータ (`LoadDate`、`SourceSystem`)、内部識別子 (`SalesOrderLineNumber`) があります。 `LoadDate` の選択を解除すると、その自動生成された日付階層も除去されますが、これは想定どおりです。時間ベースの分析は専用の `Date` テーブルによって処理されます。

1. [AI 用に準備] ダイアログの左側のナビゲーションから **[設定]** を選択します。

1. **[DAX 式を Copilot と共有する]** を有効にします。 これで、Copilot がメジャー内の基になる DAX ロジックを読み取ることができるようになるため、正しいメジャーを選択して計算を説明する能力が向上します。

## AI 指示を追加する

テーブル名とフィールドの説明には、組織のビジネス ルールと用語が常に反映されているとは限りません。 このセクションでは、ビジネス コンテキストを Copilot に渡すための AI 指示を書きます。これに含まれるものとしては、主要な用語、分析に関する好み、データ スコープなどがあります。

1. [AI 用に準備] ダイアログの左側のナビゲーションから **[AI 指示を追加]** を選択します。

1. 指示のテキスト ボックスに、次のビジネス コンテキストを入力します。

    ```text
    This model contains retail sales data for an outdoor recreation products company.

    Key business terminology:
    - "Revenue" and "sales" refer to the Total Sales measure.
    - Products are organized by Category and Subcategory.

    Analysis guidance:
    - When users ask about "top products," rank by Total Sales descending.
    - When users ask about profitability, use the Profit Margin measure (percentage), not the Profit measure (absolute amount).
    - Customers are identified by CustomerName, not CustomerKey.

    Data scope: This model covers retail sales from January 2024 through December 2025.
    ```

1. **[適用]** を選択してから、**[閉じる]** を選択して **[レポート]** ビューに戻ります。

## 確認済みの回答を作成する

事前定義済みの応答を用意しておくと、ビジネスに関する一般的な質問に対して常に同じ正確なビジュアルが返されるようになります。 このセクションでは、2 つのレポート ビジュアルに対して確認済みの回答を作成し、トリガー フレーズを割り当てます。このフレーズは、よく寄せられる質問にマップされます。

1. **売上概要**レポート ページで、`Total Sales` を表すカード ビジュアルを選択します。

1. このビジュアルの **[...]** メニューを選択し、**[確認済みの回答を設定]** を選択します。

1. 確認済みの回答のダイアログで、次のトリガー フレーズを追加します。

   - `What were total sales?`
   - `Show me total revenue`
   - `How much did we sell?`

1. **適用**を選択します。

1. `Sales by Category` を表す縦棒グラフ ビジュアルを選択し、確認済みの回答を設定するアクションを繰り返します。

   - `Show me sales by category`
   - `Which product categories have the most sales?`
   - `Break down revenue by product category`

1. **[適用]** と **[閉じる]** を選択します。

これで、2 つのビジュアルに対して確認済みの回答が作成され、ビジネスに関する一般的な質問を表すトリガー フレーズがこれに付加されました。

## 言語モデリングを使用して同意語を追加する

同意語は、自然言語ではさまざまなバリエーションがあるフィールド名やメジャー名を Copilot と Q&A が解釈するのに役立ちます。 このセクションでは、Q&A 言語モデリング セットアップを使用して主要フィールドに同意語を追加します。

1. **[モデリング]** リボンの **[Q&A の設定]** を選択します。

1. [Q&A の設定] ダイアログの **[同意語]** タブを選択します。

1. `Customer` テーブルから `CustomerName` を見つけます。 次の同意語を追加します。それぞれの後に **Enter** キーを押してください。

   - buyer
   - client

1. `Product` テーブルから `Category` を見つけます。 次の同意語を追加します。

   - 製品カテゴリ

   **[提案]** を見て `item` を追加します。

   > 提案された同意語を追加すると、承認済みの同意語を組織全体と共有するかどうかを尋ねる画面が表示されます。これを選択すると組織全体での一貫性が得られ、エンタープライズ オントロジのサポートにも役立ちます。 **[OK]** を選択して同意語を共有します。

1. `Sales` テーブルから `Profit Margin` メジャーを見つけます。 次の同意語を追加します。

   - margin
   - profit percentage
   - profitability

1. `Total Sales` メジャーを見つけます。 次の同意語を追加します。

   - 収入
   - 売り上げ
   - 合計収益別

1. [Q&A の設定] ダイアログを閉じます。

### 変更を発行して検証する

レポートを発行すると、セマンティック モデルが Power BI サービスで使用可能になるため、このサービスを使って Copilot でのテストを行い、モデルを承認済みと設定することができます。

1. レポートを**保存**して **[ホーム]** リボンの **[発行]** を選択します。

1. **[Power BI へ発行]** ダイアログで、この演習で作成したワークスペースを選択します。 発行が完了するまで待ちます。

1. 成功メッセージが表示されたら、Power BI サービスでレポートを **[開く]** リンクを選択するか、ブラウザーで自分のワークスペースに移動します。

### Copilot を使用してテストする

AI 用の準備の検証とは、Copilot で使用されるフィールド、メジャー、確認済みの回答が正しいかどうかを確認することです。 このセクションでは、発行済みのモデルを Power BI サービスの中で Copilot を使用してテストし、HCAAT 診断を使用して Copilot の推論を検査し、そのモデルを "Copilot 用に承認済み" と設定します。

1. Power BI サービスで、自分のワークスペースにある発行済みレポートを選択します。

1. レポート ツール バーの **[Copilot]** ボタンを選択して [Copilot] ペインを開きます。

1. [Copilot] ペインの **[このレポートについて質問する]** に質問を入力します。 **[データを理解する]** オプションが使用可能な場合は、これを選択してから質問を入力してください。

1. 確認済みの回答のいずれかに一致するような質問を入力します。

   - `What were total sales?`

   Copilot は、新しい応答を生成するのではなく、売上合計を示す事前定義済みのカード ビジュアルを返すはずです。

1. 2 番目の確認済み回答の質問を入力します。

   - `Show me sales by category`

   Copilot は、あなたが作成した確認済みの回答の対象である縦棒グラフ ビジュアルを返すはずです。

1. あなたが設定した同意語と AI 指示をテストするような質問を入力します。

   - `Show me the top products by profitability`

   あなたの AI 指示に基づいて、Copilot は `Profit` メジャーではなく `Profit Margin` メジャーを使用するはずです。 同意語 "profitability" が `Profit Margin` に正しくマップされているはずです。

1. 応答の下にある **[Copilot がこの回答に到達した方法]** (HCAAT) リンクを選択します。 どのフィールド、フィルター、メジャーを Copilot が使用したかを確認します。 Copilot が選択したメジャーが期待どおりであり、隠されたフィールドが使用されていないことを確認します。

1. 確認済みの回答がない、自由回答形式の質問を入力します。

   - `Which city had the highest sales?`

   HCAAT リンクをもう一度選択し、Copilot が `City` と `Total Sales` を使用していて代理キーや ETL 列は一切参照していないことを確認します。

1. 応答が不正確な場合は、質問に注目してください。 精度を向上させるには、AI 指示を調整する、確認済みの回答を追加する、同意語を改良するという方法があります。

### モデルを "Copilot 用に承認済み" と設定する

セマンティック モデルの承認とは、利用組織に対して、そのモデルが検証済みであり AI に使用できる状態であると知らせることです。 このタスクでは、発行済みのセマンティック モデルを "Copilot 用に承認済み" と設定します。

1. 自分のワークスペースに戻ります。

   > **注**: セマンティック モデルが表示されない場合は、ブラウザーを最新の情報に更新してください。

1. セマンティック モデル項目の省略記号 (**...**) を選択し、**[設定]** を選択します。

1. モデルの詳細ページの **[Copilot 用に承認済み]** セクションを展開します。

1. **[Copilot 用に承認済み]** オプションをオンにして **[適用]** を選択します。

これで、このセマンティック モデルが検索結果に現れやすくなります。 また、スタンドアロン Copilot in Power BI での、組織がデータを承認していないという警告も表示されなくなります。

## リソースをクリーンアップする

この演習では、Power BI Desktop の "AI 用に準備" 機能を構成し、同意語を追加し、モデルを Fabric ワークスペースに発行し、Copilot と HCAAT 診断を使用してテストし、モデルを "Copilot 用に承認済み" と設定しました。

1. Power BI サービスで、自分のワークスペースの画面に移動します。

1. ワークスペースの設定の画面で、**[その他]** を選択し、**[このワークスペースを削除する]** を選択します。
