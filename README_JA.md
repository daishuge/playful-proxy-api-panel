# Playful Proxy API Panel (PPAP)

> [!NOTE]
> **Playful Proxy API Panel (PPAP)** は [`router-for-me/CLIProxyAPI`](https://github.com/router-for-me/CLIProxyAPI) のコミュニティ fork です。上流互換を追いながら、管理パネル、永続化される使用量統計、Codex 向けモデル alias、ローカルコスト推定を同じ release ラインで提供します。
>
> 素の CLIProxyAPI が必要な場合は上流を使ってください。PPAP 固有の統計、価格推定、管理パネル変更が必要な場合は、このリポジトリの Releases から手動でインストールしてください。

[English](README.md) | [中文](README_CN.md) | 日本語

CLI向けのOpenAI/Gemini/Claude/Codex互換APIインターフェースを提供するプロキシサーバーです。

OAuth経由でOpenAI Codex（GPTモデル）およびClaude Codeもサポートしています。

ローカルまたはマルチアカウントのCLIアクセスを、OpenAI（Responses含む）/Gemini/Claude互換のクライアントやSDKで利用できます。

## 概要

- CLIモデル向けのOpenAI/Gemini/Claude互換APIエンドポイント
- OAuthログインによるOpenAI Codexサポート（GPTモデル）
- OAuthログインによるClaude Codeサポート
- プロバイダールーティングによるAmp CLIおよびIDE拡張機能のサポート
- ストリーミングおよび非ストリーミングレスポンス
- 関数呼び出し/ツールのサポート
- マルチモーダル入力サポート（テキストと画像）
- ラウンドロビン負荷分散による複数アカウント対応（Gemini、OpenAI、Claude）
- シンプルなCLI認証フロー（Gemini、OpenAI、Claude）
- Generative Language APIキーのサポート
- AI Studioビルドのマルチアカウント負荷分散
- Gemini CLIのマルチアカウント負荷分散
- Claude Codeのマルチアカウント負荷分散
- OpenAI Codexのマルチアカウント負荷分散
- 設定によるOpenAI互換アップストリームプロバイダー（例：OpenRouter）
- プロキシ埋め込み用の再利用可能なGo SDK（`docs/sdk-usage.md`を参照）

## PPAPを選ぶ理由

PPAPはUIだけのラッパーでも、プロトコルの全面的な書き直しでもありません。CLIProxyAPI互換のまま、自ホスト運用に必要な観測、パネル、alias、release assetsを一つのリポジトリで管理する fork です。

| 目的 | Upstream CLIProxyAPI | PPAP |
| --- | --- | --- |
| コア proxy 互換性 | 対応 | 対応し、上流更新も継続的に取り込み |
| 使用量統計 | 上流実装に依存 | management usage API、import/export、ローカル snapshot 永続化 |
| 運用メトリクス | 基本的な request 情報 | cache hit rate、first-byte latency、average latency、TPS、token breakdown、API/model rollup |
| 管理パネル | 外部パネルまたは別 release に依存 | パネル source が本 repo にあり、release で `management.html` も同時公開 |
| Codex 体験 | model 名は主に pass-through | Spark 価格推定、`model(high)`/`model-high`、自動 `-low/-medium/-high/-xhigh` alias |
| 自ホスト配布 | 上流 package channel | PPAP Releases と local Docker build で fork 固有の挙動を確認しやすい |

## PPAP固有の主な違い

- **使用量統計を内蔵。** `/v0/management/usage`、`/v0/management/usage/export`、`/v0/management/usage/import` を復元し、ローカル snapshot に保存できます。
- **管理パネルを同じ release で配布。** フロントエンド source は [`web/management-panel`](web/management-panel) にあり、release asset には対応する `management.html` が含まれます。
- **Codex Spark を価格推定に追加。** `gpt-5.3-codex-spark` は preview 中のため `gpt-5.3-codex` の推定価格を使います。参考: [Spark announcement](https://openai.com/index/introducing-gpt-5-3-codex-spark/)、[Codex rate card](https://help.openai.com/en/articles/11369540-codex-rate-card)、[API pricing](https://openai.com/api/pricing/)。
- **thinking alias を標準化。** `model(high)` と `model-high` の両方を扱い、`low`、`medium`、`high`、`xhigh` を安全に解決します。
- **上流更新も取り込み。** Redis usage queue retention 設定を取り込みつつ、PPAP の usage persistence は維持しています。

## はじめに

CLIProxyAPIガイド：[https://help.router-for.me/](https://help.router-for.me/)

## Docker Installation

This fork does not publish a separate public Docker image by default. Build it locally from this repository so the container includes the fork-specific usage statistics changes.

```bash
git clone https://github.com/daishuge/playful-proxy-api-panel.git
cd playful-proxy-api-panel
cp config.example.yaml config.yaml
mkdir -p auths logs
docker compose up -d --build
```

The default compose file builds `cliproxyapi-fork:local` from the local `Dockerfile`, maps port `8317`, and mounts `./config.yaml`, `./auths`, and `./logs` into the container. Keep your own keys, OAuth files, proxy settings, and management password in those local files/directories, and do not commit them.

## 管理API

[MANAGEMENT_API.md](https://help.router-for.me/management/api)を参照

## 使用量統計

This fork restores the built-in `/v0/management/usage`, `/v0/management/usage/export`, and `/v0/management/usage/import` endpoints. The usage snapshot records cache hit rate, first-byte latency, average latency, TPS, token breakdowns, and per-API/per-model details when the upstream provider returns enough usage data.

When `usage-statistics-enabled` is true, PPAP automatically persists the usage snapshot to `usage-statistics.json` next to `config.yaml` unless `usage-statistics-path` is set. The existing TUI usage tab displays these metrics with the same card and table style as the original interface. External tools such as [CPA Usage Keeper](https://github.com/Willxup/cpa-usage-keeper) can still be used when a separate dashboard is needed.

## Management Panel Source

The management panel frontend now lives in this repository under [`web/management-panel`](web/management-panel). Releases publish backend binaries and the single-file `management.html` from the same tag, so the backend panel auto-updater can point directly at this repository.

The previous `daishuge/CLIProxyAPI` repository name is kept as a GitHub redirect after the rename. The standalone panel repository remains available during the transition, but new changes land here.

## Amp CLIサポート

CLIProxyAPIは[Amp CLI](https://ampcode.com)およびAmp IDE拡張機能の統合サポートを含んでおり、Google/ChatGPT/ClaudeのOAuthサブスクリプションをAmpのコーディングツールで使用できます：

- Ampの APIパターン用のプロバイダールートエイリアス（`/api/provider/{provider}/v1...`）
- OAuth認証およびアカウント機能用の管理プロキシ
- 自動ルーティングによるスマートモデルフォールバック
- 利用できないモデルを代替モデルにルーティングする**モデルマッピング**（例：`claude-opus-4.5` → `claude-sonnet-4`）
- localhostのみの管理エンドポイントによるセキュリティファーストの設計

特定のバックエンド系統のリクエスト/レスポンス形状が必要な場合は、統合された `/v1/...` エンドポイントよりも provider-specific のパスを優先してください。

- messages 系のバックエンドには `/api/provider/{provider}/v1/messages`
- モデル単位の generate 系エンドポイントには `/api/provider/{provider}/v1beta/models/...`
- chat-completions 系のバックエンドには `/api/provider/{provider}/v1/chat/completions`

これらのパスはプロトコル面の選択には役立ちますが、同じクライアント向けモデル名が複数バックエンドで再利用されている場合、それだけで推論実行系が一意に固定されるわけではありません。実際の推論ルーティングは、引き続きリクエスト内の model/alias 解決に従います。厳密にバックエンドを固定したい場合は、一意な alias や prefix を使うか、クライアント向けモデル名の重複自体を避けてください。

**→ [Amp CLI統合ガイドの完全版](https://help.router-for.me/agent-client/amp-cli.html)**

## SDKドキュメント

- 使い方：[docs/sdk-usage.md](docs/sdk-usage.md)
- 上級（エグゼキューターとトランスレーター）：[docs/sdk-advanced.md](docs/sdk-advanced.md)
- アクセス：[docs/sdk-access.md](docs/sdk-access.md)
- ウォッチャー：[docs/sdk-watcher.md](docs/sdk-watcher.md)
- カスタムプロバイダーの例：`examples/custom-provider`

## コントリビューション

コントリビューションを歓迎します！お気軽にPull Requestを送ってください。

1. リポジトリをフォーク
2. フィーチャーブランチを作成（`git checkout -b feature/amazing-feature`）
3. 変更をコミット（`git commit -m 'Add some amazing feature'`）
4. ブランチにプッシュ（`git push origin feature/amazing-feature`）
5. Pull Requestを作成

## 関連プロジェクト

CLIProxyAPIをベースにした以下のプロジェクトがあります：

### [vibeproxy](https://github.com/automazeio/vibeproxy)

macOSネイティブのメニューバーアプリで、Claude CodeとChatGPTのサブスクリプションをAIコーディングツールで使用可能 - APIキー不要

### [Subtitle Translator](https://github.com/VjayC/SRT-Subtitle-Translator-Validator)

CLIProxyAPI経由でGeminiサブスクリプションを使用してSRT字幕を翻訳するブラウザベースのツール。自動検証/エラー修正機能付き - APIキー不要

### [CCS (Claude Code Switch)](https://github.com/kaitranntt/ccs)

CLIProxyAPI OAuthを使用して複数のClaudeアカウントや代替モデル（Gemini、Codex、Antigravity）を即座に切り替えるCLIラッパー - APIキー不要

### [Quotio](https://github.com/nguyenphutrong/quotio)

Claude、Gemini、OpenAI、Antigravityのサブスクリプションを統合し、リアルタイムのクォータ追跡とスマート自動フェイルオーバーを備えたmacOSネイティブのメニューバーアプリ。Claude Code、OpenCode、Droidなどのコーディングツール向け - APIキー不要

### [CodMate](https://github.com/loocor/CodMate)

CLI AIセッション（Codex、Claude Code、Gemini CLI）を管理するmacOS SwiftUIネイティブアプリ。統合プロバイダー管理、Gitレビュー、プロジェクト整理、グローバル検索、ターミナル統合機能を搭載。CLIProxyAPIと統合し、Codex、Claude、Gemini、AntigravityのOAuth認証を提供。単一のプロキシエンドポイントを通じた組み込みおよびサードパーティプロバイダーの再ルーティングに対応 - OAuthプロバイダーではAPIキー不要

### [ProxyPilot](https://github.com/Finesssee/ProxyPilot)

TUI、システムトレイ、マルチプロバイダーOAuthを備えたWindows向けCLIProxyAPIフォーク - AIコーディングツール用、APIキー不要

### [Claude Proxy VSCode](https://github.com/uzhao/claude-proxy-vscode)

Claude Codeモデルを素早く切り替えるVSCode拡張機能。バックエンドとしてCLIProxyAPIを統合し、バックグラウンドでの自動ライフサイクル管理を搭載

### [ZeroLimit](https://github.com/0xtbug/zero-limit)

CLIProxyAPIを使用してAIコーディングアシスタントのクォータを監視するTauri + React製のWindowsデスクトップアプリ。Gemini、Claude、OpenAI Codex、Antigravityアカウントの使用量をリアルタイムダッシュボード、システムトレイ統合、ワンクリックプロキシコントロールで追跡 - APIキー不要

### [CPA-XXX Panel](https://github.com/ferretgeek/CPA-X)

CLIProxyAPI向けの軽量Web管理パネル。ヘルスチェック、リソース監視、リアルタイムログ、自動更新、リクエスト統計、料金表示機能を搭載。ワンクリックインストールとsystemdサービスに対応

### [CLIProxyAPI Tray](https://github.com/kitephp/CLIProxyAPI_Tray)

PowerShellスクリプトで実装されたWindowsトレイアプリケーション。サードパーティライブラリに依存せず、ショートカットの自動作成、サイレント実行、パスワード管理、チャネル切り替え（Main / Plus）、自動ダウンロードおよび自動更新に対応

### [霖君](https://github.com/wangdabaoqq/LinJun)

霖君はAIプログラミングアシスタントを管理するクロスプラットフォームデスクトップアプリケーションで、macOS、Windows、Linuxシステムに対応。Claude Code、Gemini CLI、OpenAI Codexなどのコーディングツールを統合管理し、ローカルプロキシによるマルチアカウントクォータ追跡とワンクリック設定が可能

### [CLIProxyAPI Dashboard](https://github.com/itsmylife44/cliproxyapi-dashboard)

Next.js、React、PostgreSQLで構築されたCLIProxyAPI用のモダンなWebベース管理ダッシュボード。リアルタイムログストリーミング、構造化された設定編集、APIキー管理、Claude/Gemini/Codex向けOAuthプロバイダー統合、使用量分析、コンテナ管理、コンパニオンプラグインによるOpenCodeとの設定同期機能を搭載 - 手動でのYAML編集は不要

### [All API Hub](https://github.com/qixing-jk/all-api-hub)

New API互換リレーサイトアカウントをワンストップで管理するブラウザ拡張機能。残高と使用量のダッシュボード、自動チェックイン、一般的なアプリへのワンクリックキーエクスポート、ページ内API可用性テスト、チャネル/モデルの同期とリダイレクト機能を搭載。Management APIを通じてCLIProxyAPIと統合し、ワンクリックでプロバイダーのインポートと設定同期が可能

### [Shadow AI](https://github.com/HEUDavid/shadow-ai)

Shadow AIは制限された環境向けに特別に設計されたAIアシスタントツールです。ウィンドウや痕跡のないステルス動作モードを提供し、LAN（ローカルエリアネットワーク）を介したクロスデバイスAI質疑応答のインタラクションと制御を可能にします。本質的には「画面/音声キャプチャ + AI推論 + 低摩擦デリバリー」の自動化コラボレーションレイヤーであり、制御されたデバイスや制限された環境でアプリケーション横断的にAIアシスタントを没入的に使用できるようユーザーを支援します。

### [ProxyPal](https://github.com/buddingnewinsights/proxypal)

CLIProxyAPIをネイティブGUIでラップしたクロスプラットフォームデスクトップアプリ（macOS、Windows、Linux）。Claude、ChatGPT、Gemini、GitHub Copilot、カスタムOpenAI互換エンドポイントに対応し、使用状況分析、リクエスト監視、人気コーディングツールの自動設定機能を搭載 - APIキー不要

### [CLIProxyAPI Quota Inspector](https://github.com/AllenReder/CLIProxyAPI-Quota-Inspector)

CLIProxyAPI向けのすぐに使えるクロスプラットフォームのクォータ確認ツール。アカウントごとの codex 5h/7d クォータ表示、プラン別ソート、ステータス色分け、複数アカウントの集計分析に対応。

### [CodexCliPlus](https://github.com/C4AL/CodexCliPlus)

CLIProxyAPIを基盤にしたWindows向けのローカル優先Codex CLIデスクトップ管理プラットフォーム。ローカル設定、アカウント、実行状態の管理を簡素化し、ローカルユーザーにより包括的なCodex CLI体験を提供します。

> [!NOTE]
> CLIProxyAPIをベースにプロジェクトを開発した場合は、PRを送ってこのリストに追加してください。

## その他の選択肢

以下のプロジェクトはCLIProxyAPIの移植版またはそれに触発されたものです：

### [9Router](https://github.com/decolua/9router)

CLIProxyAPIに触発されたNext.js実装。インストールと使用が簡単で、フォーマット変換（OpenAI/Claude/Gemini/Ollama）、自動フォールバック付きコンボシステム、指数バックオフ付きマルチアカウント管理、Next.js Webダッシュボード、CLIツール（Cursor、Claude Code、Cline、RooCode）のサポートをゼロから構築 - APIキー不要

### [OmniRoute](https://github.com/diegosouzapw/OmniRoute)

コーディングを止めない。無料および低コストのAIモデルへのスマートルーティングと自動フォールバック。

OmniRouteはマルチプロバイダーLLM向けのAIゲートウェイです：スマートルーティング、負荷分散、リトライ、フォールバックを備えたOpenAI互換エンドポイント。ポリシー、レート制限、キャッシュ、可観測性を追加して、信頼性が高くコストを意識した推論を実現します。

### [Playful Proxy API Panel (PPAP)](https://github.com/daishuge/playful-proxy-api-panel)

上流に近い使い方を維持する公開CLIProxyAPI互換フォーク兼管理パネルです。内蔵の使用量統計を復元し、キャッシュヒット率、初回バイト待ち時間、TPSの記録、Docker向けのセルフホスト手順を追加しています。

> [!NOTE]
> CLIProxyAPIの移植版またはそれに触発されたプロジェクトを開発した場合は、PRを送ってこのリストに追加してください。

## ライセンス

本プロジェクトはMITライセンスの下でライセンスされています - 詳細は[LICENSE](LICENSE)ファイルを参照してください。

## フレンドリンク

- [V2EX](https://www.v2ex.com/)
- [Linux.do](https://linux.do/)
