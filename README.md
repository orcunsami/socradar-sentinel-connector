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

## Verify

```kusto
// In Sentinel → Logs
SOCRadarIncidents_CL
| take 10
```

First data appears within 10-15 minutes after deployment.

**📊 For better data analysis**, see [SENTINEL_QUERIES.md](SENTINEL_QUERIES.md) with 14+ pre-built KQL queries:
- View clean data (parsed JSON fields)
- Remove duplicates by alarm_id
- Track status changes over time
- Analytics by severity, status, type
- High-priority open alarms

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
