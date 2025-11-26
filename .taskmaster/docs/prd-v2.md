# Product Requirements Document (PRD)
# Coach Dashboard v2 - Admin Panel & Enhanced Features

**Version:** 2.0
**Date:** November 26, 2025
**Author:** Claude Code (via WWSJD validation)
**Status:** Planning

---

## 1. Executive Summary

### The Problem
The original Coach Dashboard (v1) is a working static single-file HTML dashboard. However, it requires:
- Manual JSON editing for data updates
- No historical week-by-week tracking
- No authentication (URL-based security only)
- Manual data extraction from WHOOP/MacroFactor exports
- No AI-powered insights or progress analysis
- No notification system for weekly reminders

### The Solution
Create **Coach Dashboard v2** - a copy of the original with:
1. **Admin Panel** - Protected interface for data management
2. **Authentication** - Simple coach login (username: coach, password changeable)
3. **Week-by-Week Navigation** - Historical data viewing with tabs
4. **Data Import** - Automated WHOOP/MacroFactor CSV parsing
5. **AI Insights** - Weekly analysis with week-over-week comparisons
6. **Progress Photos** - Upload and AI-powered visual analysis
7. **Notifications** - NTFY integration for Sunday/Monday reminders
8. **PDF Export** - Easy one-click PDF generation for coach

### Key Constraint
**Original index.html must remain untouched** - Coach already has access to current dashboard. All new functionality goes in a separate file structure.

---

## 2. Architecture Overview

### File Structure
```
CoachDashboard/
├── index.html                    # ORIGINAL - DO NOT MODIFY
├── v2/
│   ├── index.html               # New dashboard with week tabs + PDF export
│   ├── admin.html               # Admin panel (protected)
│   ├── login.html               # Authentication page
│   ├── css/
│   │   ├── dashboard.css        # Dashboard styles
│   │   └── admin.css            # Admin panel styles
│   ├── js/
│   │   ├── auth.js              # Authentication logic
│   │   ├── dashboard.js         # Dashboard rendering + charts
│   │   ├── admin.js             # Admin panel logic
│   │   ├── data-import.js       # WHOOP/MacroFactor CSV parsing
│   │   ├── ai-insights.js       # AI-powered analysis
│   │   └── pdf-export.js        # PDF generation
│   └── data/
│       ├── weeks/               # Historical week data (JSON)
│       │   ├── 2025-W47.json
│       │   └── ...
│       ├── config.json          # Dashboard configuration
│       └── progress-photos/     # Photo storage references
├── backend/                      # Minimal backend (optional)
│   ├── server.js                # Express server
│   ├── auth.js                  # Auth endpoints
│   └── ntfy.js                  # Notification scheduler
└── .taskmaster/docs/
    └── prd-v2.md                # This document
```

### Technology Stack
- **Frontend:** Vanilla JS, HTML5, CSS3 (matching v1 aesthetic)
- **Charts:** Chart.js 4.4.1 (same as v1)
- **PDF Export:** html2pdf.js or browser print-to-PDF with explicit button
- **Authentication:** Simple bcrypt-hashed password stored in localStorage (admin), session token
- **Data Storage:** JSON files (week data), localStorage (config/session)
- **Backend (Optional):** Minimal Node.js for NTFY cron and password changes
- **AI Integration:** Anthropic Claude API for insights generation
- **Notifications:** ntfy.sh for Sunday/Monday reminders

---

## 3. Feature Requirements

### 3.1 Authentication System (P0)

**Login Flow:**
1. User visits `/v2/` - redirected to `/v2/login.html` if no session
2. Login form: username + password
3. Default credentials: `coach` / `coach`
4. On success: store session token in localStorage, redirect to dashboard
5. Session expiry: 7 days (configurable)

**Password Management:**
- Admin panel allows password change
- Old password required to set new password
- Password stored as bcrypt hash in config
- For static hosting: hash stored in encrypted localStorage

**Implementation Notes:**
- No heavy auth library - vanilla JS with Web Crypto API
- Session token: random UUID, validated against stored value
- Logout: clear session from localStorage

### 3.2 Week-by-Week Navigation (P0)

**Week Tabs Interface:**
- Horizontal scrollable tab bar at top of dashboard
- Each tab shows: "Week of Nov 16" format
- Current week highlighted
- Click tab to load that week's data

**Data Organization:**
- Each week stored as separate JSON file: `data/weeks/2025-W47.json`
- Week key format: ISO week (YYYY-Www)
- Latest week auto-selected on load

**Week-over-Week Insights:**
- Side panel or section showing comparisons
- "vs Previous Week" indicators on key metrics
- Trend arrows (up/down/stable)

### 3.3 Admin Panel (P0)

**Access:**
- `/v2/admin.html` - requires authentication
- Navigation: Dashboard ↔ Admin via header buttons

**Admin Features:**
1. **Current Week Data Entry**
   - Form fields for all measurements
   - Training notes textarea
   - Nutrition summary
   - Well-being checklist

2. **WHOOP Data Import**
   - File upload for WHOOP CSV exports
   - Auto-parse and extract current week
   - Preview parsed data before saving
   - Support for: workouts.csv, sleep.csv, recovery.csv

3. **MacroFactor Data Import**
   - File upload for MacroFactor CSV
   - Parse daily calories, protein, carbs, fat
   - Calculate weekly averages
   - Support for nutrition export format

4. **Configuration Options**
   - Target calories, protein, carbs, fat
   - Target sleep hours
   - Recovery thresholds (green/yellow/red)
   - NTFY topic name for notifications
   - AI API key (for insights generation)

5. **Progress Photos**
   - Upload front/side/back photos per week
   - Photo preview gallery
   - AI analysis trigger button

6. **Password Change**
   - Current password field
   - New password + confirmation
   - Validation feedback

### 3.4 Data Import Automation (P1)

**WHOOP CSV Parser:**
```javascript
// Expected columns from WHOOP export
const whoopFields = {
  recovery: ['date', 'recovery_score', 'hrv', 'resting_heart_rate'],
  sleep: ['date', 'total_sleep_duration', 'efficiency', 'need', 'debt'],
  strain: ['date', 'strain', 'average_heart_rate', 'max_heart_rate']
};
```

**MacroFactor CSV Parser:**
```javascript
// Expected columns from MacroFactor export
const macroFields = {
  nutrition: ['date', 'calories', 'protein', 'carbs', 'fat'],
  weight: ['date', 'weight', 'body_fat_percentage']
};
```

**Auto-Extract Current Week:**
- Parse all rows from CSV
- Filter to current ISO week dates
- Aggregate daily data
- Calculate averages
- Populate admin form or save directly

### 3.5 AI-Powered Insights (P1)

**Weekly Summary Generation:**
- Input: Current week's full data JSON
- Output: 3-5 sentence summary of key observations
- Highlights: Recovery trends, nutrition adherence, training load balance

**Week-over-Week Analysis:**
- Input: Current week + previous 4 weeks data
- Output: Trend analysis, improvement areas, concerns
- Format: Bullet points with recommendations

**Progress Photo Analysis:**
- Input: Progress photos (front/side/back)
- Output: Visual observations (posture, composition changes)
- Privacy: Photos processed, not stored by AI
- Trigger: Manual button click only

**Implementation:**
- Use Anthropic Claude API (claude-3-sonnet or haiku for cost)
- Prompt templates stored in config
- Rate limiting: 1 insight generation per week
- Fallback: If no API key, skip AI features

### 3.6 NTFY Notification System (P1)

**Notification Schedule:**
- **Sunday 6pm:** "Don't forget to export your WHOOP and MacroFactor data for this week!"
- **Monday 9am:** "Weekly check-in reminder: Update your Coach Dashboard before your call"

**Implementation Options:**

**Option A: Backend Cron (Recommended)**
```javascript
// backend/ntfy.js
const cron = require('node-cron');
const fetch = require('node-fetch');

cron.schedule('0 18 * * 0', () => { // Sunday 6pm
  sendNtfyNotification('Export data reminder');
});

cron.schedule('0 9 * * 1', () => { // Monday 9am
  sendNtfyNotification('Weekly check-in reminder');
});

async function sendNtfyNotification(message) {
  await fetch('https://ntfy.sh/coach-dashboard-reminders', {
    method: 'POST',
    headers: { 'Title': 'Coach Dashboard', 'Priority': '3' },
    body: message
  });
}
```

**Option B: Local/Self-Hosted**
- Use local ntfy server at localhost:8889
- Shell script with systemd timer
- Reference: `~/.claude/lib/ntfy-templates.sh`

**Configuration:**
- NTFY topic configurable in admin panel
- Enable/disable toggle for notifications
- Custom reminder times (optional)

### 3.7 PDF Export (P0)

**Requirements:**
- One-click PDF generation from dashboard
- Clean, professional output matching print styles
- Include all charts, metrics, and notes
- Filename: `Coach-Dashboard-Week-of-[date].pdf`

**Implementation:**
```javascript
// Using html2pdf.js for better control than browser print
async function exportPDF() {
  const element = document.getElementById('dashboard-content');
  const options = {
    margin: 10,
    filename: `Coach-Dashboard-Week-of-${currentWeekDate}.pdf`,
    image: { type: 'jpeg', quality: 0.98 },
    html2canvas: { scale: 2 },
    jsPDF: { unit: 'mm', format: 'a4', orientation: 'portrait' }
  };
  await html2pdf().set(options).from(element).save();
}
```

**UI:**
- Prominent "Download PDF" button in header
- Icon: Download/document icon
- Loading state during generation
- Success toast on completion

### 3.8 Progress Photos (P2)

**Upload Interface:**
- Drag-and-drop zone in admin panel
- Support for: JPEG, PNG, WebP
- Max size: 5MB per photo
- Categories: Front, Side, Back

**Storage:**
- Base64 encode small photos in localStorage
- For larger: Reference external storage (S3, Cloudflare R2)
- Metadata stored in week JSON

**Gallery View:**
- Before/after comparison slider
- Select two weeks to compare
- Zoom on click

**AI Analysis:**
- Send photos to vision model
- Generate observations
- Display alongside photos

---

## 4. User Interface Design

### 4.1 Dashboard (v2/index.html)

**Header:**
- Week selector tabs (scrollable)
- Current week highlighted
- "Download PDF" button (prominent)
- "Admin" button (if authenticated)
- Logout button

**Body:**
- Identical layout to v1 (preserve design)
- Added: Week-over-week comparison indicators
- Added: AI Insight card (if generated)

**Footer:**
- "Powered by Coach Dashboard v2"
- Link to admin panel

### 4.2 Admin Panel (v2/admin.html)

**Sidebar Navigation:**
- Data Entry
- Import Data
- Progress Photos
- Configuration
- Notifications
- Account

**Main Content Area:**
- Form sections based on nav selection
- Live preview of dashboard data
- Save/Cancel buttons

### 4.3 Login Page (v2/login.html)

**Simple Form:**
- Logo/title
- Username field
- Password field
- "Remember me" checkbox
- Login button
- Error message display

---

## 5. Configuration Schema

```json
{
  "version": "2.0",
  "auth": {
    "passwordHash": "bcrypt-hash-here",
    "sessionExpiry": 604800000
  },
  "targets": {
    "calories": 2725,
    "protein": 195,
    "carbs": 314,
    "fat": 102,
    "sleep": 8.0,
    "steps": 8000
  },
  "thresholds": {
    "recovery": {
      "good": 70,
      "caution": 50
    }
  },
  "notifications": {
    "enabled": true,
    "ntfyTopic": "coach-dashboard-reminders",
    "sundayTime": "18:00",
    "mondayTime": "09:00"
  },
  "ai": {
    "enabled": false,
    "apiKey": "",
    "model": "claude-3-haiku-20240307"
  }
}
```

---

## 6. Week Data Schema (Enhanced)

```json
{
  "weekKey": "2025-W47",
  "weekOf": "2025-11-17",
  "createdAt": "2025-11-24T10:00:00Z",
  "updatedAt": "2025-11-24T10:00:00Z",

  "measurements": {
    "weight": 116,
    "waist": 86,
    "chest": 120,
    "middle": 102,
    "butt": 110,
    "leg": 68,
    "steps": 56042
  },

  "training": {
    "overall_performance": "...",
    "completed": "...",
    "difficulty": "...",
    "recovery": "...",
    "enjoyment": "..."
  },

  "nutrition": {
    "daily": [...],
    "averages": {...},
    "targets": {...},
    "notes": "..."
  },

  "whoop": {
    "daily": [...],
    "averages": {...},
    "sleep_metrics": {...},
    "interpretation": "..."
  },

  "wellbeing": {
    "pain": "...",
    "appetite": "...",
    "cravings": "...",
    "stress": "...",
    "mood": "...",
    "recovery_outside_gym": "...",
    "lifestyle_factors": "..."
  },

  "wins": [...],
  "coach_questions": "...",

  "progressPhotos": {
    "front": "base64-or-url",
    "side": "base64-or-url",
    "back": "base64-or-url"
  },

  "aiInsights": {
    "weekly": "AI-generated weekly summary...",
    "comparison": "AI-generated week-over-week analysis...",
    "photoAnalysis": "AI-generated photo observations...",
    "generatedAt": "2025-11-24T10:00:00Z"
  },

  "comparison": {
    "weightDelta": -0.5,
    "caloriesAvgDelta": +200,
    "recoveryAvgDelta": -3,
    "sleepAvgDelta": +0.2
  }
}
```

---

## 7. Implementation Phases

### Phase 1: Foundation (P0) - Week 1
1. Create v2 directory structure
2. Copy v1 styling and chart setup
3. Implement authentication system
4. Create login page
5. Create admin panel skeleton
6. Add PDF export functionality

### Phase 2: Data Management (P0) - Week 1
7. Implement week-by-week data storage
8. Create week selector tabs
9. Build admin data entry forms
10. Implement WHOOP CSV parser
11. Implement MacroFactor CSV parser
12. Add week comparison indicators

### Phase 3: Enhanced Features (P1) - Week 2
13. Integrate AI insights generation
14. Add progress photo upload
15. Implement NTFY notifications
16. Add configuration panel
17. Photo gallery and comparison

### Phase 4: Polish & Testing (P1) - Week 2
18. Responsive testing (mobile/tablet/desktop)
19. PDF export testing across browsers
20. Authentication security review
21. Performance optimization
22. Documentation and help text

---

## 8. Acceptance Criteria

### Authentication
- [ ] Login redirects unauthenticated users
- [ ] Default credentials work (coach/coach)
- [ ] Password can be changed in admin
- [ ] Session persists for 7 days
- [ ] Logout clears session

### Week Navigation
- [ ] All historical weeks accessible via tabs
- [ ] Week data loads correctly on tab click
- [ ] Current week auto-selected
- [ ] Week-over-week comparisons displayed

### Admin Panel
- [ ] All data fields editable
- [ ] Changes save correctly
- [ ] WHOOP CSV imports work
- [ ] MacroFactor CSV imports work
- [ ] Configuration changes persist

### PDF Export
- [ ] Export button visible and prominent
- [ ] PDF generates correctly
- [ ] All charts render in PDF
- [ ] Filename includes week date
- [ ] Works on mobile and desktop

### Notifications
- [ ] NTFY notifications send on schedule
- [ ] Notification content is helpful
- [ ] Can be disabled in config

### AI Insights (if enabled)
- [ ] Weekly summary generates
- [ ] Week comparison generates
- [ ] Photo analysis works
- [ ] Graceful degradation without API key

---

## 9. Security Considerations

1. **Password Storage:** bcrypt hash, never plaintext
2. **Session Tokens:** Cryptographically random UUIDs
3. **API Keys:** Stored in config, not exposed to frontend
4. **Progress Photos:** Client-side only, no server upload
5. **NTFY Topic:** Unique per user, not guessable
6. **XSS Prevention:** Sanitize all user inputs
7. **CORS:** Restrict API calls if backend used

---

## 10. Rollback Plan

If v2 causes issues:
1. Original `index.html` remains untouched
2. Coach can always access original URL
3. v2 is completely separate path
4. No data migration required - v1 continues working

---

## 11. Success Metrics

| Metric | Target |
|--------|--------|
| Weekly data update time | < 2 minutes (from 5+ min) |
| PDF export success rate | 100% |
| Week comparison accuracy | 100% |
| Notification delivery rate | 95%+ |
| AI insight usefulness | Qualitative feedback |

---

**Document Status:** Ready for Task Master Parsing
**Estimated Build Time:** 2 weeks
**Key Innovation:** Progressive enhancement without breaking existing system
