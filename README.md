# packs/ フォルダ

ここに教材JSON(`juken-pack@1`)を置くと、アプリの「GitHubから自動取得」機能で読み込めます。

## 使い方

1. `docs/PROMPTS_FOR_NEXT_AI.md` の①のプロンプトで教材JSONを作る
2. このフォルダに保存する(例:`wariai_5nen.json`)
3. `index.json` に、追加したファイル名を書き足す

```json
["wariai_5nen.json", "goi_5nen.json"]
```

4. コミットしてpush
5. アプリの「データ」画面で GitHubユーザー名・リポジトリ名を設定し、「設定を保存していますぐ取得」

詳しい手順は `docs/AI_HANDOFF.md` §6.5 と `docs/PROMPTS_FOR_NEXT_AI.md` ①-b を参照してください。
