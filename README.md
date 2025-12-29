# 🏈 NFL Picks League - Complete Documentation

## Table of Contents

1. [Overview](#overview)
2. [System Architecture](#system-architecture)
3. [Weekly Workflow](#weekly-workflow)
4. [File Reference](#file-reference)
5. [Configuration Guide](#configuration-guide)
6. [Scoring Rules](#scoring-rules)
7. [Troubleshooting](#troubleshooting)
8. [End of Season Tasks](#end-of-season-tasks)

---

## Overview

Your NFL Picks League is a fully automated system that allows 12 players to submit weekly game predictions and tracks scores throughout the season.

### Key Features

- **Player picks submission** via PIN-protected web form
- **Live score fetching** from ESPN API
- **Automatic winner detection** based on game results
- **Smart scoring** that handles normal weeks, Thanksgiving, Christmas, Week 18, and international games
- **Live schedule display** - no manual schedule maintenance needed!
- **Cloud storage** via JSONBin.io - works from any device

### URLs

|      Page       |                            URL                            |           Purpose           |
|-----------------|-----------------------------------------------------------|-----------------------------|
| Player Picks    | `https://mattjowen1991-hue.github.io/player-picks/`       | Players submit their picks  |
| Admin Dashboard | `hhttps://mattjowen1991-hue.github.io/admin-dashboard/`   | You manage scores & winners |
| League Table    | `https://mattjowen1991-hue.github.io/nfl-regular-season/` | Public standings & stats    |

---

## System Architecture

### Data Flow

```
┌─────────────────────┐
│   Players submit    │
│   picks via         │
│   player-picks.html │
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│     PICKS BIN       │
│  (JSONBin.io)       │
│  Resets each week   │
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│  Admin Dashboard    │
│  - Fetch scores     │
│  - Set winners      │
│  - Calculate points │
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│    SCORES BIN       │
│  (JSONBin.io)       │
│  NEVER resets       │
│  All-time scores    │
└─────────┬───────────┘
          │
          ▼
┌─────────────────────┐
│    League Table     │
│    index.html       │
│  Shows standings    │
└─────────────────────┘
```

### JSONBin Storage

|       Bin       |              ID            |          Purpose            | Reset?      |
|-----------------|----------------------------|-----------------------------|-------------|
| **Picks Bin**   | `69487a81ae596e708fa937cb` | Current week's player picks | Yes, weekly |
| **Scores Bin**  | `6951c2b0ae596e708fb6c625` | All-time cumulative scores  | NEVER       |

⚠️ **IMPORTANT**: Never reset the Scores Bin! It contains all historical data.

---

## Weekly Workflow

### 📅 Before Sunday (Setup)

#### Step 1: Update the Games

Edit **both** `player-picks.html` and `admin-dashboard-v2.html`:

```javascript
// player-picks.html CONFIG:
WEEK: 18,
DEADLINE: '2026-01-11T18:00:00',  // Next Sunday 6PM UK
GAMES: [
  { home: 'Team A', away: 'Team B', doublePoints: false },
  { home: 'Team C', away: 'Team D', doublePoints: false },
  // ... all games for the week
]

// admin-dashboard-v2.html CONFIG:
WEEK: 18,
WEEK_TYPE: 'week18',  // See week types below
GAMES: [
  { home: 'Team A', away: 'Team B', doublePoints: false },
  { home: 'Team C', away: 'Team D', doublePoints: false },
  // ... same games
]
```

#### Step 2: Choose the Correct Week Type

|     Week Type     |             When to Use             |     Points  | Full House? |
|-------------------|-------------------------------------|-------------|-------------|
| `'normal'`        | Regular weeks 1-17 (except special) | 1 per game  | ✅ Yes (+2) |
| `'thanksgiving'`  | Week 12.5 (Thanksgiving games only) | 2 per game  | ❌ No       |
| `'christmas'`     | Week 16.5 (Christmas games only)    | 2 per game  | ❌ No       |
| `'week18'`        | Final week of regular season        | 2 per game  | ✅ Yes (+2) |
| `'international'` | Weeks with London/Mexico games      | 1 per game* | ✅ Yes (+2) |

*For international weeks, set `doublePoints: true` on the specific international game(s).

#### Step 3: Commit & Push

```bash
git add .
git commit -m "Week 18 setup"
git push
```

#### Step 4: Notify Players

Send a WhatsApp message reminding everyone to submit picks before Sunday 6PM UK.

---

### 🏈 After Games Finish (Sunday Night/Monday)

#### Step 1: Open Admin Dashboard

Go to your admin dashboard URL and log in with PIN: `292292`

#### Step 2: Fetch Live Scores

Click **"🔄 Fetch Live Scores"** - this pulls results from ESPN and auto-selects winners for completed games.

#### Step 3: Review Winners

- ✅ Green border = winner selected
- Check any games that might not have auto-populated
- Manually click to set/change winners if needed

#### Step 4: Save to League Table

1. Click **"💾 Save to League Table"**
2. A modal appears showing:
   - Week number and type
   - All player scores for the week
   - Perfect week points calculation
3. Verify it looks correct
4. Click **"SAVE TO LEAGUE"**

You should see: `✅ Saved to JSONBin successfully!`

#### Step 5: Reset for Next Week

Click **"🔄 Reset for New Week"** → This clears the Picks Bin so players can submit fresh picks.

⚠️ **Only do this AFTER saving scores!**

---

### 📊 Verify Everything Worked

1. Open your League Table (`index.html`)
2. Check the console (F12) for:
   - `✅ Loaded scores from JSONBin`
   - `✅ Schedule updated from ESPN API`
3. Verify player totals are correct
4. Click a player card to check their stats updated

---

## File Reference

### Core Files

|            File           |             Purpose              |   Edit Weekly?   |
|---------------------------|----------------------------------|------------------|
| `player-picks.html`       | Player submission form           | ✅ Yes           |
| `admin-dashboard-v2.html` | Admin control panel              | ✅ Yes           |
| `index.html`              | Public league table              | ❌ No            |
| `players.json`            | Player profiles & favorite teams | ❌ Rarely        |
| `config.json`             | Season settings, highlights      | ❌ End of season |

### Supporting Files

|       File      |          Purpose         |                 Notes                |
|-----------------|--------------------------|--------------------------------------|
| `picks.json`    | Historical picks data    | Optional, for detailed stats         |
| `weeks.json`    | Local backup of scores   | Fallback if JSONBin fails            |
| `schedule.json` | ~~Manual game schedule~~ | **DELETED** - ESPN handles this now! |

### Asset Files

All in the `Assets/` folder:
- Team logos (e.g., `san-francisco-49ers.png`)
- Player avatars (e.g., `matt.png`)
- Background images
- Fonts

---

## Configuration Guide

### players.json Structure

```json
[
  {
    "id": "matt",
    "name": "Matt",
    "avatar": "Assets/matt.png",
    "team": "San Francisco 49ers",
    "teamLogo": "Assets/san-francisco-49ers.png",
    "superbowlWins": 0,
    "nflWins": 1
  }
]
```

|      Field      |                   Description                    |
|-----------------|--------------------------------------------------|
| `id`            | Lowercase identifier (must match CONFIG.PLAYERS) |
| `name`          | Display name                                     |
| `avatar`        | Path to player's profile picture                 |
| `team`          | Favorite NFL team (for "Next Game" feature)      |
| `teamLogo`      | Path to team logo                                |
| `superbowlWins` | League Super Bowl wins (your league's playoffs)  |
| `nflWins`       | Regular season wins                              |

### config.json Structure

```json
{
  "season": "2025",
  "repoUrl": "https://github.com/your/repo",
  "bestTeamMinPicks": 5,
  "highlight": {
    "winner": "matt",
    "spoon": "ste"
  },
  "honorary": [
    { "year": 2024, "player": "matt" },
    { "year": 2023, "player": "gaz" }
  ]
}
```

|        Field       |                    Description                    |
|--------------------|---------------------------------------------------|
| `season`           | Current season year                               |
| `highlight.winner` | Current/last season winner ID                     |
| `highlight.spoon`  | Current wooden spoon holder ID                    |
| `honorary`         | Hall of fame (past winners)                       |
| `bestTeamMinPicks` | Minimum picks to qualify for "Best Team Accuracy" |

---

## Scoring Rules

### Points Per Game

|              Week Type                  | Points per Correct Pick |
|-----------------------------------------|-------------------------|
| Normal week                             | 1 point                 |
| International game (doublePoints: true) | 2 points                |
| Thanksgiving (all games)                | 2 points                |
| Christmas (all games)                   | 2 points                |
| Week 18 (all games)                     | 2 points                |

### Full House Bonus (+2 points)

Awarded when a player gets **ALL** games correct in a week.

|   Week Type   | Full House Eligible? |
|---------------|----------------------|
| Normal        | ✅ Yes               |
| International | ✅ Yes               |
| Week 18       | ✅ Yes               |
| Thanksgiving  | ❌ No (partial week) |
| Christmas     | ❌ No (partial week) |

### Perfect Week Points Formula

```
Perfect Week Points = (Sum of all game points) + Full House Bonus (if eligible)
```

**Examples:**

|       Week Type      |     Games    |  Calculation  | Perfect Score |
|----------------------|--------------|---------------|---------------|
| Normal (7 games)     | 7            | 7×1 + 2       | **9**         |
| Normal (8 games)     | 8            | 8×1 + 2       | **10**        |
| With 1 International | 7 (1 double) | 6×1 + 1×2 + 2 | **10**        |
| Thanksgiving         | 3            | 3×2 + 0       | **6**         |
| Christmas            | 2            | 2×2 + 0       | **4**         |
| Week 18 (16 games)   | 16           | 16×2 + 2      | **34**        |

---

## Troubleshooting

### "JSONBin fetch failed"

**Cause**: Network issue or API key problem

**Fix**:
1. Check your internet connection
2. Verify API key in CONFIG matches your JSONBin account
3. Try refreshing the page

### Scores not appearing on League Table

**Cause**: Data didn't save properly

**Fix**:
1. Go back to Admin Dashboard
2. Click "Save to League Table" again
3. Check console for errors
4. Verify JSONBin has the data at jsonbin.io

### "Next Game" not showing for a player

**Cause**: ESPN API issue or team name mismatch

**Fix**:
1. Check console for ESPN errors
2. Verify player's `team` in `players.json` matches exactly (e.g., "San Francisco 49ers")
3. Clear localStorage and refresh: `localStorage.removeItem('nfl_schedule_cache')`

### Player can't submit picks

**Cause**: Deadline passed or wrong PIN

**Fix**:
1. Check if deadline has passed
2. Verify PIN in CONFIG.PLAYERS matches
3. Check DEADLINE format is correct: `'YYYY-MM-DDTHH:MM:SS'`

### Wrong scores calculated

**Cause**: Incorrect WEEK_TYPE setting

**Fix**:
1. Verify WEEK_TYPE matches the actual week type
2. Check doublePoints flags on games
3. Re-save to League Table with correct settings

### ESPN Scores not loading

**Cause**: Games not finished or ESPN API issue

**Fix**:
1. Wait for games to be marked "Final" on ESPN
2. Try clicking "Fetch Live Scores" again
3. Manually set winners if ESPN is down

---

## End of Season Tasks

### After Week 18 (Regular Season End)

1. **Update config.json** with final standings:
```json
{
  "highlight": {
    "winner": "winning_player_id",
    "spoon": "last_place_player_id"
  }
}
```

2. **Update players.json** if someone won:
```json
{
  "id": "matt",
  "nflWins": 2  // Increment by 1
}
```

3. **Add to Hall of Fame** in config.json:
```json
{
  "honorary": [
    { "year": 2025, "player": "matt" },
    { "year": 2024, "player": "matt" },
    // ... previous years
  ]
}
```

### Before Next Season

1. **DO NOT reset the Scores Bin** - historical data is preserved
2. Update `season` in config.json
3. The system will automatically start fresh when you begin Week 1
4. ESPN schedule will automatically show new season games

---

## Quick Reference Card

### Weekly Checklist

```
□ Update WEEK number in both files
□ Update DEADLINE date in player-picks.html
□ Update GAMES array in both files
□ Set correct WEEK_TYPE in admin dashboard
□ Commit and push changes
□ Notify players

After games:
□ Fetch Live Scores
□ Verify winners are correct
□ Save to League Table
□ Reset for New Week
```

### Important PINs

|      Purpose    | PIN                                     |
|-----------------|-----------------------------------------|
| Admin Dashboard | 292292                                  |
| Players         | 1234 (all players - consider changing!) |

### Week Type Quick Reference

|  Week |            Type          | Full House? |
|-------|--------------------------|-------------|
| 1-12  | `normal`                 | ✅         |
| 12.5  | `thanksgiving`           | ❌         |
| 13-16 | `normal`                 | ✅         |
| 16.5  | `christmas`              | ❌         |
| 17    | `normal`                 | ✅         |
| 18    | `week18 (double points)` | ✅         |

### Special Games

For **London/Mexico international games**, use:
- `WEEK_TYPE: 'normal'` (or `'international'`)
- `doublePoints: true` on the specific game

```javascript
GAMES: [
  { home: 'Jaguars', away: 'Dolphins', doublePoints: true }, // London!
  { home: 'Chiefs', away: 'Raiders', doublePoints: false },
  // ...
]
```

---

## Support

If something breaks or you need help:

1. Check the browser console (F12) for error messages
2. Verify JSONBin data at https://jsonbin.io
3. Check this documentation's Troubleshooting section
4. The code has fallbacks - if JSONBin fails, local files are used

---

*Documentation last updated: December 2024*
*System Version: 2.0 (with ESPN live schedule)*
