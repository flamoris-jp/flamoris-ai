# FLAMORIS AI アーキテクチャ組み換え：進捗と引継ぎ

更新日：2026-10-05（JST）  
対象：外部MCP・内部Intelligence・任意Agent・Generation・Runtimeの責務と接続の整理  
決定と受け入れ記録：[AI #18](https://github.com/flamoris-jp/flamoris-ai/issues/18)

このファイルは、ChatGPTの別スレッドや次の作業担当が、目的・現在地・残りの作業をまとめて確認する入口です。設計の基準は [ARCHITECTURE.md](docs/ARCHITECTURE.md)、担当Issueと従来の順序は [ROADMAP.md](docs/ROADMAP.md)。本書はその進捗と次の具体的な工程を記録します。

**現在地：内部Intelligence接続の組み換えと、誤って追加したGenerationのcustom/v3/Runtime委譲の削除は、検証・レビュー修正・mainへのマージまで完了。実機移行は未着手。Generation Controllerは未実装・保留です。**

全体の組み換えが実機まで完了した、またはGenerationの内部接続がすべて非MCPになったという意味ではありません。今のGenerationは基本/native生成・input・assetのMCP互換経路を残しています。

## 1. 作業の目的

制作側が必要な知能・生成機能を使えるように、各リポジトリの責務と接続を整理します。MV制作などの利用に戻れる基盤を作り、単純な推論や生成のために不要な人格・中継・実行エンジンを必須にしないことが目的です。

| 整理する点 | 到達したい状態 |
| --- | --- |
| 外部と内部の通信 | MCPはChatGPTなど外部クライアントの入口。内部サービス・アプリは小さな非MCP契約を使う |
| Hubの役割 | 外部MCPのcatalog・connection・routingを担当。内部サービスバスや実行エンジンにしない |
| Agentの役割 | 人格・会話・memory・principal/session・context policyが必要な場合だけ使う。通常推論・生成はAgentなしで使える |
| 推論の実行 | 共通のprovider adapterを再利用し、StudioやAgentに同じ通信実装を複製しない。新しい中央gatewayサービスも作らない |
| Runtimeの役割 | 推論、ExecuteFlow、コンパイル済みExecutionPlan、Job/Continuation/resource制御を維持する |
| Generationの整理 | 後付けのcustom ComfyWorkFlow登録・版管理・合成・Runtime委譲を削除。同じものをControllerやRuntimeに移植・再作成しない |
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

GPU Node Managerがhost-wideなruntime起動停止・GPU handoff・transition lockを担当します。Studio、Agent、Runtime、将来のControllerに別のhost state machineを作りません。lifecycle READY、モデルの利用可能性、特定グラフのqualification、呼出し元の認可は別の判断です。

## 3. 現在のソース接続と将来の目標

以下はソースが提供する経路です。稼働中の設定・接続を実測した表ではありません。

| 利用経路 | 現在のソース | 将来・保持する境界 |
| --- | --- | --- |
| Studioの素のIntelligence | Studio `IntelligenceGateway` → shared `flamoris_intelligence` → 設定されたlocal llama.cpp/vendor HTTP API | AgentやIntelligence MCP listenerは不要。provider詳細は共通adapterに閉じる |
| Studio Agent Support | Studio `AgentGateway` → Agent JSON HTTP `/api/v1` → Agentのapproved execution client → shared adapter | Agentがprincipal・人格・会話・モデル/送信同意を所有する |
| 外部Intelligence MCP | 外部クライアント → Hub等の外部入口 → Intelligence MCP facade → shared adapter | 外部のtool/schema/transport・結果変換を維持する |
| 外部Agent MCP | Agentの既存inbound MCP surfaceを保持 | 内部HTTPへの置換とは別の互換面。実際の外部登録・稼働状態は別途確認する |
| Studio generation | 既存Generation gateway → 設定されたMCP互換境界 → 保持されたgeneration domain/provider | 将来はStudio → 非MCPのGeneration Controller契約へ変更する。現在は未実施 |
| 外部Generation MCP | 外部クライアント → Hub → Generation MCP。基本/nativeのdomain codeは現在も同じpackageに共存 | 将来はMCPを薄い外部facadeにし、一つのController側domain authorityへ接続する |
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

Studioの元のbuiltin Imageは参照画像を受け付けません。古いcustom/v3選択は、利用者が対応builtinを明示的に選び直すまで実行不可です。既存input/upload/assetと未確定executionの削除防止は残っています。**「旧v3を削除済み」と「参照画像が新構成で使える」は別で、後者は未実装・保留です。**

詳細なinventoryと互換条件は [Generation retirement](https://github.com/flamoris-jp/flamoris-generation-mcp/blob/main/docs/LEGACY_RETIREMENT.md)、[Agent internal execution](https://github.com/flamoris-jp/flamoris-ai-agent/blob/main/docs/INTERNAL_EXECUTION.md)、[Studio cleanup](https://github.com/flamoris-jp/flamoris-studio/blob/main/docs/ARCHITECTURE_CLEANUP.md) にあります。

## 5. 詳細ロードマップ

これは工程と依存関係の計画です。将来の工程を列挙したこと自体で、保留中の実装や実機作業が再開するわけではありません。予定日・port・service方式など未決定の事項は固定しません。

| 工程 | 現在の状態 | 完了条件 / 主な依存 |
| --- | --- | --- |
| A. 設計・用語・担当整理 | 完了 | 外部/内部、任意Agent、3名称、状態所有者、Controller保留を文書とIssueに固定 |
| B. 内部Intelligence接続 | 完了・main反映済み | shared adapter、Agent HTTP、Studio raw/Agentの両経路、保持する認可・送信同意・fenceを検証 |
| C. 旧Generation機能と呼出し元の削除 | 完了・main反映済み | custom/v3/Runtime委譲と対応Studio/Hub/Runtimeを削除し、基本/native・保存データ・unknown予約を保持 |
| D. ソース受け入れ・引継ぎ | ソース受け入れ完了。本書で共有入口を整備 | 最終CI、PR/commit、実装と実機の差、次作業と保留理由を一箇所から辿れる |
| E. 実機の移行計画・受け入れ | 未着手・今回の認可範囲外 | 実状態の確認、既存debtのreconcile、backup/rollback、明示された範囲の設定切替と実測 |
| F. Generation Controllerの具体設計 | 保留・具体契約は未決定 | ユーザーが再開した後、保持domainのinventory、最小非MCP契約と一つの状態所有者を設計 |
| G. Generation内部接続の最終組み換え | 未実装・F待ち | Controllerへの内部接続と外部MCP facadeを一つのdomain authorityに接続し、互換経路を解消 |
| H. 制作機能の拡張 | 個別scope。新しい参照画像/custom構成は保留 | 利用目的、provider契約、実機受け入れを個別に定める。旧v3やmandatory Runtime bridgeを再作成しない |

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

### E3. Generation互換経路の実機更新

この工程はController実装を必要としません。保持された基本/native生成を使うための、今回のcleanup成果物の受け入れです。

1. admission停止と旧custom/delegated処理のdrain/reconcileを計画する。元journal、lease、asset/input、evidenceを保持し、起動を通すために予約を消さない。
2. Generation・Hub・Studioの対応version/catalogを揃える。廃止toolを呼ばないことと、保持23toolの外部契約・builtin/native descriptorの一致を確認する。
3. 許可された基本Imageまたは対象native profileでsubmit/status/result、preview/download、所有者確認と管理copyの削除を確認する。providerごとの本番qualificationは別に記録する。
4. restart後の保存済みasset/inputと不確定予約の扱い、古いcustom選択の拒否、参照inputの削除防止を確認する。停止していないprovider workを新authorityで再予約・再送しない。
5. liveのversion・確認したprofile・未確認profile・reconcile結果・rollback条件を記録する。

完了条件：保持されたgeneration機能が実機で確認され、旧custom/v3が実行不能で、保存データとunknown予約が保持される。Generationの非MCP cutoverが済んだとは記録しない。

### F. 将来のGeneration Controller設計

現在は [Controller #1](https://github.com/flamoris-jp/flamoris-generation-controller/issues/1) の文書上の境界だけです。実装再開・具体契約の決定は保留しています。

再開が指示された場合の設計順序：

1. 現在保持されたprovider adapter/capability、request/recipe、JobStore、Input/Asset、retention、provenanceと実際のcallerをinventoryする。削除したcustom/v3 subsystemは移植元にしない。
2. Studioと外部MCP facadeが必要とする最小のrequest/result・job observation・scoped cancellation・input/asset契約を定める。認可、ID互換、bounds、unknown/no-replayも含める。
3. library/service方式と既存状態の所有者を決める。具体endpoint・port・DB配置はcallerと運用を確認してから決め、先に新frameworkやschedulerを作らない。
4. 二frontendが同じdomain authority・journal・reservationを使う方法と、既存ID/dataの互換・移行・rollbackを定める。ControllerとMCPに独立したJobStoreを作らない。
5. provider契約テスト、二callerの同時admission/隔離、uncertain submit・restart・cancel・input/asset権限・bounded transferの受け入れ項目を決める。静的JSONと実GPU/provider qualificationを分ける。
6. 設計をレビューし、具体的な実装Issueへ分割する。新しいComfyWorkFlow機能が必要なら制作上の要件から別に設計する。旧登録/版管理/合成を自動再作成しない。

完了条件：一つの状態所有者、必要な最小契約、互換/移行条件と検証方法が決まり、実装範囲が明示される。設計完了とController実装完了を区別する。

### G. Generation内部接続の最終組み換え

Fの設計と実装再開が前提です。

1. reviewed契約に沿ったController/domain実装を作る。GPU lifecycle・Agent state・Runtime推論のauthorityを取り込まない。
2. Generation MCPを外部tool/schema/transport・入力/結果変換のfacadeとして接続する。保持された外部toolとIDの互換は維持または明示的にversion化する。
3. Studio Generation gatewayを非MCP内部契約へ置換する。ブラウザDTOとuser ownershipを維持し、内部MCP互換依存を削除する。
4. external clientとStudioが同じJob/Input/Asset authorityを使うことを、同時利用・失敗・restart・不確定結果まで検証する。
5. レビュー修正・CI・mergeを完了し、実機cutoverを別工程で行う。旧dataや未確定処理を新frontendごとのstoreにコピーして稼働させない。

完了条件：内部Generationも非MCPになり、外部MCPは保持され、状態が一つのdomain authorityに集約される。移植しないと決めた旧custom/v3は復活しない。ソース受け入れと実機受け入れを両方記録する。

### H. 制作用途に合わせた後続作業

保持されたImage・Speech・Music、raw Intelligenceと任意Agent Supportをまず利用・検証できる状態にすることが優先です。既存の利用をController完成待ちにしません。

新しい参照画像、ComfyWorkFlow機能、複数段の生成、追加providerや推論から生成への連携は、利用者の要求と対象profileを絞った別Issueで扱います。Agent・ExecuteFlow・GPU切替を単純なJSON構築の必須条件に戻しません。Irodori・YuE2・SheetSage2等は保持されたnative契約と、各実機の受け入れ状況を個別に照合します。

## 6. 次に着手する作業と未決定事項

| 優先 | 次の候補 | 開始条件 | 現在 |
| --- | --- | --- | --- |
| 1 | E1の実機棚卸しと移行計画 | 実機調査/移行を再開する指示と対象範囲 | 未着手。現在の稼働状態はこの文書から判断しない |
| 2 | E2/E3の設定切替と実機受け入れ | E1でdebt・backup/rollback・versionを確認し、変更範囲が明示される | 未着手 |
| 3 | FのController具体設計 | Controller設計を再開する指示 | 保留。library/service・API・port・migration方式は未決定 |
| 4 | GのController実装と内部Generation切替 | Fのreviewed契約と実装再開 | 未実装 |
| 5 | Hの制作機能拡張 | 個別の利用要件とscope | 新しい参照画像/custom構成は保留 |

今回受け入れたsource cleanupの未マージ実装PRはありません。親/子Issueは、受け入れた範囲と今後の境界を記録しており、openであることだけを根拠に完了済み実装をやり直しません。旧feature Issueの履歴・実機証拠も保持します。

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
| Generation Controller | [#1](https://github.com/flamoris-jp/flamoris-generation-controller/issues/1) | 将来の境界のみ。実装保留 |

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

> `flamoris-jp/flamoris-ai` のcurrent mainにある `PROGRESS.md` と `AGENTS.md`、対象Issueを読んでください。ソース完了・実機未反映・Controller保留を区別し、今回お願いする作業は「対象工程と範囲をここに記入」です。完了後はPR/検証結果・次の作業を `PROGRESS.md` に反映してください。

最初に渡すリンク：<https://github.com/flamoris-jp/flamoris-ai/blob/main/PROGRESS.md>

## 9. 更新履歴

| 日付（JST） | 到達点 |
| --- | --- |
| 2026-10-04 | 外部MCP/内部契約、任意Agent、ExecuteFlow/ExecutionPlan/ComfyWorkFlowの区別、Controller未実装と旧custom subsystem削除の方向を整理 |
| 2026-10-05 | 内部Intelligence接続と旧Generation subsystem削除の6実装PRをCI成功・main反映。全体文書AI #20と親/子Issueへ受け入れ記録を追加 |
| 2026-10-05 | 本書をスレッド間共有の入口として追加。目的・詳細工程・固定した完了基準・今後の作業・再開/更新方法を集約 |
