# webdev — Web Development Standard Skill

A Claude Code agent skill that enforces a consistent, production-ready web development standard for internal LAN applications. Covers both new project generation and refactoring of existing codebases.

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend Framework | Vue 3 CDN — Options API |
| Charts | Chart.js with animations |
| Alerts / Dialogs | SweetAlert2 |
| HTTP Client | Axios |
| Icons | Font Awesome 6 |
| CSS | Custom CSS (no Tailwind) |
| Backend | Python 3.9+ / Flask (FastAPI on request) |
| ORM | SQLAlchemy (MySQL + SQLite dual support) |
| Authentication | Windows Active Directory via `ldap3` |
| CDN Strategy | All frontend libraries downloaded locally — LAN/offline safe |

---

## Requirements

- [Claude Code](https://claude.ai/code) (for `/webdev` slash command)
- Python 3.9+ (for vendor download script and backend)
- Internet access once (for `download_vendors.py`)

---

## Installation

### Option A — Claude Code Slash Command (project-level)

Copy `.claude/commands/webdev.md` into your project:

```bash
mkdir -p your-project/.claude/commands
cp .claude/commands/webdev.md your-project/.claude/commands/webdev.md
```

Then inside your project directory, invoke with:

```
/webdev 幫我建一個庫存管理系統
```

### Option B — Universal System Prompt (all platforms)

Copy the contents of `AGENTS.md` and paste as the system prompt / custom instructions in any AI tool:

| Platform | Where to paste |
|----------|---------------|
| Claude.ai Projects | Project → Instructions |
| Cursor | `.cursor/rules/webdev.mdc` |
| Windsurf | `.windsurfrules` |
| GitHub Copilot | `.github/copilot-instructions.md` |
| ChatGPT Custom GPT | Configure → Instructions |
| OpenAI Codex CLI | `AGENTS.md` in project root (auto-loaded) |

---

## Usage

### New Project

Describe what you want to build:

```
/webdev 幫我建一個請假申請系統，功能包含申請、主管審核、查詢紀錄
```

The skill will:
1. Generate the full project structure
2. Write all backend (Flask) and frontend (Vue 3) code
3. Include `download_vendors.py` to localize all CDN files
4. Include `.env.example` for configuration

### Refactor Existing Project

Point the skill at existing code:

```
/webdev 這是現有專案，請重構成標準規範 [describe or paste existing structure]
```

The skill will:
1. Analyze the current tech stack
2. Output a **deviation report** (table of what needs to change)
3. Wait for your confirmation
4. Refactor file by file

---

## After Generating a Project

```bash
cd your-project

# 1. Download all frontend vendor files (run once)
python download_vendors.py

# 2. Copy and configure environment
cp .env.example .env
# Edit .env: set DB_TYPE, LDAP_SERVER, etc.

# 3. Install Python dependencies
pip install -r requirements.txt

# 4. Run
python app.py
```

---

## Design Rules Enforced

- **No Tailwind CSS** — custom CSS with CSS custom properties
- **No Composition API** — Vue 3 Options API only, no `setup()`, no `ref()`
- **No native alert/confirm** — SweetAlert2 for all user dialogs
- **Chart animations required** — every Chart.js instance must have `animation.duration` and `animation.easing`
- **Touch-friendly** — all interactive elements minimum 44×44px
- **Mobile-first responsive** — works at 320px minimum width
- **Python 3.9 compatible** — no `match`, no `X | Y` runtime union syntax
- **LAN safe** — zero external CDN dependencies at runtime

---

## High-Concurrency Handling

Flask is the default backend. If your requirements include:
- Real-time features / WebSocket
- Background task queues
- Simultaneous users exceeding ~500

The skill will ask: **"是否改用 FastAPI？"** before generating backend code.

---

## Authentication

Windows Active Directory authentication via LDAP (`ldap3`). Session-based (no JWT by default).

**Test mode**: If `LDAP_SERVER` is empty in `.env`, the backend accepts `admin / admin` for local development without a real AD server.

---

## File Structure

```
your-project/
├── .claude/commands/webdev.md   # Claude Code slash command
├── static/vendor/               # localized CDN files (after download_vendors.py)
├── static/css/main.css
├── static/js/app.js
├── templates/
├── models/database.py           # SQLAlchemy models
├── routes/api.py                # RESTful API  /api/v1/
├── routes/auth.py               # AD auth + login_required decorator
├── services/ldap_service.py     # LDAP connection wrapper
├── config.py                    # MySQL / SQLite switch via .env
├── app.py
├── requirements.txt
├── download_vendors.py
└── .env.example
```

---

## Test Project

A working Task Management Dashboard is included in `test-project/` to verify the skill output end-to-end.

```bash
cd test-project
python download_vendors.py
pip install -r requirements.txt
cp .env.example .env
python app.py
# open http://localhost:5000
# login: admin / admin  (test mode, no real AD needed)
```

Features demonstrated:
- Windows AD login (with test bypass)
- Task CRUD with SweetAlert2 dialogs
- Chart.js doughnut + bar charts with animations
- Responsive mobile layout
- Font Awesome icons
- Axios API calls
- Vue 3 Options API throughout

---

## Contributing

Issues and PRs welcome. When proposing changes to the skill rules, update both:
- `.claude/commands/webdev.md` (Claude Code version)
- `AGENTS.md` (universal version)

Keep both files in sync.
