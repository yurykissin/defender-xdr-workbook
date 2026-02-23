# 🛡️ MDE Device Hygiene & Health Workbook

Azure Monitor Workbook that provides a comprehensive dashboard for monitoring Microsoft Defender for Endpoint (MDE) device health across your Azure VM fleet.

## What Gets Deployed

| Resource | Type | Description |
|----------|------|-------------|
| Workbook | `Microsoft.Insights/workbooks` | Azure Monitor Workbook with device health dashboards |

## Dashboard Sections

- 📊 **Overview** — Total devices, active vs stale, healthy vs unhealthy at a glance
- 🔴 **Sensor Health Distribution** — Pie chart of Active/Inactive/Impaired states
- 👻 **Stale & Ghost Devices** — Devices not seen in 14+ days (likely decommissioned VMs)
- 🔴 **Unhealthy Devices** — Active VMs reporting non-Active sensor state
- 🌐 **Network Connectivity** — Failed connections to MDE cloud endpoints
- 🔁 **Duplicates & Cleanup** — Duplicate device names and safe-to-offboard candidates
- 📈 **Active Fleet Health** — Health trends over 30 days

## Deploy to Azure

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fyurykissin%2Fdefender-xdr-workbook%2Fmain%2Fdefender-xdr-workbook.json)

## Prerequisites

1. **Log Analytics Workspace** connected to Microsoft Defender for Endpoint
2. An Azure subscription with permissions to create Workbooks

## Post-Deployment Steps

1. Go to the Azure Portal → your Resource Group
2. Open the **Workbook** resource
3. Adjust the **Time Range** and **Stale Threshold (days)** parameters as needed
4. Pin the workbook to your Azure Dashboard for quick access

## Files

| File | Description |
|------|-------------|
| `defender-xdr-workbook.json` | ARM template — the Azure Monitor Workbook |
| `defender-xdr-dashboards.kql` | Standalone KQL queries for Advanced Hunting (reference) |

## License

MIT
