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
- ✅ **Active Devices** — Summary and list of devices seen in the last 7 days
- 📈 **Active Fleet Health** — Health trends over 30 days

## Deploy to Azure

### Workbook

[![Deploy Workbook](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fyurykissin%2Fdefender-xdr-workbook%2Fmain%2Fdefender-xdr-workbook.json)

### Device Tagging Logic App

[![Deploy Logic App](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fyurykissin%2Fdefender-xdr-workbook%2Fmain%2Fdevice-tagging-logicapp.json)

Runs weekly, queries MDE Advanced Hunting for active devices (seen in last 7 days), and manages the `ActiveDevice` tag:
- **Adds** the tag to devices that are active but not yet tagged
- **Removes** the tag from devices that are no longer active

## Prerequisites

1. **Log Analytics Workspace** connected to Microsoft Defender for Endpoint
2. An Azure subscription with permissions to create Workbooks and Logic Apps
3. **App Registration** (for the Logic App) with:
   - `AdvancedHunting.Read.All` — to query device information
   - `Machine.ReadWrite.All` — to add/remove device tags
   - Admin consent granted for both permissions

## Post-Deployment Steps

### Workbook
1. Go to the Azure Portal → your Resource Group
2. Open the **Workbook** resource
3. Select your **Subscription** from the dropdown at the top
4. Select your **Log Analytics Workspace** from the dropdown
5. Adjust the **Time Range** and **Stale Threshold (days)** parameters as needed
6. Pin the workbook to your Azure Dashboard for quick access

### Logic App
1. Go to the Azure Portal → your Resource Group
2. Open the **Logic App** resource
3. The Logic App runs automatically on the configured schedule (default: weekly)
4. Monitor runs in the Logic App's **Run history**

## Files

| File | Description |
|------|-------------|
| `defender-xdr-workbook.json` | ARM template — the Azure Monitor Workbook |
| `device-tagging-logicapp.json` | ARM template — Logic App for active device tagging |
| `defender-xdr-dashboards.kql` | Standalone KQL queries for Advanced Hunting (reference) |

## License

MIT
