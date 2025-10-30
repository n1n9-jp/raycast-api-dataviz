~~~markdown
# Raycast AI Extension ローカル開発ガイド

## 🎯 結論：ストア登録なしで開発・テスト可能！

Extension はストアに登録しなくても、**完全にローカル環境で開発・テスト可能**です。むしろ開発中は常にローカルで試します。

---

## 🛠️ ローカル開発の流れ

### 1. Extension の作成

```bash
# Raycast を開いて
# "Create Extension" コマンドを実行

# または CLI で
npm create raycast-extension
~~~

**プロンプトに従って入力：**

- Extension 名
- Description
- テンプレート選択（AI Extension なら AI Tool を選択）
- 保存場所

### 2. 開発モードで起動

```bash
cd your-extension-directory
npm install
npm run dev
```

`npm run dev` を実行すると：

- ✅ 開発モードで Extension が起動
- ✅ ホットリロード有効
- ✅ エラーレポート表示
- ✅ React Developer Tools 使用可能

### 3. Raycast で即座に使える

`npm run dev` を実行すると、**自動的に Raycast に Extension が追加**されます！

- Root Search に「Ask your-extension」として表示される
- AI Extension なら `@your-extension` で使える
- コード変更すると自動リロード

### 4. デバッグ

開発者ツールが使えます：

```typescript
export default function tool() {
  console.log("Debug info"); // ターミナルに表示される
  return "result";
}
```

- `console.log()` の出力が見える
- エラーメッセージがターミナルに表示
- React Developer Tools も使用可能

------

## 📝 具体的な開発手順

### ステップ 1: プロジェクト作成

#### Raycast UI から作成（推奨）

```
1. ⌘ + Space (Raycast起動)
2. "Create Extension" と入力
3. Enter

プロンプト:
- Name: data-visualizer
- Description: Visualize data with natural language
- Template: AI Extension (Tool)
- Location: ~/raycast-extensions/
```

#### CLI から作成

```bash
npm create raycast-extension

# 対話形式で入力
Name: data-visualizer
Description: Visualize data with natural language
Author: Your Name
Category: Developer Tools
Template: AI Extension
```

### ステップ 2: 開発開始

```bash
cd ~/raycast-extensions/data-visualizer
npm install
npm run dev
```

**ターミナルに表示される：**

```
✓ Extension loaded successfully
→ Open Raycast to see your extension
✓ Watching for changes...
```

### ステップ 3: Extension を使用

**Raycast を開いて：**

#### Root Search で

```
"Ask data-visualizer" が表示される
→ クリックして会話開始
```

#### AI Chat で

```
@data-visualizer と入力
→ AI Extension として使用可能
```

#### Quick AI で

```
@data-visualizer と入力
→ その場で使用可能
```

### ステップ 4: コード編集

```typescript
// src/tools/visualize.ts
export default function tool() {
  return "Hello from my extension!";
}

// ファイルを保存すると...
// → 自動でリロード！
// → Raycast を再起動不要
```

------

## 🔧 開発時の便利機能

### ホットリロード

- ✅ ファイル保存で自動リロード
- ✅ Raycast を再起動不要
- ✅ コードの変更が即座に反映

### デバッグログ

```typescript
export default function tool(input) {
  console.log("Tool called with:", input);
  console.log("Processing...");
  
  const result = processData(input);
  console.log("Result:", result);
  
  return result;
}
```

ターミナルに出力：

```
Tool called with: { query: "sales data" }
Processing...
Result: { chart: "..." }
```

### 開発中の Extension 管理

**Raycast で：**

```
1. "Manage Extensions" コマンド実行
2. 開発中の Extension が "Development" セクションに表示
3. 有効/無効を切り替え可能
4. ⌘ + E で Extension 設定を開く
```

### エラー確認

```typescript
export default function tool() {
  throw new Error("Something went wrong!");
  // → ターミナルにスタックトレース表示
  // → Raycast にもエラー通知
}
```

------

## 📦 ストア公開は完全にオプション

### ローカル開発だけなら：

- ✅ ストア登録不要
- ✅ 自分だけで使える
- ✅ 好きなだけ実験できる
- ✅ プライベート API キーも安全
- ✅ 公開基準を気にしなくて OK

### ストアに公開する場合のみ：

1. GitHub の raycast/extensions に PR を送る
2. レビューを受ける
3. 公開基準を満たす必要がある
4. メンテナンス責任が発生

**結論：プロトタイプや個人利用ならローカルだけで十分！**

------

## 🚀 明日の Meetup に向けて：今夜できること

### タイムライン（合計 1.5〜2 時間）

#### 1. 最小プロトタイプ作成（30分）

```bash
# Extension 作成
⌘ + Space
"Create Extension"

# 入力
Name: sales-visualizer
Description: Visualize sales data with charts
Template: AI Extension (Tool)

# 開発開始
cd ~/raycast-extensions/sales-visualizer
npm install
npm run dev
```

#### 2. Tool 実装（30〜45分）

```typescript
// src/tools/visualize-sales.ts

/**
 * 売上データを可視化する Tool
 */

type Input = {
  /** 期間（例: today, this-week, this-month） */
  period?: string;
};

export default function visualizeSales(input: Input) {
  // モックデータ（実際の API 呼び出しに置き換え可能）
  const data = [
    { label: '月', value: 100 },
    { label: '火', value: 150 },
    { label: '水', value: 120 },
    { label: '木', value: 180 },
    { label: '金', value: 200 }
  ];
  
  // QuickChart 用の設定
  const chartConfig = {
    type: 'bar',
    data: {
      labels: data.map(d => d.label),
      datasets: [{
        label: '売上',
        data: data.map(d => d.value),
        backgroundColor: 'rgba(54, 162, 235, 0.5)',
        borderColor: 'rgba(54, 162, 235, 1)',
        borderWidth: 1
      }]
    },
    options: {
      scales: {
        y: {
          beginAtZero: true
        }
      }
    }
  };
  
  // QuickChart URL 生成
  const chartUrl = `https://quickchart.io/chart?c=${encodeURIComponent(JSON.stringify(chartConfig))}`;
  
  // Markdown で返却
  return `📊 **売上グラフ（${input.period || '今週'}）**\n\n![Sales Chart](${chartUrl})\n\n各曜日の売上推移を表示しています。`;
}
```

**Tool の description を追加（package.json）:**

```json
{
  "tools": [
    {
      "name": "visualize-sales",
      "title": "Visualize Sales Data",
      "description": "売上データを棒グラフで可視化します。ユーザーが「売上」「グラフ」「可視化」「チャート」と言ったら使用してください。",
      "mode": "no-view"
    }
  ]
}
```

#### 3. 動作確認とデバッグ（30分）

**テストケース：**

1. **基本動作確認**

   ```
   Raycast AI Chat で:
   "@sales-visualizer 売上を表示して"
   ```

   期待結果：

   - Tool が呼ばれる
   - グラフの URL が返る
   - 画像が表示される（要検証！）

2. **別の表現で試す**

   ```
   "@sales-visualizer 今週のデータをグラフ化"
   "@sales-visualizer 売上チャート見せて"
   ```

3. **エラーケース**

   ```typescript
   // 意図的にエラーを起こして確認
   export default function visualizeSales() {
     throw new Error("Test error");
   }
   ```

4. **スクリーンショット撮影**

   - ✅ 動作する場合：グラフ表示の画面
   - ❌ 動作しない場合：エラーメッセージや URL 表示

#### 4. 発表資料準備（15分）

**スクリーンショットをまとめる：**

- Extension のコード
- Raycast での実行画面
- 成功 or 失敗の結果

**懸念点リストを印刷 or デバイスに保存**

------

## 💡 実装のポイント

### シンプルに始める

```typescript
// ❌ 複雑すぎ
export default async function tool(input) {
  const api = new APIClient();
  await api.authenticate();
  const data = await api.fetchWithRetry();
  // ...100行のコード
}

// ✅ シンプル
export default function tool() {
  const mockData = [1, 2, 3, 4, 5];
  const url = generateChartUrl(mockData);
  return `![chart](${url})`;
}
```

### まずはモックデータで

```typescript
// 最初はこれでOK
const data = [
  { label: 'A', value: 10 },
  { label: 'B', value: 20 }
];

// 動いたら実際のAPIに置き換え
// const data = await fetch('https://api.example.com/sales');
```

### エラーハンドリングは後回し

```typescript
// プロトタイプ段階では不要
// try-catch とか細かいエラー処理は後で追加
export default function tool() {
  // シンプルな実装だけ
  return generateChart();
}
```

------

## 📋 チェックリスト：Meetup 前

### 準備完了の基準

- [ ] Extension がローカルで動作する
- [ ] `@your-extension` で呼び出せる
- [ ] Tool が実行される（結果は問わず）
- [ ] スクリーンショット撮影済み
- [ ] 懸念点リスト持参
- [ ] 5分で説明できる内容を整理

### 最悪の場合でも大丈夫

- ❌ 画像が表示されない → 「これが最大の疑問点です」として発表
- ❌ Tool が呼ばれない → 「AI の選択精度について質問したい」
- ❌ 時間がない → 「アイデアと懸念点の共有」だけでも価値あり

------

## 🎤 発表構成案（5分）

### オープニング（30秒）

```
こんにちは、〇〇です。
今日は「AI Extension でデータ可視化に挑戦してみた」
という話をします。
```

### やりたいこと（1分）

```
自然言語で「売上を可視化して」と言うだけで
グラフが表示される Extension を作りたいと思いました。

なぜなら...
- データ分析を効率化したい
- Raycast の AI Extension を試したかった
```

### 実装（2分）

```
実装はシンプルです：
1. AI Extension の Tool として実装
2. QuickChart で画像 URL 生成
3. Markdown で返却

[コードを見せる]
[デモ or スクリーンショット]
```

### 懸念点と質問（1分30秒）

```
最大の疑問：
Tool の戻り値で Markdown 画像が表示されるのか？

その他の懸念：
- AI が適切に Tool を選択するか
- Data URI のサイズ制限
- MCP との連携は必要か

コミュニティの皆さんに聞きたいです！
```

### まとめ（30秒）

```
今日はプロトタイプの共有と
実装の懸念点について議論したくて発表しました。

フィードバックや同じようなことを
試した方がいたらぜひ教えてください！
```

------

## 🔗 参考リンク

### 公式ドキュメント

- [AI Extensions 開発ガイド](https://developers.raycast.com/ai/getting-started)
- [Create an AI Extension](https://developers.raycast.com/ai/create-an-ai-extension)
- [Tool API Reference](https://developers.raycast.com/api-reference/tool)

### サンプルコード

- [Extensions Repository](https://github.com/raycast/extensions)
- [QuickChart Documentation](https://quickchart.io/documentation/)

### コミュニティ

- [Raycast Community Japan Slack](https://join.slack.com/t/raycastcommunityjapan/shared_invite/zt-2o0futx5u-BMDYt7shHqAT2Fa82SPLfQ)
- [GitHub Discussions](https://github.com/raycast/extensions/discussions)

------

## ❓ よくある質問

### Q: ストアに公開しないと使えない？

**A: いいえ、ローカル開発だけで完全に使えます。**

### Q: npm run dev を止めたら動かなくなる？

**A: はい、開発モードを停止すると Extension も停止します。** 常に使いたい場合は `npm run build` でビルドして install。

### Q: 他の人に配布できる？

**A: できますが、相手も開発者モードで install する必要があります。** 簡単に配布したいならストアに公開が推奨。

### Q: API キーはどう管理する？

**A: Extension の Preferences に設定項目を追加できます。**

```typescript
import { getPreferenceValues } from "@raycast/api";
const { apiKey } = getPreferenceValues();
```

### Q: TypeScript 必須？

**A: Extension は TypeScript で書く必要があります。** ただし基本的な知識があれば十分。

------

## 🎯 成功の定義

### Meetup での成功とは：

1. ✅ **Extension が動く** → 最高！デモできる
2. ✅ **Extension が動かない** → 懸念点を共有、議論できる
3. ✅ **時間がなかった** → アイデアと質問だけでも価値あり

**どの状態でも、コミュニティと Thomas に質問できれば成功です！**

------

## 🚀 最後に

### 重要なマインドセット

- 完璧を目指さない
- プロトタイプで十分
- 失敗も学びの一部
- コミュニティの力を借りる

### 明日の Meetup で得るべきもの

1. **技術的な答え**
   - 画像表示の可否
   - 実装のベストプラクティス
2. **つながり**
   - 同じことに興味がある人
   - Raycast チームとの関係
3. **モチベーション**
   - 次のステップへの inspiration
   - コミュニティの熱量

**頑張ってください！楽しんできてください！** 🎉

------

**作成日:** 2025-10-30
 **イベント:** Raycast Community Meetup Tokyo #2 (2025-10-31)
 **目的:** ローカル開発とプロトタイプ作成のガイド

```

```