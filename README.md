# SOCRadar Incidents Connector for Microsoft Sentinel

Automatically ingest SOCRadar threat intelligence and incidents into Microsoft Sentinel.

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Forcunsami%2Fsocradar-sentinel-connector%2Fmaster%2Ftemplate.json)

## What It Does

- Pulls SOCRadar incidents every 5 minutes
- Writes data to `SOCRadarIncidents_CL` table in Sentinel
- Uses managed API connection for Log Analytics (automatic authentication)

## Prerequisites

1. **Azure Subscription** with Microsoft Sentinel
2. **Log Analytics Workspace** (where Sentinel is configured)
3. **Workspace Credentials**:
   - Workspace ID
   - Primary Key
4. **SOCRadar Credentials**:
   - Company ID
   - API Key

### Get Workspace Credentials

**Option 1: Azure Portal**

```
1. Go to Log Analytics Workspace
2. Settings → Agents
3. Copy Workspace ID and Primary Key
```

**Option 2: Cloud Shell**

```bash
az monitor log-analytics workspace get-shared-keys \
  --resource-group YOUR_RG \
  --workspace-name YOUR_WORKSPACE \
  --query primarySharedKey -o tsv
```

## Deployment

1. Click **Deploy to Azure** button above
2. Fill required parameters
3. Click **Review + create** → **Create**
4. Wait 2-3 minutes for deployment

## Query the Data

Run these queries in **Microsoft Sentinel → Logs** to view your SOCRadar incidents.

### 1. View All Incidents

```kusto
SOCRadarIncidents_CL
| extend ParsedData = todynamic(data_s)
| mv-expand alarm = ParsedData.data
| extend
    alarm_id = tolong(alarm.alarm_id),
    status = tostring(alarm.status),
    risk_level = tostring(alarm.alarm_risk_level),
    alarm_text = tostring(alarm.alarm_text),
    alarm_type = tostring(alarm.alarm_type_details),
    date = todatetime(alarm.date)
| project TimeGenerated, alarm_id, status, risk_level, alarm_type, date, alarm_text
| sort by TimeGenerated desc
| take 50
```

### 2. Count by Risk Level

```kusto
SOCRadarIncidents_CL
| extend ParsedData = todynamic(data_s)
| mv-expand alarm = ParsedData.data
| extend risk_level = tostring(alarm.alarm_risk_level)
| summarize count() by risk_level
| sort by count_ desc
```

### 3. Open High/Critical Incidents

```kusto
SOCRadarIncidents_CL
| extend ParsedData = todynamic(data_s)
| mv-expand alarm = ParsedData.data
| extend
    alarm_id = tolong(alarm.alarm_id),
    status = tostring(alarm.status),
    risk_level = tostring(alarm.alarm_risk_level),
    alarm_text = tostring(alarm.alarm_text)
| where status == "OPEN" and risk_level in ("HIGH", "CRITICAL")
| project TimeGenerated, alarm_id, status, risk_level, alarm_text
| sort by TimeGenerated desc
```

### 4. Count by Status

```kusto
SOCRadarIncidents_CL
| extend ParsedData = todynamic(data_s)
| mv-expand alarm = ParsedData.data
| extend status = tostring(alarm.status)
| summarize count() by status
| render columnchart
```

**Note:** First data appears within 10-15 minutes after deployment.

## Troubleshooting

**Logic App fails:**

- Check SOCRadar API key and Company ID
- Verify credentials in Logic App designer

**No data in Sentinel:**

- Wait 15 minutes (initial ingestion delay)
- Check Logic App run history shows "Succeeded"
- Verify Workspace ID/Key are correct

## Architecture

```
SOCRadar API → Logic App (5min) → Log Analytics → Sentinel
```

## Support

- Issues: GitHub Issues
- SOCRadar: support@socradar.io
