# Raycast AI Extension + MCP 実装の懸念点

## 🔴 高リスク：要検証事項

### 1. Tool の戻り値として画像が表示されるか【最重要】

**問題:**

- AI Extension の Tool から Markdown 形式で `![image](url)` を返した時、AI Chat 内で実際に画像が表示されるかが未確認
- 公式ドキュメントには明記されていない

**検証方法:**

```typescript
export default function tool() {
  return `![chart](https://quickchart.io/chart?c={...})`;
}
```

**代替案（確実に動く方法）:**

- 通常の Extension として実装し、Detail コンポーネントを使う
- ブラウザで開く形式にする
- ASCII/Unicode グラフで妥協

**影響度:** ⭐⭐⭐⭐⭐
 **優先度:** 最初に検証すべき

------

### 2. Data URI のサイズ制限

**問題:**

- SVG や Canvas で生成した画像を Base64 Data URI として埋め込む場合、URL 長の制限がある可能性
- 複雑なグラフは URI が長くなりすぎる

**検証項目:**

- Raycast の Markdown パーサーが受け付ける URI の最大長
- AI Chat の制限

**対策:**

- QuickChart などの外部サービスを使う
- 一時ファイルとして保存し `file://` プロトコルで参照

**影響度:** ⭐⭐⭐
 **優先度:** 中

------

### 3. AI が適切に Tool を呼び出すか

**問題:**

- ユーザーの入力から AI が意図した Tool を正しく選択・実行できるかは不確実
- Tool の description とユーザーの表現がマッチしない可能性

**例:**

```
ユーザー: "売上をビジュアライズして"
→ AI が "visualize-sales" Tool を呼ぶ？
→ それとも別の Tool を呼ぶ？
→ 何も呼ばない？
```

**対策:**

- 明確な Tool description を書く
- Eval（テスト）を充実させる
- Custom Instructions で誘導
- 複数の表現でテスト（可視化、グラフ化、チャート、ビジュアライズ等）

**影響度:** ⭐⭐⭐⭐
 **優先度:** 高

------

## 🟡 中リスク：設計要検討事項

### 4. MCP と AI Extension の連携方法

**現状理解:**

- MCP Server と AI Extension は**独立したシステム**
- それぞれを `@mention` で呼び出す
- AI Extension から直接 MCP を呼ぶ標準的な方法は不明

**3つの実装パターン:**

#### パターンA: 完全分離

```
ユーザー: "@mcp データ取得"
AI: [データ表示]
ユーザー: "@extension グラフ化"
AI: [グラフ表示]
```

✅ 確実に動く
 ❌ 2ステップ必要

#### パターンB: Extension内で完結

```typescript
export default async function tool() {
  const data = await fetch('https://api.example.com'); // 直接API
  return generateChart(data);
}
```

✅ 1ステップで完結
 ❌ MCPのメリットなし

#### パターンC: AI が自動連携（理想）

```
ユーザー: "@mcp @extension 売上を可視化"
AI: 自動的に両方を使ってグラフ生成
```

✅ 最高のUX
 ❓ 実現可能性不明

**推奨:** パターンB（シンプルで確実）

**影響度:** ⭐⭐⭐
 **優先度:** 設計段階で決定

------

### 5. 認証情報の管理

**問題:**

- 外部 API の認証情報（API Key等）をどこに保存するか
- MCP Server なら環境変数で管理可能
- AI Extension では？

**選択肢:**

1. **Extension の Preferences**

   ```typescript
   import { getPreferenceValues } from "@raycast/api";
   const { apiKey } = getPreferenceValues();
   ```

   ✅ Raycast の標準的な方法
    ⚠️ ユーザーが設定する必要あり

2. **環境変数**

   ```typescript
   const apiKey = process.env.API_KEY;
   ```

   ✅ 開発時は楽
    ❌ 配布時に問題

3. **MCP Server に任せる**

   - MCP で認証・データ取得
   - Extension は可視化のみ ✅ 責任分離
      ❌ 2ステップ必要

**推奨:** Preferences を使用

**影響度:** ⭐⭐⭐
 **優先度:** 中

------

### 6. データサイズの制限

**問題:**

- QuickChart の URL パラメータには長さ制限がある
- 大量のデータポイントを可視化できない可能性

**制約:**

- URL 全体で約 2000〜8000 文字が一般的な限界
- JSON をエンコードすると更に長くなる

**対策:**

1. データポイントを間引く
2. サーバーサイドでチャート生成（Canvas等）
3. 別サービス利用（Chart.io, Image-Charts）
4. POST リクエストができるサービスを使う

**影響度:** ⭐⭐
 **優先度:** 低（必要になったら対応）

------

## 🟢 低リスク：確認事項

### 7. SVG インライン埋め込み

**問題:**

- SVG を直接 Markdown に埋め込めるか
- Data URI 化が必要か

**検証:**

```typescript
// パターン1: SVG直接
return `<svg>...</svg>`;

// パターン2: Data URI
return `![chart](data:image/svg+xml;base64,...)`;
```

**影響度:** ⭐⭐
 **優先度:** 低

------

### 8. ローカルファイルパス対応

**問題:**

- `file:///path/to/chart.png` が機能するか
- セキュリティ制約で拒否される可能性

**検証:**

```typescript
const path = '/tmp/chart.svg';
await fs.writeFile(path, svgContent);
return `![chart](file://${path})`;
```

**影響度:** ⭐
 **優先度:** 低（Web URLで十分）

------

## 📋 推奨検証順序

### フェーズ1: 最小構成（1日）

```typescript
export default function tool() {
  const mockData = [{x: 1, y: 100}, {x: 2, y: 150}];
  const url = `https://quickchart.io/chart?c=${encodeURIComponent(JSON.stringify({
    type: 'bar',
    data: { labels: ['A', 'B'], datasets: [{ data: [100, 150] }] }
  }))}`;
  return `![chart](${url})`;
}
```

**検証項目:**

- ✅ Tool が実行されるか
- ✅ Markdown が解析されるか
- ✅ 画像が表示されるか ← **最重要**

------

### フェーズ2: API統合（2-3日）

```typescript
export default async function tool() {
  const response = await fetch('https://api.example.com/data');
  const data = await response.json();
  const chartUrl = generateChart(data);
  return `![chart](${chartUrl})`;
}
```

**検証項目:**

- ✅ 非同期処理が動作するか
- ✅ 外部APIへのアクセス
- ✅ エラーハンドリング

------

### フェーズ3: 高度な機能（1週間〜）

- SVG/Canvas でのローカル生成
- Data URI 対応
- MCP との連携
- インタラクティブなチャート

------

## 🎯 最重要アクション

### 今すぐやるべきこと

1. **最小構成のプロトタイプ作成**
   - Tool から QuickChart URL を返すだけ
   - 画像表示の可否を確認
2. **公式ドキュメント・既存Extensionの調査**
   - 画像を扱っている AI Extension が存在するか
   - Stable Diffusion Extension など参考になるか
3. **Raycast コミュニティで質問**
   - Slack や GitHub Discussions で事前確認
   - 同じことを試した人がいるか

------

## 📊 リスクマトリクス

| 懸念事項             | 影響度 | 発生確率 | 優先度 |
| -------------------- | ------ | -------- | ------ |
| Tool戻り値の画像表示 | 高     | 中       | 最高   |
| AIのTool選択精度     | 高     | 中       | 高     |
| MCP連携方法          | 中     | 低       | 中     |
| 認証情報管理         | 中     | 低       | 中     |
| DataURIサイズ制限    | 中     | 中       | 中     |
| データサイズ制限     | 低     | 低       | 低     |
| SVG埋め込み          | 低     | 低       | 低     |
| ローカルファイル     | 低     | 高       | 低     |

------

## 🔗 参考リンク

- [Raycast AI Extensions 公式](https://manual.raycast.com/ai-extensions)
- [Model Context Protocol](https://manual.raycast.com/model-context-protocol)
- [Tool API Reference](https://developers.raycast.com/api-reference/tool)
- [Detail Component](https://developers.raycast.com/api-reference/user-interface/detail)
- [QuickChart.io](https://quickchart.io/)

------

**最終更新:** 2025-10-30
 **作成者:** AI アシスタント