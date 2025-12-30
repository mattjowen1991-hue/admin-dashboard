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
- **Live score fetching** from ESPN API with dual status indicators
- **Automatic winner detection** based on game results
- **Smart scoring** that handles normal weeks, Thanksgiving, Christmas, Week 18, and international games
- **Live schedule display** - no manual schedule maintenance needed!
- **Cloud storage** via JSONBin.io - works from any device
- **Picks history tracking** - detailed pick-by-pick data for comprehensive stats
- **Comprehensive player stats** - win rate, best team accuracy, current form, and more

### URLs

| Page | URL | Purpose |
|------|-----|---------|
| Player Picks | `https://mattjowen1991-hue.github.io/player-picks/` | Players submit their picks |
| Admin Dashboard | `https://mattjowen1991-hue.github.io/admin-dashboard/` | You manage scores & winners |
| League Table | `https://mattjowen1991-hue.github.io/nfl-regular-season/` | Public standings & stats |

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
┌─────────────────────┐         ┌─────────────────────┐
│  Admin Dashboard    │ ◄───────│    ESPN API         │
│  - Fetch scores     │         │  (Live Scores)      │
│  - Set winners      │         │  Auto-detects       │
│  - Calculate points │         │  game winners       │
└─────────┬───────────┘         └─────────────────────┘
          │
          ├──────────────────────────────┐
          │                              │
          ▼                              ▼
┌─────────────────────┐       ┌─────────────────────┐
│    SCORES BIN       │       │  PICKS HISTORY BIN  │
│  (JSONBin.io)       │       │  (JSONBin.io)       │
│  NEVER resets       │       │  NEVER resets       │
│  All-time scores    │       │  Detailed picks     │
└─────────┬───────────┘       └─────────┬───────────┘
          │                              │
          └──────────────┬───────────────┘
                         │
                         ▼
              ┌─────────────────────┐
              │    League Table     │◄──── ESPN Schedule API
              │    index.html       │      (Next Game info)
              │  - Standings        │
              │  - Player Stats     │
              │  - Prize Pot        │
              └─────────────────────┘
```

### JSONBin Storage

| Bin | ID | Purpose | Reset? |
|-----|------|---------|--------|
| **Picks Bin** | `69487a81ae596e708fa937cb` | Current week's player picks | ✅ Yes, weekly |
| **Scores Bin** | `6951c2b0ae596e708fb6c625` | All-time cumulative scores + config | ❌ NEVER |
| **Picks History Bin** | `695317c4ae596e708fb8f9ec` | Detailed pick-by-pick history | ❌ NEVER |

⚠️ **IMPORTANT**: Never reset the Scores Bin or Picks History Bin! They contain all historical data.

### What Each Bin Stores

#### Picks Bin (resets weekly)
```json
{
  "week": 18,
  "picks": {
    "matt": { "0": "Team A", "1": "Team B" },
    "jarv": { "0": "Team B", "1": "Team A" }
  },
  "submissions": [
    { "player": "matt", "timestamp": "2024-01-04T17:30:00Z", "inGracePeriod": false }
  ]
}
```

#### Scores Bin (permanent)
```json
{
  "weeks": [
    { "week": 1, "scores": { "matt": 6, "jarv": 7 } },
    { "week": 2, "scores": { "matt": 7, "jarv": 8 } }
  ],
  "config": {
    "perfectWeekPoints": {
      "1": 10,
      "2": 11,
      "12.5": 6
    }
  }
}
```

#### Picks History Bin (permanent)
```json
{
  "weeks": [
    {
      "week": 1,
      "games": [
        {
          "home": "Indianapolis Colts",
          "away": "Miami Dolphins",
          "winner": "Indianapolis Colts",
          "doublePoints": false,
          "picks": {
            "matt": "Indianapolis Colts",
            "jarv": "Miami Dolphins"
          }
        }
      ]
    }
  ]
}
```

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
  // ... same games - MUST MATCH!
]
```

#### Step 2: Choose the Correct Week Type

| Week Type | When to Use | Points | Full House? |
|-----------|-------------|--------|-------------|
| `'normal'` | Regular weeks 1-17 (except special) | 1 per game | ✅ Yes (+2) |
| `'thanksgiving'` | Week 12.5 (Thanksgiving games only) | 2 per game | ❌ No |
| `'christmas'` | Week 16.5 (Christmas games only) | 2 per game | ❌ No |
| `'week18'` | Final week of regular season | 2 per game | ✅ Yes (+2) |
| `'international'` | Weeks with London/Mexico games | 1 per game* | ✅ Yes (+2) |

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

#### Step 2: Check Status Indicators

The header shows two connection status indicators:
- **JSONBin** - Should turn green when picks data loads
- **ESPN Scores API** - Will turn green when you fetch scores

#### Step 3: Fetch Live Scores

Click **"🔄 Fetch Live Scores"**

This will:
- Pull results from ESPN API
- Auto-select winners for completed games
- Update the ESPN status indicator to green
- Show how many winners were auto-set

#### Step 4: Review Winners

- ✅ Green border = winner selected
- Check any games that might not have auto-populated
- Manually click to set/change winners if needed
- Use the dropdown menus to manually select winners

#### Step 5: Save to League Table

1. Click **"💾 Save to League Table"**
2. A modal appears showing:
   - Week number and type
   - All player scores for the week
   - Perfect week points calculation
3. Verify it looks correct
4. Click **"SAVE TO LEAGUE"**

You should see:
- `✅ Saved successfully!`
- Console shows: `✅ Picks history saved`

**What this saves:**
- Scores to the Scores Bin
- Perfect week points to config
- Detailed picks to the Picks History Bin

#### Step 6: Reset for Next Week

Click **"🔄 Reset for New Week"** → This clears the Picks Bin so players can submit fresh picks.

⚠️ **Only do this AFTER saving scores!**

---

### 📊 Verify Everything Worked

1. Open your League Table (`index.html`)
2. Check the console (F12) for:
   - `✅ Loaded scores from JSONBin`
   - `✅ Loaded picks history from JSONBin`
   - `✅ Schedule updated from ESPN API`
3. Verify player totals are correct
4. Click a player card to check their stats updated

---

## File Reference

### Core Files

| File | Purpose | Edit Weekly? |
|------|---------|--------------|
| `player-picks.html` | Player submission form | ✅ Yes |
| `admin-dashboard-v2.html` | Admin control panel | ✅ Yes |
| `index.html` | Public league table | ❌ No |
| `players.json` | Player profiles & favorite teams | ❌ Rarely |
| `config.json` | Season settings, highlights | ❌ End of season |

### Supporting Files

| File | Purpose | Notes |
|------|---------|-------|
| `weeks.json` | Local backup of scores | Fallback if JSONBin fails |
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

| Field | Description |
|-------|-------------|
| `id` | Lowercase identifier (must match CONFIG.PLAYERS) |
| `name` | Display name |
| `avatar` | Path to player's profile picture |
| `team` | Favorite NFL team (for "Next Game" feature) |
| `teamLogo` | Path to team logo |
| `superbowlWins` | League Super Bowl wins (your league's playoffs) |
| `nflWins` | Regular season wins |

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
  ],
  "perfectWeekPoints": {
    "1": 10,
    "2": 11,
    "12.5": 6
  }
}
```

| Field | Description |
|-------|-------------|
| `season` | Current season year |
| `highlight.winner` | Current/last season winner ID |
| `highlight.spoon` | Current wooden spoon holder ID |
| `honorary` | Hall of fame (past winners) |
| `bestTeamMinPicks` | Minimum picks to qualify for "Best Team Accuracy" |
| `perfectWeekPoints` | Auto-populated by admin dashboard when saving |

---

## Scoring Rules

### Points Per Game

| Week Type | Points per Correct Pick |
|-----------|-------------------------|
| Normal week | 1 point |
| International game (doublePoints: true) | 2 points |
| Thanksgiving (all games) | 2 points |
| Christmas (all games) | 2 points |
| Week 18 (all games) | 2 points |

### Full House Bonus (+2 points)

Awarded when a player gets **ALL** games correct in a week.

| Week Type | Full House Eligible? |
|-----------|----------------------|
| Normal | ✅ Yes |
| International | ✅ Yes |
| Week 18 | ✅ Yes |
| Thanksgiving | ❌ No (partial week) |
| Christmas | ❌ No (partial week) |

### Perfect Week Points Formula

```
Perfect Week Points = (Sum of all game points) + Full House Bonus (if eligible)
```

**Examples:**

| Week Type | Games | Calculation | Perfect Score |
|-----------|-------|-------------|---------------|
| Normal (7 games) | 7 | 7×1 + 2 | **9** |
| Normal (8 games) | 8 | 8×1 + 2 | **10** |
| With 1 International | 7 (1 double) | 6×1 + 1×2 + 2 | **10** |
| Thanksgiving | 3 | 3×2 + 0 | **6** |
| Christmas | 3 | 3×2 + 0 | **6** |
| Week 18 (6 games) | 6 | 6×2 + 2 | **14** |

---

## Player Stats Explained

The league table's player stats modal shows comprehensive statistics sourced from various data:

### Win Rate 🎯
- **With Picks History:** Exact % of correct picks (e.g., "89/134 = 66%")
- **Without Picks History:** Estimated from weekly scores vs perfect week points

### Best Team Accuracy
- The team you predict most accurately
- Requires minimum 5 picks of that team (`bestTeamMinPicks` in config)
- Shows team name + accuracy % (e.g., "49ers (78%)")

### Correct Picks
- Total correct picks / total picks made
- Only available when Picks History Bin has data

### Most Picked Team
- The team you select to win most often
- Shows team name + count (e.g., "Chiefs (23)")

### Current Form
- Your performance over the last 5 weeks
- 🟢 = scored at or above league average that week
- 🔴 = scored below league average that week
- Displayed oldest to newest (left to right)

### Perfect Weeks 👌🏼
- Weeks where you scored the maximum possible points
- Compared against `perfectWeekPoints` in config/JSONBin

### NFL Regular Season Wins / Super Bowl Wins
- From `nflWins` and `superbowlWins` in players.json
- Updated manually at end of season

---

## Troubleshooting

### "JSONBin fetch failed"

**Cause**: Network issue or API key problem

**Fix**:
1. Check your internet connection
2. Verify API key in CONFIG matches your JSONBin account
3. Try refreshing the page
4. Check the JSONBin status indicator in the header

### ESPN API status shows error

**Cause**: ESPN API temporarily unavailable or network issue

**Fix**:
1. Check your internet connection
2. ESPN API may be temporarily down
3. You can still set winners manually using the dropdown menus
4. Try clicking "Fetch Live Scores" again after a few minutes

### Scores not appearing on League Table

**Cause**: Data didn't save properly

**Fix**:
1. Go back to Admin Dashboard
2. Click "Save to League Table" again
3. Check console for errors
4. Verify JSONBin has the data at jsonbin.io

### Player stats showing "—" for some fields

**Cause**: Picks History Bin doesn't have data

**Fix**:
1. Check if Picks History Bin has data
2. Look for: `✅ Loaded picks history from JSONBin` in console
3. Stats require picks history to be saved from admin dashboard
4. Some stats (Win Rate) fall back to estimates if no picks history

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
4. Check the ESPN status indicator - red means error

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

1. **DO NOT reset the Scores Bin or Picks History Bin** - historical data is preserved
2. Update `season` in config.json
3. The system will automatically start fresh when you begin Week 1
4. ESPN schedule will automatically show new season games

---

## Quick Reference Card

### Weekly Checklist

```
BEFORE GAMES:
□ Update WEEK number in both files
□ Update DEADLINE date in player-picks.html
□ Update GAMES array in both files (must match!)
□ Set correct WEEK_TYPE in admin dashboard
□ Commit and push changes
□ Notify players

AFTER GAMES:
□ Open Admin Dashboard
□ Check JSONBin status indicator is green
□ Fetch Live Scores
□ Check ESPN status indicator is green
□ Verify winners are correct
□ Save to League Table
□ Check console for "✅ Picks history saved"
□ Reset for New Week
```

### Important IDs & PINs

| Item | Value |
|------|-------|
| Admin PIN | `292292` |
| Picks Bin ID | `69487a81ae596e708fa937cb` |
| Scores Bin ID | `6951c2b0ae596e708fb6c625` |
| Picks History Bin ID | `695317c4ae596e708fb8f9ec` |
| JSONBin API Key | `$2a$10$C7BY0Kl0u74gNn/kXO7xNuayWv493/f1jAxlHUmx3ENADQKDii61C` |

### Player PINs

| Player | PIN |
|--------|-----|
| Matt | 2922 |
| Jarv | 8351 |
| Gaz | 6194 |
| Joe | 2847 |
| Ben | 9563 |
| Coley | 3718 |
| Mark | 5926 |
| Ste | 1482 |
| Tony | 7035 |
| Jay | 4263 |
| Lee | 8914 |
| Liam | 6357 |

### Week Type Quick Reference

| Week | Type | Full House? |
|------|------|-------------|
| 1-12 | `normal` | ✅ |
| 12.5 | `thanksgiving` | ❌ |
| 13-16 | `normal` | ✅ |
| 16.5 | `christmas` | ❌ |
| 17 | `normal` | ✅ |
| 18 | `week18` (double points) | ✅ |

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

## Admin Dashboard Features

### Header Status Indicators

The admin dashboard header shows two connection status indicators:

| Indicator | What It Shows | States |
|-----------|---------------|--------|
| **JSONBin** | Connection to picks data | Grey → Loading, Green → Connected, Red → Error |
| **ESPN Scores API** | ESPN connection status | Grey → Not fetched, Green → Connected, Red → Error |

### Buttons

| Button | What It Does |
|--------|--------------|
| ↻ **Refresh** | Reloads picks data from JSONBin |
| 🔄 **Fetch Live Scores** | Gets results from ESPN, auto-selects winners |
| 💾 **Save to League Table** | Saves scores + picks history to JSONBin |
| ⬇ **Download JSON** | Downloads current week data as backup |
| 🔄 **Reset for New Week** | Clears Picks Bin for new week |

### Stats Bar

Shows at-a-glance information:
- **Week** - Current week number
- **Submitted** - How many players have submitted picks
- **Games Final** - How many games have winners set
- **Pending** - Players who haven't submitted

### Games & Results Section

- Click to expand/collapse
- Shows all games with current winners
- Use "Edit Scores Manually" to change winners
- Dropdown menus for each game
- Badges show: 2x (double points), Final, Live

### All Picks & Scores Table

- Shows every player's picks for every game
- ✓ = correct pick (green)
- ✗ = incorrect pick (red)
- — = missed pick
- Points column shows current score
- 🎯 = includes full house bonus
- 🏆 FULL HOUSE = got all games correct

### Export Tabs

| Tab | Purpose |
|-----|---------|
| **League Table** | Copy-paste line for manual updates |
| **Full JSON** | Complete week data for backup |
| **Validator** | Format for bulk validation |
| **WhatsApp** | Shareable summary for group chat |

---

## Support

If something breaks or you need help:

1. Check the browser console (F12) for error messages
2. Verify JSONBin data at https://jsonbin.io
3. Check this documentation's Troubleshooting section
4. Check both status indicators in the admin dashboard header
5. The code has fallbacks - if JSONBin fails, local files are used

---

*Documentation last updated: December 2024*
*System Version: 3.0 (Picks History + Enhanced Stats + Dual Status Indicators)*
