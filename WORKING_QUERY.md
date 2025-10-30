# Working Query for SOCRadar Incidents

## ✅ TESTED AND WORKING

```kusto
SOCRadarIncidents_CL
| extend alarms = parse_json(data_s)
| mv-expand alarm = alarms
| extend
    alarm_id = tostring(alarm.alarm_id),
    alarm_text = substring(tostring(alarm.alarm_text), 0, 80),
    status = tostring(alarm.status),
    risk = tostring(alarm.alarm_risk_level),
    alarm_type = tostring(alarm.alarm_type_details.alarm_main_type),
    asset = tostring(alarm.alarm_asset)
| where isnotempty(alarm_id)
| project TimeGenerated, alarm_id, status, risk, alarm_type, asset, alarm_text
| take 50
```

## Key Points

- **Data location**: `data_s` field (NOT RawData)
- **Data structure**: Array of alarms (use `mv-expand`)
- **Time filter**: Use "Last 1 hour" or ignore time picker (use `take`)
- **Field access**: `alarm.alarm_id`, `alarm.status`, etc.

## More Queries

### Latest Unique Alarms (No Duplicates)
```kusto
SOCRadarIncidents_CL
| extend alarms = parse_json(data_s)
| mv-expand alarm = alarms
| extend alarm_id = tostring(alarm.alarm_id)
| where isnotempty(alarm_id)
| summarize arg_max(TimeGenerated, *) by alarm_id
| extend
    alarm_text = substring(tostring(alarm.alarm_text), 0, 80),
    status = tostring(alarm.status),
    risk = tostring(alarm.alarm_risk_level)
| project TimeGenerated, alarm_id, status, risk, alarm_text
| take 50
```

### Open Alarms Only
```kusto
SOCRadarIncidents_CL
| extend alarms = parse_json(data_s)
| mv-expand alarm = alarms
| extend
    alarm_id = tostring(alarm.alarm_id),
    status = tostring(alarm.status)
| where status == "OPEN"
| extend
    alarm_text = substring(tostring(alarm.alarm_text), 0, 80),
    risk = tostring(alarm.alarm_risk_level)
| project TimeGenerated, alarm_id, status, risk, alarm_text
| take 50
```

### High/Critical Risk Alarms
```kusto
SOCRadarIncidents_CL
| extend alarms = parse_json(data_s)
| mv-expand alarm = alarms
| extend
    alarm_id = tostring(alarm.alarm_id),
    risk = tostring(alarm.alarm_risk_level)
| where risk in ("HIGH", "CRITICAL")
| extend
    alarm_text = substring(tostring(alarm.alarm_text), 0, 80),
    status = tostring(alarm.status)
| project TimeGenerated, alarm_id, status, risk, alarm_text
| take 50
```

### Count by Status
```kusto
SOCRadarIncidents_CL
| extend alarms = parse_json(data_s)
| mv-expand alarm = alarms
| extend
    alarm_id = tostring(alarm.alarm_id),
    status = tostring(alarm.status)
| where isnotempty(alarm_id)
| summarize arg_max(TimeGenerated, status) by alarm_id
| summarize count() by status
```

### Count by Risk Level
```kusto
SOCRadarIncidents_CL
| extend alarms = parse_json(data_s)
| mv-expand alarm = alarms
| extend
    alarm_id = tostring(alarm.alarm_id),
    risk = tostring(alarm.alarm_risk_level)
| where isnotempty(alarm_id)
| summarize arg_max(TimeGenerated, risk) by alarm_id
| summarize count() by risk
```

## Data Structure

SOCRadar API returns:
```json
{
  "data": [
    {
      "alarm_id": 77627281,
      "alarm_text": "...",
      "status": "OPEN",
      "alarm_risk_level": "MEDIUM",
      "alarm_asset": "TestGreenanimalsbank",
      "date": "2025-10-24 10:01:13",
      ...
    }
  ]
}
```

In Sentinel, this is stored in `data_s` field as JSON array.
