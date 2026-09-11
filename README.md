# FANY Ticket Watch（めぞん版）

`warakurasan` と同じ仕組みの、友人用の別インスタンスです。
FANYチケットで「めぞん」の新しい公演が追加されたらLINEに通知します。
（Googleカレンダー連携は今回は未設定。あとから追加可能です）

## 仕組み

1. GitHub Actionsが30分おき（cronは変更可）に起動
2. `config/comedians.json` に書いた名前（今回は「めぞん」）で
   `https://ticket.fany.lol/search/event?keywords=...` を検索
3. 前回チェック時の一覧（`data/fany-state.json`、リポジトリにコミットして永続化）と比較
4. 新規公演があればLINEに通知
5. 更新した状態ファイルをリポジトリに自動コミット

## セットアップ

### 1. リポジトリを作成してpush

このフォルダの中身を、新しいGitHubリポジトリ（例: `fany-watch-mezon`）に
そのままpushしてください。

### 2. LINEグループIDを調べる

`../line-group-id-helper` の手順で、友人のLINEグループのグループIDを取得してください。

### 3. GitHub Secretsを設定

リポジトリの Settings → Secrets and variables → Actions で以下を登録:

| Secret名 | 内容 |
|---|---|
| `LINE_CHANNEL_ACCESS_TOKEN` | 使うLINE公式アカウントのチャネルアクセストークン（既存のものを流用可） |
| `LINE_TO_ID` | 手順2で取得した友人グループのグループID |

`GOOGLE_SERVICE_ACCOUNT_JSON` / `GOOGLE_CALENDAR_ID` は今回は設定しません
（未設定だと自動でスキップされるので、後で使いたくなったら追加するだけでOKです）。

### 4. 動作確認

Actionsタブから `FANY Ticket Watch（めぞん）` を選び「Run workflow」で手動実行できます。

初回実行時は既存の公演が全部「新規」扱いになって大量通知が飛びます。
一度 `LINE_TO_ID` を設定しない状態（もしくはSecrets未設定のまま）で1回実行して
`data/fany-state.json` を埋めてから、Secretsを設定して通知を有効化するのがおすすめです。

## 監視対象を増やしたいとき

`config/comedians.json` に名前を追加するだけです:

```json
[
  "めぞん",
  "別の芸人名"
]
```

## 注意点

FANYチケットのページ構造が変わるとスクレイピング部分
（`scripts/check-fany.js` の `fetchEventsFor` / `parseHeading`）の調整が必要になることがあります。
これは本家 `warakurasan` と共通のコードなので、片方を直したらもう片方も同じように直してください。
