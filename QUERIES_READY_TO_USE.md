# Ready-to-Use SOCRadar Queries (Actual Field Names)

## 📋 Actual Fields in Your Data

Based on your SOCRadar API, each alarm contains:
- `alarm_id` - Unique identifier
- `alarm_text` - Alarm description/name
- `status` - Current status (e.g., "Open", "Closed")
- `alarm_risk_level` - Risk/severity level
- `alarm_type_details` - Type of alarm
- `alarm_asset` - Affected asset
- `date` - Creation date
- `notification_id`, `tags`, `notes`, etc.

---

## 🎯 COPY-PASTE QUERIES (50 Char Limit Per Field)

### Query 1: Clean View with 50-Char Limit

```kusto
SOCRadarIncidents_CL
| extend
    alarm_id = tostring(parse_json(RawData).alarm_id),
    alarm_text = substring(tostring(parse_json(RawData).alarm_text), 0, 50),
    status = substring(tostring(parse_json(RawData).status), 0, 50),
    risk_level = substring(tostring(parse_json(RawData).alarm_risk_level), 0, 50),
    alarm_type = substring(tostring(parse_json(RawData).alarm_type_details), 0, 50),
    asset = substring(tostring(parse_json(RawData).alarm_asset), 0, 50),
    created = todatetime(parse_json(RawData).date)
| project
    TimeGenerated,
    alarm_id,
    alarm_text,
    status,
    risk_level,
    alarm_type,
    asset,
    created
| take 20
```

**What it does**: Shows 20 alarms with key fields, each limited to 50 characters

---

### Query 2: Latest Unique Alarms (No Duplicates)

```kusto
SOCRadarIncidents_CL
| extend alarm_id = tostring(parse_json(RawData).alarm_id)
| summarize arg_max(TimeGenerated, *) by alarm_id
| extend
    alarm_text = substring(tostring(parse_json(RawData).alarm_text), 0, 50),
    status = substring(tostring(parse_json(RawData).status), 0, 50),
    risk_level = substring(tostring(parse_json(RawData).alarm_risk_level), 0, 50),
    alarm_type = substring(tostring(parse_json(RawData).alarm_type_details), 0, 50),
    asset = substring(tostring(parse_json(RawData).alarm_asset), 0, 50),
    created = todatetime(parse_json(RawData).date)
| project
    LastSeen = TimeGenerated,
    alarm_id,
    alarm_text,
    status,
    risk_level,
    alarm_type,
    asset,
    created
| sort by LastSeen desc
| take 20
```

**What it does**: Shows each unique alarm once (latest version), no duplicates

---

### Query 3: Open Alarms Only

```kusto
SOCRadarIncidents_CL
| extend
    alarm_id = tostring(parse_json(RawData).alarm_id),
    status = tostring(parse_json(RawData).status)
| where status == "open" or status == "Open" or status == "OPEN"
| summarize arg_max(TimeGenerated, *) by alarm_id
| extend
    alarm_text = substring(tostring(parse_json(RawData).alarm_text), 0, 50),
    risk_level = substring(tostring(parse_json(RawData).alarm_risk_level), 0, 50),
    alarm_type = substring(tostring(parse_json(RawData).alarm_type_details), 0, 50),
    asset = substring(tostring(parse_json(RawData).alarm_asset), 0, 50),
    created = todatetime(parse_json(RawData).date)
| project
    LastSeen = TimeGenerated,
    alarm_id,
    alarm_text,
    status,
    risk_level,
    alarm_type,
    asset,
    created
| sort by created desc
```

**What it does**: Shows only open alarms (removes duplicates)

---

### Query 4: High Risk Alarms

```kusto
SOCRadarIncidents_CL
| extend
    alarm_id = tostring(parse_json(RawData).alarm_id),
    risk_level = tostring(parse_json(RawData).alarm_risk_level),
    status = tostring(parse_json(RawData).status)
| where risk_level in ("High", "high", "HIGH", "Critical", "critical", "CRITICAL")
| summarize arg_max(TimeGenerated, *) by alarm_id
| extend
    alarm_text = substring(tostring(parse_json(RawData).alarm_text), 0, 50),
    alarm_type = substring(tostring(parse_json(RawData).alarm_type_details), 0, 50),
    asset = substring(tostring(parse_json(RawData).alarm_asset), 0, 50),
    created = todatetime(parse_json(RawData).date)
| project
    LastSeen = TimeGenerated,
    alarm_id,
    alarm_text,
    status,
    risk_level,
    alarm_type,
    asset,
    created
| sort by created desc
```

**What it does**: Shows only high/critical risk alarms

---

### Query 5: Count by Status

```kusto
SOCRadarIncidents_CL
| extend
    alarm_id = tostring(parse_json(RawData).alarm_id),
    status = tostring(parse_json(RawData).status)
| summarize arg_max(TimeGenerated, status) by alarm_id
| summarize count() by status
| sort by count_ desc
```

**What it does**: Shows how many alarms per status (Open, Closed, etc.)

---

### Query 6: Count by Risk Level

```kusto
SOCRadarIncidents_CL
| extend
    alarm_id = tostring(parse_json(RawData).alarm_id),
    risk_level = tostring(parse_json(RawData).alarm_risk_level)
| summarize arg_max(TimeGenerated, risk_level) by alarm_id
| summarize count() by risk_level
| sort by count_ desc
```

**What it does**: Shows how many alarms per risk level

---

### Query 7: Count by Alarm Type

```kusto
SOCRadarIncidents_CL
| extend
    alarm_id = tostring(parse_json(RawData).alarm_id),
    alarm_type = tostring(parse_json(RawData).alarm_type_details)
| summarize arg_max(TimeGenerated, alarm_type) by alarm_id
| summarize count() by alarm_type
| sort by count_ desc
```

**What it does**: Shows alarm distribution by type

---

### Query 8: See One Complete Alarm (All Fields)

```kusto
SOCRadarIncidents_CL
| take 1
| project RawData
```

**What it does**: Shows all fields of one alarm (expand the JSON to explore)

---

### Query 9: Search by Alarm Text

```kusto
let searchTerm = "phishing";  // Change this to your search term
SOCRadarIncidents_CL
| extend
    alarm_id = tostring(parse_json(RawData).alarm_id),
    alarm_text = tostring(parse_json(RawData).alarm_text)
| where alarm_text contains searchTerm
| summarize arg_max(TimeGenerated, *) by alarm_id
| extend
    alarm_text_short = substring(alarm_text, 0, 50),
    status = substring(tostring(parse_json(RawData).status), 0, 50),
    risk_level = substring(tostring(parse_json(RawData).alarm_risk_level), 0, 50)
| project
    alarm_id,
    alarm_text_short,
    status,
    risk_level
| take 20
```

**What it does**: Search for alarms containing specific text

---

### Query 10: Alarms Created in Last 24 Hours

```kusto
SOCRadarIncidents_CL
| extend
    alarm_id = tostring(parse_json(RawData).alarm_id),
    created = todatetime(parse_json(RawData).date)
| where created > ago(24h)
| summarize arg_max(TimeGenerated, *) by alarm_id
| extend
    alarm_text = substring(tostring(parse_json(RawData).alarm_text), 0, 50),
    status = substring(tostring(parse_json(RawData).status), 0, 50),
    risk_level = substring(tostring(parse_json(RawData).alarm_risk_level), 0, 50)
| project
    alarm_id,
    alarm_text,
    status,
    risk_level,
    created
| sort by created desc
```

**What it does**: Shows alarms created in the last 24 hours

---

### Query 11: Status Change Tracking

```kusto
SOCRadarIncidents_CL
| extend
    alarm_id = tostring(parse_json(RawData).alarm_id),
    status = tostring(parse_json(RawData).status)
| summarize
    StatusHistory = make_set(status),
    Count = count(),
    FirstSeen = min(TimeGenerated),
    LastSeen = max(TimeGenerated)
    by alarm_id
| where array_length(StatusHistory) > 1
| project alarm_id, StatusHistory, Count, FirstSeen, LastSeen
```

**What it does**: Shows alarms that changed status over time

---

### Query 12: Count Duplicates

```kusto
SOCRadarIncidents_CL
| extend alarm_id = tostring(parse_json(RawData).alarm_id)
| summarize
    count = count(),
    FirstSeen = min(TimeGenerated),
    LastSeen = max(TimeGenerated)
    by alarm_id
| sort by count desc
| take 20
```

**What it does**: Shows how many times each alarm was ingested

---

### Query 13: All Available Fields List

```kusto
SOCRadarIncidents_CL
| take 1
| extend parsed = parse_json(RawData)
| mv-expand bagexpansion=array parsed
| project key = tostring(bag_keys(parsed)[0])
| distinct key
```

**What it does**: Lists all available field names in the data

---

## 🎯 Most Useful Queries

**For daily monitoring:**
- Query 2: Latest unique alarms (overview)
- Query 3: Open alarms only (action items)
- Query 4: High risk alarms (priorities)

**For analysis:**
- Query 5: Status distribution
- Query 6: Risk level breakdown
- Query 7: Alarm type analysis

**For troubleshooting:**
- Query 8: See complete alarm data
- Query 12: Check duplicates

---

## 💡 Tips

1. **Always deduplicate** with `summarize arg_max(TimeGenerated, *) by alarm_id`
2. **50-char limit** uses `substring(field, 0, 50)` - adjust number as needed
3. **Time filters** use `where TimeGenerated > ago(24h)` or `where created > ago(1d)`
4. **Search** uses `where field contains "text"`
5. **Save useful queries** as Sentinel bookmarks (star icon)

---

## 🔧 Customize Field Length

To change the character limit, modify the `substring()` function:

```kusto
// 50 characters
substring(tostring(parse_json(RawData).alarm_text), 0, 50)

// 100 characters
substring(tostring(parse_json(RawData).alarm_text), 0, 100)

// 30 characters
substring(tostring(parse_json(RawData).alarm_text), 0, 30)
```
