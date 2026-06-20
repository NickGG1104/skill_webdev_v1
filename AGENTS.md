# Name: web-developer

## Description

Generates or refactors web projects using a standardized LAN-friendly stack — Vue 3 CDN Options API, Flask (Python 3.9+), SQLAlchemy with MySQL/SQLite dual support, Windows AD authentication via ldap3, Chart.js with animations, SweetAlert2, and inline SVG icons. All frontend files are served locally so the app runs in offline/intranet environments without any CDN dependency.

Use when: building a new web application from scratch, or refactoring an existing website into this standard.

---

## Instructions

### Input

| Type | Content |
|------|---------|
| New project | System name, feature requirements, data fields |
| Refactor | Existing code, project structure, or directory listing |
| Optional | DB preference (MySQL/SQLite), auth required, specific page needs |

---

### Steps

**Step 1 — Determine mode**

- User describes a new system or features → **MODE A: New Project**
- User provides existing code or project structure → **MODE B: Refactor**

---

**Step 2A — New Project workflow**

1. List proposed data models (fields, types, relationships) and page inventory
2. Check for high-concurrency signals (WebSocket, real-time updates, >500 simultaneous users)
   - If detected: ask "此需求可能需要高併發支援，是否改用 FastAPI？" before proceeding
3. Generate the full project following all constraints below

---

**Step 2B — Refactor workflow**

1. Read and analyze the existing project structure and tech stack
2. Output a deviation report — then **stop and wait for user confirmation**:

```
## Refactor Analysis

### Current stack
- Frontend: ...
- Backend: ...
- Database: ...

### Items to change
| Item | Current | Target | Scope |
|------|---------|--------|-------|

### Items already compliant
- ...

Proceed with refactor?
```

3. After confirmation: refactor file by file, announce each file before editing
4. Add `download_vendors.py` and `.env.example` after all files are complete

---

**Step 3 — Tech stack constraints**

**Frontend**

| Rule | Detail |
|------|--------|
| Framework | Vue 3 CDN, Options API only — no `setup()`, no `ref()`, no `reactive()`, no `.vue` files, no build tools |
| Icons | Inline SVG sprite only — define icons as `<symbol>` in a hidden `<svg>` block at the top of `<body>`, reference with `<svg class="icon"><use href="#ic-name"/></svg>`. No icon font libraries, no webfonts |
| Alerts | SweetAlert2 only — `alert()`, `confirm()`, `prompt()` are forbidden |
| Charts | Chart.js only — every instance must include `animation: { duration: 800, easing: 'easeInOutQuart' }` |
| HTTP | Axios for all API calls |
| CDN policy | All `<script>` and `<link>` tags must point to `static/vendor/` — no external CDN URLs in production HTML |
| CSS | Custom CSS with CSS custom properties — Tailwind is forbidden |
| Touch targets | All interactive elements minimum 44x44px |
| Responsive | Mobile-first, minimum supported width 320px |
| Typography | System font stack: `-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif` |

**Backend**

| Rule | Detail |
|------|--------|
| Framework | Flask (default); ask about FastAPI only when high-concurrency signals are detected |
| Python | 3.9+ compatible — no `match` statements, no `X \| Y` runtime union types |
| ORM | SQLAlchemy for all DB access — raw SQL strings are forbidden |
| DB switching | `DB_TYPE=sqlite` or `DB_TYPE=mysql` via `.env`; `config.py` handles the URI |
| API prefix | All routes under `/api/v1/` |
| API response | Always `{ "success": bool, "data": any, "message": str }` |
| Auth | `@login_required` decorator on all protected routes |
| Auth method | Windows AD via `ldap3` — session-based; include test-mode bypass when `LDAP_SERVER` is empty (accept `admin`/`admin`) |
| Jinja2 delimiter | In `create_app()`, always set `app.jinja_env.variable_start_string = '[['` and `app.jinja_env.variable_end_string = ']]'` to avoid conflict with Vue's `{{ }}` interpolation syntax |

---

**Step 4 — Pre-output checklist**

Verify before writing any file:

- [ ] No external CDN URLs in any HTML file
- [ ] No Tailwind classes
- [ ] No Composition API syntax
- [ ] No `alert()` / `confirm()` / `prompt()`
- [ ] No icon font libraries — inline SVG sprite only
- [ ] Every Chart.js instance has `animation: { duration, easing }`
- [ ] All interactive elements >= 44px touch target
- [ ] Python 3.9 compatible
- [ ] All DB access via SQLAlchemy ORM
- [ ] `download_vendors.py` included
- [ ] `.env.example` included

---

### Output

**A. Project structure**

```
{project_name}/
├── static/
│   ├── vendor/               # vue3, chart.js, sweetalert2, axios (local)
│   ├── css/main.css          # CSS custom properties + responsive layout
│   └── js/app.js             # Vue Options API app
├── templates/
│   └── index.html            # SVG sprite block + refs to static/vendor/ only
├── models/database.py        # SQLAlchemy models with to_dict()
├── routes/api.py             # RESTful CRUD returning standard JSON
├── routes/auth.py            # login_required + AD login/logout/me
├── services/ldap_service.py  # ldap3 wrapper with test-mode bypass
├── config.py                 # DB_TYPE switch + LDAP config from .env
├── app.py                    # create_app() factory
├── requirements.txt
├── download_vendors.py
└── .env.example
```

**B. Standard file templates**

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

def dl(url, path):
    os.makedirs(os.path.dirname(path), exist_ok=True)
    print(f'  {os.path.basename(path)}', end=' ... ', flush=True)
    urllib.request.urlretrieve(url, path)
    print('OK')

if __name__ == '__main__':
    for path, url in VENDORS.items(): dl(url, path)
    print('Done. Icons are inline SVG — no extra downloads needed.')
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

`services/ldap_service.py` — test-mode pattern
```python
def authenticate_ad(username: str, password: str):
    if not current_app.config.get('LDAP_SERVER'):
        if username == 'admin' and password == 'admin':
            return {'username': 'admin', 'display_name': 'Test Admin',
                    'email': 'admin@test.local', 'groups': []}
        return None
    # real ldap3 logic follows
```

**C. Deviation report** (Refactor mode — output then wait)

```
## Refactor Analysis

### Current stack
- Frontend: ...
- Backend: ...

### Items to change
| Item | Current | Target | Scope |
|------|---------|--------|-------|

### Items already compliant
- ...

Proceed with refactor?
```
