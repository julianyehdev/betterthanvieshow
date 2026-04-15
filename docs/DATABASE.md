# 資料模型

---

## 實體關聯圖 (ERD)

```
┌─────────┐         ┌──────────┐
│  User   │         │  Movie   │
├─────────┤         ├──────────┤
│ UserId  │         │ MovieId  │
│ Email   │         │ Title    │
│ Password│         │ Duration │
│ Role    │         │ Status   │
└────┬────┘         └────┬─────┘
     │                    │
     │ 1:N                │ N:M
     │                    │
     │              ┌─────────────┐
     │              │MovieShowTime│
     │              ├─────────────┤
     │              │ MovieId     │
     │              │ ShowtimeId  │
     │              └──────┬──────┘
     │                     │
     │ 1:N                 │ 1:N
     │                     │
     └─────────┬───────────┘
               │
          ┌────┴────┐
          │  Order  │
          ├─────────┤
          │ OrderId │
          │ UserId  │
          │ Status  │
          └────┬────┘
               │ 1:N
               │
          ┌────┴─────┐
          │  Ticket  │
          ├──────────┤
          │ TicketId │
          │ OrderId  │
          │ SeatId   │
          │ Status   │
          └──────────┘

┌─────────────┐        ┌──────────┐
│  Theater    │────────│  Seat    │
├─────────────┤ 1:N    ├──────────┤
│ TheaterId   │        │ SeatId   │
│ Name        │        │ TheaterId│
│ Type        │        │ Row      │
│ Floor       │        │ Column   │
└─────────────┘        │ Type     │
                       └──────────┘

┌──────────────┐        ┌────────────┐
│ DailySchedule│────────│ Showtime   │
├──────────────┤ 1:N    ├────────────┤
│ ScheduleId   │        │ ShowtimeId │
│ Date         │        │ StartTime  │
│ Status       │        │ TheaterId  │
└──────────────┘        │ Price      │
                        └────────────┘

┌───────────────────┐
│TicketValidateLog  │
├───────────────────┤
│ LogId             │
│ TicketId          │
│ ValidatedAt       │
│ ValidatedBy       │
└───────────────────┘
```

---

## 資料表詳細說明

### 1. Users（用戶表）

| 欄位名 | 資料類型 | 約束 | 說明 |
|--------|---------|------|------|
| UserId | UNIQUEIDENTIFIER | PRIMARY KEY | 用戶唯一識別碼 |
| Email | NVARCHAR(255) | UNIQUE, NOT NULL | 登入電郵，唯一 |
| PasswordHash | NVARCHAR(MAX) | NOT NULL | BCrypt 雜湊密碼 |
| Name | NVARCHAR(100) | NOT NULL | 用戶名稱 |
| Role | NVARCHAR(50) | NOT NULL | `Customer` 或 `Admin` |
| IsActive | BIT | NOT NULL, DEFAULT(1) | 帳號啟用狀態 |
| CreatedAt | DATETIME2 | NOT NULL | 建立時間 |
| UpdatedAt | DATETIME2 | NULL | 最後更新時間 |

**索引**:
- `UK_Users_Email` on Email (UNIQUE)
- `IX_Users_Role` on Role

---

### 2. Movies（電影表）

| 欄位名 | 資料類型 | 約束 | 說明 |
|--------|---------|------|------|
| MovieId | UNIQUEIDENTIFIER | PRIMARY KEY | 電影唯一識別碼 |
| Title | NVARCHAR(200) | NOT NULL | 電影名稱 |
| Genre | NVARCHAR(100) | NOT NULL | 類型（Action, Comedy 等） |
| Rating | NVARCHAR(20) | NOT NULL | 分級（G, PG, PG-13, R 等） |
| Duration | INT | NOT NULL | 片長（分鐘） |
| ReleaseDate | DATE | NOT NULL | 上映日期 |
| Director | NVARCHAR(200) | NULL | 導演 |
| Cast | NVARCHAR(MAX) | NULL | 演員（JSON 陣列或分號分隔） |
| Description | NVARCHAR(MAX) | NULL | 電影概述 |
| PosterUrl | NVARCHAR(500) | NULL | 海報圖片 URL |
| TrailerUrl | NVARCHAR(500) | NULL | 預告片 URL |
| IsFeatured | BIT | NOT NULL, DEFAULT(0) | 是否為推薦電影 |
| Status | NVARCHAR(50) | NOT NULL | `OnSale`, `ComingSoon`, `Archived` |
| CreatedAt | DATETIME2 | NOT NULL | 建立時間 |
| UpdatedAt | DATETIME2 | NULL | 最後更新時間 |

**索引**:
- `IX_Movies_Status` on Status
- `IX_Movies_ReleaseDate` on ReleaseDate
- `IX_Movies_IsFeatured` on IsFeatured

---

### 3. Theaters（影廳表）

| 欄位名 | 資料類型 | 約束 | 說明 |
|--------|---------|------|------|
| TheaterId | UNIQUEIDENTIFIER | PRIMARY KEY | 影廳唯一識別碼 |
| Name | NVARCHAR(100) | NOT NULL | 影廳名稱（如 Digital Hall 1） |
| Type | NVARCHAR(50) | NOT NULL | 影廳類型：`Digital`, `4DX`, `IMAX` |
| Floor | INT | NOT NULL | 樓層編號 |
| Capacity | INT | NOT NULL | 座位總數 |
| Rows | INT | NOT NULL | 座位排數 |
| Columns | INT | NOT NULL | 座位列數 |
| CreatedAt | DATETIME2 | NOT NULL | 建立時間 |
| UpdatedAt | DATETIME2 | NULL | 最後更新時間 |

**索引**:
- `IX_Theaters_Type` on Type
- `IX_Theaters_Floor` on Floor

---

### 4. Seats（座位表）

| 欄位名 | 資料類型 | 約束 | 說明 |
|--------|---------|------|------|
| SeatId | UNIQUEIDENTIFIER | PRIMARY KEY | 座位唯一識別碼 |
| TheaterId | UNIQUEIDENTIFIER | FOREIGN KEY | 所屬影廳 |
| Row | NCHAR(1) | NOT NULL | 列標識（A, B, C ...） |
| Column | INT | NOT NULL | 列號 (1, 2, 3 ...) |
| Type | NVARCHAR(50) | NOT NULL | `Standard`, `Recliner`, `Accessible`, `Aisle` |
| IsAccessible | BIT | NOT NULL, DEFAULT(0) | 是否為無障礙座位 |
| CreatedAt | DATETIME2 | NOT NULL | 建立時間 |

**外鍵**:
- `FK_Seats_Theaters` -> Theaters(TheaterId)

**索引**:
- `IX_Seats_TheaterId` on TheaterId
- `UK_Seats_TheaterRow_Column` on TheaterId, Row, Column (UNIQUE)

---

### 5. Showtimes（場次表）

| 欄位名 | 資料類型 | 約束 | 說明 |
|--------|---------|------|------|
| ShowtimeId | UNIQUEIDENTIFIER | PRIMARY KEY | 場次唯一識別碼 |
| MovieId | UNIQUEIDENTIFIER | FOREIGN KEY | 電影 ID |
| TheaterId | UNIQUEIDENTIFIER | FOREIGN KEY | 影廳 ID |
| StartTime | DATETIME2 | NOT NULL | 開始時間 |
| EndTime | DATETIME2 | NOT NULL | 結束時間（自動計算） |
| Price | DECIMAL(10,2) | NOT NULL | 票價 |
| Status | NVARCHAR(50) | NOT NULL | `Active`, `Cancelled` |
| CreatedAt | DATETIME2 | NOT NULL | 建立時間 |
| UpdatedAt | DATETIME2 | NULL | 最後更新時間 |

**外鍵**:
- `FK_Showtimes_Movies` -> Movies(MovieId)
- `FK_Showtimes_Theaters` -> Theaters(TheaterId)

**索引**:
- `IX_Showtimes_TheaterId_StartTime` on TheaterId, StartTime
- `IX_Showtimes_MovieId` on MovieId
- `UK_Showtimes_TheaterStartTime` on TheaterId, StartTime (UNIQUE，防止時間衝突)

---

### 6. DailySchedules（日期時刻表）

| 欄位名 | 資料類型 | 約束 | 說明 |
|--------|---------|------|------|
| ScheduleId | UNIQUEIDENTIFIER | PRIMARY KEY | 時刻表唯一識別碼 |
| Date | DATE | NOT NULL, UNIQUE | 日期 |
| Status | NVARCHAR(50) | NOT NULL | `Draft`, `OnSale`, `Closed` |
| CreatedAt | DATETIME2 | NOT NULL | 建立時間 |
| UpdatedAt | DATETIME2 | NULL | 最後更新時間 |

**索引**:
- `UK_DailySchedules_Date` on Date (UNIQUE)

---

### 7. Orders（訂單表）

| 欄位名 | 資料類型 | 約束 | 說明 |
|--------|---------|------|------|
| OrderId | UNIQUEIDENTIFIER | PRIMARY KEY | 訂單唯一識別碼 |
| OrderNumber | NVARCHAR(50) | UNIQUE, NOT NULL | 訂單號（如 #ABC-12345） |
| UserId | UNIQUEIDENTIFIER | FOREIGN KEY | 購票用戶 |
| ShowtimeId | UNIQUEIDENTIFIER | FOREIGN KEY | 場次 |
| TotalPrice | DECIMAL(10,2) | NOT NULL | 總金額 |
| Status | NVARCHAR(50) | NOT NULL | `Pending`, `Paid`, `Cancelled` |
| ExpiresAt | DATETIME2 | NULL | 過期時間（5分鐘後） |
| CreatedAt | DATETIME2 | NOT NULL | 建立時間 |
| UpdatedAt | DATETIME2 | NULL | 最後更新時間 |

**外鍵**:
- `FK_Orders_Users` -> Users(UserId)
- `FK_Orders_Showtimes` -> Showtimes(ShowtimeId)

**索引**:
- `IX_Orders_UserId` on UserId
- `IX_Orders_Status` on Status
- `IX_Orders_CreatedAt` on CreatedAt
- `IX_Orders_ExpiresAt` on ExpiresAt

---

### 8. Tickets（票券表）

| 欄位名 | 資料類型 | 約束 | 說明 |
|--------|---------|------|------|
| TicketId | UNIQUEIDENTIFIER | PRIMARY KEY | 票券唯一識別碼 |
| OrderId | UNIQUEIDENTIFIER | FOREIGN KEY | 訂單 ID |
| SeatId | UNIQUEIDENTIFIER | FOREIGN KEY | 座位 ID |
| QRCode | NVARCHAR(500) | NOT NULL | QR Code 編碼（加密） |
| Status | NVARCHAR(50) | NOT NULL | `Pending`, `Unused`, `Used`, `Expired` |
| ValidatedAt | DATETIME2 | NULL | 驗票時間 |
| CreatedAt | DATETIME2 | NOT NULL | 建立時間 |

**外鍵**:
- `FK_Tickets_Orders` -> Orders(OrderId)
- `FK_Tickets_Seats` -> Seats(SeatId)

**索引**:
- `IX_Tickets_OrderId` on OrderId
- `IX_Tickets_Status` on Status
- `IX_Tickets_QRCode` on QRCode (UNIQUE)

---

### 9. TicketValidateLogs（驗票日誌表）

| 欄位名 | 資料類型 | 約束 | 說明 |
|--------|---------|------|------|
| LogId | UNIQUEIDENTIFIER | PRIMARY KEY | 日誌唯一識別碼 |
| TicketId | UNIQUEIDENTIFIER | FOREIGN KEY | 票券 ID |
| ValidatedBy | UNIQUEIDENTIFIER | FOREIGN KEY | 驗票人（Admin） |
| ValidatedAt | DATETIME2 | NOT NULL | 驗票時間 |
| Status | NVARCHAR(50) | NOT NULL | `Success`, `Duplicate`, `Invalid` |
| ErrorMessage | NVARCHAR(500) | NULL | 錯誤訊息（如重複掃描） |

**外鍵**:
- `FK_TicketValidateLogs_Tickets` -> Tickets(TicketId)
- `FK_TicketValidateLogs_Users` -> Users(ValidatedBy)

**索引**:
- `IX_TicketValidateLogs_TicketId` on TicketId
- `IX_TicketValidateLogs_ValidatedAt` on ValidatedAt

---

### 10. LinePayTransactions（LINE Pay 交易記錄表）

| 欄位名 | 資料類型 | 約束 | 說明 |
|--------|---------|------|------|
| TransactionId | UNIQUEIDENTIFIER | PRIMARY KEY | 交易唯一識別碼 |
| OrderId | UNIQUEIDENTIFIER | FOREIGN KEY | 訂單 ID |
| LinePayTransactionId | BIGINT | UNIQUE, NOT NULL | LINE Pay 交易 ID |
| Amount | DECIMAL(10,2) | NOT NULL | 交易金額 |
| Currency | NCHAR(3) | NOT NULL | 幣別（TWD） |
| Status | NVARCHAR(50) | NOT NULL | `Reserved`, `Confirmed`, `Captured`, `Voided`, `Failed` |
| PaymentUrl | NVARCHAR(500) | NULL | LINE Pay 支付 URL |
| RequestKey | NVARCHAR(500) | NULL | Request 冪等鑰匙 |
| CreatedAt | DATETIME2 | NOT NULL | 建立時間 |
| UpdatedAt | DATETIME2 | NULL | 最後更新時間 |

**外鍵**:
- `FK_LinePayTransactions_Orders` -> Orders(OrderId)

**索引**:
- `IX_LinePayTransactions_OrderId` on OrderId
- `IX_LinePayTransactions_Status` on Status

---

## 關鍵業務規則

### 座位鎖定流程

```
1. 用戶選座 → 建立 Order (Status = Pending)
2. SignalR 廣播座位狀態為 "Locked"
3. Background Service 每秒掃描過期訂單
4. 若 Order.ExpiresAt < Now:
   - Order.Status = Cancelled
   - Tickets.Status = Expired
   - SignalR 廣播座位為 "Released"
5. 用戶支付 → Order.Status = Paid
   - Tickets.Status = Unused
```

### LINE Pay 支付流程（冪等性設計）

```
1. POST /payments/linepay/request
   - 生成 Request Key = OrderId + Timestamp
   - 存儲於 LinePayTransactions.RequestKey
   - 呼叫 LINE Pay API Request

2. POST /payments/linepay/confirm
   - 檢查 RequestKey 是否已存在
   - 若存在 → 直接返回之前的結果（冪等）
   - 若不存在 → 呼叫 LINE Pay API Confirm
   - 更新 Order.Status = Paid

3. 背景任務
   - 每小時掃描 Reserved 但超過 3600 秒的交易
   - 調用 LINE Pay API Void（取消預留）
```

### 票券驗票規則

```
1. 掃描 QR Code → 查詢 Ticket
2. 檢查：
   - Ticket.Status == Unused（未使用）
   - Order.Status == Paid（已支付）
   - Showtime.StartTime <= Now（場次已開始）
3. 若檢查通過：
   - Ticket.Status = Used
   - 建立 TicketValidateLog (Status = Success)
4. 若重複掃描：
   - 建立 TicketValidateLog (Status = Duplicate)
   - 返回錯誤
```

---

## 查詢優化建議

### 常用查詢及推薦索引

| 查詢用途 | 建議複合索引 | 備註 |
|---------|-----------|------|
| 查詢某日期場次 | (TheaterId, StartTime) | Showtime 高頻查詢 |
| 查詢用戶訂單 | (UserId, CreatedAt DESC) | 支援分頁排序 |
| 查詢過期訂單 | (Status, ExpiresAt) | Background Service 掃描 |
| 查詢驗票日誌 | (ValidatedAt DESC) | 統計報表查詢 |
| 檢查座位可用性 | (ShowtimeId, SeatId) | 選座驗證 |

### 資料分割建議（大流量場景）

```
Orders 表可按 CreatedAt 進行日期分割：
- Orders_2024_Q1
- Orders_2024_Q2
- ...

TicketValidateLogs 表可按 Date 進行日期分割：
- TicketValidateLogs_2024_04
- TicketValidateLogs_2024_05
- ...
```

---

## 備份與恢復

### 定期備份策略

```bash
# 每日 02:00 完整備份
BACKUP DATABASE BetterThanVieShow TO DISK = '/var/opt/mssql/backup/BetterThanVieShow_$(date +%Y%m%d).bak';

# 每 6 小時差異備份
BACKUP DATABASE BetterThanVieShow TO DISK = '/var/opt/mssql/backup/BetterThanVieShow_Diff_$(date +%Y%m%d_%H%M%S).bak' WITH DIFFERENTIAL;
```

