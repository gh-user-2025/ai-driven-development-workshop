# 開発環境セットアップ手順

## 🖥️ 概要

この文書では、工場設備管理アプリケーションの開発を行うための開発環境セットアップ手順を詳細に説明します。

## 📋 前提条件

### システム要件
- **OS**: Windows 10/11、macOS 10.15+、または Ubuntu 18.04+
- **メモリ**: 8GB以上推奨
- **ストレージ**: 10GB以上の空き容量
- **ネットワーク**: インターネット接続

### 必要なソフトウェア
- Visual Studio Code
- Node.js (v16以降)
- Python (v3.9以降)
- Azure CLI
- Git

## 🚀 セットアップ手順

### ステップ 1: 基本ツールのインストール

#### 1.1 Visual Studio Code のインストール
```bash
# Windows (winget利用)
winget install Microsoft.VisualStudioCode

# macOS (Homebrew利用)
brew install --cask visual-studio-code

# Ubuntu/Debian
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg
sudo install -o root -g root -m 644 packages.microsoft.gpg /etc/apt/trusted.gpg.d/
sudo sh -c 'echo "deb [arch=amd64,arm64,armhf signed-by=/etc/apt/trusted.gpg.d/packages.microsoft.gpg] https://packages.microsoft.com/repos/code stable main" > /etc/apt/sources.list.d/vscode.list'
sudo apt update
sudo apt install code
```

#### 1.2 Node.js のインストール
```bash
# Node.jsバージョン管理ツール（nvm）のインストール
# Windows
winget install CoreyButler.NVMforWindows

# macOS/Linux
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# nvm経由でNode.js最新LTSをインストール
nvm install --lts
nvm use --lts

# インストール確認
node --version
npm --version
```

#### 1.3 Python のインストール
```bash
# Windows
winget install Python.Python.3.11

# macOS
brew install python@3.11

# Ubuntu/Debian
sudo apt update
sudo apt install python3.11 python3.11-pip python3.11-venv

# インストール確認
python3 --version
pip3 --version
```

#### 1.4 Azure CLI のインストール
```bash
# Windows
winget install Microsoft.AzureCLI

# macOS
brew install azure-cli

# Ubuntu/Debian
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash

# インストール確認
az --version
```

#### 1.5 Git のインストール
```bash
# Windows
winget install Git.Git

# macOS
brew install git

# Ubuntu/Debian
sudo apt install git

# Git設定
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

### ステップ 2: Visual Studio Code 拡張機能のインストール

#### 2.1 必須拡張機能
```bash
# VS Code拡張機能のインストール
code --install-extension ms-vscode.vscode-json
code --install-extension ms-python.python
code --install-extension ms-vscode.azure-account
code --install-extension ms-azuretools.vscode-azurefunctions
code --install-extension ms-azuretools.vscode-cosmosdb
code --install-extension ms-mssql.mssql
code --install-extension ms-vscode.vscode-node-azure-pack
code --install-extension Vue.volar
code --install-extension bradlc.vscode-tailwindcss
code --install-extension esbenp.prettier-vscode
code --install-extension ms-vscode.vscode-typescript-next
```

#### 2.2 推奨拡張機能
```bash
# 開発効率向上のための拡張機能
code --install-extension GitHub.copilot
code --install-extension ms-vscode.vscode-github-workflow
code --install-extension humao.rest-client
code --install-extension ms-vscode.vscode-thunder-client
code --install-extension formulahendry.auto-rename-tag
code --install-extension ms-vscode.vscode-eslint
```

### ステップ 3: プロジェクトクローンと初期設定

#### 3.1 プロジェクトリポジトリのクローン
```bash
# プロジェクトディレクトリの作成
mkdir -p ~/projects
cd ~/projects

# リポジトリのクローン（実際のリポジトリURLに置き換え）
git clone https://github.com/your-org/factory-equipment-management.git
cd factory-equipment-management

# プロジェクト構造の作成
mkdir -p {functions,web-app,data-models,deployment,tests,scripts}
```

#### 3.2 Python 仮想環境の設定
```bash
# Python仮想環境の作成
cd functions
python3 -m venv venv

# 仮想環境の有効化
# Windows
venv\Scripts\activate

# macOS/Linux
source venv/bin/activate

# 必要なPythonパッケージのインストール
pip install --upgrade pip
pip install azure-functions
pip install azure-cosmos
pip install azure-storage-blob
pip install azure-iot-device
pip install azure-iot-hub
pip install pyodbc
pip install pandas
pip install numpy
pip install requests
pip install python-dotenv

# requirements.txt の作成
pip freeze > requirements.txt
```

#### 3.3 Node.js プロジェクトの初期化
```bash
# Vue.jsプロジェクトの作成
cd ../web-app
npm init vue@latest factory-equipment-dashboard -- --typescript --pwa --router --pinia --vitest --cypress

cd factory-equipment-dashboard
npm install

# 追加ライブラリのインストール
npm install @azure/msal-browser
npm install chart.js vue-chartjs
npm install @vueuse/core
npm install axios
npm install date-fns
npm install @headlessui/vue @heroicons/vue
npm install tailwindcss @tailwindcss/forms @tailwindcss/typography

# 開発用ライブラリのインストール
npm install --save-dev @types/node
npm install --save-dev autoprefixer postcss
```

### ステップ 4: 環境設定ファイルの作成

#### 4.1 Azure設定ファイルの準備
```bash
# ルートディレクトリに戻る
cd ~/projects/factory-equipment-management

# 環境変数設定ファイルの作成
cat > .env.local << 'EOF'
# Azure リソース設定
AZURE_RESOURCE_GROUP=rg-factory-equipment-mgmt
AZURE_LOCATION=japaneast
AZURE_SUBSCRIPTION_ID=your-subscription-id

# Cosmos DB設定
COSMOSDB_ACCOUNT_NAME=factory-equipment-cosmosdb
COSMOSDB_DATABASE_NAME=EquipmentData
COSMOSDB_CONNECTION_STRING=your-cosmosdb-connection-string

# SQL Database設定
SQL_SERVER_NAME=factory-equipment-sqlserver
SQL_DATABASE_NAME=EquipmentManagement
SQL_CONNECTION_STRING=your-sql-connection-string

# IoT Hub設定
IOT_HUB_NAME=factory-equipment-iothub
IOT_HUB_CONNECTION_STRING=your-iothub-connection-string

# Storage Account設定
STORAGE_ACCOUNT_NAME=factoryequipmentstorage
STORAGE_CONNECTION_STRING=your-storage-connection-string

# Application Insights設定
APP_INSIGHTS_INSTRUMENTATION_KEY=your-app-insights-key
EOF

# .envファイルをgitignoreに追加
echo ".env.local" >> .gitignore
echo "azure-config.env" >> .gitignore
```

#### 4.2 VS Code ワークスペース設定
```bash
# .vscode ディレクトリの作成
mkdir -p .vscode

# VS Code設定ファイルの作成
cat > .vscode/settings.json << 'EOF'
{
    "python.defaultInterpreterPath": "./functions/venv/bin/python",
    "python.terminal.activateEnvironment": true,
    "python.linting.enabled": true,
    "python.linting.pylintEnabled": true,
    "typescript.preferences.importModuleSpecifier": "relative",
    "editor.formatOnSave": true,
    "editor.codeActionsOnSave": {
        "source.fixAll.eslint": true
    },
    "files.associations": {
        "*.vue": "vue"
    },
    "azure.resourceFilter": [
        "your-subscription-id"
    ]
}
EOF

# タスク設定ファイルの作成
cat > .vscode/tasks.json << 'EOF'
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "Install Python Dependencies",
            "type": "shell",
            "command": "pip",
            "args": ["install", "-r", "requirements.txt"],
            "options": {
                "cwd": "${workspaceFolder}/functions"
            },
            "group": "build"
        },
        {
            "label": "Install Node Dependencies",
            "type": "shell",
            "command": "npm",
            "args": ["install"],
            "options": {
                "cwd": "${workspaceFolder}/web-app/factory-equipment-dashboard"
            },
            "group": "build"
        },
        {
            "label": "Start Vue Dev Server",
            "type": "shell",
            "command": "npm",
            "args": ["run", "dev"],
            "options": {
                "cwd": "${workspaceFolder}/web-app/factory-equipment-dashboard"
            },
            "group": "test",
            "isBackground": true
        },
        {
            "label": "Start Azure Functions",
            "type": "shell",
            "command": "func",
            "args": ["start"],
            "options": {
                "cwd": "${workspaceFolder}/functions"
            },
            "group": "test",
            "isBackground": true
        }
    ]
}
EOF
```

### ステップ 5: Azure Functions Core Tools のインストール

#### 5.1 Azure Functions Core Tools
```bash
# Windows
winget install Microsoft.AzureFunctionsCoreTools

# macOS
brew tap azure/functions
brew install azure-functions-core-tools@4

# Ubuntu/Debian
curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
sudo mv microsoft.gpg /etc/apt/trusted.gpg.d/microsoft.gpg
sudo sh -c 'echo "deb [arch=amd64] https://packages.microsoft.com/repos/microsoft-ubuntu-$(lsb_release -cs)-prod $(lsb_release -cs) main" > /etc/apt/sources.list.d/dotnetdev.list'
sudo apt-get update
sudo apt-get install azure-functions-core-tools-4

# インストール確認
func --version
```

### ステップ 6: 開発用スクリプトの作成

#### 6.1 開発環境起動スクリプト
```bash
# 開発環境起動スクリプトの作成
cat > scripts/start-dev.sh << 'EOF'
#!/bin/bash

echo "🚀 Factory Equipment Management - 開発環境起動"

# Azure環境変数の読み込み
if [ -f .env.local ]; then
    echo "📄 環境変数を読み込み中..."
    source .env.local
fi

# Python仮想環境の有効化
echo "🐍 Python仮想環境を有効化中..."
cd functions
source venv/bin/activate
cd ..

# バックエンド（Azure Functions）の起動
echo "⚡ Azure Functions を起動中..."
cd functions
func start --port 7071 &
FUNCTIONS_PID=$!
cd ..

# フロントエンド（Vue.js）の起動
echo "🌐 Vue.js開発サーバーを起動中..."
cd web-app/factory-equipment-dashboard
npm run dev &
VUE_PID=$!
cd ../..

echo "✅ 開発環境が起動しました！"
echo "   📊 ダッシュボード: http://localhost:5173"
echo "   ⚡ API: http://localhost:7071"
echo ""
echo "停止するには Ctrl+C を押してください"

# シグナルハンドリング
trap 'echo "🛑 開発環境を停止中..."; kill $FUNCTIONS_PID $VUE_PID; exit 0' INT

wait
EOF

chmod +x scripts/start-dev.sh
```

#### 6.2 Azure接続テストスクリプト
```bash
# Azure接続テストスクリプトの作成
cat > scripts/test-azure-connection.sh << 'EOF'
#!/bin/bash

echo "🔍 Azure接続テスト"

# Azure CLI ログイン確認
echo "1. Azure CLI認証状況..."
az account show --output table

# リソースグループの確認
echo "2. リソースグループの確認..."
az group list --query "[?name=='rg-factory-equipment-mgmt']" --output table

# Cosmos DBの接続テスト
echo "3. Cosmos DB接続テスト..."
az cosmosdb list --resource-group rg-factory-equipment-mgmt --output table

# SQL Databaseの接続テスト
echo "4. SQL Database接続テスト..."
az sql db list --resource-group rg-factory-equipment-mgmt --server factory-equipment-sqlserver --output table

# IoT Hubの確認
echo "5. IoT Hub接続テスト..."
az iot hub list --resource-group rg-factory-equipment-mgmt --output table

echo "✅ Azure接続テスト完了"
EOF

chmod +x scripts/test-azure-connection.sh
```

### ステップ 7: 開発環境の動作確認

#### 7.1 Azure接続の確認
```bash
# Azureにログイン
az login

# 接続テストの実行
./scripts/test-azure-connection.sh
```

#### 7.2 開発環境の起動テスト
```bash
# 開発環境の起動
./scripts/start-dev.sh

# 別ターミナルで動作確認
curl http://localhost:7071/api/health
curl http://localhost:5173
```

## 🔧 トラブルシューティング

### よくある問題と解決方法

#### Python仮想環境の問題
```bash
# 仮想環境の削除と再作成
rm -rf functions/venv
cd functions
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

#### Node.js依存関係の問題
```bash
# node_modulesの削除と再インストール
cd web-app/factory-equipment-dashboard
rm -rf node_modules package-lock.json
npm install
```

#### Azure CLI認証の問題
```bash
# Azure CLIの再認証
az logout
az login --use-device-code
```

## 📝 次のステップ

1. **データモデルの設計**: [data-model.md](./data-model.md)を参照
2. **API仕様の設計**: [api-specification.md](./api-specification.md)を参照
3. **フロントエンド開発の開始**: Vue.jsコンポーネントの実装
4. **Azure Functions開発の開始**: データ処理ロジックの実装

## 💡 開発のヒント

- **ホットリロード**: Vue.jsとAzure Functionsの両方でホットリロードが有効
- **デバッグ**: VS CodeでPythonとTypeScriptの両方をデバッグ可能
- **テスト**: 単体テストと統合テストの環境が整備済み
- **コード品質**: ESLintとPylintによる自動コード品質チェック