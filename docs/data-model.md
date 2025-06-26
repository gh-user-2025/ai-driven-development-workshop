# データモデル設計

## 📊 概要

この文書では、工場設備管理アプリケーションで使用するデータモデルの詳細設計を記載しています。システム全体のデータ構造、リレーション、および最適化戦略を含みます。

## 🏗️ データアーキテクチャ

### データストレージ戦略

#### Azure Cosmos DB（NoSQL）
- **用途**: リアルタイムIoTセンサーデータ、アラート情報
- **利点**: 高スループット、低レイテンシ、自動スケーリング
- **パーティション戦略**: equipmentId によるパーティション分割

#### Azure SQL Database（RDBMS）  
- **用途**: マスターデータ、履歴データ、レポート用集計データ
- **利点**: ACID準拠、複雑なクエリ、レポーティング機能
- **最適化**: インデックス戦略、パーティション テーブル

## 📋 エンティティ設計

### 1. 設備マスター（Equipment Master）

#### SQL Database: Equipment テーブル
```sql
-- 設備マスターテーブル
CREATE TABLE Equipment (
    EquipmentId NVARCHAR(50) PRIMARY KEY,
    EquipmentName NVARCHAR(200) NOT NULL,
    EquipmentType NVARCHAR(100) NOT NULL,
    Manufacturer NVARCHAR(100),
    ModelNumber NVARCHAR(100),
    SerialNumber NVARCHAR(100),
    InstallationDate DATE,
    Location NVARCHAR(200),
    Department NVARCHAR(100),
    Status NVARCHAR(20) CHECK (Status IN ('Active', 'Inactive', 'Maintenance', 'Decommissioned')),
    CreatedAt DATETIME2 DEFAULT GETDATE(),
    UpdatedAt DATETIME2 DEFAULT GETDATE(),
    INDEX IX_Equipment_Location (Location),
    INDEX IX_Equipment_Department (Department),
    INDEX IX_Equipment_Status (Status)
);
```

#### JSON スキーマ（Cosmos DB用）
```json
{
  "id": "equipment-001",
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
      "thresholds": {
        "warning": 35,
        "critical": 40
      }
    },
    {
      "sensorId": "vibration-001", 
      "sensorType": "Vibration",
      "unit": "g",
      "thresholds": {
        "warning": 5.0,
        "critical": 10.0
      }
    }
  ],
  "_ts": 1640995200
}
```

### 2. センサーデータ（Sensor Data）

#### Cosmos DB: SensorData コンテナ
```json
{
  "id": "sensor-data-20240101120000-001",
  "equipmentId": "equipment-001",
  "sensorId": "temp-001",
  "timestamp": "2024-01-01T12:00:00.000Z",
  "sensorType": "Temperature",
  "value": 32.5,
  "unit": "celsius",
  "quality": "Good",
  "location": {
    "latitude": 35.6762,
    "longitude": 139.6503
  },
  "metadata": {
    "deviceId": "iot-device-001",
    "firmwareVersion": "1.2.3",
    "batteryLevel": 85
  },
  "ttl": 2592000,
  "_ts": 1640995200
}
```

#### SQL Database: SensorDataHistory テーブル（集計用）
```sql
-- センサーデータ履歴テーブル（時間単位集計）
CREATE TABLE SensorDataHistory (
    Id BIGINT IDENTITY(1,1) PRIMARY KEY,
    EquipmentId NVARCHAR(50) NOT NULL,
    SensorId NVARCHAR(50) NOT NULL,
    HourTimestamp DATETIME2 NOT NULL,
    SensorType NVARCHAR(50) NOT NULL,
    AvgValue DECIMAL(10,3),
    MinValue DECIMAL(10,3),
    MaxValue DECIMAL(10,3),
    SampleCount INT,
    CreatedAt DATETIME2 DEFAULT GETDATE(),
    FOREIGN KEY (EquipmentId) REFERENCES Equipment(EquipmentId),
    INDEX IX_SensorDataHistory_Equipment_Time (EquipmentId, HourTimestamp),
    INDEX IX_SensorDataHistory_Sensor_Time (SensorId, HourTimestamp)
);

-- パーティション化（月単位）
ALTER TABLE SensorDataHistory
ADD PARTITION BY RANGE (MONTH(HourTimestamp));
```

### 3. アラート管理（Alert Management）

#### Cosmos DB: Alerts コンテナ
```json
{
  "id": "alert-20240101120500-001",
  "equipmentId": "equipment-001",
  "alertId": "alert-20240101120500-001",
  "timestamp": "2024-01-01T12:05:00.000Z",
  "alertType": "Temperature",
  "severity": "Warning",
  "status": "Active",
  "threshold": {
    "type": "UpperLimit",
    "value": 35.0,
    "unit": "celsius"
  },
  "currentValue": 37.2,
  "sensorId": "temp-001",
  "description": "設備温度が警告閾値を超過しました",
  "recommendations": [
    "冷却システムの動作確認",
    "環境温度の確認",
    "負荷状況の点検"
  ],
  "acknowledgedBy": null,
  "acknowledgedAt": null,
  "resolvedAt": null,
  "escalationLevel": 1,
  "_ts": 1640995500
}
```

#### SQL Database: AlertHistory テーブル
```sql
-- アラート履歴テーブル
CREATE TABLE AlertHistory (
    Id BIGINT IDENTITY(1,1) PRIMARY KEY,
    AlertId NVARCHAR(100) NOT NULL,
    EquipmentId NVARCHAR(50) NOT NULL,
    AlertType NVARCHAR(50) NOT NULL,
    Severity NVARCHAR(20) CHECK (Severity IN ('Info', 'Warning', 'Critical')),
    Status NVARCHAR(20) CHECK (Status IN ('Active', 'Acknowledged', 'Resolved')),
    CreatedAt DATETIME2 NOT NULL,
    AcknowledgedBy NVARCHAR(100),
    AcknowledgedAt DATETIME2,
    ResolvedAt DATETIME2,
    ResponseTimeMinutes AS DATEDIFF(MINUTE, CreatedAt, AcknowledgedAt),
    ResolutionTimeMinutes AS DATEDIFF(MINUTE, CreatedAt, ResolvedAt),
    FOREIGN KEY (EquipmentId) REFERENCES Equipment(EquipmentId),
    INDEX IX_AlertHistory_Equipment_Created (EquipmentId, CreatedAt DESC),
    INDEX IX_AlertHistory_Status_Created (Status, CreatedAt DESC)
);
```

### 4. メンテナンス管理（Maintenance Management）

#### SQL Database: MaintenanceSchedule テーブル
```sql
-- メンテナンススケジュールテーブル
CREATE TABLE MaintenanceSchedule (
    Id BIGINT IDENTITY(1,1) PRIMARY KEY,
    EquipmentId NVARCHAR(50) NOT NULL,
    MaintenanceType NVARCHAR(50) NOT NULL,
    ScheduledDate DATETIME2 NOT NULL,
    EstimatedDurationHours DECIMAL(4,2),
    Priority NVARCHAR(20) CHECK (Priority IN ('Low', 'Medium', 'High', 'Critical')),
    Status NVARCHAR(20) CHECK (Status IN ('Scheduled', 'InProgress', 'Completed', 'Cancelled')),
    AssignedTechnician NVARCHAR(100),
    Description NVARCHAR(1000),
    RequiredParts NVARCHAR(2000),
    CreatedAt DATETIME2 DEFAULT GETDATE(),
    UpdatedAt DATETIME2 DEFAULT GETDATE(),
    FOREIGN KEY (EquipmentId) REFERENCES Equipment(EquipmentId),
    INDEX IX_MaintenanceSchedule_Equipment_Date (EquipmentId, ScheduledDate),
    INDEX IX_MaintenanceSchedule_Status_Date (Status, ScheduledDate)
);

-- メンテナンス実績テーブル
CREATE TABLE MaintenanceHistory (
    Id BIGINT IDENTITY(1,1) PRIMARY KEY,
    ScheduleId BIGINT NOT NULL,
    EquipmentId NVARCHAR(50) NOT NULL,
    StartDateTime DATETIME2 NOT NULL,
    EndDateTime DATETIME2,
    ActualDurationHours AS DATEDIFF(MINUTE, StartDateTime, EndDateTime) / 60.0,
    TechnicianName NVARCHAR(100),
    WorkDescription NVARCHAR(2000),
    PartsUsed NVARCHAR(2000),
    TotalCost DECIMAL(10,2),
    NextMaintenanceDate DATETIME2,
    Notes NVARCHAR(2000),
    CreatedAt DATETIME2 DEFAULT GETDATE(),
    FOREIGN KEY (ScheduleId) REFERENCES MaintenanceSchedule(Id),
    FOREIGN KEY (EquipmentId) REFERENCES Equipment(EquipmentId),
    INDEX IX_MaintenanceHistory_Equipment_Date (EquipmentId, StartDateTime DESC)
);
```

### 5. ユーザー管理（User Management）

#### SQL Database: Users テーブル
```sql
-- ユーザーテーブル
CREATE TABLE Users (
    UserId NVARCHAR(100) PRIMARY KEY,
    UserName NVARCHAR(100) NOT NULL,
    Email NVARCHAR(200) NOT NULL UNIQUE,
    Department NVARCHAR(100),
    Role NVARCHAR(50) CHECK (Role IN ('Operator', 'Technician', 'Supervisor', 'Manager', 'Admin')),
    IsActive BIT DEFAULT 1,
    LastLoginAt DATETIME2,
    CreatedAt DATETIME2 DEFAULT GETDATE(),
    UpdatedAt DATETIME2 DEFAULT GETDATE(),
    INDEX IX_Users_Department (Department),
    INDEX IX_Users_Role (Role)
);

-- ユーザー権限テーブル
CREATE TABLE UserPermissions (
    Id BIGINT IDENTITY(1,1) PRIMARY KEY,
    UserId NVARCHAR(100) NOT NULL,
    EquipmentId NVARCHAR(50),
    Permission NVARCHAR(50) CHECK (Permission IN ('Read', 'Write', 'Maintenance', 'Admin')),
    GrantedBy NVARCHAR(100),
    GrantedAt DATETIME2 DEFAULT GETDATE(),
    FOREIGN KEY (UserId) REFERENCES Users(UserId),
    FOREIGN KEY (EquipmentId) REFERENCES Equipment(EquipmentId),
    UNIQUE (UserId, EquipmentId, Permission)
);
```

## 🔄 データフロー設計

### リアルタイムデータフロー
```
IoTセンサー → IoT Hub → Azure Functions → Cosmos DB
                              ↓
                         アラート判定 → アラート通知
                              ↓
                         データ集計 → SQL Database
```

### バッチデータフロー
```
Cosmos DB → Azure Functions (時間実行) → データ集計処理 → SQL Database
                                              ↓
                                         Power BI → レポート生成
```

## 📊 データ保持ポリシー

### Cosmos DB
- **センサーデータ**: 30日間（TTL設定）
- **アクティブアラート**: 90日間
- **解決済みアラート**: 365日間

### SQL Database
- **マスターデータ**: 永続保持
- **履歴データ**: 5年間
- **メンテナンス記録**: 永続保持（法的要件）

## 🚀 パフォーマンス最適化

### インデックス戦略

#### Cosmos DB
```json
{
  "indexingMode": "consistent",
  "automatic": true,
  "includedPaths": [
    {
      "path": "/equipmentId/?"
    },
    {
      "path": "/timestamp/?"
    },
    {
      "path": "/sensorType/?"
    }
  ],
  "excludedPaths": [
    {
      "path": "/metadata/*"
    }
  ]
}
```

#### SQL Database
```sql
-- 時系列クエリ最適化
CREATE INDEX IX_SensorData_Equipment_Time_Covering 
ON SensorDataHistory (EquipmentId, HourTimestamp DESC)
INCLUDE (SensorType, AvgValue, MinValue, MaxValue);

-- アラート分析用
CREATE INDEX IX_Alert_Severity_Status_Time
ON AlertHistory (Severity, Status, CreatedAt DESC)
INCLUDE (EquipmentId, ResponseTimeMinutes);
```

### パーティション戦略

#### Cosmos DB パーティション
- **パーティションキー**: `/equipmentId`
- **理由**: 設備ごとの負荷分散、クエリの局所性

#### SQL Database パーティション
```sql
-- 月単位パーティション（センサーデータ）
CREATE PARTITION FUNCTION PF_Monthly (DATETIME2)
AS RANGE RIGHT FOR VALUES 
('2024-01-01', '2024-02-01', '2024-03-01', ...);

CREATE PARTITION SCHEME PS_Monthly
AS PARTITION PF_Monthly ALL TO ([PRIMARY]);

ALTER TABLE SensorDataHistory 
ADD CONSTRAINT PK_SensorDataHistory_Partitioned
PRIMARY KEY (Id, HourTimestamp);
```

## 📝 データ品質管理

### バリデーションルール

#### センサーデータ検証
```json
{
  "rules": [
    {
      "field": "value",
      "type": "range",
      "min": -50,
      "max": 100,
      "description": "温度センサーの有効範囲"
    },
    {
      "field": "timestamp",
      "type": "timestamp",
      "maxAge": "5m",
      "description": "5分以内のデータのみ受け入れ"
    }
  ]
}
```

#### データクレンジング
```sql
-- 異常値の検出と除外
WITH OutlierDetection AS (
    SELECT *,
           AVG(AvgValue) OVER (PARTITION BY EquipmentId, SensorType 
                               ORDER BY HourTimestamp 
                               ROWS BETWEEN 23 PRECEDING AND CURRENT ROW) as MovingAvg,
           STDEV(AvgValue) OVER (PARTITION BY EquipmentId, SensorType 
                                ORDER BY HourTimestamp 
                                ROWS BETWEEN 23 PRECEDING AND CURRENT ROW) as MovingStdev
    FROM SensorDataHistory
)
SELECT *
FROM OutlierDetection
WHERE ABS(AvgValue - MovingAvg) <= 3 * MovingStdev;
```

## 🔒 データセキュリティ

### 暗号化設定
- **保存時暗号化**: Azure Cosmos DB、SQL Database両方で有効
- **転送時暗号化**: TLS 1.2以上で全通信を暗号化
- **フィールドレベル暗号化**: 機密データ（個人情報等）

### アクセス制御
```sql
-- 行レベルセキュリティ（RLS）の設定
CREATE FUNCTION fn_SecurityPredicate(@DepartmentColumn NVARCHAR(100))
RETURNS TABLE
WITH SCHEMABINDING
AS
RETURN SELECT 1 AS AccessResult
WHERE @DepartmentColumn = USER_NAME() 
   OR IS_MEMBER('db_manager') = 1;

CREATE SECURITY POLICY EquipmentAccessPolicy
ADD FILTER PREDICATE fn_SecurityPredicate(Department)
ON Equipment WITH (STATE = ON);
```

## 📈 監視とメトリクス

### データベースメトリクス
- **Cosmos DB**: RU消費量、レイテンシ、可用性
- **SQL Database**: DTU使用率、接続数、クエリパフォーマンス

### データ品質メトリクス
- データ欠損率
- 異常値検出率
- リアルタイム性（遅延時間）

## 📋 次のステップ

1. **API仕様の設計**: [api-specification.md](./api-specification.md)を参照
2. **データベーススキーマの実装**: SQLスクリプトの実行
3. **データアクセス層の実装**: Repository パターンの実装
4. **データバリデーションの実装**: 入力データ検証ロジック