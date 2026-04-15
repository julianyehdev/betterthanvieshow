# API 文件

所有 API 均支援 JSON 請求與回應。認證需使用 JWT Bearer token。

**Base URL**: `http://localhost:5000/api`

---

## 📘 認證 API (Auth)

### POST `/auth/register`
註冊新用戶

**Request Body**:
```json
{
  "email": "user@example.com",
  "password": "Password@123",
  "name": "John Doe"
}
```

**Response** (200):
```json
{
  "success": true,
  "message": "註冊成功",
  "data": {
    "userId": "uuid",
    "email": "user@example.com",
    "name": "John Doe"
  }
}
```

### POST `/auth/login`
登入獲取 JWT token

**Request Body**:
```json
{
  "email": "user@example.com",
  "password": "Password@123"
}
```

**Response** (200):
```json
{
  "success": true,
  "message": "登入成功",
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIs...",
    "expiresIn": 3600,
    "user": {
      "userId": "uuid",
      "email": "user@example.com",
      "role": "Customer"
    }
  }
}
```

**Headers**: 需要在後續 API 請求加入
```
Authorization: Bearer <token>
```

---

## 🎬 電影 API (Movies)

### GET `/movies`
取得所有電影列表（支援篩選）

**Query Parameters**:
- `status` (optional): `OnSale` | `ComingSoon` | `Archived`
- `page` (optional, default: 1)
- `pageSize` (optional, default: 10)

**Response** (200):
```json
{
  "success": true,
  "data": [
    {
      "movieId": "uuid",
      "title": "復仇者聯盟",
      "genre": "Action",
      "rating": "PG-13",
      "duration": 181,
      "releaseDate": "2024-04-15",
      "director": "Joss Whedon",
      "cast": ["Robert Downey Jr.", "Chris Evans"],
      "description": "...",
      "posterUrl": "https://...",
      "trailerUrl": "https://...",
      "status": "OnSale"
    }
  ],
  "pagination": {
    "currentPage": 1,
    "totalPages": 5,
    "totalRecords": 50
  }
}
```

### GET `/movies/{movieId}`
取得單部電影詳細資訊

**Response** (200): 同上單筆

### POST `/movies` (Admin Only)
新增電影

**Headers**: `Authorization: Bearer <token>` (Admin token required)

**Request Body**:
```json
{
  "title": "復仇者聯盟",
  "genre": "Action",
  "rating": "PG-13",
  "duration": 181,
  "releaseDate": "2024-04-15",
  "director": "Joss Whedon",
  "cast": ["Robert Downey Jr.", "Chris Evans"],
  "description": "The Avengers assemble...",
  "posterUrl": "https://...",
  "trailerUrl": "https://...",
  "isFeatured": true
}
```

### PUT `/movies/{movieId}` (Admin Only)
編輯電影

**Request Body**: 同上

---

## 🏢 影廳 API (Theaters)

### GET `/theaters`
取得所有影廳列表

**Response** (200):
```json
{
  "success": true,
  "data": [
    {
      "theaterId": "uuid",
      "name": "Digital Hall 1",
      "type": "Digital",
      "floor": 1,
      "capacity": 150,
      "rows": 10,
      "columns": 15,
      "seats": [
        {
          "seatId": "uuid",
          "row": "A",
          "column": 1,
          "type": "Standard",
          "isAccessible": false
        }
      ]
    }
  ]
}
```

### GET `/theaters/{theaterId}`
取得單間影廳詳細資訊

### POST `/theaters` (Admin Only)
新增影廳

**Request Body**:
```json
{
  "name": "Digital Hall 1",
  "type": "Digital",
  "floor": 1,
  "rows": 10,
  "columns": 15
}
```

### PUT `/theaters/{theaterId}` (Admin Only)
編輯影廳配置

### DELETE `/theaters/{theaterId}` (Admin Only)
刪除影廳（須無場次）

---

## 📅 場次 API (Showtimes)

### GET `/showtimes`
取得所有場次（支援日期篩選）

**Query Parameters**:
- `date` (optional): `YYYY-MM-DD`
- `theaterId` (optional)

**Response** (200):
```json
{
  "success": true,
  "data": [
    {
      "showtimeId": "uuid",
      "movieId": "uuid",
      "theaterId": "uuid",
      "startTime": "2024-04-15T14:00:00Z",
      "endTime": "2024-04-15T15:51:00Z",
      "price": 300,
      "availableSeats": 45,
      "totalSeats": 150
    }
  ]
}
```

### GET `/showtimes/{showtimeId}`
取得單場次詳細資訊（含座位狀態）

### POST `/showtimes` (Admin Only)
新增場次

**Request Body**:
```json
{
  "movieId": "uuid",
  "theaterId": "uuid",
  "startTime": "2024-04-15T14:00:00Z"
}
```

### PUT `/showtimes/{showtimeId}` (Admin Only)
編輯場次

### DELETE `/showtimes/{showtimeId}` (Admin Only)
刪除場次

---

## 🎫 訂單 API (Orders)

### POST `/orders`
建立訂單

**Headers**: `Authorization: Bearer <token>` (Customer token required)

**Request Body**:
```json
{
  "showtimeId": "uuid",
  "seatIds": ["uuid", "uuid", "uuid"],
  "quantity": 3
}
```

**Response** (201):
```json
{
  "success": true,
  "data": {
    "orderId": "uuid",
    "orderNumber": "#ABC-12345",
    "userId": "uuid",
    "showtimeId": "uuid",
    "seats": [
      {
        "seatId": "uuid",
        "row": "A",
        "column": 1
      }
    ],
    "totalPrice": 900,
    "status": "Pending",
    "createdAt": "2024-04-15T12:00:00Z",
    "expiresAt": "2024-04-15T12:05:00Z"
  }
}
```

### GET `/orders/{orderId}`
取得訂單詳細資訊

**Headers**: `Authorization: Bearer <token>`

### GET `/orders`
取得目前用戶的訂單列表

**Headers**: `Authorization: Bearer <token>` (Customer)

**Query Parameters**:
- `status` (optional): `Pending` | `Paid` | `Cancelled`
- `page` (optional)
- `pageSize` (optional)

---

## 💳 支付 API (Payments)

### POST `/payments/linepay/request`
LINE Pay 支付請求

**Headers**: `Authorization: Bearer <token>`

**Request Body**:
```json
{
  "orderId": "uuid",
  "redirectUri": "https://yourdomain.com/payment/callback"
}
```

**Response** (200):
```json
{
  "success": true,
  "data": {
    "transactionId": "2024041512345678",
    "paymentUrl": "https://web-pay.line.me/web/payment/wait?transactionReserveId=...",
    "expiresAt": "2024-04-15T12:05:00Z"
  }
}
```

### POST `/payments/linepay/confirm`
LINE Pay 支付確認

**Headers**: `Authorization: Bearer <token>`

**Request Body**:
```json
{
  "transactionId": "2024041512345678",
  "orderId": "uuid"
}
```

**Response** (200):
```json
{
  "success": true,
  "message": "支付成功",
  "data": {
    "orderId": "uuid",
    "transactionId": "2024041512345678",
    "status": "Paid",
    "paidAt": "2024-04-15T12:02:00Z"
  }
}
```

---

## 🎟️ 票券 API (Tickets)

### GET `/tickets/{orderId}`
取得訂單的票券資訊（含 QR Code）

**Headers**: `Authorization: Bearer <token>`

### POST `/tickets/validate`
驗票（掃描 QR Code）

**Headers**: `Authorization: Bearer <token>` (Admin token required)

**Request Body**:
```json
{
  "ticketId": "uuid"
}
```

**Response** (200):
```json
{
  "success": true,
  "data": {
    "ticketId": "uuid",
    "orderNumber": "#ABC-12345",
    "status": "Used",
    "validatedAt": "2024-04-15T14:15:00Z"
  }
}
```

### GET `/tickets/validate-logs` (Admin Only)
取得驗票日誌

---

## 📅 時刻表 API (DailySchedules)

### GET `/daily-schedules/{date}` (Admin Only)
取得指定日期的時刻表

**Headers**: `Authorization: Bearer <token>` (Admin token required)

**Path Parameters**: `date` (format: YYYY-MM-DD)

**Response** (200):
```json
{
  "success": true,
  "data": {
    "date": "2024-04-15",
    "showtimes": [
      {
        "showtimeId": "uuid",
        "movieId": "uuid",
        "movieTitle": "復仇者聯盟",
        "theaterId": "uuid",
        "theaterName": "Digital Hall 1",
        "startTime": "2024-04-15T14:00:00Z",
        "price": 300,
        "availableSeats": 45
      }
    ]
  }
}
```

### POST `/daily-schedules/{date}/copy` (Admin Only)
複製時刻表到其他日期

**Request Body**:
```json
{
  "targetDate": "2024-04-16"
}
```

---

## 📊 統計 API (Admin Only)

### GET `/admin/statistics/daily/{date}`
取得指定日期的統計資訊

**Headers**: `Authorization: Bearer <token>` (Admin token required)

**Response** (200):
```json
{
  "success": true,
  "data": {
    "date": "2024-04-15",
    "totalShowtimes": 12,
    "totalOrders": 150,
    "totalRevenue": 45000,
    "totalTicketsUsed": 120,
    "averageOccupancyRate": 0.75
  }
}
```

### GET `/admin/statistics/revenue`
取得營收統計

**Query Parameters**:
- `startDate` (required): `YYYY-MM-DD`
- `endDate` (required): `YYYY-MM-DD`

---

## 錯誤回應格式

所有錯誤均返回以下格式 (非 2xx status code):

```json
{
  "success": false,
  "message": "錯誤說明",
  "errors": [
    {
      "field": "email",
      "message": "Email 已被使用"
    }
  ]
}
```

**常見 HTTP Status Codes**:
- `200` — 成功
- `201` — 資源建立成功
- `400` — 請求格式錯誤
- `401` — 未授權（缺少或無效 token）
- `403` — 禁止（權限不足）
- `404` — 資源不存在
- `409` — 衝突（如座位已被選）
- `500` — 伺服器錯誤

---

## 即時通訊 (SignalR)

**Hub**: `/hubs/seat-hub`

### Client 連線後接收事件
```javascript
connection.on("SeatLocked", (showtimeId, seatId) => {
  // 座位被鎖定
});

connection.on("SeatReleased", (showtimeId, seatId) => {
  // 座位被釋放
});

connection.on("OrderExpired", (orderId, seats) => {
  // 訂單過期，座位已釋放
});
```

### Client 發送事件
```javascript
// 進入場次觀看
connection.invoke("JoinShowtime", showtimeId);

// 離開場次
connection.invoke("LeaveShowtime", showtimeId);
```

---

## 節流限制

- 登入失敗 5 次後 15 分鐘內無法再試
- 支付重試最多 3 次
- 單個 IP 每分鐘最多 60 個 API 請求

