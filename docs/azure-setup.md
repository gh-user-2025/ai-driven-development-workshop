# Azure リソースセットアップ手順

## 🚀 概要

この文書では、工場設備管理アプリケーションで使用するAzureリソースの詳細なセットアップ手順を説明します。Azure初心者でも理解できるよう、ステップごとに詳細に記載しています。

## 📋 前提条件

### 必要なツール
- Azure CLI (バージョン 2.30以降)
- 有効なAzureサブスクリプション
- 適切なAzure権限 (リソース作成権限)

### Azure CLIの確認
```bash
# Azure CLIのバージョン確認
az --version

# Azureにログイン
az login

# サブスクリプションの確認
az account list --output table

# 使用するサブスクリプションを設定
az account set --subscription "YOUR_SUBSCRIPTION_ID"
```

## 🏗️ リソースセットアップ手順

### ステップ 1: 基本設定

#### 1.1 環境変数の設定
```bash
# プロジェクト基本設定
export RESOURCE_GROUP="rg-factory-equipment-mgmt"
export LOCATION="japaneast"
export PROJECT_NAME="factory-equipment"
export ENVIRONMENT="dev"  # dev, test, prod

# 一意性を保つためのサフィックス（現在の日時を使用）
export UNIQUE_SUFFIX=$(date +%Y%m%d%H%M)
```

#### 1.2 リソースグループの作成
```bash
# リソースグループの作成
az group create \
    --name $RESOURCE_GROUP \
    --location $LOCATION \
    --tags Environment=$ENVIRONMENT Project=$PROJECT_NAME

# 作成されたリソースグループの確認
az group show --name $RESOURCE_GROUP --output table
```

### ステップ 2: ストレージアカウントの作成

#### 2.1 ストレージアカウントの作成
```bash
# ストレージアカウント名の設定（全世界で一意である必要があります）
export STORAGE_ACCOUNT_NAME="${PROJECT_NAME}storage${UNIQUE_SUFFIX}"

# ストレージアカウントの作成
az storage account create \
    --name $STORAGE_ACCOUNT_NAME \
    --resource-group $RESOURCE_GROUP \
    --location $LOCATION \
    --sku Standard_LRS \
    --kind StorageV2 \
    --access-tier Hot \
    --tags Environment=$ENVIRONMENT Project=$PROJECT_NAME

# ストレージアカウントキーの取得
export STORAGE_KEY=$(az storage account keys list \
    --resource-group $RESOURCE_GROUP \
    --account-name $STORAGE_ACCOUNT_NAME \
    --query '[0].value' \
    --output tsv)

echo "Storage Account: $STORAGE_ACCOUNT_NAME"
echo "Storage Key: $STORAGE_KEY"
```

#### 2.2 Blobコンテナーの作成
```bash
# データ保存用コンテナーの作成
az storage container create \
    --name "sensor-data" \
    --account-name $STORAGE_ACCOUNT_NAME \
    --account-key $STORAGE_KEY \
    --public-access off

az storage container create \
    --name "equipment-images" \
    --account-name $STORAGE_ACCOUNT_NAME \
    --account-key $STORAGE_KEY \
    --public-access off

# 作成されたコンテナーの確認
az storage container list \
    --account-name $STORAGE_ACCOUNT_NAME \
    --account-key $STORAGE_KEY \
    --output table
```

### ステップ 3: Azure Cosmos DB の作成

#### 3.1 Cosmos DB アカウントの作成
```bash
# Cosmos DB アカウント名の設定
export COSMOSDB_ACCOUNT_NAME="${PROJECT_NAME}-cosmosdb-${UNIQUE_SUFFIX}"

# Cosmos DB アカウントの作成（SQL API）
az cosmosdb create \
    --name $COSMOSDB_ACCOUNT_NAME \
    --resource-group $RESOURCE_GROUP \
    --locations regionName=$LOCATION \
    --default-consistency-level Session \
    --enable-automatic-failover true \
    --tags Environment=$ENVIRONMENT Project=$PROJECT_NAME

# 接続文字列の取得
export COSMOSDB_CONNECTION_STRING=$(az cosmosdb keys list \
    --name $COSMOSDB_ACCOUNT_NAME \
    --resource-group $RESOURCE_GROUP \
    --type connection-strings \
    --query 'connectionStrings[0].connectionString' \
    --output tsv)

echo "Cosmos DB Account: $COSMOSDB_ACCOUNT_NAME"
```

#### 3.2 データベースとコンテナーの作成
```bash
# データベースの作成
az cosmosdb sql database create \
    --account-name $COSMOSDB_ACCOUNT_NAME \
    --resource-group $RESOURCE_GROUP \
    --name "EquipmentData" \
    --throughput 400

# IoTセンサーデータ用コンテナーの作成
az cosmosdb sql container create \
    --account-name $COSMOSDB_ACCOUNT_NAME \
    --resource-group $RESOURCE_GROUP \
    --database-name "EquipmentData" \
    --name "SensorData" \
    --partition-key-path "/equipmentId" \
    --throughput 400

# アラートデータ用コンテナーの作成
az cosmosdb sql container create \
    --account-name $COSMOSDB_ACCOUNT_NAME \
    --resource-group $RESOURCE_GROUP \
    --database-name "EquipmentData" \
    --name "Alerts" \
    --partition-key-path "/equipmentId" \
    --throughput 400
```

### ステップ 4: Azure SQL Database の作成

#### 4.1 SQL Server の作成
```bash
# SQL Server名の設定
export SQL_SERVER_NAME="${PROJECT_NAME}-sqlserver-${UNIQUE_SUFFIX}"
export SQL_ADMIN_USER="factory_admin"
export SQL_ADMIN_PASSWORD="Factory@Admin2024!"

# SQL Serverの作成
az sql server create \
    --name $SQL_SERVER_NAME \
    --resource-group $RESOURCE_GROUP \
    --location $LOCATION \
    --admin-user $SQL_ADMIN_USER \
    --admin-password $SQL_ADMIN_PASSWORD \
    --tags Environment=$ENVIRONMENT Project=$PROJECT_NAME

# ファイアウォール規則の設定（Azure内からのアクセスを許可）
az sql server firewall-rule create \
    --resource-group $RESOURCE_GROUP \
    --server $SQL_SERVER_NAME \
    --name "AllowAzureServices" \
    --start-ip-address 0.0.0.0 \
    --end-ip-address 0.0.0.0

echo "SQL Server: $SQL_SERVER_NAME"
echo "SQL Admin User: $SQL_ADMIN_USER"
```

#### 4.2 SQL Database の作成
```bash
# データベース名の設定
export SQL_DATABASE_NAME="EquipmentManagement"

# SQL Databaseの作成
az sql db create \
    --resource-group $RESOURCE_GROUP \
    --server $SQL_SERVER_NAME \
    --name $SQL_DATABASE_NAME \
    --service-objective Basic \
    --tags Environment=$ENVIRONMENT Project=$PROJECT_NAME

# 接続文字列の取得
export SQL_CONNECTION_STRING="Server=tcp:${SQL_SERVER_NAME}.database.windows.net,1433;Initial Catalog=${SQL_DATABASE_NAME};Persist Security Info=False;User ID=${SQL_ADMIN_USER};Password=${SQL_ADMIN_PASSWORD};MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;"

echo "SQL Database: $SQL_DATABASE_NAME"
```

### ステップ 5: Azure IoT Hub の作成

#### 5.1 IoT Hub の作成
```bash
# IoT Hub名の設定
export IOT_HUB_NAME="${PROJECT_NAME}-iothub-${UNIQUE_SUFFIX}"

# IoT Hubの作成
az iot hub create \
    --name $IOT_HUB_NAME \
    --resource-group $RESOURCE_GROUP \
    --location $LOCATION \
    --sku F1 \
    --partition-count 2 \
    --tags Environment=$ENVIRONMENT Project=$PROJECT_NAME

# IoT Hub接続文字列の取得
export IOT_HUB_CONNECTION_STRING=$(az iot hub connection-string show \
    --hub-name $IOT_HUB_NAME \
    --resource-group $RESOURCE_GROUP \
    --query 'connectionString' \
    --output tsv)

echo "IoT Hub: $IOT_HUB_NAME"
```

#### 5.2 IoT デバイスの作成（テスト用）
```bash
# テスト用IoTデバイスの作成
az iot hub device-identity create \
    --hub-name $IOT_HUB_NAME \
    --device-id "equipment-001" \
    --edge-enabled false

az iot hub device-identity create \
    --hub-name $IOT_HUB_NAME \
    --device-id "equipment-002" \
    --edge-enabled false

# デバイス接続文字列の取得
export DEVICE_001_CONNECTION_STRING=$(az iot hub device-identity connection-string show \
    --hub-name $IOT_HUB_NAME \
    --device-id "equipment-001" \
    --query 'connectionString' \
    --output tsv)

echo "Device 001 Connection String: $DEVICE_001_CONNECTION_STRING"
```

### ステップ 6: Azure Functions の準備

#### 6.1 Function App の作成
```bash
# Function App名の設定
export FUNCTION_APP_NAME="${PROJECT_NAME}-functions-${UNIQUE_SUFFIX}"

# Function Appの作成
az functionapp create \
    --resource-group $RESOURCE_GROUP \
    --consumption-plan-location $LOCATION \
    --runtime python \
    --runtime-version 3.9 \
    --functions-version 4 \
    --name $FUNCTION_APP_NAME \
    --storage-account $STORAGE_ACCOUNT_NAME \
    --tags Environment=$ENVIRONMENT Project=$PROJECT_NAME

echo "Function App: $FUNCTION_APP_NAME"
```

#### 6.2 Application Settings の設定
```bash
# 環境変数の設定
az functionapp config appsettings set \
    --name $FUNCTION_APP_NAME \
    --resource-group $RESOURCE_GROUP \
    --settings \
        COSMOSDB_CONNECTION_STRING="$COSMOSDB_CONNECTION_STRING" \
        SQL_CONNECTION_STRING="$SQL_CONNECTION_STRING" \
        IOT_HUB_CONNECTION_STRING="$IOT_HUB_CONNECTION_STRING" \
        STORAGE_CONNECTION_STRING="DefaultEndpointsProtocol=https;AccountName=$STORAGE_ACCOUNT_NAME;AccountKey=$STORAGE_KEY;EndpointSuffix=core.windows.net"
```

### ステップ 7: Azure Monitor と Application Insights

#### 7.1 Application Insights の作成
```bash
# Application Insights名の設定
export APP_INSIGHTS_NAME="${PROJECT_NAME}-insights-${UNIQUE_SUFFIX}"

# Application Insightsの作成
az monitor app-insights component create \
    --app $APP_INSIGHTS_NAME \
    --location $LOCATION \
    --resource-group $RESOURCE_GROUP \
    --tags Environment=$ENVIRONMENT Project=$PROJECT_NAME

# Instrumentation Keyの取得
export APP_INSIGHTS_KEY=$(az monitor app-insights component show \
    --app $APP_INSIGHTS_NAME \
    --resource-group $RESOURCE_GROUP \
    --query 'instrumentationKey' \
    --output tsv)

echo "Application Insights: $APP_INSIGHTS_NAME"
echo "Instrumentation Key: $APP_INSIGHTS_KEY"
```

#### 7.2 Function App にApplication Insights を追加
```bash
# Function App にApplication Insights を接続
az functionapp config appsettings set \
    --name $FUNCTION_APP_NAME \
    --resource-group $RESOURCE_GROUP \
    --settings \
        APPINSIGHTS_INSTRUMENTATIONKEY="$APP_INSIGHTS_KEY" \
        APPLICATIONINSIGHTS_CONNECTION_STRING="InstrumentationKey=$APP_INSIGHTS_KEY"
```

## 📊 リソース確認

### 作成されたリソースの確認
```bash
# リソースグループ内のすべてのリソースを表示
az resource list \
    --resource-group $RESOURCE_GROUP \
    --output table

# コストの確認（課金情報）
az consumption usage list \
    --start-date $(date -d '7 days ago' +%Y-%m-%d) \
    --end-date $(date +%Y-%m-%d) \
    --output table
```

## 🔧 環境変数の保存

### 設定値のエクスポート
```bash
# 重要な接続情報をファイルに保存（機密情報の取り扱いに注意）
cat > azure-config.env << EOF
RESOURCE_GROUP=$RESOURCE_GROUP
LOCATION=$LOCATION
STORAGE_ACCOUNT_NAME=$STORAGE_ACCOUNT_NAME
COSMOSDB_ACCOUNT_NAME=$COSMOSDB_ACCOUNT_NAME
SQL_SERVER_NAME=$SQL_SERVER_NAME
SQL_DATABASE_NAME=$SQL_DATABASE_NAME
IOT_HUB_NAME=$IOT_HUB_NAME
FUNCTION_APP_NAME=$FUNCTION_APP_NAME
APP_INSIGHTS_NAME=$APP_INSIGHTS_NAME

# 接続文字列（機密情報）
COSMOSDB_CONNECTION_STRING="$COSMOSDB_CONNECTION_STRING"
SQL_CONNECTION_STRING="$SQL_CONNECTION_STRING"
IOT_HUB_CONNECTION_STRING="$IOT_HUB_CONNECTION_STRING"
DEVICE_001_CONNECTION_STRING="$DEVICE_001_CONNECTION_STRING"
APP_INSIGHTS_KEY="$APP_INSIGHTS_KEY"
EOF

echo "設定値が azure-config.env に保存されました"
echo "⚠️  このファイルには機密情報が含まれています。セキュリティに注意してください。"
```

## 🗑️ リソースの削除（必要時）

### 全リソースの削除
```bash
# ⚠️ 注意: この操作は元に戻せません
# リソースグループ内のすべてのリソースを削除
az group delete \
    --name $RESOURCE_GROUP \
    --yes \
    --no-wait

echo "リソースグループ $RESOURCE_GROUP の削除を開始しました"
```

## 📝 次のステップ

1. **開発環境のセットアップ**: [development-setup.md](./development-setup.md)を参照
2. **Azure Functions の実装**: functions フォルダーでの開発開始
3. **IoT シミュレーターの作成**: テストデータ生成ツールの実装
4. **フロントエンドの開発**: Vue.js アプリケーションの作成

## 💡 ヒントとベストプラクティス

- **コスト管理**: 開発中は適切なサイズでリソースを作成し、本番移行時にスケールアップ
- **セキュリティ**: 接続文字列やキーはAzure Key Vaultで管理することを推奨
- **監視**: Application Insights でアプリケーションの健全性を常に監視
- **バックアップ**: 重要なデータは定期的にバックアップを実行