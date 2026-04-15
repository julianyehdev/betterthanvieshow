# 🎬 BetterThanVieShow - 電影院訂票系統
[![.NET](https://img.shields.io/badge/.NET-9.0-512BD4?logo=dotnet)](https://dotnet.microsoft.com/)
[![C#](https://img.shields.io/badge/C%23-12.0-239120?logo=csharp)](https://docs.microsoft.com/en-us/dotnet/csharp/)
[![SQL Server](https://img.shields.io/badge/SQL%20Server-2019+-CC2927?logo=microsoft-sql-server)](https://www.microsoft.com/sql-server)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

BetterThanVieShow 是以威秀影城為原型設計的電影院訂票系統，目標是改善傳統售票 App 在行動裝置上體驗不直覺的問題。

系統涵蓋完整的前台訂票流程與後台營運管理，採用 ASP.NET Core 9.0 Web API + SignalR + LINE Pay，實現即時座位同步、金流整合與 QR Code 驗票。

> **本人負責**：後端 API 設計（32 個 API 端點）、EF Core 資料庫設計、SignalR 即時通訊、LINE Pay 金流整合、GitHub Actions CI/CD 部署。
---

## 線上 Demo

| | 連結 | 帳號 | 密碼 |
|--|------|------|------|
| 前台 | [前台網站](https://better-than-vieshow-user.vercel.app/) | `guest@test.com` | `Abcd1234` |
| 後台 | [後台管理](https://better-than-vieshow-admin.vercel.app/login) | `guest@test.com` | `Abcd1234` |
| API 文件 | [Swagger](https://better-than-vieshow-api.rocket-coding.com/swagger/index.html) / [Scalar](https://better-than-vieshow-api.rocket-coding.com/scalar/v1) | — | — |



## 🎯 架構亮點

| 亮點 | 解決問題 | 技術方案 | 成果 |
|-----|--------|--------|------|
| **即時座位同步** | 多人同時搶票導致超額售出 | SignalR + Background Service 定時釋放 | 座位狀態 <100ms 同步；選座後 5 分鐘未付款自動釋放，完全避免售出問題 |
| **LINE Pay HMAC 簽章驗證** | API 請求遭竄改風險 | HMAC-SHA256 + Nonce + Request → Confirm 完整流程 | 每次請求加入簽章 header；支付流程完整端對端 |
| **Clean Architecture** | 業務邏輯與資料存取混雜難以測試 | Controllers → Services → Repositories 三層分離 | 32 個 API 端點模組化；業務邏輯完全解耦；易於單測和維護 |
| **JWT 雙角色權限控制** | 前後台存取權限混亂 | Token 驗證 + [Authorize] 集中管理 | Customer / Admin 角色完全隔離 |
| **GitHub Actions CI/CD** | 手動部署易出錯 | push to main 自動 build → deploy 至 Azure VM | 完整的 build 驗証 → 自動部署流程 |

---

## 🛠️ 技術棧

| 層級 | 技術 | 版本 | 用途 |
|-----|-----|------|------|
| **Web Framework** | ASP.NET Core Web API | 9.0 | RESTful API |
| **Language** | C# | 12.0 | 開發語言 |
| **ORM** | Entity Framework Core | 9.0 | 資料存取層 |
| **Database** | SQL Server | 2019+ | 關聯式資料庫 |
| **Real-time** | SignalR | 9.0 | 座位狀態同步 |
| **Auth** | JWT Bearer + BCrypt | 9.0 | 身份驗證、密碼雜湊 |
| **Payment** | LINE Pay API | — | 第三方支付 |
| **API Docs** | Swagger + Scalar | 7.2.0 / 2.11.10 | 互動式文件 |
| **CI/CD** | GitHub Actions | — | 自動化部署 |

---

## ✨ 核心功能

### 前台（Customer）
- 🎬 電影瀏覽、篩選、推薦
- 🎫 即時選座、座位鎖定、5 分鐘付款倒數
- 💳 LINE Pay 支付、QR Code 票券
- 📊 訂單查詢、票券狀態追蹤

### 後台（Admin）
- 🏢 影廳 / 座位配置管理
- 🎥 電影資訊維護
- 📅 時刻表與場次排程
- 🔍 訂單統計、驗票日誌

---

## 🚀 快速開始

### 環境要求
- .NET SDK 9.0+
- SQL Server 2019+
- Node.js 20+ (if frontend included)

### 啟動步驟

```bash
# 1. Clone 專案
git clone https://github.com/julianyehdev/betterthanvieshow.git
cd betterthanvieshow

# 2. 設定 SQL Server 連線字串
# 編輯 appsettings.json：
#   "ConnectionStrings": {
#     "DefaultConnection": "Server=YOUR_SERVER;Database=BetterThanVieShow;Trusted_Connection=true;"
#   }

# 3. 資料庫遷移
dotnet ef database update

# 4. 執行應用
dotnet run

# 5. 訪問 API 文件
# Swagger: http://localhost:{PORT}/swagger
# Scalar:  http://localhost:{PORT}/scalar/v1
# 其中 {PORT} 為應用執行的埠號
```

---

## 📚 詳細文件

- **[API 端點列表](docs/API.md)** — 32 個 API 的完整參數、回應、範例
- **[部署指南](docs/DEPLOYMENT.md)** — 本地開發、Azure 部署、GitHub Actions 設定
- **[資料模型](docs/DATABASE.md)** — 實體關聯圖、表結構、字段說明

---

## 📄 授權

MIT License — 詳見 [LICENSE](LICENSE) 檔案
