# SOCRadar Incidents → Microsoft Sentinel

Automatically ingest SOCRadar threat intelligence into Microsoft Sentinel every 5 minutes.

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Forcunsami%2Fsocradar-sentinel-connector%2Fmaster%2Ftemplate.json)

## Quick Start

1. Click **Deploy to Azure** button
2. Fill in 4 parameters:
   - **Workspace ID** (from Log Analytics → Properties)
   - **Workspace Key** (from Azure CLI: `az monitor log-analytics workspace get-shared-keys --resource-group <RG> --workspace-name <NAME> --query primarySharedKey -o tsv`)
   - **SOCRadar Company ID** (from SOCRadar portal)
   - **SOCRadar API Key** (from SOCRadar → Settings → API)
3. Wait 2-3 minutes for deployment
4. Wait 15 minutes for first data
5. Query in Sentinel: `SOCRadarIncidents_CL | take 10`

## Data Safety & Optimization

**Automatic Field Truncation:** All string fields are automatically limited to 30,000 characters before sending to Azure. This prevents data loss due to Azure Log Analytics limits (32 KB per field).

**Time-based Filtering:** The connector fetches only incidents from the last 10 minutes on each run (5-minute recurrence with 5-minute overlap for safety). This ensures:
- No duplicate data ingestion
- Efficient API usage
- Reduced costs
- Maximum 1,000 incidents per run (configurable limit)

**Reference:** [Microsoft Azure Monitor Service Limits](https://learn.microsoft.com/en-us/azure/azure-monitor/fundamentals/service-limits) - Data Collector API section states: "Fields longer than 32 KB are truncated."

## Query Examples

**Recent alarms:**
```kusto
SOCRadarIncidents_CL
| where TimeGenerated > ago(24h)
| project TimeGenerated, tolong(alarm_id_d), tostring(status_s), tostring(alarm_risk_level_s)
| take 50
```

**Risk level distribution:**
```kusto
SOCRadarIncidents_CL
| summarize count() by tostring(alarm_risk_level_s)
```

## Workbook (Optional)

Import `workbook-template.json` for instant visualization:
- Azure Portal → Workbooks → Import
- Select `workbook-template.json` from this repo

---

**Issues?** Check Logic App run history for errors. Most common: wrong Workspace Key (403 error).
