# Security Policy

## Reporting a vulnerability

Please do not post suspected security vulnerabilities, credentials, API keys, tokens, private prompts, personal data, private generated media, or other sensitive information in a public Issue.

If GitHub private vulnerability reporting is available for this repository, please use it.

If private reporting is not available, avoid publishing exploit details or secrets publicly. Contact the FLAMORIS maintainers through an appropriate private channel before disclosing sensitive details.

For non-sensitive security hardening, dependency updates, provider-safety discussions, or general security design, a normal GitHub Issue is welcome.

## AI-specific security scope

AI integrations can cross trust boundaries between applications, local runtimes, remote providers, tools, filesystems, and generated assets.

Changes should treat the following as security-sensitive:

- provider credentials and API keys;
- prompts and tool inputs that may contain private information;
- model and dataset provenance;
- local filesystem paths and arbitrary file access;
- tool permissions and capability scope;
- network endpoints and tunnel configuration;
- provider responses and generated filenames;
- prompt injection or untrusted content entering tool-enabled agents;
- resource exhaustion from generation or inference workloads.

Do not commit live credentials, private deployment details, model weights, private datasets, or private generated outputs.

## Supported versions

FLAMORIS is developed as an open-source project without a guaranteed support window or security-response SLA.

Security fixes are generally applied to the current maintained codebase rather than to every historical version.

## Scope

This policy applies to code and documentation maintained by FLAMORIS.

Third-party dependencies, AI models, model weights, datasets, providers, hosted services, generated media, and other assets may have their own security, privacy, licensing, and support policies.

---

# セキュリティポリシー

## 脆弱性の報告

脆弱性の可能性がある情報、認証情報、APIキー、token、private prompt、個人情報、非公開の生成物、その他の機密情報を公開Issueへ投稿しないでください。

このリポジトリでGitHubのPrivate vulnerability reportingが利用できる場合は、そちらを使用してください。

Private reportingが利用できない場合も、攻撃手順や秘密情報を公開せず、機密情報を共有する前にFLAMORISのメンテナへ適切な非公開手段で連絡してください。

機密性のないsecurity hardening、依存関係更新、provider安全性、一般的なsecurity設計については、通常のGitHub Issueを利用して構いません。

## AI固有の注意点

AI連携では、アプリ、local runtime、remote provider、tool、filesystem、生成assetの間にtrust boundaryがあります。

特に以下をsecurity-sensitiveとして扱ってください。

- provider credential / API key
- private informationを含む可能性のあるpromptやtool input
- model / datasetの出所
- local filesystem pathや任意ファイルアクセス
- tool permission / capability scope
- network endpoint / tunnel設定
- provider response / generated filename
- tool利用可能なAgentへ入るuntrusted contentやprompt injection
- generation / inferenceによるresource exhaustion

live credential、個人環境のdeployment情報、model weights、private dataset、非公開生成物はcommitしないでください。

## サポート対象

FLAMORISはオープンソースプロジェクトとして開発されており、サポート期間やsecurity対応時間を保証していません。

security fixは、原則として現在保守しているcodebaseへ適用します。

## 対象範囲

このpolicyはFLAMORISが管理するコードとドキュメントに適用されます。

第三者dependency、AI model、model weights、dataset、provider、hosted service、generated mediaなどには、それぞれ別のsecurity、privacy、license、support条件が適用される場合があります。
