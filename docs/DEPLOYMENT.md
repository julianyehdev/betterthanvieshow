# 部署指南

---

## 本地開發環境

### 系統需求
- **.NET SDK**: 9.0 以上
- **SQL Server**: 2019 以上（或 SQL Server Express）
- **Git**: 最新版本
- **VS Code** 或 **Visual Studio 2022**

### 第一次設置

#### 1. Clone 專案
```bash
git clone https://github.com/julianyehdev/betterthanvieshow.git
cd betterthanvieshow
```

#### 2. 設定資料庫連線字串
編輯 `appsettings.Development.json`：

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=.;Database=BetterThanVieShow;Trusted_Connection=true;"
  },
  "JwtSettings": {
    "SecretKey": "your-super-secret-key-at-least-32-characters-long",
    "Issuer": "betterthanvieshow",
    "Audience": "betterthanvieshow-users",
    "ExpirationMinutes": 60
  },
  "LinePaySettings": {
    "ChannelId": "YOUR_CHANNEL_ID",
    "ChannelSecret": "YOUR_CHANNEL_SECRET",
    "IsSandbox": true
  }
}
```

**注意**: 不要提交 `appsettings.json` 到版本控制，敏感資訊應存儲在 `secrets.json`：

```bash
dotnet user-secrets init
dotnet user-secrets set "ConnectionStrings:DefaultConnection" "Server=.;Database=BetterThanVieShow;Trusted_Connection=true;"
dotnet user-secrets set "JwtSettings:SecretKey" "your-secret-key"
dotnet user-secrets set "LinePaySettings:ChannelId" "YOUR_CHANNEL_ID"
```

#### 3. 建立資料庫
```bash
# 安裝 dotnet-ef 工具（若還未安裝）
dotnet tool install --global dotnet-ef

# 執行遷移，建立資料庫與表
dotnet ef database update
```

#### 4. 啟動應用
```bash
dotnet run

# 或使用 watch 模式（程式碼變更自動重載）
dotnet watch run
```

應用應在 `http://localhost:5000` 啟動

#### 5. 驗證安裝
- **Swagger 文件**: http://localhost:5000/swagger
- **Scalar 文件**: http://localhost:5000/scalar
- **健康檢查**: `GET /health`

---

## Docker 部署

### 建立 Docker Image

在專案根目錄創建 `Dockerfile`：

```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:9.0 AS build
WORKDIR /app

COPY . .
RUN dotnet restore
RUN dotnet publish -c Release -o out

FROM mcr.microsoft.com/dotnet/aspnet:9.0
WORKDIR /app
COPY --from=build /app/out .

EXPOSE 5000
ENV ASPNETCORE_URLS=http://+:5000
ENTRYPOINT ["dotnet", "betterthanvieshow.dll"]
```

### 建置與執行

```bash
# 建置 image
docker build -t betterthanvieshow:latest .

# 運行容器（需連接本機 SQL Server）
docker run -d \
  -p 5000:5000 \
  -e "ConnectionStrings:DefaultConnection=Server=host.docker.internal;Database=BetterThanVieShow;User=sa;Password=YourPassword;" \
  -e "JwtSettings:SecretKey=your-secret-key" \
  --name betterthanvieshow \
  betterthanvieshow:latest
```

### Docker Compose （推薦）

創建 `docker-compose.yml`：

```yaml
version: '3.8'

services:
  sqlserver:
    image: mcr.microsoft.com/mssql/server:2019-latest
    environment:
      SA_PASSWORD: "YourPassword@123"
      ACCEPT_EULA: "Y"
    ports:
      - "1433:1433"
    volumes:
      - sqlserver_data:/var/opt/mssql

  app:
    build: .
    depends_on:
      - sqlserver
    environment:
      ConnectionStrings__DefaultConnection: "Server=sqlserver;Database=BetterThanVieShow;User=sa;Password=YourPassword@123;"
      JwtSettings__SecretKey: "your-super-secret-key-at-least-32-characters-long"
      LinePaySettings__ChannelId: "YOUR_CHANNEL_ID"
      LinePaySettings__ChannelSecret: "YOUR_CHANNEL_SECRET"
      LinePaySettings__IsSandbox: "true"
    ports:
      - "5000:5000"

volumes:
  sqlserver_data:
```

啟動：
```bash
docker-compose up -d
```

---

## Azure 部署

### 前置準備

1. **Azure 帳戶** 與 **Azure CLI** 已安裝
2. **SQL Server** 已在 Azure 上建立
3. **Azure VM** 或 **App Service** 已建立

### 方法 A：使用 GitHub Actions CI/CD

在 `.github/workflows/` 下創建 `deploy.yml`：

```yaml
name: Deploy to Azure

on:
  push:
    branches: [ main ]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3

    - name: Setup .NET
      uses: actions/setup-dotnet@v3
      with:
        dotnet-version: '9.0.x'

    - name: Restore dependencies
      run: dotnet restore

    - name: Build
      run: dotnet build --configuration Release --no-restore

    - name: Test
      run: dotnet test --configuration Release --no-build --verbosity normal

    - name: Publish
      run: dotnet publish -c Release -o ./publish

    - name: Deploy to Azure VM
      uses: appleboy/ssh-action@master
      with:
        host: ${{ secrets.AZURE_VM_IP }}
        username: ${{ secrets.AZURE_VM_USER }}
        key: ${{ secrets.AZURE_VM_SSH_KEY }}
        script: |
          cd /var/www/betterthanvieshow
          git pull origin main
          dotnet publish -c Release -o ./publish
          systemctl restart betterthanvieshow
```

### 方法 B：手動部署至 Azure VM

1. **連接 VM**
```bash
ssh user@your-vm-ip
```

2. **安裝 .NET Runtime**
```bash
wget https://dot.net/v1/dotnet-install.sh -O dotnet-install.sh
chmod +x dotnet-install.sh
./dotnet-install.sh --version 9.0 --install-dir ~/.dotnet
```

3. **複製應用檔案**
```bash
# 本機
dotnet publish -c Release -o ./publish

# 上傳到 VM
scp -r ./publish/* user@your-vm-ip:/var/www/betterthanvieshow/
```

4. **設定 SQL Server 連線**
```bash
# 在 VM 上編輯 appsettings.json
nano /var/www/betterthanvieshow/appsettings.json
```

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=YOUR_SQL_SERVER_IP;Database=BetterThanVieShow;User=sa;Password=YourPassword;"
  }
}
```

5. **啟動應用（使用 systemd）**

創建服務檔案 `/etc/systemd/system/betterthanvieshow.service`：

```ini
[Unit]
Description=BetterThanVieShow API
After=network.target

[Service]
Type=notify
User=www-data
WorkingDirectory=/var/www/betterthanvieshow
ExecStart=/home/user/.dotnet/dotnet /var/www/betterthanvieshow/betterthanvieshow.dll
Restart=always
RestartSec=10
StandardOutput=journal

[Install]
WantedBy=multi-user.target
```

啟動服務：
```bash
sudo systemctl daemon-reload
sudo systemctl enable betterthanvieshow
sudo systemctl start betterthanvieshow
sudo systemctl status betterthanvieshow
```

查看日誌：
```bash
sudo journalctl -u betterthanvieshow -f
```

---

## Nginx 反向代理設定

在 Azure VM 上使用 Nginx 作為反向代理：

```nginx
upstream betterthanvieshow {
    server 127.0.0.1:5000;
}

server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://betterthanvieshow;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection keep-alive;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_buffering off;
        proxy_request_buffering off;
        proxy_redirect off;
    }

    # SignalR WebSocket 支援
    location /hubs/seat-hub {
        proxy_pass http://betterthanvieshow/hubs/seat-hub;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

啟用 SSL（使用 Let's Encrypt）：
```bash
sudo apt-get install certbot python3-certbot-nginx
sudo certbot --nginx -d your-domain.com
```

---

## 資料庫遷移

### 本機建立新的遷移

```bash
# 修改實體後，建立遷移檔案
dotnet ef migrations add MigrationName

# 應用遷移到資料庫
dotnet ef database update

# 查看待應用的遷移
dotnet ef migrations list
```

### 生產環境應用遷移

```bash
# 在伺服器上執行
dotnet ef database update --project . --startup-project . --context AppDbContext

# 或透過 SQL 腳本（更安全）
dotnet ef migrations script -i -o ./migrations.sql
# 將 migrations.sql 複製到伺服器，在 SQL Server Management Studio 執行
```

---

## 環境變數

### 開發環境
使用 `appsettings.Development.json` 與 `secrets.json`

### 生產環境
設定環境變數（推薦）：

```bash
export ASPNETCORE_ENVIRONMENT=Production
export ConnectionStrings__DefaultConnection="Server=...;Database=BetterThanVieShow;..."
export JwtSettings__SecretKey="your-production-secret-key"
export LinePaySettings__ChannelId="PROD_CHANNEL_ID"
export LinePaySettings__ChannelSecret="PROD_CHANNEL_SECRET"
export LinePaySettings__IsSandbox=false
```

---

## 監控與日誌

### 應用日誌

應用日誌輸出到 `logs/` 目錄。啟用詳細日誌：

編輯 `appsettings.json`：
```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "BetterThanVieShow": "Debug"
    }
  }
}
```

### 健康檢查

```bash
curl http://localhost:5000/health
```

應返回：
```json
{
  "status": "Healthy"
}
```

---

## 備份與復原

### 定期備份 SQL Server

```bash
# 備份資料庫
sqlcmd -S YOUR_SERVER -U sa -P YOUR_PASSWORD -Q "BACKUP DATABASE BetterThanVieShow TO DISK = '/var/opt/mssql/backup/BetterThanVieShow.bak';"

# 定時備份（cron）
0 2 * * * sqlcmd -S YOUR_SERVER -U sa -P YOUR_PASSWORD -Q "BACKUP DATABASE BetterThanVieShow TO DISK = '/var/opt/mssql/backup/BetterThanVieShow_$(date +\%Y\%m\%d).bak';"
```

### 復原資料庫

```bash
sqlcmd -S YOUR_SERVER -U sa -P YOUR_PASSWORD -Q "RESTORE DATABASE BetterThanVieShow FROM DISK = '/var/opt/mssql/backup/BetterThanVieShow.bak';"
```

---

## 故障排除

### 資料庫連線失敗
- 檢查 `appsettings.json` 中的連線字串
- 確保 SQL Server 已啟動並可訪問
- 檢查防火牆規則（SQL Server 預設連接埠 1433）

### SignalR 連線失敗
- 確保 Nginx 配置支援 WebSocket（proxy_set_header Upgrade）
- 檢查防火牆是否阻止 WebSocket

### 應用崩潰
- 查看 `systemd` 日誌：`journalctl -u betterthanvieshow -f`
- 檢查應用日誌：`logs/` 目錄

