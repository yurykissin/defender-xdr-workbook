# 🛡️ MDE Device Hygiene & Health Dashboard

Azure Monitor Workbook + Logic App for monitoring and managing Microsoft Defender for Endpoint (MDE) device health across your Azure VM fleet.

---

## What Gets Deployed

This repo contains two ARM templates that can be deployed independently:

| Template | Resource | Type | Purpose |
|----------|----------|------|---------|
| `defender-xdr-workbook.json` | Workbook | `Microsoft.Insights/workbooks` | Interactive dashboard for device health visibility |
| `device-tagging-logicapp.json` | Logic App + O365 Connection | `Microsoft.Logic/workflows` + `Microsoft.Web/connections` | Automated device reporting and tagging |

---

## 1. Workbook — Device Health Dashboard

An Azure Monitor Workbook that queries MDE Advanced Hunting data via a Log Analytics workspace.

[![Deploy Workbook](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fyurykissin%2Fdefender-xdr-workbook%2Fmain%2Fdefender-xdr-workbook.json)

### Dashboard Sections

| Section | What it shows |
|---------|--------------|
| 📊 **Overview** | Total devices, active last 24h/7d, stale 14d/30d, healthy vs unhealthy — tile view |
| 🔴 **Sensor Health Distribution** | Pie chart of Active / Inactive / ImpairedCommunication / NoSensorData |
| 🖥️ **OS Platform Distribution** | Pie chart of Windows / Linux / macOS breakdown |
| 👻 **Stale & Ghost Devices** | Bar chart of device staleness buckets (0-7d, 7-14d, 14-30d, 30-60d, 60-90d, 90d+) |
| 📋 **Stale Device List** | Table of all devices past the stale threshold with color-coded days and health icons |
| 🔴 **Unhealthy Devices** | Active VMs reporting non-Active sensor — likely blocked network traffic |
| 🔌 **Onboarded But Never Healthy** | Devices onboarded via Terraform/script but agent never connected |
| 🌐 **Network Connectivity** | Failed connections to MDE cloud endpoints (securitycenter.windows.com, winatp-gw, etc.) |
| 📈 **Daily Failed Connection Trend** | Line chart of failed connections by MDE endpoint over 7 days |
| 🔥 **Firewall/NSG Blocks** | Table of blocked connections to MDE service URLs with block counts |
| 🔁 **Duplicate Device Names** | Same hostname with multiple MDE records — common after VM reimage |
| 🗑️ **Cleanup Candidates** | Devices 30+ days stale and not Active — safe to offboard |
| ✅ **Active Devices** | Summary tiles and full table of devices seen in last 7 days |
| 📈 **Fleet Health Trend** | Area chart of healthy vs unhealthy devices over 30 days |

### Post-Deployment

1. Open the deployed **Workbook** resource
2. Select your **Subscription** from the dropdown
3. Select your **Log Analytics Workspace**
4. Adjust **Time Range** and **Stale Threshold (days)** as needed

---

## 2. Logic App — Device Tagging & Reporting

A Logic App that runs on a schedule, queries MDE for active/inactive devices, sends a styled HTML email report with CSV attachments, and optionally manages the `ActiveDevice` tag on devices.

[![Deploy Logic App](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fyurykissin%2Fdefender-xdr-workbook%2Fmain%2Fdevice-tagging-logicapp.json)

### How It Works

The Logic App executes the following actions in order:

| # | Action | What it does |
|---|--------|-------------|
| 1 | **Query_Active_Devices** | Calls MDE Advanced Hunting API to find devices seen within the threshold (default: 7 days) with `SensorHealthState == 'Active'` |
| 2 | **Query_Inactive_Devices** | Finds devices not seen within the threshold or with non-Active sensor health, includes days since last seen |
| 3 | **Query_Currently_Tagged_Devices** | Finds devices that currently have the `ActiveDevice` tag via `DeviceManualTags` |
| 4 | **Parse results** | Parses JSON results from all three queries |
| 5 | **Build HTML tables** | Creates styled HTML tables for active and inactive devices |
| 6 | **Build CSV files** | Creates CSV tables for both device lists (attached to email) |
| 7 | **Build_Report** | Composes a styled HTML report with summary badges, device counts, and both tables |
| 8 | **Send_Report_Email** | Sends the report via Office 365 Outlook (Send email V2) with CSV attachments |
| 9 | **Apply_Tags_If_Live** | If `dryRun` is `false`: adds `ActiveDevice` tag to active untagged devices, removes it from inactive tagged devices |

### Dry Run vs Live Mode

| Mode | `dryRun` | Behavior |
|------|----------|----------|
| **Dry Run** (default) | `true` | Sends email report only — no tags are added or removed |
| **Live** | `false` | Sends email report AND applies tag changes to devices |

**To switch to live mode:**
1. Open the Logic App → **Logic app code view**
2. In the `"parameters"` section (at the bottom, outside `"definition"`), change `"DryRun": { "value": true }` to `"DryRun": { "value": false }`
3. Click **Save** → **Run Trigger** → **Recurrence**

### Email Report Example

The email includes:
- **Subject:** `MDE Device Report — 2 active, 5 inactive [DRY RUN]` (or `[LIVE]`)
- **Body:** Styled HTML with summary badges:

```
🛡️ MDE Device Tagging Report
━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Mode:           🔍 DRY RUN — report only, no changes
Tag name:       ActiveDevice
Active threshold: 7 days
Active devices:   2
Inactive devices: 5
Currently tagged: 0

🔴 Inactive / Unhealthy Devices (5)
┌─────────────────┬──────────────────┬───────────────┬─────────────┬──────────────┐
│ Device Name     │ OS               │ Sensor Health │ Last Seen   │ Days Inactive│
├─────────────────┼──────────────────┼───────────────┼─────────────┼──────────────┤
│ web-server-03   │ WindowsServer2019│ Inactive      │ 2026-02-10  │ 13           │
│ db-replica-02   │ Linux            │ NoSensorData  │ 2026-01-15  │ 39           │
└─────────────────┴──────────────────┴───────────────┴─────────────┴──────────────┘

✅ Active Devices (2)
┌─────────────────┬──────────────────┬─────────────────────────────┐
│ Device Name     │ OS               │ Last Seen                   │
├─────────────────┼──────────────────┼─────────────────────────────┤
│ svr19-dc1       │ WindowsServer2019│ 2026-02-22T15:43:57Z        │
│ srv             │ Linux            │ 2026-02-23T11:22:10Z        │
└─────────────────┴──────────────────┴─────────────────────────────┘
```

- **Attachments:** `active-devices.csv` and `inactive-devices.csv`

### Post-Deployment

1. Go to your Resource Group → find the **API Connection** (`<logicAppName>-office365`)
2. Click **Edit API connection** → **Authorize** → sign in with your Office 365 account → **Save**
3. The Logic App runs automatically on schedule (default: weekly)
4. To run immediately: open Logic App → **Run Trigger** → **Recurrence**

---

## Prerequisites

1. **Log Analytics Workspace** connected to Microsoft Defender for Endpoint (for the Workbook)
2. An Azure subscription with permissions to create Workbooks and Logic Apps
3. **App Registration** in Entra ID (for the Logic App) — see below

---

## App Registration Setup

The Logic App authenticates against two Microsoft APIs:

| API | Audience | Permission | Used for |
|-----|----------|------------|----------|
| **Microsoft Threat Protection** | `https://api.security.microsoft.com` | `AdvancedHunting.Read.All` | Querying device information |
| **WindowsDefenderATP** | `https://api.securitycenter.microsoft.com` | `Machine.ReadWrite.All` | Adding/removing device tags |

> **💡 Already have an App Registration for [xdrnotifications](https://github.com/yurykissin/xdrnotifications)?** You can reuse the same one — just add `WindowsDefenderATP → Application permissions → Machine.ReadWrite.All` and grant admin consent. No need for a second app.

### Create a new App Registration

1. Go to **Azure Portal** → **Microsoft Entra ID** → **App registrations** → **New registration**
2. Name it (e.g., `MDE-DeviceTagger`) → click **Register**
3. On the **Overview** page, copy the **Application (client) ID**

### Add a client secret

4. Go to **Certificates & secrets** → **New client secret**
5. Set a description and expiry → click **Add**
6. **Copy the Secret Value immediately** — it won't be shown again

### Configure API permissions

7. Go to **API permissions** → **Add a permission** → **APIs my organization uses**
8. Search for **Microsoft Threat Protection** → select it → **Application permissions** → check:
   - `AdvancedHunting.Read.All`
9. Click **Add a permission** again → **APIs my organization uses**
10. Search for **WindowsDefenderATP** → select it → **Application permissions** → check:
    - `Machine.ReadWrite.All`
11. Click **Add permissions**
12. Click **Grant admin consent for [your organization]** — requires Global Admin or Privileged Role Administrator

### Values needed for deployment

| Parameter | Where to find it |
|-----------|-----------------|
| **Client ID** | App registration → Overview → Application (client) ID |
| **Client Secret** | The value you copied in step 6 |
| **Tenant ID** | App registration → Overview → Directory (tenant) ID (auto-filled by default) |
| **Report Email** | The email address to receive the device report |

---

## Pricing

| Resource | Cost |
|----------|------|
| **Workbook** | Free — Azure Monitor Workbooks have no additional charge. You pay only for the underlying Log Analytics data ingestion. |
| **Logic App (Consumption plan)** | ~$0.000025 per action execution. A weekly run with ~100 devices ≈ a few cents/month. See [Logic Apps pricing](https://azure.microsoft.com/pricing/details/logic-apps/). |
| **Office 365 Connector** | Included with your Microsoft 365 license — no extra cost. |
| **API Connection** | Free — the `Microsoft.Web/connections` resource has no charge. |
| **MDE Advanced Hunting API** | Included with your Defender for Endpoint license. |

---

## Files

| File | Description |
|------|-------------|
| `defender-xdr-workbook.json` | ARM template — Azure Monitor Workbook with all dashboard sections |
| `device-tagging-logicapp.json` | ARM template — Logic App + O365 connection for device reporting and tagging |
| `defender-xdr-dashboards.kql` | Standalone KQL queries for Advanced Hunting (reference / manual use) |

## License

MIT

---

## Disclaimer

This ARM template is provided "as is", without warranties or guarantees of any kind.
Use at your own risk. You are responsible for reviewing, validating, and testing this template in a non‑production environment before deploying it to production.
The author assumes no liability for resource costs, configuration issues, or service disruptions resulting from the use of this template.

### Security Considerations

- Review all resource configurations before deployment.
- Validate role assignments, network rules, and identity configurations.
- Confirm compliance with your organization's security and governance standards.
- Ensure secrets/keys are not hard‑coded in templates or parameter files.

## Contributing

Contributions are welcome! When contributing:

- Do not include sensitive information.
- Ensure resource configurations follow Azure best practices.
- Confirm the template passes ARM validation (`az deployment ... --validate`).
- Follow the Code of Conduct (below).

## Code of Conduct

This project adopts the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct).
