# Claude Code Instructions

## Task Master AI Instructions
**Import Task Master's development workflow commands and guidelines, treat as if import is in the main CLAUDE.md file.**
@./.taskmaster/CLAUDE.md

## Project: Coach Dashboard

**Live URL**: https://anombyte93.github.io/CoachDashboard/
**Repo**: https://github.com/anombyte93/CoachDashboard

### Authentication
- **Coach login**: `coach` / `coach` (Dashboard view only)
- **Admin login**: `admin` / `hayden123` (Dashboard + Check-In + Admin Panel)
- Hashes are SHA-256 in `v2/login.html`
- Sessions stored in localStorage as `coachDashboardSession` with 7-day expiry

### Key Files
| File | Purpose |
|------|---------|
| `v2/index.html` | Main dashboard (auth-protected) |
| `v2/login.html` | Login page with SHA-256 auth |
| `v2/admin.html` | Admin panel (admin-only) |
| `checkin.html` | Weekly check-in form (admin-only, has CSV import) |
| `data/weeks/*.json` | Weekly data files |

### Features
- **File Import**: WHOOP CSV and MacroFactor CSV import in checkin.html
- **Multi-week History**: 8 weeks shown dynamically via `generateRecentWeeks()`
- **Role-based Nav**: Admin users see "⚙️ Admin Panel" and "Check-In" buttons
- **Graceful No-Data**: Shows helpful message when no data for selected week

### Common Issues & Fixes

**JS Crash - Duplicate Variable Declaration**
- **Symptom**: Page stuck on "Loading insight...", no admin buttons visible
- **Cause**: Duplicate `const` declaration in same scope (e.g., `const recoveryEl` declared twice)
- **Fix**: Remove duplicate declaration, reuse existing variable
- **Lesson**: Always check for duplicate `const`/`let` declarations when JS silently fails

**Login Not Working**
- **Symptom**: Correct password rejected
- **Cause**: SHA-256 hash mismatch between code and actual password
- **Debug**: `echo -n "password" | sha256sum` to verify hash
- **Fix**: Update hash in CREDENTIALS object in login.html

**Session Persisting After Logout**
- **Symptom**: Auto-redirects to dashboard after logout
- **Fix**: Logout redirects to `login.html?logout=true` which forces session clear

**Admin Buttons Not Showing**
- **Symptom**: Logged in as admin but no Admin Panel button
- **Cause**: JS error before `checkAdminRole()` runs, or session role !== 'admin'
- **Debug**: Browser console for errors, check `localStorage.getItem('coachDashboardSession')`

### Slash Command: `/coach-checkin`

**Automated Weekly Check-in Workflow**

Located at `~/.claude/commands/coach-checkin.md`

**Weekly Workflow:**
1. Export WHOOP/MacroFactor → email to self
2. Run `/coach-checkin` in Claude Code
3. Auto-imports data from Gmail via MCP
4. Answer wellbeing/training questions (form-based)
5. AI generates coach summary with insights
6. Data saved to dashboard (validated automatically)
7. Optional: Email notification to coach

**Helper Scripts** at `~/.claude/scripts/coach-checkin/`:
| Script | Purpose |
|--------|---------|
| `fetch-whoop.sh` | Gmail MCP instructions for WHOOP export |
| `fetch-macrofactor.sh` | Gmail MCP instructions for MacroFactor |
| `aggregate-metrics.py` | Combines all data into dashboard JSON |
| `validate-dashboard.sh` | Validates JSON schema (auto-fixes issues) |
| `send-coach-email.sh` | Generates coach notification email |

**Data Files:**
- Weekly data: `data/weeks/YYYY-MM-DD.json` and `v2/data/weeks/YYYY-MM-DD.json`
- Coach config: `data/coach-config.json` (email, notifications_enabled)

**Dashboard Features:**
- AI Coach Summary section displays insights, risk flags, recommendations
- Traffic light indicators for key metrics
- Auto-generated interpretation of WHOOP/nutrition data

### Training Data Export Research (Dec 2025)
- **TrainHeroic**: NO export capability (closed system)
- **Liftoff**: NO export capability (closed system)
- Both require manual data entry - no programmatic import possible

### JSON Schema Reference

Weekly data files must include:
```json
{
  "weekOf": "YYYY-MM-DD",
  "measurements": { "weight": N, ... },
  "training": { "overall_performance": "...", ... },
  "nutrition": { "daily": [...], "averages": {...}, "targets": {...} },
  "whoop": { "daily": [...], "averages": {...}, "interpretation": "..." },
  "wellbeing": { "pain": "...", "stress": "...", ... },
  "wins": ["...", ...],
  "coach_summary": {
    "executive_summary": "...",
    "risk_flags": ["...", ...],
    "recommendations": ["...", ...]
  },
  "insights": {
    "readiness": "Good/Moderate/Poor",
    "fatigue_risk": "Low/Moderate/High",
    "recovery_notes": "...",
    "nutrition_notes": "...",
    "sleep_notes": "..."
  },
  "recommendations": ["...", ...]
}
```
