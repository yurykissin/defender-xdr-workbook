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

Runs weekly, queries MDE Advanced Hunting for device status, and emails a styled report with CSV attachments:
- **Active devices** — healthy sensors seen in the last 7 days
- **Inactive devices** — stale or unhealthy sensors, sorted by days since last seen
- **CSV attachments** — `active-devices.csv` and `inactive-devices.csv` for further analysis

> **Deploys in dry-run mode (default).** The Logic App sends the report email but does **not** modify any tags. To enable tagging, redeploy with `dryRun` set to `false`, or change the `DryRun` parameter to `false` in the Logic App parameters and trigger a manual run.

## Prerequisites

1. **Log Analytics Workspace** connected to Microsoft Defender for Endpoint
2. An Azure subscription with permissions to create Workbooks and Logic Apps
3. **App Registration** in Azure AD (required for the Logic App) — see setup below

## App Registration Setup

The Logic App needs an Azure AD App Registration to authenticate against the Microsoft 365 Defender API.

> **💡 Already have an App Registration for [xdrnotifications](https://github.com/yurykissin/xdrnotifications)?** You can reuse the same one — just add `WindowsDefenderATP → Application permissions → Machine.ReadWrite.All` (step 9 below). No need for a second app.

### Create a new App Registration

1. Go to **Azure Portal** → **Microsoft Entra ID** → **App registrations** → **New registration**
2. Name it (e.g., `MDE-DeviceTagger`) → click **Register**
3. On the **Overview** page, copy the **Application (client) ID** — you'll need it during deployment

### Add a client secret

4. Go to **Certificates & secrets** → **New client secret**
5. Set a description and expiry → click **Add**
6. **Copy the Secret Value immediately** — it won't be shown again

### Configure API permissions

7. Go to **API permissions** → **Add a permission** → **APIs my organization uses**
8. Search for **Microsoft Threat Protection** → select it → **Application permissions** → check:
   - `AdvancedHunting.Read.All` — query device information
9. Click **Add a permission** again → **APIs my organization uses**
10. Search for **WindowsDefenderATP** → select it → **Application permissions** → check:
    - `Machine.ReadWrite.All` — add/remove device tags
11. Click **Add permissions**
12. Click **Grant admin consent for [your organization]** — requires Global Admin or Privileged Role Administrator

### Values needed for deployment

| Value | Where to find it |
|-------|-----------------|
| **Client ID** | App registration → Overview → Application (client) ID |
| **Client Secret** | The value you copied in step 6 |
| **Tenant ID** | App registration → Overview → Directory (tenant) ID (auto-filled by default) |

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
2. Find the **API Connection** resource (named `<logicAppName>-office365`)
3. Click **Edit API connection** → click **Authorize** → sign in with your Office 365 account → **Save**
4. Open the **Logic App** resource → it runs automatically on schedule (default: weekly)
5. You'll receive a device report email with HTML tables and CSV attachments
6. To enable actual tagging, change the `DryRun` parameter to `false` in the Logic App and trigger a run

## Files

| File | Description |
|------|-------------|
| `defender-xdr-workbook.json` | ARM template — the Azure Monitor Workbook |
| `device-tagging-logicapp.json` | ARM template — Logic App for active device tagging |
| `defender-xdr-dashboards.kql` | Standalone KQL queries for Advanced Hunting (reference) |

## License

MIT
