# web-developer

A standardized Agent Skill for building or refactoring web applications on LAN/intranet environments. All frontend libraries are served locally — no CDN dependency at runtime.

---

## Tech stack

| Layer | Technology |
|-------|-----------|
| Frontend framework | Vue 3 CDN, Options API |
| Charts | Chart.js with animations |
| Alerts / Dialogs | SweetAlert2 |
| HTTP client | Axios |
| Icons | Inline SVG sprite (no webfonts, no icon libraries) |
| CSS | Custom CSS, no Tailwind |
| Backend | Python 3.9+ / Flask |
| ORM | SQLAlchemy (MySQL + SQLite dual support) |
| Authentication | Windows Active Directory via ldap3 |

---

## File structure

```
Skills/
├── .claude/skills/web-developer/
│   └── SKILL.md          # Agent Skills format (Claude Code / claude.ai / API)
├── AGENTS.md             # Universal system prompt (Cursor, Windsurf, ChatGPT, etc.)
├── README.md
└── test-project/         # Working demo — Task Management Dashboard
```

---

## Installation

### Claude Code (Agent Skills — auto-triggered)

Copy the skill directory into your project:

```bash
mkdir -p your-project/.claude/skills/web-developer
cp .claude/skills/web-developer/SKILL.md your-project/.claude/skills/web-developer/SKILL.md
```

Claude will automatically use the skill when you ask to build or refactor a web project. No slash command needed.

### Other platforms (universal system prompt)

Paste the contents of `AGENTS.md` as the system prompt / custom instructions:

| Platform | Where to paste |
|----------|---------------|
| Claude.ai Projects | Project > Instructions |
| Cursor | `.cursor/rules/webdev.mdc` |
| Windsurf | `.windsurfrules` |
| GitHub Copilot | `.github/copilot-instructions.md` |
| ChatGPT Custom GPT | Configure > Instructions |
| OpenAI Codex CLI | `AGENTS.md` in project root |

---

## Usage

### New project

Describe what you want to build. Claude detects the skill and applies the standard:

```
幫我做一個請假申請系統，功能包含申請、主管審核、查詢紀錄
```

### Refactor existing project

```
把這個 React 專案重構成標準規範
```

Claude will output a deviation report first and wait for confirmation before making changes.

---

## After generating a project

```bash
cd your-project

# 1. Download all frontend vendor files (run once, requires internet)
python download_vendors.py

# 2. Configure environment
cp .env.example .env
# Edit .env — set DB_TYPE, LDAP_SERVER, etc.

# 3. Install Python dependencies
pip install -r requirements.txt

# 4. Run
python app.py
```

---

## Test project

A working Task Management Dashboard is in `test-project/` to verify the skill output end-to-end.

```bash
cd test-project
python download_vendors.py
pip install -r requirements.txt
cp .env.example .env
python app.py
# open http://localhost:5100
# login: admin / admin  (test mode, no AD server required)
```

Features demonstrated:

- Windows AD login with test-mode bypass
- Task CRUD with SweetAlert2 dialogs and toast notifications
- Chart.js doughnut and bar charts with animations
- Inline SVG icon sprite (zero network requests for icons)
- Responsive mobile layout with sidebar drawer
- Axios API calls with consistent JSON response format
- Vue 3 Options API throughout

---

## Rules enforced

- No Tailwind CSS
- No Composition API (`setup()`, `ref()`, `reactive()`)
- No `alert()` / `confirm()` — SweetAlert2 only
- No icon font libraries or webfonts — inline SVG sprite only
- Chart animations required on every chart instance
- Touch targets minimum 44x44px
- Python 3.9 compatible syntax
- All DB queries through SQLAlchemy ORM
- No external CDN URLs in production HTML

---

## High-concurrency handling

Flask is the default. If requirements involve WebSocket, real-time features, or more than ~500 simultaneous users, the skill will ask before proceeding:

"此需求可能需要高併發支援，是否改用 FastAPI？"

---

## Contributing

When updating rules, keep both files in sync:

- `.claude/skills/web-developer/SKILL.md` — Agent Skills format
- `AGENTS.md` — universal system prompt format
