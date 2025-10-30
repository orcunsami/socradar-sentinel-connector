# How to Run Queries in Sentinel - SIMPLE GUIDE

## 🎯 THE PROBLEM YOU'RE HAVING

You see the "Query history" panel with old queries and blue "Run" buttons.
The query editor (where you TYPE new queries) is HIDDEN behind this panel.

---

## ✅ SOLUTION 1: Use Query History (EASIEST!)

### Just Click a "Run" Button!

Look at your screen - you see queries like:
```
SOCRadarIncidents_CL
7 results - 10/30/2025, 5:20 PM          [Run] ← Click this!
```

**Just click any blue "Run" button** next to a query you want to re-run!

That's it! The query will execute and show results.

---

## ✅ SOLUTION 2: Type New Queries

### Step 1: Close Query History Panel

The "Query history" panel is blocking the query editor.

**How to close it:**
- Look for **"Query history"** text at the top of that panel
- Find the **X** icon or **trash icon** near it (top-right of the panel)
- Click it
- The panel will disappear

### Step 2: You'll See the Query Editor

After closing Query history, you'll see a **big empty box** (white or gray background).

This is where you type queries!

### Step 3: Type Your Query

Click in that empty box and type (or copy-paste):

```kusto
SOCRadarIncidents_CL
| take 10
```

### Step 4: Click Run

Look for the blue **"Run"** button (usually top-right area).

Click it!

Results will appear below.

---

## 🖼️ WHAT YOU SHOULD SEE

### Before (Query History Open):
```
┌─────────────────────────────────────┐
│ New Query 1*                        │
├─────────────────────────────────────┤
│                                     │
│  ╔═══════════════════════════════╗  │ ← This panel is in the way
│  ║ Query history           [X]   ║  │
│  ║                               ║  │
│  ║ SOCRadarIncidents_CL  [Run]   ║  │
│  ║                               ║  │
│  ╚═══════════════════════════════╝  │
│                                     │
└─────────────────────────────────────┘
```

### After (Query History Closed):
```
┌─────────────────────────────────────┐
│ New Query 1*                        │
├─────────────────────────────────────┤
│                                     │
│  ┌───────────────────────────────┐  │
│  │ // Type your query here       │  │ ← Empty box for typing!
│  │                               │  │
│  │                               │  │
│  └───────────────────────────────┘  │
│                           [Run] ←   │ ← Blue Run button
└─────────────────────────────────────┘
```

---

## 🎯 TRY THIS NOW

### Quick Test:

**Option A (Easiest):**
1. Don't close anything
2. Just click any blue **"Run"** button in Query history
3. See results!

**Option B (Type new query):**
1. Close Query history panel (click X)
2. Type: `SOCRadarIncidents_CL | take 10`
3. Click **Run** button
4. See results!

---

## 📋 Common Queries to Try

After you get the editor working, try these:

### 1. Simple View
```kusto
SOCRadarIncidents_CL
| take 10
```

### 2. See One Alarm's Full Data
```kusto
SOCRadarIncidents_CL
| take 1
| project RawData
```
Then expand the JSON in results to see all fields.

### 3. Clean Parsed View
```kusto
SOCRadarIncidents_CL
| extend
    alarm_id = tostring(parse_json(RawData).alarm_id),
    alarm_name = tostring(parse_json(RawData).alarm_name),
    alarm_status = tostring(parse_json(RawData).alarm_status)
| project TimeGenerated, alarm_id, alarm_name, alarm_status
| take 10
```

### 4. Remove Duplicates (Latest Only)
```kusto
SOCRadarIncidents_CL
| extend alarm_id = tostring(parse_json(RawData).alarm_id)
| summarize arg_max(TimeGenerated, *) by alarm_id
| take 10
```

---

## ❓ Still Confused?

### The tabs system:
- **"New Query 1", "New Query 2"** etc. are TABS
- Each tab has its own query editor
- You can have multiple tabs open
- The **+** button creates a new tab
- The **X** on a tab closes that tab (but you can't close the LAST tab)

### The "Query history" panel:
- This is NOT a tab
- It's a PANEL that slides over the editor
- You can close this panel with X
- It shows your previously run queries
- You can re-run old queries by clicking their "Run" buttons

---

## 💡 Pro Tip

If you're still stuck, just **click any "Run" button** in Query history!

That will show you results immediately, even without typing anything.

Then you can explore from there.
