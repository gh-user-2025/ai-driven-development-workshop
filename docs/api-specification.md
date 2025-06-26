# API仕様書

## 🚀 概要

この文書では、工場設備管理アプリケーションのREST API仕様を詳細に定義しています。Azure Functionsを使用したサーバーレスアーキテクチャに基づく設計となっています。

## 📋 API設計原則

### RESTful設計
- リソース指向のURL構成
- 適切なHTTPメソッドの使用
- ステータスコードの統一
- JSON形式でのデータ交換

### セキュリティ
- Azure AD認証・認可
- CORS設定
- レート制限
- 入力値検証

### パフォーマンス
- ページネーション対応
- キャッシュ戦略
- 非同期処理
- バッチ操作サポート

## 🔗 ベースURL

```
https://factory-equipment-functions-{unique-suffix}.azurewebsites.net/api
```

## 🔐 認証

### Azure AD Bearer Token
```http
Authorization: Bearer {access_token}
```

### スコープ
- `Equipment.Read`: 設備情報の読み取り
- `Equipment.Write`: 設備情報の更新
- `SensorData.Read`: センサーデータの読み取り
- `Maintenance.Manage`: メンテナンス管理
- `Alert.Manage`: アラート管理

## 📊 共通レスポンス形式

### 成功レスポンス
```json
{
  "success": true,
  "data": { /* レスポンスデータ */ },
  "message": "処理が正常に完了しました",
  "timestamp": "2024-01-01T12:00:00.000Z",
  "requestId": "uuid-v4"
}
```

### エラーレスポンス
```json
{
  "success": false,
  "error": {
    "code": "EQUIPMENT_NOT_FOUND",
    "message": "指定された設備が見つかりません",
    "details": {
      "equipmentId": "equipment-001"
    }
  },
  "timestamp": "2024-01-01T12:00:00.000Z",
  "requestId": "uuid-v4"
}
```

### ページネーションレスポンス
```json
{
  "success": true,
  "data": {
    "items": [ /* データ配列 */ ],
    "pagination": {
      "page": 1,
      "pageSize": 20,
      "totalItems": 150,
      "totalPages": 8,
      "hasNext": true,
      "hasPrevious": false
    }
  }
}
```

## 🏭 設備管理API

### 設備一覧取得
```http
GET /equipment
```

#### クエリパラメータ
| パラメータ | 型 | 必須 | 説明 |
|-----------|---|------|------|
| page | integer | × | ページ番号（デフォルト: 1） |
| pageSize | integer | × | ページサイズ（デフォルト: 20、最大: 100） |
| location | string | × | 設置場所でフィルタ |
| department | string | × | 部署でフィルタ |
| status | string | × | ステータスでフィルタ（Active, Inactive, Maintenance） |
| equipmentType | string | × | 設備タイプでフィルタ |

#### レスポンス例
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "equipmentId": "equipment-001",
        "equipmentName": "生産ライン A1",
        "equipmentType": "Production Line",
        "location": "工場棟A-1F",
        "department": "生産部",
        "status": "Active",
        "lastMaintenanceDate": "2024-01-01",
        "nextMaintenanceDate": "2024-02-01",
        "currentAlertCount": 2,
        "operationHours": 1250.5
      }
    ],
    "pagination": {
      "page": 1,
      "pageSize": 20,
      "totalItems": 45,
      "totalPages": 3
    }
  }
}
```

### 設備詳細取得
```http
GET /equipment/{equipmentId}
```

#### パスパラメータ
| パラメータ | 型 | 説明 |
|-----------|---|------|
| equipmentId | string | 設備ID |

#### レスポンス例
```json
{
  "success": true,
  "data": {
    "equipmentId": "equipment-001",
    "equipmentName": "生産ライン A1",
    "equipmentType": "Production Line",
    "manufacturer": "株式会社ABC機械",
    "modelNumber": "PL-2024",
    "serialNumber": "PL2024001",
    "installationDate": "2024-01-15",
    "location": "工場棟A-1F",
    "department": "生産部",
    "status": "Active",
    "specifications": {
      "maxCapacity": 1000,
      "powerConsumption": 50,
      "operatingTemperature": {
        "min": 10,
        "max": 40
      }
    },
    "sensors": [
      {
        "sensorId": "temp-001",
        "sensorType": "Temperature",
        "unit": "celsius",
        "currentValue": 32.5,
        "lastUpdated": "2024-01-01T12:00:00.000Z",
        "status": "Normal"
      }
    ]
  }
}
```

### 設備ステータス更新
```http
PATCH /equipment/{equipmentId}/status
```

#### リクエストボディ
```json
{
  "status": "Maintenance",
  "reason": "定期メンテナンスのため",
  "estimatedDowntime": 240
}
```

## 📊 センサーデータAPI

### リアルタイムセンサーデータ取得
```http
GET /equipment/{equipmentId}/sensors/realtime
```

#### クエリパラメータ
| パラメータ | 型 | 必須 | 説明 |
|-----------|---|------|------|
| sensorTypes | string[] | × | 取得するセンサータイプ（Temperature, Vibration, Pressure） |
| latest | boolean | × | 最新値のみ取得（デフォルト: true） |

#### レスポンス例
```json
{
  "success": true,
  "data": {
    "equipmentId": "equipment-001",
    "timestamp": "2024-01-01T12:00:00.000Z",
    "sensors": [
      {
        "sensorId": "temp-001",
        "sensorType": "Temperature",
        "value": 32.5,
        "unit": "celsius",
        "quality": "Good",
        "timestamp": "2024-01-01T12:00:00.000Z"
      },
      {
        "sensorId": "vibration-001",
        "sensorType": "Vibration",
        "value": 3.2,
        "unit": "g",
        "quality": "Good",
        "timestamp": "2024-01-01T12:00:00.000Z"
      }
    ]
  }
}
```

### 履歴センサーデータ取得
```http
GET /equipment/{equipmentId}/sensors/history
```

#### クエリパラメータ
| パラメータ | 型 | 必須 | 説明 |
|-----------|---|------|------|
| startDate | string(ISO8601) | ○ | 開始日時 |
| endDate | string(ISO8601) | ○ | 終了日時 |
| sensorType | string | × | センサータイプ |
| aggregation | string | × | 集計タイプ（raw, hourly, daily） |
| page | integer | × | ページ番号 |
| pageSize | integer | × | ページサイズ |

#### レスポンス例
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "timestamp": "2024-01-01T12:00:00.000Z",
        "sensorType": "Temperature",
        "avgValue": 32.5,
        "minValue": 30.2,
        "maxValue": 35.1,
        "sampleCount": 60
      }
    ],
    "pagination": {
      "page": 1,
      "pageSize": 100,
      "totalItems": 2400
    }
  }
}
```

### センサーデータ一括投稿
```http
POST /sensors/data/batch
```

#### リクエストボディ
```json
{
  "deviceId": "iot-device-001",
  "timestamp": "2024-01-01T12:00:00.000Z",
  "data": [
    {
      "equipmentId": "equipment-001",
      "sensorId": "temp-001",
      "sensorType": "Temperature",
      "value": 32.5,
      "unit": "celsius",
      "quality": "Good"
    },
    {
      "equipmentId": "equipment-001", 
      "sensorId": "vibration-001",
      "sensorType": "Vibration",
      "value": 3.2,
      "unit": "g",
      "quality": "Good"
    }
  ]
}
```

## 🚨 アラート管理API

### アクティブアラート一覧取得
```http
GET /alerts
```

#### クエリパラメータ
| パラメータ | 型 | 必須 | 説明 |
|-----------|---|------|------|
| severity | string | × | 重要度フィルタ（Info, Warning, Critical） |
| status | string | × | ステータスフィルタ（Active, Acknowledged, Resolved） |
| equipmentId | string | × | 設備IDフィルタ |
| startDate | string(ISO8601) | × | 開始日時 |
| endDate | string(ISO8601) | × | 終了日時 |

#### レスポンス例
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "alertId": "alert-20240101120500-001",
        "equipmentId": "equipment-001",
        "equipmentName": "生産ライン A1",
        "alertType": "Temperature",
        "severity": "Warning",
        "status": "Active",
        "timestamp": "2024-01-01T12:05:00.000Z",
        "description": "設備温度が警告閾値を超過しました",
        "currentValue": 37.2,
        "thresholdValue": 35.0,
        "unit": "celsius",
        "duration": "00:05:30",
        "acknowledgedBy": null
      }
    ]
  }
}
```

### アラート詳細取得
```http
GET /alerts/{alertId}
```

#### レスポンス例
```json
{
  "success": true,
  "data": {
    "alertId": "alert-20240101120500-001",
    "equipmentId": "equipment-001",
    "timestamp": "2024-01-01T12:05:00.000Z",
    "alertType": "Temperature",
    "severity": "Warning",
    "status": "Active",
    "description": "設備温度が警告閾値を超過しました",
    "currentValue": 37.2,
    "threshold": {
      "type": "UpperLimit",
      "value": 35.0,
      "unit": "celsius"
    },
    "recommendations": [
      "冷却システムの動作確認",
      "環境温度の確認",
      "負荷状況の点検"
    ],
    "relatedData": {
      "sensorId": "temp-001",
      "recentValues": [35.1, 36.2, 37.2],
      "trend": "increasing"
    }
  }
}
```

### アラート確認
```http
POST /alerts/{alertId}/acknowledge
```

#### リクエストボディ
```json
{
  "acknowledgedBy": "user@company.com",
  "comment": "現地確認を実施します"
}
```

### アラート解決
```http
POST /alerts/{alertId}/resolve
```

#### リクエストボディ
```json
{
  "resolvedBy": "user@company.com",
  "resolutionComment": "冷却システムの清掃により正常値に復帰",
  "actionTaken": "清掃作業実施"
}
```

## 🔧 メンテナンス管理API

### メンテナンススケジュール一覧取得
```http
GET /maintenance/schedules
```

#### クエリパラメータ
| パラメータ | 型 | 必須 | 説明 |
|-----------|---|------|------|
| equipmentId | string | × | 設備IDフィルタ |
| startDate | string(ISO8601) | × | 開始日時 |
| endDate | string(ISO8601) | × | 終了日時 |
| status | string | × | ステータスフィルタ |
| priority | string | × | 優先度フィルタ |

#### レスポンス例
```json
{
  "success": true,
  "data": {
    "items": [
      {
        "id": 1001,
        "equipmentId": "equipment-001",
        "equipmentName": "生産ライン A1",
        "maintenanceType": "定期点検",
        "scheduledDate": "2024-02-01T09:00:00.000Z",
        "estimatedDurationHours": 4.0,
        "priority": "Medium",
        "status": "Scheduled",
        "assignedTechnician": "田中太郎",
        "description": "四半期定期点検",
        "requiredParts": ["フィルター x2", "オイル 5L"]
      }
    ]
  }
}
```

### メンテナンススケジュール作成
```http
POST /maintenance/schedules
```

#### リクエストボディ
```json
{
  "equipmentId": "equipment-001",
  "maintenanceType": "定期点検",
  "scheduledDate": "2024-02-01T09:00:00.000Z",
  "estimatedDurationHours": 4.0,
  "priority": "Medium",
  "assignedTechnician": "田中太郎",
  "description": "四半期定期点検",
  "requiredParts": ["フィルター x2", "オイル 5L"],
  "specialInstructions": "安全装具着用必須"
}
```

### メンテナンス実績記録
```http
POST /maintenance/schedules/{scheduleId}/complete
```

#### リクエストボディ
```json
{
  "startDateTime": "2024-02-01T09:00:00.000Z",
  "endDateTime": "2024-02-01T13:30:00.000Z",
  "technicianName": "田中太郎",
  "workDescription": "定期点検を実施。全項目正常を確認。",
  "partsUsed": ["フィルター x2", "オイル 5L"],
  "totalCost": 15000.00,
  "nextMaintenanceDate": "2024-05-01T09:00:00.000Z",
  "findings": [
    {
      "item": "ベルト",
      "condition": "良好",
      "notes": "摩耗なし"
    },
    {
      "item": "温度センサー",
      "condition": "交換推奨",
      "notes": "次回メンテナンス時に交換"
    }
  ]
}
```

## 📊 レポート・分析API

### 設備稼働率レポート
```http
GET /reports/equipment-utilization
```

#### クエリパラメータ
| パラメータ | 型 | 必須 | 説明 |
|-----------|---|------|------|
| equipmentId | string | × | 設備IDフィルタ |
| startDate | string(ISO8601) | ○ | 開始日時 |
| endDate | string(ISO8601) | ○ | 終了日時 |
| groupBy | string | × | グループ化（hour, day, week, month） |

#### レスポンス例
```json
{
  "success": true,
  "data": {
    "reportPeriod": {
      "startDate": "2024-01-01T00:00:00.000Z",
      "endDate": "2024-01-31T23:59:59.000Z"
    },
    "summary": {
      "totalEquipment": 10,
      "averageUtilization": 85.5,
      "totalDowntime": 120.5,
      "plannedDowntime": 80.0,
      "unplannedDowntime": 40.5
    },
    "equipmentData": [
      {
        "equipmentId": "equipment-001",
        "equipmentName": "生産ライン A1",
        "utilizationRate": 88.5,
        "operatingHours": 636.0,
        "downtime": 12.0,
        "maintenanceHours": 8.0,
        "alertCount": 15
      }
    ]
  }
}
```

### アラート統計レポート
```http
GET /reports/alert-statistics
```

#### レスポンス例
```json
{
  "success": true,
  "data": {
    "summary": {
      "totalAlerts": 245,
      "criticalAlerts": 12,
      "warningAlerts": 156,
      "infoAlerts": 77,
      "averageResponseTime": 15.5,
      "averageResolutionTime": 45.2
    },
    "alertsByType": [
      {
        "alertType": "Temperature",
        "count": 89,
        "avgResolutionTime": 32.1
      },
      {
        "alertType": "Vibration",
        "count": 67,
        "avgResolutionTime": 28.7
      }
    ],
    "trend": [
      {
        "date": "2024-01-01",
        "totalAlerts": 8,
        "criticalAlerts": 1
      }
    ]
  }
}
```

## 🔄 リアルタイムAPI（WebSocket）

### WebSocket接続
```
wss://factory-equipment-functions-{unique-suffix}.azurewebsites.net/api/realtime
```

### 認証
```json
{
  "type": "auth",
  "token": "bearer_token_here"
}
```

### サブスクリプション
```json
{
  "type": "subscribe",
  "topics": [
    "equipment.equipment-001.sensors",
    "alerts.critical",
    "maintenance.schedules"
  ]
}
```

### リアルタイムデータ受信
```json
{
  "type": "sensor-data",
  "equipmentId": "equipment-001",
  "timestamp": "2024-01-01T12:00:00.000Z",
  "data": {
    "sensorId": "temp-001",
    "sensorType": "Temperature",
    "value": 32.5,
    "unit": "celsius"
  }
}
```

## 🚫 エラーコード一覧

| エラーコード | HTTPステータス | 説明 |
|-------------|---------------|------|
| EQUIPMENT_NOT_FOUND | 404 | 設備が見つかりません |
| INVALID_SENSOR_DATA | 400 | センサーデータが無効です |
| ALERT_ALREADY_ACKNOWLEDGED | 409 | アラートは既に確認済みです |
| MAINTENANCE_CONFLICT | 409 | メンテナンススケジュールが重複しています |
| AUTHORIZATION_REQUIRED | 401 | 認証が必要です |
| INSUFFICIENT_PERMISSIONS | 403 | 権限が不足しています |
| RATE_LIMIT_EXCEEDED | 429 | レート制限を超過しました |
| INTERNAL_SERVER_ERROR | 500 | 内部サーバーエラー |

## 🔧 API使用例

### Python（requests）
```python
import requests
import json

# 認証トークンの取得（Azure AD）
headers = {
    'Authorization': 'Bearer {access_token}',
    'Content-Type': 'application/json'
}

# 設備一覧取得
response = requests.get(
    'https://factory-equipment-functions.azurewebsites.net/api/equipment',
    headers=headers,
    params={'status': 'Active', 'page': 1}
)

equipment_list = response.json()
```

### JavaScript（axios）
```javascript
const axios = require('axios');

const apiClient = axios.create({
  baseURL: 'https://factory-equipment-functions.azurewebsites.net/api',
  headers: {
    'Authorization': `Bearer ${accessToken}`,
    'Content-Type': 'application/json'
  }
});

// リアルタイムセンサーデータ取得
const getSensorData = async (equipmentId) => {
  try {
    const response = await apiClient.get(`/equipment/${equipmentId}/sensors/realtime`);
    return response.data;
  } catch (error) {
    console.error('API Error:', error.response.data);
  }
};
```

## 📝 次のステップ

1. **Azure Functions実装**: 各APIエンドポイントの実装
2. **認証・認可の実装**: Azure AD統合
3. **データアクセス層の実装**: データベース接続
4. **バリデーション実装**: 入力データ検証
5. **テスト実装**: 単体テスト・統合テスト
6. **API ドキュメント生成**: OpenAPI仕様書の作成