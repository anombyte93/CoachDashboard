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

### Slash Command
- `/coach-checkin` - Interactive weekly check-in via Claude terminal
- Located at `~/.claude/commands/coach-checkin.md`
- Collects WHOOP, nutrition, wellbeing, training data
- Generates AI insights and saves JSON to `data/weeks/`

### Training Data Export Research (Dec 2025)
- **TrainHeroic**: NO export capability (closed system)
- **Liftoff**: NO export capability (closed system)
- Both require manual data entry - no programmatic import possible
