# 工場設備管理アプリ プロトタイプ開発プロジェクト

## 🏭 プロジェクト概要

工場内の設備の稼働状況やメンテナンス情報を一元管理し、効率的な運用をサポートする設備管理アプリケーションのプロトタイプ開発プロジェクトです。

### 🎯 主要機能
- **リアルタイム監視**: IoTセンサーを活用した設備稼働状況の監視
- **予防保全管理**: データ分析に基づくメンテナンススケジュールの最適化
- **異常検知**: 機械学習による設備故障の予兆検知
- **データ可視化**: Power BIを活用したレポート生成とダッシュボード

### 🛠️ 技術スタック
- **フロントエンド**: Vue.js 3.x
- **バックエンド**: Azure Functions (Python 3.9+)
- **データベース**: Azure Cosmos DB, Azure SQL Database
- **IoTプラットフォーム**: Azure IoT Hub
- **可視化**: Power BI
- **監視**: Azure Monitor, Application Insights

## 📚 ドキュメント

### 設計ドキュメント
- **[プロジェクト概要](./docs/project-overview.md)** - プロジェクトの詳細説明と要件定義
- **[データモデル設計](./docs/data-model.md)** - データベース設計とデータフロー
- **[API仕様書](./docs/api-specification.md)** - REST API詳細仕様

### セットアップガイド
- **[Azureリソースセットアップ](./docs/azure-setup.md)** - Azureリソースの作成手順
- **[開発環境セットアップ](./docs/development-setup.md)** - ローカル開発環境構築手順

## 🚀 クイックスタート

### 1. 前提条件
- Azure サブスクリプション
- Azure CLI がインストール済み
- Node.js 16+ がインストール済み
- Python 3.9+ がインストール済み

### 2. Azureリソースのセットアップ
```bash
# Azure CLIでログイン
az login

# リソース作成スクリプトの実行
./scripts/setup-azure-resources.sh
```

### 3. 開発環境のセットアップ
```bash
# リポジトリのクローン
git clone https://github.com/your-org/factory-equipment-management.git
cd factory-equipment-management

# 開発環境の起動
./scripts/start-dev.sh
```

### 4. アプリケーションへのアクセス
- **ダッシュボード**: http://localhost:5173
- **API**: http://localhost:7071

## 🏗️ プロジェクト構成

```
factory-equipment-management/
├── docs/                           # プロジェクトドキュメント
├── azure-resources/                # Azureリソース定義ファイル
├── functions/                      # Azure Functions
│   ├── data-ingestion/            # データ取り込み関数
│   ├── data-processing/           # データ処理関数
│   └── alert-management/          # アラート管理関数
├── web-app/                       # Vue.js フロントエンド
├── data-models/                   # データモデル定義
├── deployment/                    # デプロイメント設定
├── scripts/                       # セットアップ・管理スクリプト
└── tests/                         # テストファイル
```

## 📊 主要な機能

### リアルタイムダッシュボード
- 設備稼働状況の可視化
- センサーデータのリアルタイム表示
- アラート通知と対応状況

### メンテナンス管理
- 予防保全スケジュールの管理
- メンテナンス履歴の記録
- 部品交換とコスト管理

### データ分析
- 設備稼働率の分析
- 故障予兆の検知
- 運用改善提案の生成

## 🔒 セキュリティ

- Azure Active Directory による認証・認可
- データ暗号化（保存時・転送時）
- ネットワークセキュリティグループによるアクセス制御
- 監査ログの記録と保持

## 📈 監視とメトリクス

- Application Insights によるアプリケーション監視
- Azure Monitor によるインフラ監視
- リアルタイムアラート機能
- パフォーマンスメトリクス収集

## 🤝 開発への参加

### 開発フロー
1. Issue の作成または確認
2. Feature ブランチの作成
3. 実装とテスト
4. Pull Request の作成
5. コードレビューとマージ

### コーディング規約
- **Python**: PEP 8準拠
- **TypeScript/Vue.js**: ESLint + Prettier設定
- **コミットメッセージ**: Conventional Commits形式

## 📄 ライセンス

このプロジェクトは MIT ライセンスの下で公開されています。詳細は [LICENSE](./LICENSE) ファイルを参照してください。

## 📞 サポート

プロジェクトに関する質問や問題がある場合は、以下の方法でお問い合わせください：

- **Issues**: GitHub Issues で問題報告や機能要求
- **Discussions**: 一般的な質問や議論
- **Email**: project-team@company.com

---

## AI-Driven Development Workshop 
このプロジェクトは AI-Driven Development のアプローチを採用しています。
👉 [Workshop Link](https://dev-lab-io.github.io/aoai/scenario2/home)
