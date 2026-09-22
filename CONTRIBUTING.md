# Contributing to FLAMORIS AI

Thank you for your interest in FLAMORIS.

FLAMORIS AI is the integration and architecture home for FLAMORIS AI projects. Please keep this repository small and use it for boundaries, shared direction, and cross-repository documentation rather than runtime code that belongs in a dedicated project.

## Before contributing

For small documentation fixes, feel free to open a pull request directly.

For larger architecture changes, new repositories, changes to repository ownership, or changes that affect multiple FLAMORIS AI projects, please open an Issue first.

Before proposing a new abstraction or repository, check whether the work belongs in:

- `flamoris-ai-agent`;
- `flamoris-intelligence-hub`;
- `flamoris-generation-mcp`;
- a product repository such as FLAMORIS 2D, Cutwork, Kachinco, or Studio;
- FLAMORIS Commons or one of its shared infrastructure repositories.

## Pull requests

Please:

- keep changes focused;
- preserve explicit ownership and dependency direction;
- distinguish current implementation from future plans;
- update the repository map when boundaries change;
- avoid introducing unnecessary dependencies;
- document architectural, compatibility, licensing, and security impact where relevant.

AI-assisted contributions are welcome. The contributor remains responsible for reviewing, testing, and understanding the submitted change.

## Licensing

Unless explicitly stated otherwise, code and documentation contributions are submitted under the Apache License 2.0.

Do not add third-party code, AI models, model weights, datasets, prompts, media, generated assets, or provider-hosted material unless their licenses and redistribution terms are compatible and clearly documented.

## Security and privacy

Do not include credentials, API keys, private hostnames, personal deployment details, private datasets, generated private media, or developer-specific machine paths in Issues, examples, tests, or documentation.

Use SECURITY.md for vulnerability-reporting guidance.

## Support

FLAMORIS does not provide guaranteed individual support.

Use repository documentation, Issues, tests, logs, and source code as primary references. AI-assisted self-support is encouraged.

---

# FLAMORIS AI へのコントリビューション

FLAMORISに興味を持っていただきありがとうございます。

FLAMORIS AIは、AI関連プロジェクトの実装を全部集める場所ではなく、リポジトリ同士の境界、共通方針、横断設計を整理するための場所です。

## 変更を始める前に

小さな文書修正は、そのままPull Requestを送っていただいて構いません。

大きな設計変更、新しいリポジトリ、責務の移動、複数のAI系リポジトリへ影響する変更は、先にIssueで意図と境界を整理してください。

新しい実装や共通化を始める前に、次のどこへ置くべきか確認してください。

- `flamoris-ai-agent`
- `flamoris-intelligence-hub`
- `flamoris-generation-mcp`
- FLAMORIS 2D / Cutwork / Kachinco / Studioなどの製品リポジトリ
- FLAMORIS Commonsまたは共通基盤リポジトリ

## Pull Request

以下を意識してください。

- 変更範囲を絞る
- authorityと依存方向を明確に保つ
- 現在の実装と将来構想を混同しない
- 境界が変わったらrepository mapを更新する
- 不要な依存関係を増やさない
- 必要に応じて設計、互換性、ライセンス、セキュリティへの影響を書く

AIを使ったコントリビューションも歓迎します。提出する変更の確認、テスト、内容の理解については、コントリビュータ自身が責任を持ってください。

## ライセンス

明記がない限り、コードとドキュメントへのコントリビューションはApache License 2.0の条件で提供されます。

第三者のコード、AIモデル、model weights、データセット、prompt、画像・音声・動画、生成物、provider側のassetなどを追加する場合は、利用条件と再配布条件を確認し、適用されるライセンスを明記してください。

## セキュリティとプライバシー

認証情報、APIキー、private hostname、個人環境のdeployment情報、private dataset、非公開の生成物、開発者固有の絶対パスをIssue、example、test、documentationへ含めないでください。

脆弱性報告についてはSECURITY.mdを参照してください。

## サポート

FLAMORISは個別サポートを保証しません。

README、Issue、テスト、ログ、ソースコードを主な参照先として利用してください。AIによる自己サポートも歓迎します。
