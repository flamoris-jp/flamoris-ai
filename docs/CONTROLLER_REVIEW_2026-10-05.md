# Controller実装のレビュー・修正記録

日付：2026-10-05。対象はController #5、Generation #71、Studio #65、AI #25のopen PRです。ユーザーのレビュー・修正ループ指示に基づくソースレビューで、merge／実機受け入れとは区別します。[PROGRESS §4.7](../PROGRESS.md#47-controllerのレビュー修正ループ2026-10-05) が最新の引継ぎです。

## 基準とレビュー範囲

| 対象 | レビュー開始head | 修正後head |
| --- | --- | --- |
| [Controller #5](https://github.com/flamoris-jp/flamoris-generation-controller/pull/5) | `640a5bd48c76e4bf736e9a3589c216123ccd18b3` | `b57140954c8bc8176d882a053df70f00fad2be31` |
| [Generation #71](https://github.com/flamoris-jp/flamoris-generation-mcp/pull/71) | `15a7b203b9149c11967acaf59bcec9273bdd553d` | `a580b15fe9f29ec31d50558744d33858c309181a` |
| [Studio #65](https://github.com/flamoris-jp/flamoris-studio/pull/65) | `5f059c6b1e28f69865fbd488da31a1f1f9ca9147` | `e7c050c9f8067f7f976703ada139cdaf5301b7a8` |
| [AI #25](https://github.com/flamoris-jp/flamoris-ai/pull/25) | `4ec778862289dc0820a09ba08f2c515a770444c1` | 本記録を含む最新PR headを参照 |

coreのconstruction／ownership／shutdown、23 operationのstrict DTO、HTTP認証とbounds／safe errors、MCP facade・signed ingress・binary変換、Studio直接HTTPと既存account／request／publication fenceを確認しました。retained domainは移動前ソースとnamespace差分を照合し、provider／jobs／inputs／assets／retentionの保持形式とunknown／no-replay、文書と依存pinを確認しました。モデル／GPUの実動作、稼働中設定、private運用環境はこのレビューの対象ではありません。

## 発見・再現・修正

| 優先度 | 問題と利用への影響 | 修正と回帰検証 |
| --- | --- | --- |
| P1 | Controllerのcloseが、submit acknowledgement待ちの呼び出しより先にproviderと所有権lockを解放した。別ownerを起動でき、旧呼び出しの保存状態更新が後から走る可能性があった | 新規呼び出しを拒否してadmitted callをdrainし、providerを閉じてからlockを解放。一つのcleanup taskへ全closerをjoinし、close待機側の取消をshield。submit途中のclose／close取消、submit取消後のunknown recoveryを検証 |
| P2 | StudioがI/O待ちごとのtimeoutだけを使い、少しずつ届く応答で全体期限を超えて待つ可能性があった | 接続setupを含むwhole-call期限とexchange期限を設定。遅いheaders／JSON／binary／接続を拒否し、streamを閉じ、再送しない。upload exchangeにも15秒期限を適用。identity encodingを要求し、圧縮応答を引き続き拒否 |
| P2 | 新しい`models.get` DTOが512文字までに制限され、既存カタログが返す有効な長いmodel IDをHTTP 400で拒否した | 元のmodel nameの1024 UTF-8 byte上限とkind prefixに合わせて1040文字まで受付。nested modelのlist→get roundtripを実際のHTTP adapterで検証。domainのpath／UTF-8 byte検証は維持 |

修正前にshutdown 2ケース、whole-call期限4ケース、長いmodel ID取得1ケースの失敗を再現しました。修正後は追加11 testsを含むsuiteが成功。取消後のunknown journalとexchange期限も追加検証しています。既知の3指摘は修正済みで、再レビューで追加の修正必須事項は見つかっていません。

## 検証とソース状態

| 対象 | ローカル検証 | 最終head CI |
| --- | --- | --- |
| Controller | 280 tests、Ruff check/format、sdist/wheel、MCP／Starletteなしのinstalled core import・同時close | [Controller CI](https://github.com/flamoris-jp/flamoris-generation-controller/actions/runs/37287862226)：success。Python 3.11/3.12 |
| Generation MCP | 194 tests、Ruff check/format、sdist/wheel、installed stdio／HTTP smoke | [Python CI](https://github.com/flamoris-jp/flamoris-generation-mcp/actions/runs/37288168566)：success。installed wheel／Dockerも成功 |
| Studio | PostgreSQL 16で372 backend tests、sdist/wheel、変更Pythonのformat | [Python Studio CI](https://github.com/flamoris-jp/flamoris-studio/actions/runs/37288175782)：success。PostgreSQL 17で372 tests、web 101 tests、build、production Docker |
| AI文書 | 変更9文書、相対リンク45件・見出し・diffを検証。current head／固定source／contractと照合 | CI workflowなし |

両callerのcore依存を`b57140954c8bc8176d882a053df70f00fad2be31`へ揃えました。Generation Dockerの依存固定も同じcommitです。依存・active設計リンクは修正後のcoreを指し、初回実装の固定CI記録はPROGRESS §4.6に保持します。

4PRは未マージで、実機も未変更です。source acceptance後のlive切替は、一つのowner、旧versionのdrain/reconcile、保存状態のbackup/rollback、明示されたendpoint／private token／空namespace設定を別scopeで確認します。新しい参照画像／custom機能、DB migration、provider／GPU lifecycle、data／grant／credential変更は追加していません。
