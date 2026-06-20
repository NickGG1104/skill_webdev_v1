# Name: web-developer

## Description

當使用者需要**建立新網頁專案**或將**現有專案重構**為統一規範時觸發。
套用固定技術棧：Vue 3 CDN（Options API）、Flask、SQLAlchemy、Windows AD 認證、Chart.js、SweetAlert2，所有前端套件本地化以支援區網離線運行。

觸發情境：
- 「幫我做一個 XX 系統」
- 「把這個專案重構成標準規範」
- 「照規範建立 XX 功能」

---

## Instructions

### Input

| 類型 | 內容 |
|------|------|
| 新建專案 | 使用者描述的系統名稱、功能需求、資料欄位 |
| 重構專案 | 現有程式碼、專案結構描述、或目錄清單 |
| 選填補充 | 資料庫偏好（MySQL/SQLite）、是否需要認證、特定頁面需求 |

---

### Steps

**Step 1 — 判斷模式**

- 有新需求描述 → **MODE A（新建）**
- 有現有程式碼/專案 → **MODE B（重構）**

---

**Step 2A — 新建模式**

1. 根據需求列出資料模型（欄位、型別、關聯）
2. 列出頁面清單與功能對應
3. 確認是否涉及高併發（WebSocket / 即時更新 / 大量並發請求）
   - 是 → 詢問使用者：「此需求可能需要高併發支援，是否改用 FastAPI？」
   - 否 → 使用 Flask
4. 依照輸出規格產生完整專案

---

**Step 2B — 重構模式**

1. 讀取並分析現有檔案結構與技術棧
2. 輸出**偏差報告**（格式見 Output 區塊），停止並等待使用者確認
3. 確認後逐檔重構，每個檔案動作前先宣告「正在處理：[檔名]」
4. 全部完成後補充 `download_vendors.py` 與 `.env.example`

---

**Step 3 — 套用技術規範**（新建與重構皆適用）

**前端限制**
- Vue 3 CDN，**只用 Options API**（禁止 `setup()`、`ref()`、`reactive()`、Composition API）
- 禁止 Tailwind CSS，使用 CSS 自定義屬性
- 所有 alert/confirm 使用 **SweetAlert2**，禁止原生 `alert()` / `confirm()`
- 所有圖表使用 **Chart.js**，每個圖表必須設定 `animation: { duration: 800, easing: 'easeInOutQuart' }`
- 所有 HTML `<script>` / `<link>` 只指向 `static/vendor/`，禁止外部 CDN URL
- 互動元素最小觸控區域 44×44px，RWD Mobile-first（最小支援 320px）

**後端限制**
- Python 3.9 相容（禁止 `match`、`X | Y` 執行期型別聯集）
- 所有 DB 操作透過 SQLAlchemy ORM，禁止原始 SQL 字串
- API 路由統一前綴 `/api/v1/`
- 所有 API 回傳格式：`{ "success": bool, "data": any, "message": str }`
- 保護路由使用 `@login_required` 裝飾器

**字型**：系統字型堆疊 `-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Arial, sans-serif`

---

**Step 4 — 執行前檢查清單**

輸出任何程式碼前，逐項確認：
- [ ] HTML 無外部 CDN URL
- [ ] 無 Tailwind 類別
- [ ] 無 Composition API 語法
- [ ] 無原生 `alert()` / `confirm()`
- [ ] 每個 Chart.js 實例有 `animation` 設定
- [ ] 觸控區域 ≥ 44px
- [ ] Python 3.9 相容
- [ ] 所有 DB 存取透過 ORM
- [ ] 包含 `download_vendors.py`
- [ ] 包含 `.env.example`

---

### Output

**A. 新建專案 — 標準目錄結構**

```
{project_name}/
├── static/
│   ├── vendor/           # 本地化套件（vue3、chart.js、sweetalert2、axios、fontawesome）
│   ├── css/main.css      # CSS 變數 + RWD 樣式
│   └── js/app.js         # Vue Options API 主應用
├── templates/
│   └── index.html        # 僅引用 static/vendor/ 路徑
├── models/database.py    # SQLAlchemy models + to_dict()
├── routes/api.py         # RESTful CRUD，回傳標準 JSON
├── routes/auth.py        # login_required 裝飾器 + AD 路由
├── services/ldap_service.py  # ldap3 封裝（含測試模式 bypass）
├── config.py             # DB_TYPE 環境變數切換 MySQL/SQLite
├── app.py                # create_app() + Blueprint 註冊
├── requirements.txt
├── download_vendors.py   # 一鍵下載所有前端套件至 static/vendor/
└── .env.example
```

**B. 重構模式 — 偏差報告格式**（輸出後停止，等待確認）

```
## 重構分析報告

### 現有技術棧
- 前端：...
- 後端：...
- 資料庫：...

### 需調整項目
| 項目 | 現況 | 目標 | 影響範圍 |
|------|------|------|---------|
| ... | ... | ... | ... |

### 保留項目（已符合規範）
- ...

確認後開始重構，是否繼續？
```

**C. 必要樣板**

`config.py` — DB 切換
```python
import os
from dotenv import load_dotenv
load_dotenv()

class Config:
    SECRET_KEY = os.getenv('SECRET_KEY', 'change-me')
    DB_TYPE = os.getenv('DB_TYPE', 'sqlite')
    if DB_TYPE == 'mysql':
        SQLALCHEMY_DATABASE_URI = (
            f"mysql+pymysql://{os.getenv('DB_USER')}:{os.getenv('DB_PASS')}"
            f"@{os.getenv('DB_HOST')}:{os.getenv('DB_PORT', 3306)}/{os.getenv('DB_NAME')}"
        )
    else:
        SQLALCHEMY_DATABASE_URI = 'sqlite:///app.db'
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    LDAP_SERVER  = os.getenv('LDAP_SERVER', '')
    LDAP_DOMAIN  = os.getenv('LDAP_DOMAIN', '')
    LDAP_BASE_DN = os.getenv('LDAP_BASE_DN', '')
```

`services/ldap_service.py` — 含測試模式
```python
import ldap3
from flask import current_app

def authenticate_ad(username: str, password: str):
    # 測試模式：LDAP_SERVER 未設定時允許 admin/admin
    if not current_app.config.get('LDAP_SERVER'):
        if username == 'admin' and password == 'admin':
            return {'username': 'admin', 'display_name': '測試帳號',
                    'email': 'admin@test.local', 'groups': []}
        return None
    server = ldap3.Server(current_app.config['LDAP_SERVER'], get_info=ldap3.ALL)
    try:
        conn = ldap3.Connection(
            server,
            user=f"{current_app.config['LDAP_DOMAIN']}\\{username}",
            password=password, auto_bind=True,
        )
        conn.search(current_app.config['LDAP_BASE_DN'],
                    f'(sAMAccountName={username})',
                    attributes=['displayName', 'mail', 'memberOf'])
        if conn.entries:
            e = conn.entries[0]
            return {'username': username, 'display_name': str(e.displayName),
                    'email': str(e.mail), 'groups': [str(g) for g in e.memberOf]}
    except ldap3.core.exceptions.LDAPBindError:
        return None
    return None
```

`download_vendors.py`
```python
"""Run once to download all frontend vendor files for LAN/offline use."""
import urllib.request, os

VENDORS = {
    'static/vendor/vue3.global.prod.min.js':
        'https://cdn.jsdelivr.net/npm/vue@3/dist/vue.global.prod.min.js',
    'static/vendor/chart.umd.min.js':
        'https://cdn.jsdelivr.net/npm/chart.js/dist/chart.umd.min.js',
    'static/vendor/sweetalert2.all.min.js':
        'https://cdn.jsdelivr.net/npm/sweetalert2@11/dist/sweetalert2.all.min.js',
    'static/vendor/axios.min.js':
        'https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js',
}
FA = 'https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.0'
FA_FONTS = ['fa-solid-900.woff2','fa-regular-400.woff2','fa-brands-400.woff2',
            'fa-solid-900.ttf','fa-regular-400.ttf','fa-brands-400.ttf']

def dl(url, path):
    os.makedirs(os.path.dirname(path), exist_ok=True)
    print(f'  {os.path.basename(path)}', end=' ... ', flush=True)
    urllib.request.urlretrieve(url, path)
    print('OK')

if __name__ == '__main__':
    for path, url in VENDORS.items(): dl(url, path)
    dl(f'{FA}/css/all.min.css', 'static/vendor/fontawesome/css/all.min.css')
    for f in FA_FONTS: dl(f'{FA}/webfonts/{f}', f'static/vendor/fontawesome/webfonts/{f}')
    print('Done.')
```

`requirements.txt`
```
flask>=2.3.0
flask-cors>=4.0.0
flask-sqlalchemy>=3.1.0
sqlalchemy>=2.0.0
pymysql>=1.1.0
ldap3>=2.9.1
python-dotenv>=1.0.0
cryptography>=41.0.0
```
