# APISIX Docker Compose 配置

這個 Docker Compose 配置包含了完整的 APISIX 生態系統，包括：

- **APISIX Gateway** (端口 9080, 9091, 9443, 9092)
- **etcd** (端口 2379) - 配置存儲
- **APISIX Dashboard** (端口 9000) - Web 管理界面
- **Prometheus** (端口 9090) - 監控數據收集
- **Grafana** (端口 3000) - 監控數據可視化

## 快速開始

### 方式一：使用啟動腳本（推薦）
```powershell
# 啟動所有服務
.\start.ps1

# 停止所有服務
.\stop.ps1
```

### 方式二：手動啟動
1. 先啟動 etcd 並等待就緒：
   ```powershell
   docker-compose up -d etcd
   # 等待 30 秒讓 etcd 完全啟動
   Start-Sleep -Seconds 30
   ```

2. 啟動其他服務：
   ```powershell
   docker-compose up -d
   ```

3. 檢查服務狀態：
   ```powershell
   docker-compose ps
   ```

### 故障排除
如果遇到 etcd 連接問題，請嘗試：
```powershell
# 完全重新啟動
docker-compose down
docker-compose up -d etcd
Start-Sleep -Seconds 30
docker-compose up -d
```

## 限流功能

APISIX 提供了多種限流插件，類似於 Kong 的限流功能：

### 快速開始限流
```powershell
# 運行限流測試
.\test-rate-limit.ps1

# 使用限流配置工具
.\rate-limit.ps1 -Action help
```

### 限流類型
1. **limit-req**: 請求頻率限流（類似 Nginx limit_req）
2. **limit-conn**: 連接數限流
3. **limit-count**: 請求計數限流（時間窗口）

### 配置示例
```powershell
# 創建上游
.\rate-limit.ps1 -Action create-upstream -Route 1

# 配置每秒5個請求的限流
.\rate-limit.ps1 -Action limit-req -Route 1 -Rate 5 -Uri "/api/*"

# 測試限流效果
.\rate-limit.ps1 -Action test -Route 1
```

詳細配置請參考：
- `rate-limiting-examples.md` - 命令行配置示例
- `dashboard-rate-limiting.md` - Dashboard 配置指南

## 訪問地址

- **APISIX Gateway**: http://localhost:9080
- **APISIX Admin API**: http://localhost:9091
- **APISIX Dashboard**: http://localhost:9000
  - 用戶名: admin
  - 密碼: admin
- **Prometheus**: http://localhost:9090
- **Grafana**: http://localhost:3000
  - 用戶名: admin
  - 密碼: admin

## 配置說明

### APISIX 配置
- 配置文件位於 `apisix_conf/config.yaml`
- 默認 Admin API Key: `edd1c9f034335f136f87ad84b625c8f1`
- 日誌存儲在 `apisix_log/` 目錄

### 基本使用示例

1. **創建上游服務**：
   ```bash
   curl "http://127.0.0.1:9091/apisix/admin/upstreams/1" \
   -H "X-API-KEY: edd1c9f034335f136f87ad84b625c8f1" \
   -X PUT -d '
   {
     "type": "roundrobin",
     "nodes": {
       "httpbin.org:80": 1
     }
   }'
   ```

2. **創建路由**：
   ```bash
   curl "http://127.0.0.1:9091/apisix/admin/routes/1" \
   -H "X-API-KEY: edd1c9f034335f136f87ad84b625c8f1" \
   -X PUT -d '
   {
     "uri": "/get",
     "upstream_id": 1
   }'
   ```

3. **測試路由**：
   ```bash
   curl -i http://127.0.0.1:9080/get
   ```

## 停止和清理

停止服務：
```bash
docker-compose down
```

停止服務並刪除數據卷：
```bash
docker-compose down -v
```

## 故障排除

如果遇到問題，可以：

1. 檢查所有容器狀態：
   ```bash
   docker-compose ps
   ```

2. 查看特定服務的日誌：
   ```bash
   docker-compose logs apisix
   docker-compose logs etcd
   ```

3. 重新啟動服務：
   ```bash
   docker-compose restart apisix
   ```

## 自定義配置

- 修改 `apisix_conf/config.yaml` 來調整 APISIX 配置
- 修改 `dashboard_conf/conf.yaml` 來調整 Dashboard 配置
- 修改 `prometheus_conf/prometheus.yml` 來調整監控配置

配置修改後需要重新啟動相應的服務：
```bash
docker-compose restart apisix
```
