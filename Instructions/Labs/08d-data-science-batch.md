---
lab:
  title: バッチ予測を生成して保存する
  module: Generate batch predictions using a deployed model in Microsoft Fabric
---

# バッチ予測を生成して保存する

このラボでは、機械学習モデルを使用して、糖尿病の定量的尺度を予測します。 Fabric の PREDICT 関数を使用して、登録済みモデルで予測を生成します。

このラボを完了すると、予測の生成と結果の視覚化に関するハンズオン経験が得られます。

このラボは完了するまで、約 **20** 分かかります。

> **注**: この演習を完了するには、Microsoft Fabric ライセンスが必要です。 無料の Fabric 試用版ライセンスを有効にする方法の詳細については、[Fabric の概要](https://learn.microsoft.com/fabric/get-started/fabric-trial)に関するページを参照してください。 これを行うには、Microsoft の "*学校*" または "*職場*" アカウントが必要です。 お持ちでない場合は、[Microsoft Office 365 E3 以降の試用版にサインアップ](https://www.microsoft.com/microsoft-365/business/compare-more-office-365-for-business-plans)できます。

## ワークスペースの作成

Fabric でモデルを操作する前に、有効な Fabric 試用版を使用してワークスペースを作成します。

1. `https://app.fabric.microsoft.com` で [Microsoft Fabric](https://app.fabric.microsoft.com) にサインインし、 **[Power BI]** を選択してください。
2. 左側のメニュー バーで、 **[ワークスペース]** を選択します (アイコンは &#128455; に似ています)。
3. 任意の名前で新しいワークスペースを作成し、Fabric 容量を含むライセンス モード ("試用版"、*Premium*、または *Fabric*) を選択してください。**
4. 新しいワークスペースを開くと次に示すように空のはずです。

    ![Power BI の空のワークスペースのスクリーンショット。](./Images/new-workspace.png)

## Notebook のアップロード

データの取り込み、モデルのトレーニング、登録を行うには、ノートブックでセルを実行します。 ノートブックをワークスペースにアップロードできます。

1. 新しいブラウザー タブで、[Generate-Predictions](https://github.com/MicrosoftLearning/mslearn-fabric/blob/main/Allfiles/Labs/08/Generate-Predictions.ipynb) ノートブックに移動し、それを任意のフォルダーにダウンロードします。
1. Fabric ブラウザー タブに戻り、Fabric ポータルの左下にある **[Power BI]** アイコンを選択し、 **[データ サイエンス]** エクスペリエンスに切り替えます。
1. **[データ サイエンス]** ホーム ページで、 **[ノートブックのインポート]** を選択します。

    ノートブックが正常にインポートされると、通知を受け取ります。

1. `Generate-Predictions` という名前のインポートされたノートブックに移動します。
1. ノートブック内の指示を注意深く読み、各セルを個別に実行します。

## リソースをクリーンアップする

この演習では、モデルを使用してバッチ予測を生成しました。

ノートブックの詳細の確認が終了したら、この演習用に作成したワークスペースを削除して構いません。

1. 左側のバーで、ワークスペースのアイコンを選択して、それに含まれるすべての項目を表示します。
2. ツール バーの **[...]** メニューで、 **[ワークスペースの設定]** を選択します。
3. **[その他]** セクションで、 **[このワークスペースの削除]** を選択します。
