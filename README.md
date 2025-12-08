# 🚀 安裝及啟動 Nginx

## 📦 下載 Nginx
下載網址：https://nginx.org/en/download.html

## 測試URL
http://localhost:8080
127.0.0.1:8080

## cd進入 nginx-1.28.0資料夾
```sh
cd nginx-1.28.0
```

## 啟動Nginx
```sh
start nginx
```

## 重啟Nginx
```sh
.\nginx -s reload
```

## 最後查詢啟動是否成功。
輸入指令：
```sh
tasklist /fi "imagename eq nginx.exe"
```

## 停止Nginx
```sh
.\nginx -s stop
```


# 🚀 使用winsw部署將程式改成開機時自動啟動服務

1. 先安裝 .NET Framework 3.5 以上功能
2. 下載winsw 下載路徑：http://repo.jenkins-ci.org/releases/com/sun/winsw/winsw/2.9.0/
3. 基本設定（複製winsw.exe 及 建立 winsw.xml 檔）
 - 將下載後的文件（如 winsw-2.9.0-bin.exe ）拷貝至nginx資料夾下，將重新命名為nginx-winsw.exe
 - 在nginx資料夾下創建 nginx-winsw.xml 文件，並輸入以下內容後存檔
```html
<?xml version="1.0" encoding="UTF-8" ?>
<service>
  <id>Nginx</id>
  <name>Nginx</name>
  <description>本服務用於加載Nginx服務，請確保開機啟動。</description>
  <logpath>C:\Users\yiru\Downloads\nginx-1.19.6\nginx\logs</logpath>
  <executable>nginx.exe</executable>
  <stopexecutable>nginx.exe</stopexecutable>
  <stopargument>-s</stopargument>
  <stopargument>stop</stopargument>
  <logmode>rotate</logmode>
</service>
```
4. 安裝及卸載指令
 - 開啟cmd （命令提示字元）→ 接著cd 到 nginx 目錄
 - 輸入安裝指令：`nginx-winsw.exe install`
 - 輸入卸載 指令：`nginx-winsw.exe uninstall`

5. 部署將程式改成開機時自動啟動服務
- 方法一：首先開啟應用程式 → 服務
  - 開啟服務視窗後，這邊已經多了 Nginx 服務了
  - 按下右鍵 → 直接啟動
  - 當成功啟動時，會自動跳出瀏覽器，並打開127.0.0.1的歡迎畫面。

- 方法二：也可以使用指令啟動
  - 首先開啟cmd → 右鍵以系統管理員身分執行
  - 常用指令如下：
    - 啟動服務：`net start nginx`
    - 查看狀態指令：`tasklist /fi "imagename eq nginx.exe"`
    - 停止服務：`net stop nginx`

---

# 🚀 Windows Nginx 正確的資料夾結構（推薦）
```sh
C:\nginx\
│
├── nginx.exe
├── conf\
│   ├── nginx.conf              # 主設定檔
│   └── conf.d\                 # 子站台放這裡
│       ├── frontend.conf       # 前端站台設定
│       └── backend.conf        # 後端站台設定（API）
│
├── www\                        # 靜態網站根目錄
│   ├── frontend\               # 前端 dist 或 html
│   └── backend\                # 後端靜態或 API 子站台
│
└── logs\
    ├── access.log
    └── error.log
```

## 🟥 Windows Nginx 路徑使用注意事項
### Windows 必須使用 正斜線 / 或雙反斜線：
✔ `C:/nginx/ssl/server.crt`

✔ `C:\\nginx\\ssl\\server.crt`

✘ `C:\nginx\ssl\server.crt`（會報錯）


## 🟦 各資料夾用途說明
## 📁 C:\nginx\conf\nginx.conf

Nginx 主設定檔
你只需要讓它 include 子設定即可：
```sh
worker_processes 1;

events {
    worker_connections 1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile on;
    keepalive_timeout 65;

    client_max_body_size 50m;

    include conf.d/*.conf;   # 所有站台統一從這裡載入
}
```

## 📁 C:\nginx\conf\conf.d*

放每個站台的設定檔（推薦方式！）

範例：

| 檔案              | 說明                        |
| --------------- | ------------------------- |
| `frontend.conf` | 前端網站，例如 Vue/React/Angular |
| `backend.conf`  | 後端 API 服務（反向代理）           |

## 📁 *C:\nginx\www\frontend*
前端網站的根目錄，比如：
```sh
C:\nginx\www\frontend\index.html
C:\nginx\www\frontend\assets\...
```
在 Nginx 內設定：
```sh
root C:/nginx/www/frontend;
```

## 📁 *C:\nginx\www\backend*
如果後端也是靜態 OR 後端跑在 Node/Python/等，也可以反代：
```sh
location /api/ {
    proxy_pass http://127.0.0.1:5000;
}
```
如果你的後端是靜態 HTML 或 API docs 也可以放：
```sh
C:\nginx\www\backend\
```

## 🟨 完整前端設定（純 HTTP）
C:\nginx\conf\conf.d\frontend.conf
```sh
server {
    listen 3000;
    server_name frontend.local;

    root C:/Users/Len/Desktop/nginx-1.28.0/www/front-end/dist;    # ← 修改成你的前端目錄
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

## 🟧 完整後端設定（純 HTTP + 反向代理）
C:\nginx\conf\conf.d\backend.conf
```sh
server {
    listen 5241;
    server_name backend.local;

    # 如果後端是 API
    location / {
        proxy_pass http://127.0.0.1:5000;  # 後端程式端口默認為 5000
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    # -------------------------
    # WebSocket 支援
    # -------------------------
    # location /ws/ {
    #     proxy_pass http://backend_api;
    #     proxy_set_header Host $host;
    #     proxy_http_version 1.1;
    #     proxy_set_header Upgrade $http_upgrade;
    #     proxy_set_header Connection "Upgrade";
    # }
}
```
→ 前端透過 http://localhost/api
 → 可 proxy 到 5000 port

## 🟩 如果你的需求是 “前後端都用同一個 Nginx 下同 port”
放一起就可以：
```sh
C:\nginx\www\frontend\
C:\nginx\www\backend\ (可選)
```
Nginx 設定：
```sh
server {
    listen 80;

    root C:/nginx/www/frontend;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /api/ {
        proxy_pass http://127.0.0.1:5000;
    }
}
```

---

## 🟩 ASP.NET Core（Kestrel）啟動方式
Nginx 只是反向代理，真正跑程式的是 Kestrel。
### ✅ 方法 A：直接執行（本機測試用）
在 cmd 執行：
```sh
cd C:\deploy\myapi\
MyApi.exe
```
API 預設跑在：
```sh
http://localhost:5000
```

### ✅ 方法 B：指定 Port（推薦）
在 appsettings.json 設定 Kestrel port：
```sh
"Kestrel": {
  "Endpoints": {
    "Http": {
      "Url": "http://localhost:5000"
    }
  }
}
```

### ✅ 方法 C：讓後端常駐（推薦）
用 NSSM 或 Windows Service：
```sh
nssm install MyApiService
```

設定：
- Path → C:\deploy\myapi\MyApi.exe
- Startup dir → C:\deploy\myapi\

然後：
```sh
nssm start MyApiService
```

---

# 🚀 刪除windows 服務
## ✅ 方法一：用 sc delete 刪除（最常用）
1. 開啟 命令提示字元（CMD），使用 系統管理員 執行
2. 查看服務名稱（不是顯示名稱）
```sh
sc query type= service state= all
```
找到你要刪除的「ServiceName」
3. 執行刪除命令：
```cpp
sc delete <ServiceName>
```

範例：
```cpp
sc delete MyService
```
➡️ 刪除後需 重新開機 才會完全消失。

## ✅ 方法二：使用 PowerShell 刪除
1. 用 service name：
```powershell
Get-Service -Name "MyService" | Remove-Service
```
PowerShell 7+ 可直接使用 Remove-Service。

## ✅ 方法三：從註冊表移除（最後手段）
### ⚠️ 只有當 service 卡住或無法刪除時使用。
1. Win+R → 輸入：regedit
2. 進入：
```sql
HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services
```
3. 找到你的服務資料夾（ServiceName）
4. 右鍵 → Delete
5. 重新開機

## ❗ 常見問題
Q1：刪除時回應「OpenService FAILED 5: Access is denied」？

原因：CMD 沒用管理員執行

✔ 右鍵「命令提示字元 → 以系統管理員執行」

Q2：刪除後仍然看到服務？

✔ 重開機

✔ 若是 Windows 在保護該服務（例如 Defender、系統服務）→ 無法刪除

Q3：服務刪除後 exe 檔案可以手動刪除嗎？

可以。

服務刪除後不會影響程式檔案，你可到原安裝路徑自行刪除。