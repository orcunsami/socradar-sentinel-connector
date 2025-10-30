# Sentinel Queries for SOCRadar Incidents

## Understanding the Data Structure

The `SOCRadarIncidents_CL` table stores raw JSON from SOCRadar API. Each row contains:
- `TimeGenerated`: When data was ingested into Sentinel
- `RawData`: Complete JSON response from SOCRadar (all alarm fields)
- `Type`: Table name (SOCRadarIncidents_CL)

To see specific fields, you need to **parse the JSON** using KQL (Kusto Query Language).

---

## 🔍 Basic Queries

### 1. View Last 10 Alarms (Clean Format)

```kusto
SOCRadarIncidents_CL
| extend
    alarm_id = tostring(parse_json(RawData).alarm_id),
    alarm_name = tostring(parse_json(RawData).alarm_name),
    alarm_status = tostring(parse_json(RawData).alarm_status),
    alarm_severity = tostring(parse_json(RawData).alarm_severity),
    alarm_type = tostring(parse_json(RawData).alarm_type),
    created_date = todatetime(parse_json(RawData).created_date)
| project
    TimeGenerated,
    alarm_id,
    alarm_name,
    alarm_status,
    alarm_severity,
    alarm_type,
    created_date
| take 10
```

**What it does**: Shows key fields in readable columns instead of raw JSON.

---

### 2. See ALL Available Fields (Explore Structure)

```kusto
SOCRadarIncidents_CL
| take 1
| project RawData
```

**What it does**: Shows the complete JSON structure of one alarm so you can see all available fields.

**Then expand the JSON** in the results to see fields like:
- `alarm_id`, `alarm_name`, `alarm_status`
- `alarm_asset`, `alarm_category`, `alarm_severity`
- `created_date`, `updated_date`
- Any other fields SOCRadar provides

---

## 🎯 Handling Duplicates

### 3. Latest Status of Each Unique Alarm

```kusto
SOCRadarIncidents_CL
| extend
    alarm_id = tostring(parse_json(RawData).alarm_id),
    alarm_name = tostring(parse_json(RawData).alarm_name),
    alarm_status = tostring(parse_json(RawData).alarm_status),
    alarm_severity = tostring(parse_json(RawData).alarm_severity),
    created_date = todatetime(parse_json(RawData).created_date)
| summarize arg_max(TimeGenerated, *) by alarm_id
| project
    LastSeen = TimeGenerated,
    alarm_id,
    alarm_name,
    alarm_status,
    alarm_severity,
    created_date
| sort by LastSeen desc
```

**What it does**:
- Groups by `alarm_id` (unique identifier)
- Shows only the **latest version** of each alarm
- Removes duplicates - if an alarm was fetched 10 times, you see it once

**Use this for**: Current state of all alarms (no duplicates)

---

### 4. Count How Many Times Each Alarm Was Sent

```kusto
SOCRadarIncidents_CL
| extend alarm_id = tostring(parse_json(RawData).alarm_id)
| summarize
    count = count(),
    FirstSeen = min(TimeGenerated),
    LastSeen = max(TimeGenerated)
    by alarm_id
| sort by count desc
```

**What it does**: Shows how many duplicate entries exist for each alarm.

**Example output**:
```
alarm_id     | count | FirstSeen           | LastSeen
-------------|-------|---------------------|-------------------
12345        | 24    | 2025-10-30 15:00:00 | 2025-10-30 17:00:00
67890        | 18    | 2025-10-30 15:05:00 | 2025-10-30 16:50:00
```

This means alarm `12345` was sent 24 times over 2 hours (every 5 minutes).

---

## 📊 Analytics & Summaries

### 5. Alarm Count by Status

```kusto
SOCRadarIncidents_CL
| extend
    alarm_id = tostring(parse_json(RawData).alarm_id),
    alarm_status = tostring(parse_json(RawData).alarm_status)
| summarize arg_max(TimeGenerated, alarm_status) by alarm_id
| summarize count() by alarm_status
| render piechart
```

**What it does**: Shows distribution of alarms by status (Open, Closed, etc.)
- Deduplicates first (latest status per alarm)
- Counts by status
- Renders as pie chart

---

### 6. Alarm Count by Severity

```kusto
SOCRadarIncidents_CL
| extend
    alarm_id = tostring(parse_json(RawData).alarm_id),
    alarm_severity = tostring(parse_json(RawData).alarm_severity)
| summarize arg_max(TimeGenerated, alarm_severity) by alarm_id
| summarize count() by alarm_severity
| render barchart
```

**What it does**: Shows how many alarms per severity level (Critical, High, Medium, Low)

---

### 7. Alarm Count by Type

```kusto
SOCRadarIncidents_CL
| extend
    alarm_id = tostring(parse_json(RawData).alarm_id),
    alarm_type = tostring(parse_json(RawData).alarm_type)
| summarize arg_max(TimeGenerated, alarm_type) by alarm_id
| summarize count() by alarm_type
| sort by count_ desc
```

**What it does**: Groups alarms by type (e.g., "Data Breach", "Phishing", "Malware")

---

## 🔄 Tracking Status Changes

### 8. Alarms That Changed Status

```kusto
SOCRadarIncidents_CL
| extend
    alarm_id = tostring(parse_json(RawData).alarm_id),
    alarm_status = tostring(parse_json(RawData).alarm_status)
| summarize
    StatusChanges = make_set(alarm_status),
    Count = count()
    by alarm_id
| where Count > 1  // Only alarms seen multiple times
| where array_length(StatusChanges) > 1  // Only alarms with different statuses
| project alarm_id, StatusChanges, Count
```

**What it does**: Finds alarms that had status changes (e.g., Open → Closed)

**Example output**:
```
alarm_id | StatusChanges        | Count
---------|----------------------|------
12345    | ["Open", "Closed"]   | 24
67890    | ["Open"]             | 18
```

Alarm `12345` changed status, alarm `67890` stayed "Open".

---

### 9. Status Change Timeline for Specific Alarm

```kusto
let targetAlarmId = "12345";  // Replace with actual alarm_id
SOCRadarIncidents_CL
| extend alarm_id = tostring(parse_json(RawData).alarm_id)
| where alarm_id == targetAlarmId
| extend alarm_status = tostring(parse_json(RawData).alarm_status)
| project TimeGenerated, alarm_status
| sort by TimeGenerated asc
```

**What it does**: Shows when a specific alarm's status changed over time.

---

## 🚨 Actionable Queries

### 10. New Alarms in Last Hour

```kusto
SOCRadarIncidents_CL
| extend
    alarm_id = tostring(parse_json(RawData).alarm_id),
    alarm_name = tostring(parse_json(RawData).alarm_name),
    alarm_severity = tostring(parse_json(RawData).alarm_severity),
    created_date = todatetime(parse_json(RawData).created_date)
| where created_date > ago(1h)
| summarize arg_max(TimeGenerated, *) by alarm_id
| project alarm_id, alarm_name, alarm_severity, created_date
| sort by created_date desc
```

**What it does**: Shows alarms **created** in the last hour (not ingested, but actually created in SOCRadar)

---

### 11. Critical & High Severity Alarms (Open Only)

```kusto
SOCRadarIncidents_CL
| extend
    alarm_id = tostring(parse_json(RawData).alarm_id),
    alarm_name = tostring(parse_json(RawData).alarm_name),
    alarm_status = tostring(parse_json(RawData).alarm_status),
    alarm_severity = tostring(parse_json(RawData).alarm_severity)
| where alarm_status == "Open" or alarm_status == "open"
| where alarm_severity in ("Critical", "High", "critical", "high")
| summarize arg_max(TimeGenerated, *) by alarm_id
| project alarm_id, alarm_name, alarm_severity, alarm_status
| sort by alarm_severity asc
```

**What it does**: Filters for actionable high-priority alarms that need attention.

---

### 12. All Fields for Specific Alarm

```kusto
let targetAlarmId = "12345";  // Replace with actual alarm_id
SOCRadarIncidents_CL
| extend alarm_id = tostring(parse_json(RawData).alarm_id)
| where alarm_id == targetAlarmId
| summarize arg_max(TimeGenerated, *) by alarm_id
| project RawData
```

**What it does**: Shows complete details of a specific alarm (all JSON fields).

---

## 📈 Time-Based Analysis

### 13. Alarm Ingestion Over Time

```kusto
SOCRadarIncidents_CL
| summarize count() by bin(TimeGenerated, 5m)
| render timechart
```

**What it does**: Shows how many records were ingested every 5 minutes (should be consistent if Logic App runs properly).

---

### 14. Unique Alarms Added Over Time

```kusto
SOCRadarIncidents_CL
| extend alarm_id = tostring(parse_json(RawData).alarm_id)
| summarize FirstSeen = min(TimeGenerated) by alarm_id
| summarize count() by bin(FirstSeen, 1h)
| render timechart
```

**What it does**: Shows when **new unique alarms** first appeared (not duplicates).

---

## 🛠️ Advanced: Create Parsed View

If you want to avoid parsing JSON every time, create a **function**:

```kusto
// Step 1: Run this once to create a function
.create-or-alter function SOCRadarParsed() {
    SOCRadarIncidents_CL
    | extend
        alarm_id = tostring(parse_json(RawData).alarm_id),
        alarm_name = tostring(parse_json(RawData).alarm_name),
        alarm_status = tostring(parse_json(RawData).alarm_status),
        alarm_severity = tostring(parse_json(RawData).alarm_severity),
        alarm_type = tostring(parse_json(RawData).alarm_type),
        alarm_asset = tostring(parse_json(RawData).alarm_asset),
        created_date = todatetime(parse_json(RawData).created_date),
        updated_date = todatetime(parse_json(RawData).updated_date)
}
```

**Then use it like a table:**
```kusto
SOCRadarParsed()
| where alarm_severity == "Critical"
| take 10
```

**Note**: Function creation might require permissions. If it fails, just use the regular queries above.

---

## 🎯 Quick Reference

| Task | Query Number |
|------|--------------|
| View clean data | #1 |
| See all fields | #2 |
| Remove duplicates | #3 |
| Count duplicates | #4 |
| Status distribution | #5 |
| Severity breakdown | #6 |
| Find status changes | #8 |
| New alarms only | #10 |
| High priority open | #11 |
| Alarm details | #12 |

---

## 💡 Pro Tips

1. **Always deduplicate** with `summarize arg_max(TimeGenerated, *) by alarm_id` when analyzing current state
2. **Use `where TimeGenerated > ago(24h)`** to limit results for performance
3. **Adjust field names** based on what you see in Query #2 (explore structure)
4. **Save useful queries** as Sentinel workbook or alert rules
5. **Duplicates are OK** - they provide history and status change tracking

---

## 🚀 Next Steps

1. Run Query #2 to see actual field names from your SOCRadar data
2. Adjust other queries if field names differ
3. Create Sentinel alerts using Query #11 (high severity open alarms)
4. Build a dashboard with Queries #5, #6, #7 (status, severity, type charts)
