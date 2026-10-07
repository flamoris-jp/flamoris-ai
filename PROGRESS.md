# FLAMORIS AI アーキテクチャ組み換え：進捗と引継ぎ

更新日：2026-10-07（JST）

対象：外部MCP・内部Intelligence・任意Agent・Generation・Runtimeの責務と接続の整理

決定と受け入れ記録：[AI #18](https://github.com/flamoris-jp/flamoris-ai/issues/18)

このファイルは、ChatGPTの別スレッドや次の作業担当が、目的・現在地・残りの作業をまとめて確認する入口です。設計の基準は [ARCHITECTURE.md](docs/ARCHITECTURE.md)、担当Issueと従来の順序は [ROADMAP.md](docs/ROADMAP.md)。本書はその進捗と次の具体的な工程を記録します。

**現在地：cleanupとController・対応caller、bounded ComfyWorkFlow／参照画像と会話内LLM切替のソースはmain受け入れ済み（4.8／4.9）。2026-10-07 22:15 JSTまでにDB migration、人格import・承認済みlocal権限、GPU Node Managerと5アプリの切替、Studio gatewayのTLS／認証API経路を確認しました（4.12）。基本Imageとbounded schema7参照画像の実推論・PNG取得も確認しました（4.13／4.14）。GPU handoff／VRAM観測とAgent directモデル解決も確認しました（4.15）。外部Intelligence alias補正、LLM推論・会話継続、異なるモデルへの切替、ログイン後UIは残っています。**

以下の新しいGeneration経路はmain受け入れ済みのソースです。稼働済み設定や実機切替の保証ではありません。限定checkpoint profileの参照画像ソースは4.9で受け入れ済みです。実機導入は2026-10-06の明示されたリリース作業として開始しました（4.11）。任意custom node・追加model familyなどの拡張や未承認のremote／有料APIは別scopeです。2026-10-05は実機作業を翌日に延期し、文書整合を行いました（4.10）。

2026-10-05の追加全体レビューで、旧機構の手順書13本、未使用request field、Studio Imageの廃止済み参照／固定サイズ分岐と認可再確認の不足を発見・修正しました。**追加修正7PRと組織READMEはmain反映済み。全体の受け入れ記録は [AI #23](https://github.com/flamoris-jp/flamoris-ai/pull/23) です。** [リポジトリ別の全体レビューと残作業](docs/REVIEW_2026-10-05.md) に変更・検証・機能の可否を記録しています。最初のcleanupと追加レビューの受け入れ基準は4.2／4.4で個別に固定しています。

## 1. 作業の目的

制作側が必要な知能・生成機能を使えるように、各リポジトリの責務と接続を整理します。MV制作などの利用に戻れる基盤を作り、単純な推論や生成のために不要な人格・中継・実行エンジンを必須にしないことが目的です。

| 整理する点 | 到達したい状態 |
| --- | --- |
| 外部と内部の通信 | MCPはChatGPTなど外部クライアントの入口。内部サービス・アプリは小さな非MCP契約を使う |
| Hubの役割 | 外部MCPのcatalog・connection・routingを担当。内部サービスバスや実行エンジンにしない |
| Agentの役割 | 人格・会話・memory・principal/session・context policyが必要な場合だけ使う。通常推論・生成はAgentなしで使える |
| 推論の実行 | 共通のprovider adapterを再利用し、StudioやAgentに同じ通信実装を複製しない。新しい中央gatewayサービスも作らない |
| Runtimeの役割 | 推論、ExecuteFlow、コンパイル済みExecutionPlan、Job/Continuation/resource制御を維持する |
| Generationの整理 | 保持された基本/native生成・generic recipe・Job/Input/Assetの責務を明確にする。対応PRの非MCP契約・main受け入れ・実機設定を区別する |
| 状態の所有者 | 会話・生成Job/Input/Asset・GPU lifecycleごとに一つのauthorityを保つ。複数frontendが別々の予約や状態を作らない |
| 既存の保護 | ユーザー分離、送信同意、上限、provenance、不確定処理の予約、保存済みデータを保持する |

この作業は、native kernelやMaidionis・Arbitrium・Oblivionisのモデル研究、人格DBの再設計、新しい生成providerの追加をまとめて行う計画ではありません。それぞれの独立した課題は維持します。

## 2. 固定した名称と責務

| 名称 | 意味 | 所有者 |
| --- | --- | --- |
| `ExecuteFlow` | 推論の依存・データ・制御フロー | AI Runtime |
| `ExecutionPlan` | 既存のコンパイル済みRuntime表現。ExecuteFlowへ改名しない | AI Runtime |
| `ComfyWorkFlow` | ComfyUI実行グラフ・API-format JSON | generation領域。実際のグラフ実行はComfyUI |

新しい設計文では単独のWorkflowという語を避けます。一方、既存の `WorkflowStore`、`workflows.*`、`workflow_id`、型・wire field・設定・ファイル名や歴史的引用は、実装と一致する綴りを保ちます。文書だけでAPIや設定を改名したことにしません。ComfyUI以外のrequest/recipeを一律にComfyWorkFlowとは呼びません。

GPU Node Managerがhost-wideなruntime起動停止・GPU handoff・transition lockを担当します。Studio、Agent、Runtime、Controllerに別のhost state machineを作りません。lifecycle READY、モデルの利用可能性、特定グラフのqualification、呼出し元の認可は別の判断です。

## 3. 現在のソース接続と将来の目標

以下はソースが提供する経路です。稼働中の設定・接続を実測した表ではありません。

| 利用経路 | 現在のソース | 将来・保持する境界 |
| --- | --- | --- |
| Studioの素のIntelligence | Studio `IntelligenceGateway` → shared `flamoris_intelligence` → 設定されたlocal llama.cpp/vendor HTTP API | AgentやIntelligence MCP listenerは不要。provider詳細は共通adapterに閉じる |
| Studio Agent Support | Studio `AgentGateway` → Agent JSON HTTP `/api/v1` → Agentのapproved execution client → shared adapter | Agentがprincipal・人格・会話・モデル/送信同意を所有する |
| 外部Intelligence MCP | 外部クライアント → Hub等の外部入口 → Intelligence MCP facade → shared adapter | 外部のtool/schema/transport・結果変換を維持する |
| 外部Agent MCP | Agentの既存inbound MCP surfaceを保持 | 内部HTTPへの置換とは別の互換面。実際の外部登録・稼働状態は別途確認する |
| Studio generation | main受け入れ済みのStudio → 認証付きController HTTP v1 → 一つのController → provider | Studioがuser ownership・request/publication fenceを保持。main反映済み。2026-10-07にgateway経路確認、実生成は未確認（4.12） |
| 外部Generation MCP | main受け入れ済みの外部client → Hub → Generation MCP facade → 同じController → provider | 保持23tool＋comfy.register/getの25tool。signed ingress／SDK変換を保持し、Studioとは同じruntime／予約を利用（4.9） |
| Runtime | native推論・ExecuteFlow・ExecutionPlan・Job/Continuationを保持 | 必要なら将来、生成能力への任意の内部連携を別scopeで設計する |
| GPU lifecycle | 既存RuntimeManagerと共通lock。CLI/HTTP/MCPは同じmanagerを利用する | 既存authorityを維持する |

shared libraryのdistribution名は `flamoris-intelligence-mcp`、内部import namespaceは `flamoris_intelligence` です。base wheelと任意のMCP extraを分離しました。名前にMCPを含むdistributionを使うことと、内部通信をMCPにすることは別です。StudioのAgent Python package依存は接続テスト用extraだけで、production StudioはHTTP契約を使います。

## 4. 完了した作業と根拠

### 4.1 設計・用語・Issue整理

2026-10-04の設計決定を [AI #18](https://github.com/flamoris-jp/flamoris-ai/issues/18) に集約しました。旧内部MCP図とmandatory bridgeの指示を置き換え、Controllerを今は実装しないこと、旧custom subsystemを移植せず削除することを固定しました。

文書PRは [AI #19](https://github.com/flamoris-jp/flamoris-ai/pull/19)、[Controller #2](https://github.com/flamoris-jp/flamoris-generation-controller/pull/2)、[Intelligence #11](https://github.com/flamoris-jp/flamoris-intelligence-mcp/pull/11)、[Agent #39](https://github.com/flamoris-jp/flamoris-ai-agent/pull/39)、[Runtime #24](https://github.com/flamoris-jp/flamoris-ai-runtime/pull/24)、[Generation #68](https://github.com/flamoris-jp/flamoris-generation-mcp/pull/68)。過去の設計と証拠は [固定した履歴](docs/LEGACY_DESIGN.md) に残しています。

### 4.2 内部接続・削除・main反映

以下は2026-10-05に再確認した受け入れ基準です。実装6PRはすべてmerged、最終PR CIはsuccessです。コミットは完了時点の固定基準で、将来の最新mainを表すものではありません。

| リポジトリ | 完了内容 | PR / merge commit | 検証の記録 |
| --- | --- | --- | --- |
| Intelligence | neutral provider libraryを分離。外部6toolを保持し、MCPを任意extraへ分離 | [#12](https://github.com/flamoris-jp/flamoris-intelligence-mcp/pull/12) / [5da9736](https://github.com/flamoris-jp/flamoris-intelligence-mcp/commit/5da9736e60c29ed549d16656ff7acac5d54bdb83) | ローカル103件。Python 3.11/3.12、base/extras・installed packaging・Docker CI成功 |
| Agent | outbound Intelligence MCP clientを削除。approved直接推論と内部JSON HTTP APIを追加。外部inbound MCPを保持 | [#40](https://github.com/flamoris-jp/flamoris-ai-agent/pull/40) / [c6adca2](https://github.com/flamoris-jp/flamoris-ai-agent/commit/c6adca2727805a62348e2aade219ea20a8e2ee99) | PostgreSQLを含むCI 202件。lint/format・installed wheel・Docker成功 |
| Studio | raw直接adapter・Agent HTTP接続。custom/v3 Image discovery/dispatchを廃止 | [#63](https://github.com/flamoris-jp/flamoris-studio/pull/63) / [abba596](https://github.com/flamoris-jp/flamoris-studio/commit/abba5963a5e680f74b7fda717de5c7dc96a7d571) | PostgreSQLを含むCI 320件、画面100件、TypeScript/Vite build・production Docker成功 |
| Generation MCP | custom登録/版管理・v3合成/qualification・Runtime evidence/delegation/cleanupを削除。基本/native生成と状態保護を保持 | [#69](https://github.com/flamoris-jp/flamoris-generation-mcp/pull/69) / [c3f8eee](https://github.com/flamoris-jp/flamoris-generation-mcp/commit/c3f8eee64283aab3e148c20ec737bc8002edb66a) | ローカル444件。lint/format・wheel/sdist・installed wheel・Docker CI成功 |
| AI Runtime | Generation専用media loweringと依存テスト/文書だけを削除。native ExecuteFlow/ExecutionPlanを維持 | [#25](https://github.com/flamoris-jp/flamoris-ai-runtime/pull/25) / [f5a3de5](https://github.com/flamoris-jp/flamoris-ai-runtime/commit/f5a3de5a1f53622f0fc8ca9124901ee7d1345e7d) | ローカル27 CTest。Developer/native・警告をerror扱いするbuildとC++ CI成功 |
| MCP Hub | v3例と7custom toolをcatalogから削除。保持23toolと整合。外部routingを維持 | [#37](https://github.com/flamoris-jp/flamoris-mcp-hub/pull/37) / [e529145](https://github.com/flamoris-jp/flamoris-mcp-hub/commit/e529145fe3ee3b35281715b6e74d9b329630dc7c) | ローカル157件。lint/format・wheel/sdistとCI成功 |
| 全体文書 | 実装済み内部接続・Generation互換経路・将来Controllerを区別し、親/子Issueへ証拠を記録 | [AI #20](https://github.com/flamoris-jp/flamoris-ai/pull/20) / [b105fb1](https://github.com/flamoris-jp/flamoris-ai/commit/b105fb1164b7bcbe0797c4f3aa707f70e6be9ca7) | 変更7文書のレビュー、diff/local link検証。このrepoにはCI workflowなし |

GPU Node Managerは [#11](https://github.com/flamoris-jp/flamoris-gpu-node-manager/issues/11) で境界監査だけを行いました。基準は [e2de4bd](https://github.com/flamoris-jp/flamoris-gpu-node-manager/commit/e2de4bd4e24502dede6f0446c99de309f9ca9afa)。共通RuntimeManager/host lockと任意evidence安全portを保持し、ファイル・profile・実機・evidenceは変更していません。この監査で新しいテストやlive runtime確認を行ったとはしていません。

### 4.3 接続と保存データについて確認した範囲

Studioの5件の接続テストは、実際のGateway → Agent HTTP app → Agent session → shared provider adapterを通します。principal/conversation storeとprovider HTTPはsynthetic fixtureです。二principal、messageのtrusted/untrusted role、request/session/conversation identity、public provenance、送信前の拒否、権限失効、異常/partial結果と自動再送しない動作を確認しました。durable DBの保証は各repoのPostgreSQL suiteで別に検証しました。

保持したgenerationは、元のschema-1 Image/LoRAと、schema-4 Speech・schema-5 Music・schema-6 transcriptionのnative recipeです。設定やCIの成功は、実際のモデル/GPUでの品質・生成成功を証明しません。

旧custom recipeの実行は明示的に拒否します。旧custom/delegatedのactive debtは元の `active.json` とunknown/busy予約を保持し、新packageからpoll・resubmit・cancel・clearしません。固定エラーは `retired_feature_requires_reconciliation`。保存済み定義・input・asset・会話・grant・evidenceをソース削除で消していません。

Studioの元のbuiltin Imageは参照画像を受け付けません。古いcustom/v3選択は、利用者が対応builtinを明示的に選び直すまで実行不可です。既存input/upload/assetと未確定executionの削除防止は残っています。これはcleanup受け入れ時の記録です。後続の外部ChatGPT向け参照画像実装は4.9を参照し、Studio Image UIの追加とは区別します。

詳細なinventoryと互換条件は [Generation retirement](https://github.com/flamoris-jp/flamoris-generation-mcp/blob/main/docs/LEGACY_RETIREMENT.md)、[Agent internal execution](https://github.com/flamoris-jp/flamoris-ai-agent/blob/main/docs/INTERNAL_EXECUTION.md)、[Studio cleanup](https://github.com/flamoris-jp/flamoris-studio/blob/main/docs/ARCHITECTURE_CLEANUP.md) にあります。

### 4.4 追加の全体レビュー（2026-10-05、main反映）

| リポジトリ | 追加修正 | 受け入れmerge commit |
| --- | --- | --- |
| Generation | [#70](https://github.com/flamoris-jp/flamoris-generation-mcp/pull/70)：旧runbook 9本とdefinition/evidence request seamを削除。native／data／unknown保護を保持 | [02ce5e2](https://github.com/flamoris-jp/flamoris-generation-mcp/commit/02ce5e23f2c6cd311181169e36e58a0eb6eb6ff4) |
| Studio | [#64](https://github.com/flamoris-jp/flamoris-studio/pull/64)：旧runbook 4本と参照／固定サイズ実行分岐を削除。Imageの認可・設定再確認と異常応答のunknown保存を追加 | [deb0dd9](https://github.com/flamoris-jp/flamoris-studio/commit/deb0dd9f796e3f47b3eb35ed631ea9cacb18ebdd) |
| Agent | [#41](https://github.com/flamoris-jp/flamoris-ai-agent/pull/41)：現行HTTP／shared direct executionへphase・運用文書を一致 | [4b8245a](https://github.com/flamoris-jp/flamoris-ai-agent/commit/4b8245a32d093d91ac737cb70cf5396050bd3bca) |
| Controller | [#3](https://github.com/flamoris-jp/flamoris-generation-controller/pull/3)：削除済みscopeと将来の最小contractを区別 | [c29d5cf](https://github.com/flamoris-jp/flamoris-generation-controller/commit/c29d5cff565669c6e0c771e5923f211cdbd0b26f) |
| Runtime | [#26](https://github.com/flamoris-jp/flamoris-ai-runtime/pull/26)：実装済みPhase Cと将来external facadeを区別 | [8227502](https://github.com/flamoris-jp/flamoris-ai-runtime/commit/8227502c77ac6cf13b1a412c984f429787b77963) |
| Hub | [#38](https://github.com/flamoris-jp/flamoris-mcp-hub/pull/38)：Studio GenerationだけのMCP互換例外を明記 | [5b54c77](https://github.com/flamoris-jp/flamoris-mcp-hub/commit/5b54c77d3df97613723b53eb5ce1931c93833268) |
| GPU Node Manager | [#12](https://github.com/flamoris-jp/flamoris-gpu-node-manager/pull/12)：独立host portと廃止済みGeneration consumerを区別 | [2f55fb7](https://github.com/flamoris-jp/flamoris-gpu-node-manager/commit/2f55fb74b986b7fa9c0eceb00cab2bd94187fdca) |
| Intelligence | 現行source／103 testsを再確認。追加ソース変更なし | 追加PRなし。4.2の基準を維持 |
| AI | [#23](https://github.com/flamoris-jp/flamoris-ai/pull/23)：レビュー記録に加え、README／AGENTS／現行設計から完了済み削除手順・旧機構の説明を整理。ROADMAPを残作業中心へ更新 | 本記録を含むPRのmerge commitを参照 |
| 組織 `.github` | [#28](https://github.com/flamoris-jp/.github/pull/28)：組織READMEの日英責務マップを現行接続へ一致。曖昧なWorkflow説明・旧階層図を整理し、Controller未実装とGeneration互換経路を明記 | [94d40e8](https://github.com/flamoris-jp/.github/commit/94d40e8a4f5f5628591c1fddef9f7ae0303bb438) |

2026-10-05の「全部マージ」指示に基づき、レビュー済みheadを指定してowner7PRと組織READMEをsquash mergeしました。各merge treeが検証済みheadと一致することも確認。AI #23に全体の受け入れ記録を集約しています。マージ前の6 owner workflowは正確なPR headで全て成功。ローカル検証との差は[レビュー記録](docs/REVIEW_2026-10-05.md#ci受け入れ記録)を参照。Controller／実機操作／新しいreference featureは再開していません。

組織READMEの追補確認とAI文書整理では、完了したソース監査を繰り返さず、現行案内の記述だけを更新しました。開発ステータスの生成領域は変更せず、既存sync helperのoffline idempotence・日英リンク・相対／ownerリンク・見出し・diffを確認。受け入れ履歴、実機の互換／回復条件と保存データの保護は保持しています。

### 4.5 Controller実装準備の再開（2026-10-05）

この節は準備scopeの受け入れ履歴です。続くコード実装と現在の状態は4.6を参照します。

ユーザーはControllerを共通の生成制御層として進める方針を選び、まずControllerのREADME・AGENTSとAI全体の実装方針の更新を指示しました。Fの一律保留を今回の文書整備へ変更します。Controllerコード、Generation/Studioソースの切り出しと実機cutoverはまだ行っていません。

今回の文書PRで、[Controller実装方針](https://github.com/flamoris-jp/flamoris-generation-controller/blob/ba9f3856517b56dad509f757b73cdcdc00ed5b6e/docs/IMPLEMENTATION.md)にretained sourceのclass/module inventory、MCP/Studioに残す責務、単一state owner、未決定のDTO/auth/hosting、段階的な受け入れを記録します。Generation `02ce5e23f2c6cd311181169e36e58a0eb6eb6ff4`、Studio `deb0dd9f796e3f47b3eb35ed631ea9cacb18ebdd`を確認した準備基準で、将来のcode taskでは最新ソースを再照合します。

[Controller #4](https://github.com/flamoris-jp/flamoris-generation-controller/pull/4) と [AI #24](https://github.com/flamoris-jp/flamoris-ai/pull/24) が今回の更新です。両PRはmergedです。Controller main `4b6babf08cf7613fc15d71b541a57296790fa4a0`、AI main `bebbe3d753b4b3f3544b1393def7947c8c8c9932`で準備を受け入れました。以前のcleanup受け入れは4.2／4.4のままです。具体的なコード実装はFの最小契約・状態所有者の決定に続くscopeとし、live作業や新しい参照画像を自動追加しません。

検証：変更したMarkdown 13文書、相対リンク50件・見出しanchor 7件、inventoryのソースpath 29件とdiff whitespaceを確認しました。runtime/live providerテストはこの文書更新では実行していません。

### 4.6 Controller実装と対応するcallerソース（2026-10-05）

この節は初回実装・検証の固定記録です。続くレビュー修正後の最新headと検証は4.7を参照します。

続く「controllerの実装お願い」というユーザー指示で、文書準備からコード実装と対応caller接続へ進みました。抽出前にController #1へpackage／最小契約／service permission／同居hosting／一つのreservation ownerを記録し、最新mainのGeneration・Studioソースと保存形式を照合しました。

| リポジトリ | 実装した内容 | open PR / 固定head |
| --- | --- | --- |
| Controller | retained domainを一度だけ抽出。MCP-free package、shared runtime／close、output-root lifetime lock、strict authenticated HTTP API | [#5](https://github.com/flamoris-jp/flamoris-generation-controller/pull/5) / `640a5bd48c76e4bf736e9a3589c216123ccd18b3` |
| Generation MCP | 23tool／signed ingress／SDK変換を保持し、同じControllerへdelegate。内部HTTPを同居。秘密・transport設定をcoreから分離、固定依存とoffline Docker wheel install | [#71](https://github.com/flamoris-jp/flamoris-generation-mcp/pull/71) / `15a7b203b9149c11967acaf59bcec9273bdd553d` |
| Studio | MCP client／Hub namespaceを削除し、直接JSON／binary HTTPへ変更。既存認可・history・opaque mapping・unknown fenceを保持。DB migrationなし | [#65](https://github.com/flamoris-jp/flamoris-studio/pull/65) / `5f059c6b1e28f69865fbd488da31a1f1f9ca9147` |
| AI | 本書とREADME／AGENTS／architecture／roadmap／ecosystem／Studio案内を実装PRと整合 | 本記録を含む文書PR。旧cleanupの受け入れ記録は保持 |

Generation／Studio test extraはControllerの上記commitを正確にpinします。最初のhostは既存Generation MCP HTTP processで、一つのControllerを外部MCPと内部HTTPから共有します。独立daemon／port／JobStoreを追加しません。`controller-authority/owner.lock`はprovider構築・recoveryの前に取得し、別processも拒否します。dispatchではroot／namespace／lock identityを確認し、shutdownでもjob journalを消しません。違うroot／hostや旧pre-lock binaryの排他は保証しないため、実機では旧ownerをdrain/reconcile・停止してから切り替えます。

内部APIはPOST `/api/v1/generation/{operation}`。32–512文字のprivate service tokenで認証し、trusted Studio backendにbounded operation surfaceを許可します。Studio user identity／provenance／upstream IDをアクセス権に変換しません。JSON／headersからtrusted contextを注入できず、Studioがuser ownershipと送信前／公開前の再確認を維持します。設定のない内部APIはunavailable、外部MCPは利用可能です。外部Hub credential／signed provenance secretとは別の設定です。

保持するschema 1/4/5/6、recipe/job/input/asset ID、provider/storage key、active.json／archive／lease／copy ledgerの形式を変更せず、旧custom/v3を復活させていません。timeout／未確定acknowledgement／journal commit／restartをresubmitの根拠にせず、ControllerとStudioがそれぞれのunknown reservationを保持します。

ローカル検証：Controller 276件、MCP facade／ingress 194件、Studio PostgreSQL backend 362件に加えてmalformed response 3件、HTTP／実adapter接続32件、画面101件とTypeScript/Vite buildが成功。元のGeneration domain 444件を移動先込みで維持し、新しい単一所有者／認証／共有admissionを追加しました。core／facadeのlint/format・sdist/wheel build、MCP/Starletteなしのinstalled coreとinstalled MCP stdio／HTTP smokeが成功しています。実GPU／provider／本番DB／有料APIは使っていません。

CI：上記の正確な最終headですべて成功。[Controller CI](https://github.com/flamoris-jp/flamoris-generation-controller/actions/runs/37264463332) はPython 3.11/3.12とMCP-free installed wheel、[Generation CI](https://github.com/flamoris-jp/flamoris-generation-mcp/actions/runs/37265896760) は194 tests・lint/format・installed stdio/HTTP・Docker、[Studio CI](https://github.com/flamoris-jp/flamoris-studio/actions/runs/37265651569) はPostgreSQL 17の365 tests・画面101 tests・build・production Dockerを確認しました。Generationの初回CIはdomain移動後のDocker smoke import残存とprivate lock directoryのtest cleanupで失敗し、follow-up headで修正済みです。AI文書にはCI workflowがなく、変更8文書の差分・相対リンク42件と見出し・実装との整合を確認しました。ソースのmergeと実機受け入れは未完了です。

### 4.7 Controllerのレビュー・修正ループ（2026-10-05）

この節は初回レビュー修正の固定記録です。後続の再レビュー・main受け入れは4.8を参照します。

ユーザーのレビュー・修正ループ指示に基づき、4つのopen PRの正確なhead、core／HTTP／MCP／Studioの責務・lifecycle・認証・保存形式・不確定処理と文書を再確認しました。[詳細レビュー記録](docs/CONTROLLER_REVIEW_2026-10-05.md) に再現・修正・検証を記録しています。

3件を修正しました。終了時に実行中の呼び出しより先に所有権lockを解放する問題は、admitted callのdrainと一つのshield付きcleanup taskで解消。Studioのslow responseは、接続／headers／全streamの絶対期限とupload exchangeの期限で制限。モデル一覧に出る長いIDの取得拒否は、保持された1024 UTF-8 byteのmodel nameとkind prefixを受け付けるDTOで解消しました。新しい型名や保存形式への移行ではありません。

| PR | レビュー修正後の固定head | 最終CI |
| --- | --- | --- |
| [Controller #5](https://github.com/flamoris-jp/flamoris-generation-controller/pull/5) | `b57140954c8bc8176d882a053df70f00fad2be31` | [Controller CI](https://github.com/flamoris-jp/flamoris-generation-controller/actions/runs/37287862226)：success |
| [Generation #71](https://github.com/flamoris-jp/flamoris-generation-mcp/pull/71) | `a580b15fe9f29ec31d50558744d33858c309181a` | [Python CI](https://github.com/flamoris-jp/flamoris-generation-mcp/actions/runs/37288168566)：success |
| [Studio #65](https://github.com/flamoris-jp/flamoris-studio/pull/65) | `e7c050c9f8067f7f976703ada139cdaf5301b7a8` | [Python Studio CI](https://github.com/flamoris-jp/flamoris-studio/actions/runs/37288175782)：success |
| [AI #25](https://github.com/flamoris-jp/flamoris-ai/pull/25) | 本記録とレビュー文書を含むPR | CI workflowなし。文書・リンク・source整合を検証 |

ローカルでController 280件、Generation 194件、Studio PostgreSQL 16で372件が成功。11件の境界／回帰テストを追加し、lint/format、3 packageのsdist/wheel、MCP/Starletteなしのinstalled core lifecycleとinstalled MCP stdio／HTTP smokeも成功しました。両dependentのcore pinは `b57140954c8bc8176d882a053df70f00fad2be31` へ揃えています。最新headのCIは全てsuccessで、StudioのPostgreSQL 17・画面101件・build・production Docker、Generationのinstalled wheelとDocker、ControllerのPython 3.11/3.12も確認しました。レビュー修正は完了し、merge／実機受け入れは未実施のまま区別します。

AI側は変更9文書、相対リンク45件と見出し、固定source／CI記録、文書と実装の整合、diff whitespaceを確認しました。初回の受け入れ履歴と今回の修正結果を分け、親／担当Issueと4PRへ最新head・検証を記録します。

### 4.8 Controllerの再レビューとソース受け入れ（2026-10-05）

ユーザーの「レビュー・修正ループと、問題なくなったらマージ」指示に基づき、4.7の修正後head、CI全job／step、競合とレビュー指摘を再確認しました。core lifecycle／strict HTTP／MCP同居runtime／Studio期限・認証・共有状態を再レビューし、追加の修正必須事項は見つかっていません。重点テストを再実行し、Controller 26件、GenerationのMCP／HTTP 27件、Studioの接続／実契約39件、合計92件が成功しました。全suiteとpackaging／Docker／画面の最終CIは4.7の正確なheadでsuccessのままです。

Controller → Generation MCP → Studioの順にexpected head SHAを指定してsquash mergeしました。全merge treeが検証済みheadのtreeと一致することをGitHubで確認しています。

| 対象 | accepted merge commit | 検証済みtree |
| --- | --- | --- |
| [Controller #5](https://github.com/flamoris-jp/flamoris-generation-controller/pull/5) | [bbabc38](https://github.com/flamoris-jp/flamoris-generation-controller/commit/bbabc38240e3af2bf4b2093951da90273b5659d8) | `ca27eaab5f84207d753e811a2f7932b65521f553` |
| [Generation #71](https://github.com/flamoris-jp/flamoris-generation-mcp/pull/71) | [e592457](https://github.com/flamoris-jp/flamoris-generation-mcp/commit/e59245723ca0208b69cc8d3de4461a1eace596c9) | `99b0aea20abe630b4dec6a76ab6b30141de3876a` |
| [Studio #65](https://github.com/flamoris-jp/flamoris-studio/pull/65) | [cf11ef4](https://github.com/flamoris-jp/flamoris-studio/commit/cf11ef421bef4c944bb2aca51378ec21200d8a9a) | `f0b2f255ef33a4663a1d72608e28974eb30a6186` |
| 全体文書 | 本記録・active設計・レビュー履歴を含む[AI #25](https://github.com/flamoris-jp/flamoris-ai/pull/25)のmerge commitを参照 | CI workflowなし。変更9文書・相対リンク46件／見出し・source整合とdiffを検証 |

両dependentとDockerのcore pinは、CIで検証した`b57140954c8bc8176d882a053df70f00fad2be31`を維持します。マージ後も同じsource artifactを導入でき、履歴上のsquash commitへ依存を再指定する必要はありません。Controller #1のソースscopeを受け入れ、運用工程は本書Eと親AI #18へ引き継ぎます。既存Issue履歴・旧cleanup証拠は保持します。

F/Gのソース作業は完了です。この節の受け入れ時点では実機作業は未着手でした。後続の明示リリース指示に基づく固定成果物切替・gateway経路確認と、残るprovider／UI等の受け入れは4.12を参照します。任意custom構成の拡張は別scopeです。

### 4.9 実機投入前の2機能（2026-10-05、main反映）

ユーザーの「PROGRESS.mdを前提に、ChatGPTによるComfyWorkFlow登録と参照画像生成、LLM切替時の会話引継ぎ」という依頼に基づくソース実装です。当初は実装・テスト・レビュー可能なDraft PRまで進め、続く「マージもお願い」の明示指示でmainへ反映しました。最新headとbase、全CI成功、競合なし・レビュー指摘なしを再確認し、検証済みheadを指定してsquash mergeしました。実機deploy／restart、migration適用、grant／credential変更やlive推論は行っていません。既存main受け入れ記録4.8はそのまま保持します。

| 担当 | merged PR | 検証対象head | 最終CI |
| --- | --- | --- | --- |
| Controller | [#7](https://github.com/flamoris-jp/flamoris-generation-controller/pull/7) | `69b05d0897a94f5d174335f8d849ca9e1f9488ac` | Python 3.11/3.12各291件、lint/format・build・installed MCP-free core成功 |
| Generation MCP | [#72](https://github.com/flamoris-jp/flamoris-generation-mcp/pull/72) | `bdc043f75db4455b59c3c08f94fe71b7ae4bd967` | 195件、lint/format・wheel/sdist成功 |
| Hub | [#39](https://github.com/flamoris-jp/flamoris-mcp-hub/pull/39) | `6d23bd1fb695349e21f27963963485bb949b68c7` | 159件、lint/format成功。25 schema/annotationの実MCP export一致を確認 |
| Agent | [#43](https://github.com/flamoris-jp/flamoris-ai-agent/pull/43) | `be43e282843c787544a481fccf60c548951e2b23` | PostgreSQLを含む206件、lint/format・wheel/sdist・installed wheel・container成功 |
| Studio | [#66](https://github.com/flamoris-jp/flamoris-studio/pull/66) | `87bb2ce2396ecb9347858432298be0e29b987c26` | PostgreSQL backend 376件、web 103件、TypeScript/Vite・container成功 |

最終headの成功run：[generation-controller](https://github.com/flamoris-jp/flamoris-generation-controller/actions/runs/37305721265)、[generation-mcp](https://github.com/flamoris-jp/flamoris-generation-mcp/actions/runs/37305804278)、[ai-agent](https://github.com/flamoris-jp/flamoris-ai-agent/actions/runs/37304935183)、[studio](https://github.com/flamoris-jp/flamoris-studio/actions/runs/37304326508)、[mcp-hub](https://github.com/flamoris-jp/flamoris-mcp-hub/actions/runs/37305563302)。ログから上記件数を確認しました。これらはsynthetic image／fake providerとPostgreSQL／UIのソース検証で、実GPU・live APIの受け入れではありません。

| 担当 | main受け入れcommit | 完全なmerge SHA |
| --- | --- | --- |
| generation-controller | [`8997cef`](https://github.com/flamoris-jp/flamoris-generation-controller/commit/8997cef0b575fec3657337dc97a9f90094279b13) | `8997cef0b575fec3657337dc97a9f90094279b13` |
| generation-mcp | [`962090a`](https://github.com/flamoris-jp/flamoris-generation-mcp/commit/962090ab73ccbe44d29744f279e0eb7418f0a918) | `962090ab73ccbe44d29744f279e0eb7418f0a918` |
| mcp-hub | [`a2aa33a`](https://github.com/flamoris-jp/flamoris-mcp-hub/commit/a2aa33adeb37f5fb43b1b5bf91ae8843f3080731) | `a2aa33adeb37f5fb43b1b5bf91ae8843f3080731` |
| ai-agent | [`78e5b92`](https://github.com/flamoris-jp/flamoris-ai-agent/commit/78e5b92fe474f7b05d12c6cbbc5569dfbaa22c1e) | `78e5b92fe474f7b05d12c6cbbc5569dfbaa22c1e` |
| studio | [`df8a2f0`](https://github.com/flamoris-jp/flamoris-studio/commit/df8a2f045554d05b62a176857aec0fc833c6bfaf) | `df8a2f045554d05b62a176857aec0fc833c6bfaf` |

各merge treeと上記CI検証済みheadのtreeが一致し、main refもmerge SHAと一致することを確認しました。Generation package／Dockerは検証済みController artifact `69b05d0897a94f5d174335f8d849ca9e1f9488ac`のpinを維持します。受け入れ記録を含む文書は[AI #26](https://github.com/flamoris-jp/flamoris-ai/pull/26)のmerge commitを参照します。このrepoにはCI workflowがなく、変更6文書の相対リンク・見出し・文書整合を確認しました。

**ComfyWorkFlowと参照画像：** 外部Generationに`comfy.register/get`を追加し、Hubの公開catalogを保持23tool＋2toolの25に整合しました。登録対象は標準checkpointを用いるbounded txt2img／img2imgのAPI graphです。参照画像はinit-imageとして受け取り、IPAdapter／ControlNet／任意custom nodeへの対応は含みません。新schema 7の不変・digest IDの定義を専用`comfy-definitions-v1`に保存し、`workflows.list`のdescriptorから選んで既存build/save/submitを利用します。旧schema 2/3、custom版管理、v3、qualification/Runtime bridgeは復活させません。

既存managed input契約でPNG/JPEG/WebP（8 MiB上限、pixel上限付き）を受け、leaseとComfyUI input-copy ledgerを共通Controllerが管理します。実機ではComfyUIと一致する`COMFYUI_INPUT_ROOT`が必要です。未確定submitはcopyと予約を保持し、自動replayしません。descriptorの`validated`と`live_provider_verified:false`は静的確認で、実GPU上の生成成功とは区別します。Studio Imageは既存builtinのままです。Hubはgraphを解釈せず、25toolのschema/annotationを実際のMCP exportと一致させ、古い23tool upstreamとの混在をforward前に拒否します。

**LLM切替と会話：** Studio Agent assistantの同じlogical session key、既存の応答、未送信質問を維持します。Agent内部`POST /api/v1/sessions/continue`で同じ認証principalの不変な子sessionを作り、元sessionを同一transactionでrevokeします。元の人格snapshotと発言provenanceを保持し、新しい発言には切替先modelを記録します。履歴は最大12message／64 KiBのrolling windowで、保存済み会話を削除しません。既存15分認可期限・session容量と最大31handoffは維持します。raw Intelligenceはone-shot、外部Agent MCP契約は変更しません。

切替先の利用許可、current membershipとremoteへの全context送信同意を再確認します。未確定／進行中turnは引き継ぎません。Studioはmetadata-onlyのdurable switch fenceをdispatch前に保存し、不確定時は新質問を止め、利用者の「切替確認」で同じrequest IDだけを再確認します。availability確認失敗後も新conversationを開始しないよう修正しました。definite reject後の再確認も再dispatch前にuncertainへ戻し、publicationではrow lock後にbindingを再照合します。Agent shutdownはhandoff worker失敗でもdrainを完了します。

Agent migration `005`とStudio Alembic revision `20261005_12`はソースのみ追加しました。Agentのlineageをretentionで失わないよう子から削除し、Studio downgradeは保存済みfenceを黙って消しません。MCP schema export用の一時CI workflowは最終差分から削除済みです。

ソースの採用は完了です。次はE1の実機棚卸しとmatched cutover計画で、対象が明示された範囲で進めます。Controllerの検証済みartifact → Generationと同じ25tool Hub catalog、Agentの対応migration/source → Studioの対応migration/sourceを組にし、既存data・unknown予約のreconcileとrollback条件を確認します。新機能の実機受け入れ完了とはまだ記録しません。

### 4.10 実機延期と文書整合（2026-10-05）

ユーザーの「実機は明日にしよう。文書整合お願い」に基づき、AI系と組織READMEの10リポジトリ、現行Markdown 142本を照合しました。9リポジトリの37文書を更新し、旧「Controller未実装」「Generation互換MCP経路」「登録は保留」「モデル変更には新conversation」という現在の案内を、4.8／4.9のmain受け入れ済みソースに揃えました。過去のcleanup／抽出／CI記録は当時のscopeとして保持しています。GPU Node Managerの文書は照合のみで変更しません。

- 現行Generationは保持23操作＋`comfy.register/get`の25操作。API一覧、Hub paired rollout、Studio実機受け入れを一致させ、schema 7と`COMFYUI_INPUT_ROOT`、unknown予約／protected copyを保持するrollback条件を明記しました。Studio Imageはbuiltin txt2imgのままです。
- 会話内LLM切替はAgentの不変child-session lineageとStudioのlogical conversation／switch fenceで維持します。未関連fresh sessionとの区別、人格snapshot、送信同意、不確定時の同一ID確認とleaf-first retentionを揃えました。
- Agent migration READMEを予定例から実在する002→003→004→005へ修正し、Studioはsettings `20261004_11`に続く`20261005_12`までのchainを必要とすることを整合しました。両方ともlive適用は未実施です。
- 組織READMEの日英責務マップとowner AGENTSの現在地を更新し、進捗の正本へ辿れるようにしました。本書の更新履歴tableの分断も修正しました。

| Repository | 文書PR | main受け入れcommit | CI |
| --- | --- | --- | --- |
| flamoris-ai-agent | [#44](https://github.com/flamoris-jp/flamoris-ai-agent/pull/44) | `d5dd2e0a922e08e3054a71defb7f79b40d754a28` | [CI](https://github.com/flamoris-jp/flamoris-ai-agent/actions/runs/37312666002) 成功 |
| flamoris-ai-runtime | [#27](https://github.com/flamoris-jp/flamoris-ai-runtime/pull/27) | `e2f4ff709246922674d99344e02c8ce364f9e8f2` | [C++ contract and qualification](https://github.com/flamoris-jp/flamoris-ai-runtime/actions/runs/37312674102) 成功 |
| flamoris-generation-controller | [#8](https://github.com/flamoris-jp/flamoris-generation-controller/pull/8) | `2e85caac885a84a851d92a7864e7b94c4f95ad86` | [Controller CI](https://github.com/flamoris-jp/flamoris-generation-controller/actions/runs/37312595986) 成功 |
| flamoris-generation-mcp | [#73](https://github.com/flamoris-jp/flamoris-generation-mcp/pull/73) | `0ee424599f4a7d7d81453b59921fc9cbe9d64062` | [Python CI](https://github.com/flamoris-jp/flamoris-generation-mcp/actions/runs/37312680673) 成功 |
| flamoris-studio | [#67](https://github.com/flamoris-jp/flamoris-studio/pull/67) | `d03028df892f7b1dc658534d22abaffcc34a5134` | [Python Studio CI](https://github.com/flamoris-jp/flamoris-studio/actions/runs/37312685723) 成功 |
| flamoris-mcp-hub | [#40](https://github.com/flamoris-jp/flamoris-mcp-hub/pull/40) | `e74f2cfa45e5ea57ec0996c6d7a6e83c5c4b4aee` | [CI](https://github.com/flamoris-jp/flamoris-mcp-hub/actions/runs/37312693613) 成功 |
| flamoris-intelligence-mcp | [#13](https://github.com/flamoris-jp/flamoris-intelligence-mcp/pull/13) | `405507b20258c9fd1313b7be519c85bb7f01b4ba` | [CI](https://github.com/flamoris-jp/flamoris-intelligence-mcp/actions/runs/37312702394) 成功 |
| .github | [#29](https://github.com/flamoris-jp/.github/pull/29) | `693b7c53eafccf29dedd61dbd3d039e6bfa54324` | CI workflowなし（文書検証） |

上記owner文書8PRはmainへ反映済みです。全体文書とこの受け入れ記録は [AI #27](https://github.com/flamoris-jp/flamoris-ai/pull/27) にまとめました。AIと組織READMEにはCI workflowがなく、文書検証で確認しています。

検証：変更対象の相対／既知ownerのcurrent-mainファイルリンク174件、見出し参照2件、Markdown table／空白を確認しました。各PRの変更ファイルがMarkdownだけであること、head／baseとレビュー状態を照合しました。owner8PRはmerge後の受け入れtreeとレビュー済みheadのtree、main refの一致も確認済みです。実機情報の取得、deploy／restart、DB／grant／credential変更、GPU／有料provider callは行っていません。

次は翌日のE1棚卸しとmatched cutover計画です。Controller／Generation／Hubの対応catalogとschema、Agent／Studioの対応migration、既存data／unknown予約／rollback条件を組にして確認し、実機受け入れはその結果を別途記録します。

### 4.11 実機リリースの進捗（2026-10-06〜07 JST）

ユーザーの明示された実機更新指示に基づき、固定成果物の準備・バックアップ・Agent DB migrationまで進めました。23:42のoperator-pasted SSH結果を最後の確認点として、就寝のため切替前に中断しています。以下はその時点の証拠であり、将来の稼働状態の保証ではありません。private配置・設定・証拠ファイルと再開点はInternalのdated maintenance記録に置き、公開文書には秘密値・private topology・個人の会話内容を記載しません。

| 工程 | 2026-10-06の確認範囲 |
| --- | --- |
| 固定成果物 | Generation `0ee424599f4a7d7d81453b59921fc9cbe9d64062`、Intelligence `405507b20258c9fd1313b7be519c85bb7f01b4ba`、Agent `d5dd2e0a922e08e3054a71defb7f79b40d754a28`、Studio `d03028df892f7b1dc658534d22abaffcc34a5134`、Hub `e74f2cfa45e5ea57ec0996c6d7a6e83c5c4b4aee`の候補5イメージをビルド済み。切替未実施 |
| rollback準備 | システム／DB／既存設定・生成データと旧イメージを退避。旧Studio・HubのアーカイブはOCI indexから26参照blobのchecksum、稼働構成とlayersを照合。再保存せず既存アーカイブを受け入れ |
| 内部HTTP準備 | 既存proxyの旧MCP経路を保持して内部API経路を追加・reload。Controller秘密の生成側と転送ファイルの一致、受け渡し先の0600保護を確認。アプリrecreateと実API到達性の確認は未実施 |
| Agent DB復元検証 | 保存済みcustom-format dumpを専用検証DBへ同じrolesで復元。保護対象の内容・件数、既存table所有者・app権限を比較。004→005を検証DBで試適用し、既存内容と権限を保持。検証DBは削除済み |
| Agent DB本番 | `004_assistant_settings` → `005_model_continuations`を適用済み。元の`0.1`／002／003を保持。保護対象の内容・件数が適用前後で一致。retention timerは元のactive/waiting・enabledへ復帰 |
| 未実施 | Studio `20261005_12`、人格DB import、model/personality追加grant、Agent direct/registry/default設定、Studio/Hub設定とcatalog更新、GPU Node Managerと5アプリの切替、実GPU推論・会話継続・参照画像の実機受け入れ |

保存／読込経路の検証と実推論は別工程です。既存のprovider pin、保存状態、unknown予約、persona／conversation／grant identityを維持します。AI Runtimeの独立リリースは今回の必須依存に追加していません。

再開時は新しいlive状態を確認し、Internalの完了記録と適用版を照合します。候補イメージの再ビルド、完了済みbackup・復元検証・Agent 004/005適用を最初から繰り返しません。Studio migrationは翌日の再開で完了しました。次は設定準備、Agentの設定／人格／grant、Managerとアプリの固定成果物切替、その後の認証・catalog・local推論・会話継続を進めます。未完了のアプリ切替を「実機更新完了」と扱わないでください。


### 2026-10-07の再開とStudio DB適用

ユーザーの明示された再開指示で続行。operator-pasted SSH結果により、固定候補Studio imageと専用migration roleを用いたAlembic `20261005_12` の本番適用を確認しました。既存の保護対象12テーブルの内容・件数、table所有者とruntime DML権限を保持し、新しい `assistant_model_switches` の所有者・DML権限と0件を確認しています。期限付きlogin session／throttleは内容比較から除外しています。

初回はmigrationジョブのAlembic設定読取PermissionErrorで停止し、旧DB版と新テーブル未作成を確認後、当該一時ジョブだけroot userで再開しました。常駐アプリのuser、イメージ、設定は変更していません。ユーザー指示により既存の日次backupを前提とし、追加backupとStudio dumpの復元試験は実施していません。Agentの前日復元検証と004/005を繰り返していません。

旧Studio／Hubの同一コンテナ継続稼働を確認。この19:05時点では人格import・追加grant・アプリ切替は未完了でした。後続の切替結果と現在の残作業は4.12を参照します。

### 4.12. 2026-10-07の固定成果物切替と経路確認

operator-pasted SSH結果により、GPU Node ManagerとGeneration／Intelligence／Agent／Hub／Studioの切替を22:15 JSTまでに確認しました。準備済み人格をDBへimportし、既存の承認された2組のprincipalにlocalモデル・人格権限を設定しました。既存identity、人格のordered content、grant、会話、生成物を保持しています。

| 対象 | 固定source | 確認した範囲 |
| --- | --- | --- |
| GPU Node Manager core | `2f55fb74b986b7fa9c0eceb00cab2bd94187fdca` | core 1.0.0、既存native service・共通lock・privilege境界保持 |
| Generation | `0ee424599f4a7d7d81453b59921fc9cbe9d64062` | Controller内部API認証、output-root単一authority、managed input readiness、25tool、既存内容/inode保持 |
| Intelligence | `405507b20258c9fd1313b7be519c85bb7f01b4ba` | 起動と外部MCP 6tool。provider readiness／実推論は未確認 |
| Agent | `d5dd2e0a922e08e3054a71defb7f79b40d754a28` | direct execution、DB context、settings、API認証／Host／Origin guards、内部9操作・外部8tool、localモデル権限 |
| Hub | `e74f2cfa45e5ea57ec0996c6d7a6e83c5c4b4aee` | Generation直結とHubの25toolの名前・schema・annotationがfresh connectionで一致 |
| Studio | `d03028df892f7b1dc658534d22abaffcc34a5134` | runtime UID、DB role／revision、Generation／Agent gatewayのTLS・認証API、localモデル権限、Webと未認証asset拒否 |

Studioの元候補は非root runtimeからソースを読めず起動に失敗しました。旧image／envへのrollbackを検証し、固定候補から読込modeだけを補正した派生imageを作成。42件のmode変更以外はアプリ内容・ownership・runtime設定を保持し、runtime UIDで隔離factory初期化を確認しました。採用imageは `sha256:e6376961a3f07f9e66ca3d5ae9acd9de44b33c89b5af4a317b8a906c497d23e8`。常駐userをrootへ変更せず、DB migrationを繰り返していません。

Studio切替では既存envのinode／owner／mode／ACL、tokenとbindings、保護対象DB内容・権限、既存thumbnail、周辺サービスを保持。gatewayから実TLS経路でreadinessと権限を確認しました。Hub catalogは件数だけでなく25toolのschema／annotationも一致しています。

この切替工程ではsession／conversationを作成せず、実推論は行っていません。後続の基本Image検証は4.13を参照します。ログイン後UI、最小local GPU推論、会話継続、異なるモデルへのhandoff、bounded schema7の参照画像実行を次に個別確認します。承認済みlocalモデルは1つのため異モデルhandoffは現構成で検証できるかを確認し、未承認モデル追加で埋めません。raw Intelligenceの接続は未設定。remote／有料API、任意custom node、追加providerの受け入れは含みません。

本記録の更新は[AI #28](https://github.com/flamoris-jp/flamoris-ai/pull/28)で進行中で、main反映とは区別します。

### 4.13. 2026-10-07の基本Image実行

GPU Node Managerの正規MCP経路で、画像runtimeのREADY、遷移なし・異常なしをoperator結果から確認。外部Hub経由でbuiltin text-to-imageをbuildし、1回submit、status/result completed、asset一覧・PNG取得まで確認しました。512×512・4 stepsの最小動作確認で、画質の受け入れではありません。完了後のControllerは空き状態で、生成物は保持しています。

この結果は外部Hub／facade／Controller／ComfyUIの基本画像実行を確認するものです。GPU telemetry・runtime handoff、Studioログイン後UI、LLM推論・会話継続、異なるモデルへの切替、schema7の参照画像実行は未確認のままです。次は既存生成画像を管理参照入力として、bounded checkpoint img2img登録とHTTP経路のschema7実行を検証します。新しいStudio Image UIの機能追加を確認したとは記録しません。

### 4.14. 2026-10-07のschema7参照画像実行

22:37 JSTのoperator結果により、bounded checkpoint img2img ComfyWorkFlowの登録・digest／graph照合、管理参照入力、schema7 build、1回submit、completed、512×512 PNGの取得・decodeを確認しました。HTTPの認証済み接続から外部Hubで作成済みの画像を読めることを確認し、そのimmutable snapshotの内容を元画像と照合して参照に使っています。後続の外部Hub asset取得も成功しました。

参照画像出力は219208 bytes、SHA256 `1e39d0aa6c19c116470de21c8298003dffc7d146c1bf8d95214969420a261499`。完了後に今回の参照snapshotだけ削除し、生成物と登録定義・既存データを保持。Controllerの空き状態と周辺アプリの同一コンテナを確認しました。登録時のstatic validated／live_provider_verified=falseは維持し、個別の実行成功と区別しています。

これは512×512・4 stepsのbounded img2img実行確認です。IPAdapter／ControlNet／任意custom node、画質、Studio Imageの参照画像UI・ギャラリー登録は確認範囲に含みません。GPU handoff／telemetry、local LLM推論・会話継続、異なるモデルへのhandoff、ログイン後UIは残っています。

### 4.15. 2026-10-07のGPU handoffとAgentモデル解決

22:56 JSTのoperator結果により、正規Manager MCPから画像runtime→local LLM runtimeを1回だけ切り替え、LLM READY／healthy、他runtime OFF、遷移なし・異常なしを確認しました。旧画像runtimeのowned process解放、新LLM processとVRAM使用量を記録しています。GPUメモリが途中で完全に0だったとは記録しません。

最初のreadiness検査は公開model IDとproviderのserved aliasを混同して停止しました。起動停止を再送せず、固定Agentソースと実機default／registryの公開ID→承認済みproviderモデルmappingを照合し、direct adapterによる正確なモデル解決に成功しました。この修正は検査側で、AgentやLLMの設定変更は行っていません。

外部Intelligence facadeには公開IDをserved aliasとして使う設定が残っていることも判明しました。公開IDと既存上限を保ったalias補正を次に行います。Agent経路のモデル解決を外部facadeの成功と同一視しません。LLM実推論・会話継続、異なるモデルへの切替、ログイン後UIはまだ未確認です。

## 5. 詳細ロードマップ

これは工程と依存関係の計画です。将来の工程を列挙したこと自体で、保留中の実装や実機作業が再開するわけではありません。予定日・port・service方式など未決定の事項は固定しません。

| 工程 | 現在の状態 | 完了条件 / 主な依存 |
| --- | --- | --- |
| A. 設計・用語・担当整理 | 完了 | 外部/内部、任意Agent、3名称、状態所有者を固定。Controllerの当時の保留は4.5で更新 |
| B. 内部Intelligence接続 | 完了・main反映済み | shared adapter、Agent HTTP、Studio raw/Agentの両経路、保持する認可・送信同意・fenceを検証 |
| C. 旧Generation機能と呼出し元の削除 | 完了・main反映済み | custom/v3/Runtime委譲と対応Studio/Hub/Runtimeを削除し、基本/native・保存データ・unknown予約を保持 |
| D. ソース受け入れ・引継ぎ | 追加全体レビューを含め受け入れ完了。本書で共有入口を整備 | 最終PR CI、受け入れmerge commit、実装と実機の差、次作業と保留理由を一箇所から辿れる |
| E. 実機の移行計画・受け入れ | DB・人格／権限・Manager／5アプリ切替とgateway確認済み。UI／実推論等の受け入れが残る（4.12） | 実状態の確認、既存debtのreconcile、backup/rollback、明示された範囲の設定切替と実測 |
| F. Generation Controllerの実装準備・具体設計 | 完了。具体契約／hosting／service permissionを4.6で固定 | README/AGENTS、retained inventoryと実装方針を整備し、最小非MCP契約と一つの状態所有者を決定 |
| G. Generation内部接続の最終組み換え | 完了・main反映済み。最終head CI成功とmerge tree一致を4.8に記録 | Controllerへの内部接続と外部MCP facadeを一つのdomain authorityに接続し、互換経路を解消 |
| H. 制作機能の拡張 | 参照画像／ComfyWorkFlowと会話内LLM切替を4.9でmain受け入れ。実機受け入れは未完了 | bounded profileで個別に検証。追加provider／複数段／任意custom構成は別scope |

### E1. 実機移行前の棚卸し

再開時は、まず変更を伴わない確認から始めます。

1. 対象サービスのinstalled commit/package、設定の方式、接続先の種類、既存migration適用状況、grants、未確定request/jobを確認する。
2. 現在のserver/service/firewall/runtime healthは `flamoris-server-manager` を一次情報源にする。取得できない項目は未確認と記録し、過去の会話やrepoの例から実状態を推測しない。
3. Agentの旧 `AGENT_INTELLIGENCE_TRANSPORT=mcp` / MCP endpoint、Studioの旧 `/mcp` endpoint設定を識別する。実際に変更するまで新しい経路へ移行済みとはしない。
4. Generationのcustom/delegated active journal、input lease、unknown予約と保存済みデータを確認する。未解決なら実機の旧matched versionでdrain/reconcileする計画を作る。単にqueue recordが見えないことをreleaseの根拠にしない。
5. 既存のbackup/restore・rollback手順と必要な設定差分を、実測したversionに対してまとめる。公開文書には秘密値、個人情報、private topologyを記載しない。

完了条件：対象version・接続方式・未確定処理・データ保護・切替/rollbackの手順が具体化され、未確認項目と作業範囲が分かる。調査だけでruntime切替、再起動、DB/data/grant/credential変更を行わない。

### E2. Intelligence/Agentの実機切替と確認

実機作業が明示的に再開された範囲で行います。既存migrationの適用が必要かどうかは実機の状態で判断し、今回のcleanupに新しいDB migrationがあると仮定しません。

1. reviewed commitを固定した成果物と設定差分を準備する。shared libraryの導入と外部Intelligence MCP listenerの起動は別で、内部推論のために後者を必須にしない。
2. Agentのapproved target、provider/served model identity、送信同意と既存grantを確認し、内部HTTP `/api/v1` と直接executionを設定する。retired MCP modeを残して黙ってfallbackさせない。
3. Studio raw Intelligenceは実vendor origin・exact alias・operator上限・approved-local方針を確認する。Studio Agent SupportはAgent HTTP base URLとcore/settings catalogを整合させる。credentialsはbackendに留める。
4. 旧sessionや不確定requestを新targetへ自動付け替えせず、明示的な再開始・既存のexpiry/revocation/reconcile方針で扱う。
5. 許可されたlocal推論を最小の入力で確認し、二accountの隔離、CSRF、availability、request identity、logout/config/grant変更時の拒否と結果非公開を確認する。
6. 外部MCPの保持toolが利用できることを、内部接続の検証と分けて確認する。live/有料APIやGPU lifecycle操作を必要な確認範囲へ自動追加しない。

完了条件：rawとAgent Supportが実機で非MCPの内部経路を使い、実測したversion・結果・隔離・uncertainty・rollbackの証拠が残る。OpenAI等の有料provider受け入れは、別途認可された場合だけ実施する。

### E3. Generationの実機切替と確認

保持された基本/native生成と保存状態を実機で受け入れる工程です。cleanup基準のみの更新と、4.6のController対応版へのcutoverを区別し、指示された対象versionと経路を記録します。Controller対応版ではGの設定差分と単一owner条件も満たします。

1. admission停止と旧custom/delegated処理のdrain/reconcileを計画する。元journal、lease、asset/input、evidenceを保持し、起動を通すために予約を消さない。
2. Generation・Hub・Studioの対応version/catalogを揃える。Controller対応版では固定core artifact、内部HTTPの認証と直接接続も確認する。廃止toolを呼ばないことと、選定versionの23toolまたは4.9の25tool契約・descriptorの一致を確認する。
3. 許可された基本Imageまたは対象native profileでsubmit/status/result、preview/download、所有者確認と管理copyの削除を確認する。providerごとの本番qualificationは別に記録する。
4. restart後の保存済みasset/inputと不確定予約の扱い、古いcustom選択の拒否、参照inputの削除防止を確認する。停止していないprovider workを新authorityで再予約・再送しない。
5. liveのversion・確認したprofile・未確認profile・reconcile結果・rollback条件を記録する。

完了条件：保持されたgeneration機能が選定したversion・経路で確認され、旧custom/v3が実行不能で、保存データとunknown予約が保持される。非MCP cutoverの完了はController対応版とStudioの実設定・認証・両callerの共有状態を実測した場合だけ記録する。

### F. Generation Controller実装準備と具体設計

完了。[Controller #1](https://github.com/flamoris-jp/flamoris-generation-controller/issues/1)、[実装inventory](https://github.com/flamoris-jp/flamoris-generation-controller/blob/b57140954c8bc8176d882a053df70f00fad2be31/docs/IMPLEMENTATION.md)と[API契約](https://github.com/flamoris-jp/flamoris-generation-controller/blob/b57140954c8bc8176d882a053df70f00fad2be31/docs/API.md)に、MCP-free package、同居runtime、local ownership lock、strict request/result/error、service permissionと保存形式の維持を定めました。元の未決定DTO／hosting工程を繰り返さず、変更が必要なら現行契約を基準にレビューします。

### G. Generation内部接続の最終組み換え

Controller／外部facade／Studio gatewayのソースを4.6で実装し、4.7でレビュー修正、4.8で再レビューしてmain受け入れしました。最終headのCIは4.7、merge commitと検証済みtreeの一致は4.8を参照します。固定成果物のinstalled version切替とgateway確認は4.12、前段のAPI準備とDB適用は4.11を参照します。live cutoverはEの棚卸し、旧ownerのreconcile、backup/rollback、固定artifactと明示された設定変更に続きます。

実機ではStudioの旧MCP／Hub endpointから正確な `/api/v1/generation` baseへ変更し、hostの `FLAMORIS_CONTROLLER_TOKEN` と同じprivate `STUDIO_GENERATION_TOKEN`、空の `STUDIO_GENERATION_NAMESPACE` を設定します。これは旧endpointの自動fallbackではありません。old/new ownerを併走させず、既存unknown journalを消しません。

### H. 制作用途に合わせた後続作業

保持されたImage・Speech・Music、raw Intelligenceと任意Agent Supportをまず利用・検証できる状態にすることが優先です。既存の利用をController完成待ちにしません。

参照画像／ComfyWorkFlowと会話内LLM切替は4.9に実装・検証・main受け入れを記録しています。複数段の生成、追加providerや推論から生成への連携は対象profileを絞った別scopeです。Agent・ExecuteFlow・GPU切替を単純なJSON構築の必須条件に戻しません。Irodori・YuE2・SheetSage2等は保持されたnative契約と各実機の受け入れ状況を個別に照合します。

## 6. 次に着手する作業と未決定事項

| 優先 | 次の候補 | 開始条件 | 現在 |
| --- | --- | --- | --- |
| 1 | E1の実機棚卸し・matched cutover計画 | 実機調査／移行の指示と対象範囲 | 2026-10-06に開始。固定artifact・backup・Agent DB適用を確認（4.11）。停止／切替直前のbusy・request状態は再確認 |
| 2 | E2/E3の実機受け入れ | debt・backup/rollback・versionを確認し、変更範囲が明示される | DB・persona/grant・Manager／アプリ切替とgateway確認済み。基本Imageとschema7実行済み。ログイン後UI／GPU handoff／LLM推論・会話継続が残る（4.14） |
| 3 | Hの追加制作機能 | 個別の利用要件とscope | 任意custom node・追加provider・複数段生成は別scope |

4.2の実装と4.4の追加レビュー修正・README整理は受け入れ済みです。全体文書の採用commitはAI #23を参照します。親/子Issueは、受け入れた範囲と今後の境界を記録しており、openであることだけを根拠に完了済み実装をやり直しません。旧feature Issueの履歴・実機証拠も保持します。

## 7. 担当Issueと読む文書

| 担当 | 受け入れ / 継続Issue | 詳細 |
| --- | --- | --- |
| 全体 | [AI #18](https://github.com/flamoris-jp/flamoris-ai/issues/18) | [Architecture](docs/ARCHITECTURE.md)、[Roadmap](docs/ROADMAP.md)、[Studio boundary](docs/MULTIMODAL_STUDIO_ARCHITECTURE.md) |
| Intelligence | [#10](https://github.com/flamoris-jp/flamoris-intelligence-mcp/issues/10) | [Internal execution](https://github.com/flamoris-jp/flamoris-intelligence-mcp/blob/main/docs/INTERNAL_EXECUTION.md) |
| Agent | [#38](https://github.com/flamoris-jp/flamoris-ai-agent/issues/38) | [Internal execution](https://github.com/flamoris-jp/flamoris-ai-agent/blob/main/docs/INTERNAL_EXECUTION.md)、[settings](https://github.com/flamoris-jp/flamoris-ai-agent/blob/main/docs/ASSISTANT_SETTINGS_V1.md) |
| Studio | [#62](https://github.com/flamoris-jp/flamoris-studio/issues/62) | [Cleanup](https://github.com/flamoris-jp/flamoris-studio/blob/main/docs/ARCHITECTURE_CLEANUP.md)、[raw Intelligence](https://github.com/flamoris-jp/flamoris-studio/blob/main/docs/RAW_INTELLIGENCE.md) |
| Generation MCP | [#67](https://github.com/flamoris-jp/flamoris-generation-mcp/issues/67) | [Retirement/data/recovery](https://github.com/flamoris-jp/flamoris-generation-mcp/blob/main/docs/LEGACY_RETIREMENT.md) |
| Runtime | [#23](https://github.com/flamoris-jp/flamoris-ai-runtime/issues/23) | [Architecture](https://github.com/flamoris-jp/flamoris-ai-runtime/blob/main/docs/ARCHITECTURE.md) |
| Hub | [#36](https://github.com/flamoris-jp/flamoris-mcp-hub/issues/36) | [Generation rollout](https://github.com/flamoris-jp/flamoris-mcp-hub/blob/main/docs/GENERATION_ROLLOUT.md) |
| GPU Node Manager | [#11](https://github.com/flamoris-jp/flamoris-gpu-node-manager/issues/11) | [Architecture](https://github.com/flamoris-jp/flamoris-gpu-node-manager/blob/main/docs/ARCHITECTURE.md) |
| Generation Controller | [#1](https://github.com/flamoris-jp/flamoris-generation-controller/issues/1) | [実装方針・inventory](https://github.com/flamoris-jp/flamoris-generation-controller/blob/b57140954c8bc8176d882a053df70f00fad2be31/docs/IMPLEMENTATION.md)。共通core／API／単一所有者と対応PRを実装。source acceptance完了（4.8）。固定成果物cutoverとgateway確認は4.12、実生成受け入れは未完了 |
| 参照画像／ComfyWorkFlow | [Controller #6](https://github.com/flamoris-jp/flamoris-generation-controller/issues/6) | Controller #7・Generation #72・Hub #39。bounded登録／init-image、25tool、未確定submit保護（4.9） |
| 会話内LLM切替 | [Agent #42](https://github.com/flamoris-jp/flamoris-ai-agent/issues/42) | Agent #43・Studio #66。同じconversation、immutable lineage、durable switch fence（4.9） |

## 8. 別スレッドでの再開・更新方法

ChatGPTの会話記憶だけで自動同期するものではありません。新しいスレッドには [current mainのPROGRESS.md](https://github.com/flamoris-jp/flamoris-ai/blob/main/PROGRESS.md) を渡し、実際に読むよう指定します。リポジトリの [AGENTS.md](AGENTS.md) とREADMEからも本書へ辿れるようにします。

再開時の順序：

1. GitHubのcurrent mainで本書・AGENTS.md・対象の設計/Issueを読む。関連PRのmerge/CIと対象repoの最新ソースを照合し、過去スレッドの結果だけを現在の根拠にしない。
2. ユーザーの最新の指示、今回のtask scope、保留中の作業、現行ソースと実機の差を確認する。将来のロードマップを作業認可と取り違えない。
3. 同じ領域の作業が別スレッド/PRで進行中か確認し、独立したstore・予約・重複実装を作らない。
4. 実機情報が必要なときだけserver-managerで実状態を確認する。確認できなければ未確認のままにし、勝手な切替・再送・data削除で埋めない。
5. 作業後に本書の更新日、工程/状態、実際のPR/commit/検証、未確認・保留事項、次の一手を更新する。仕様変更はARCHITECTURE.md、詳細なinventoryは担当Issueへ記録してリンクする。
6. 更新をレビュー・GitHubへ反映してから共有リンクを渡す。ローカル下書き、未マージPR、main反映済みを区別する。

新スレッドに渡す例：

> `flamoris-jp/flamoris-ai` のcurrent mainにある `PROGRESS.md` と `AGENTS.md`、対象Issueを読んでください。既存ソース完了・実機未受け入れ・Controllerのmain受け入れ済みソース・実機未反映を区別し、今回お願いする作業は「対象工程と範囲をここに記入」です。完了後はPR/検証結果・次の作業を `PROGRESS.md` に反映してください。

最初に渡すリンク：<https://github.com/flamoris-jp/flamoris-ai/blob/main/PROGRESS.md>

## 9. 更新履歴

| 日付（JST） | 到達点 |
| --- | --- |
| 2026-10-04 | 外部MCP/内部契約、任意Agent、ExecuteFlow/ExecutionPlan/ComfyWorkFlowの区別、Controller未実装と旧custom subsystem削除の方向を整理 |
| 2026-10-05 | 内部Intelligence接続と旧Generation subsystem削除の6実装PRをCI成功・main反映。全体文書AI #20と親/子Issueへ受け入れ記録を追加 |
| 2026-10-05 | 本書をスレッド間共有の入口として追加。目的・詳細工程・固定した完了基準・今後の作業・再開/更新方法を集約 |
| 2026-10-05 | 9リポジトリの全体レビュー。旧runbook 13本・残存source分岐・文書矛盾とStudio Imageの認可／異常応答を追加修正し、open PRと検証を本書／レビュー記録へ集約 |
| 2026-10-05 | 組織 `.github` READMEを追補確認し、日英マップとAI現行案内から旧機構の説明・完了済み削除手順を整理。AI #23／組織 #28のopen PRと文書検証を記録 |
| 2026-10-05 | ユーザーの全件マージ指示でowner7PRと組織READMEをmain反映。マージ前CI／headとmerge tree一致を確認し、AI #23に受け入れcommit・残作業を集約 |
| 2026-10-05 | Controller #4／AI #24の準備受け入れ後、明示されたController実装指示でcore／APIとGeneration #71／Studio #65の対応接続を実装・検証。open PRと固定head、単一所有者／service permission／残る実機切替を4.6へ記録 |
| 2026-10-05 | Controller対応4PRを再レビュー。終了時の所有権、HTTP絶対期限、長いmodel IDの3件を修正し、11 testsとsource pinを更新。最新head・検証とレビュー記録を4.7へ追加 |
| 2026-10-05 | 再レビュー・重点92 testsと最終CIを確認後、Controller #5／Generation #71／Studio #65をmainへマージ。全merge tree一致と受け入れcommit、残る実機工程を4.8へ記録 |
| 2026-10-05 | 実機投入前のComfyWorkFlow登録／参照画像と会話内LLM切替を実装・レビュー。Controller #7／Generation #72／Hub #39／Agent #43／Studio #66のDraft PR、最終CIと残る採用・実機工程を4.9へ記録。今回のmerge／live変更なし |
| 2026-10-05 | 続く明示merge指示で実装5PRをmainへ反映。最新head／CI／レビュー状態を再確認し、全merge treeと検証済みsourceの一致を確認。AI #26へ受け入れcommitと残る実機棚卸し・migration／catalog整合を記録。live変更なし |
| 2026-10-05 | 実機を翌日へ延期。10repoの現行文書を照合し、9repoの37文書をソース受け入れ／25tool／会話内LLM切替／migration順序に整合。文書PRと検証・受け入れcommitを4.10／AI #27へ記録 |
| 2026-10-06 | 明示された実機リリースを再開。固定5候補・rollback／内部API準備、Agent DB復元・004/005試適用と本番適用、保護対象の保持・retention timer復帰を確認。アプリ／Manager切替前で中断し、残工程を4.11へ記録 |
| 2026-10-07 | DB・人格／権限・GPU Node Managerと5アプリ切替を確認。Studio起動時のソース読込権限を補正し、既存データ・設定・権限を保持。gateway TLS／API確認と未実施のUI／推論／会話／schema7受け入れを4.12へ記録 |
| 2026-10-07 | 外部Hubからbuiltin Imageを1回実行し、completed・PNG取得とController空き状態を確認。基本実行と未確認のschema7／LLM／UI／GPU handoffを4.13で区別 |
| 2026-10-07 | bounded checkpoint img2imgの登録・管理参照・schema7実行・PNG取得を確認。参照snapshotだけ完了後に削除し既存データを保持。残るGPU／LLM／UI受け入れを4.14へ記録 |
| 2026-10-07 | 正規Managerの画像→LLM handoff、owned process解放／VRAM観測、Agent directモデル解決を確認。外部Intelligence alias不一致と残る実推論／会話／UIを4.15へ記録 |
