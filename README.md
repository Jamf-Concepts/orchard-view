# Orchard View — Admin Guide

> Enriched, unified visibility into your Apple device fleet across Jamf Pro, Jamf School, and Apple School/Business Manager.

---

## Table of contents

1. [Overview](#overview)
2. [Requirements](#requirements)
3. [Installation](#installation)
4. [First launch](#first-launch)
5. [Connecting your MDM sources](#connecting-your-mdm-sources)
   - [Jamf Pro](#jamf-pro)
   - [Jamf School](#jamf-school)
   - [Apple School Manager / Apple Business Manager](#apple-school-manager--apple-business-manager)
6. [Navigating the main window](#navigating-the-main-window)
   - [Header and connection status](#header-and-connection-status)
   - [Device type and warranty charts](#device-type-and-warranty-charts)
   - [Device table](#device-table)
   - [Searching and filtering](#searching-and-filtering)
7. [Device detail view](#device-detail-view)
   - [Installed apps](#installed-apps)
8. [Generating reports](#generating-reports)
   - [Filtering the report view](#filtering-the-report-view)
   - [Exporting to CSV or PDF](#exporting-to-csv-or-pdf)
9. [Static group management](#static-group-management)
   - [Adding devices to an existing group](#adding-devices-to-an-existing-group)
   - [Creating a new static group](#creating-a-new-static-group)
10. [Clearing MDM assignments in ASM/ABM](#clearing-mdm-assignments-in-asmabm)
11. [Remote commands](#remote-commands)
12. [Appearance settings](#appearance-settings)
13. [Credential management and security](#credential-management-and-security)
14. [Logging](#logging)
15. [Troubleshooting](#troubleshooting)
16. [Getting help](#getting-help)
17. [Dependencies](#dependencies)
18. [License, copyright, and privacy](#license-copyright-and-privacy)

---

## Overview

Orchard View is a native macOS application that gives IT administrators a single, enriched view of their Apple device fleet. It pulls inventory data from **Jamf Pro** and **Jamf School**, then layers in coverage and assignment data from **Apple School Manager (ASM)** or **Apple Business Manager (ABM)** — giving you a more complete picture of each device than any one platform provides on its own.

Orchard View is not a replacement for your MDM. Jamf Pro and Jamf School remain your source of truth for device management. Orchard View sits alongside them, consolidating and enriching their data so you can answer questions like: which devices have expired warranties, what apps are installed across my fleet, which devices are in which groups, and which devices are assigned to which MDM server in Apple's systems — all without switching between multiple admin portals.

### Key capabilities

| Capability | Details |
|---|---|
| Unified inventory | Computers, iPads, iPhones, and Apple TVs from Jamf Pro and Jamf School in one table |
| Warranty tracking | Live AppleCare coverage data pulled directly from ASM/ABM |
| Installed app inventory | Per-device app lists with bundle IDs and versions (Jamf Pro computers and mobile devices) |
| Device detail | Hardware specs, OS version, IP address, MDM status, group memberships |
| Remote commands | Lock, Restart, and Erase — sent via MDM with confirmation dialogs |
| Static group management | Add selected devices to existing groups or create new ones across Jamf Pro and Jamf School |
| ASM/ABM MDM assignment | Clear MDM server assignments directly from within the app |
| Reports | CSV and PDF export with flexible filtering and sorting |
| Deduplication | Devices that appear in multiple sources are merged into a single record |

---

## Requirements

- macOS 13 Ventura or later (Apple Silicon or Intel)
- Network access to your Jamf Pro and/or Jamf School instance
- API credentials for each source you want to connect (see [Connecting your MDM sources](#connecting-your-mdm-sources))
- An ASM or ABM account with an API account configured (optional, for warranty data and MDM assignment management)

---

## Installation

1. Download `Orchard View.pkg` from [GitHub Releases](https://github.com/Jamf-Concepts/orchard-view/releases/latest).
2. Double-click the `.pkg` and follow the installer prompts. The package installs Orchard View to `/Applications`.
3. Launch **Orchard View**. 

---

## First launch

When you open Orchard View for the first time, a welcome screen appears. Click **Get Started** to open the API Settings sheet and begin connecting your MDM sources. You can dismiss the welcome screen without configuring anything and return to settings at any time using the **API Settings** button in the top-right corner of the main window.

---

## Connecting your MDM sources

Open **API Settings** from the toolbar. The sheet has three tabs — one per source. You only need to configure the sources you use. At least one Jamf source (Jamf Pro or Jamf School) is required before any devices will load.

### Jamf Pro

Orchard View uses **OAuth 2.0 Client Credentials** to authenticate with Jamf Pro. No username or password is required.

**To create an API client in Jamf Pro:**

1. In Jamf Pro, go to **Settings → API Roles and Clients**.
2. Create an **API Role** with at minimum the following permissions:

   | Feature | Required Permission |
   |---|---|
   | Device inventory | Read Computers |
   | Device inventory | Read Mobile Devices |
   | Static group listing | Read Static Computer Groups |
   | Static group listing | Read Static Mobile Device Groups |
   | Create new static group (Report → Create New) | Create Static Computer Groups |
   | Add devices to existing group (Report → Existing Group) | Update Static Computer Groups |
   | Create new static group (mobile) | Create Static Mobile Device Groups |
   | Add devices to existing group (mobile) | Update Static Mobile Device Groups |
   | Remote Lock (computers) | Send Computer Remote Lock Command |
   | Remote Restart (computers) | Send Computer Restart Command |
   | Remote Erase (computers) | Send Computer Remote Wipe Command |
   | Remote Lock (mobile) | Send Mobile Device Remote Lock Command |
   | Remote Restart (mobile) | Send Mobile Device Restart Device Command |
   | Remote Erase (mobile) | Send Mobile Device Remote Wipe Command |

3. Create an **API Client**, assign the role above, and note the **Client ID** and **Client Secret**.

**In the Jamf Pro tab of API Settings:**

| Field | Value |
|---|---|
| Server URL | Your Jamf Pro instance URL, e.g. `https://yourserver.jamfcloud.com` (no trailing slash) |
| Client ID | The Client ID from the API Client you created |
| Client Secret | The Client Secret from the API Client |

Click **Test Connection** to verify credentials, then **Save**. A successful test saves automatically.

---

### Jamf School

Orchard View uses **HTTP Basic Auth** (Network ID + API Key) for Jamf School.

**To find your credentials in Jamf School:**

- **Network ID:** Go to **Organisation → Details**. The Network ID is displayed there.
- **API Key:** Go to **Settings → API** and generate or copy your API key.

**In the Jamf School tab of API Settings:**

| Field | Value |
|---|---|
| Server URL | Your Jamf School instance URL, e.g. `https://yourschool.jamfcloud.com` |
| Network ID | The numeric Network ID from Organisation → Details |
| API Key | Your API key from Settings → API |

Click **Test Connection** to verify, then **Save**.

---

### Apple School Manager / Apple Business Manager

ASM/ABM connectivity is optional. When configured, Orchard View:

- Fetches live **AppleCare coverage data** for each device (warranty expiry, plan type, remaining time)
- Resolves which **MDM server** each device is assigned to, correcting source attribution where needed
- Enables **MDM assignment clearing** directly from the device table and detail view

Orchard View uses **OAuth 2.0 with a signed JWT** (ES256, P-256 EC private key). This is the same authentication method used by Apple's own tools.

**To create an API account in ASM or ABM:**

1. In ASM/ABM, go to **Settings → API**.
2. Create a new API Account. Download the **private key (.pem file)** when prompted — this is only available at creation time.
3. Note the **Client ID** and **Key ID** shown on the account page.

**In the Apple School Manager tab of API Settings:**

| Field | Value |
|---|---|
| Account Type | Choose Apple School Manager or Apple Business Manager |
| Organisation Name | Your organisation name (for display purposes) |
| Client ID | Starts with `SCHOOLAPI.` or `BUSINESSAPI.` |
| Key ID | The Key ID from the same page as the Client ID |
| Private Key File | Click **Choose File** and select the `.pem` file you downloaded |

> **Note:** Orchard View accepts both **PKCS#8** (`-----BEGIN PRIVATE KEY-----`) and **SEC1** (`-----BEGIN EC PRIVATE KEY-----`) key formats. Both are issued by Apple depending on account type.

Click **Test Connection** to verify, then **Save**.

---

## Navigating the main window

### Header and connection status

The top of the main window shows the **Orchard View** logo and title on the left, and a toolbar on the right containing:

- **Source status badges** — green dots for each connected source (Jamf Pro, Jamf School, Apple School Manager). These appear only for configured sources.
- **Appearance menu** — toggle between System, Light, and Dark mode.
- **API Settings** — opens the credentials sheet.

Below the charts, a narrow status bar shows per-source fetch status as devices load: *Fetching…*, *N devices loaded*, or an error message with a red indicator.

### Device type and warranty charts

Two interactive pie charts appear below the header:

**Device Types** — breaks down your fleet by Laptops, Desktops, Mobile devices, and Tablets. The center shows the total device count.

**Warranty Status** — segments devices by warranty state: Expired, ≤30 days, 31–60 days, 61–90 days, 91–180 days, 180+ days, and Unknown (no ASM/ABM data available).

**Interacting with charts:**

- Click or tap a segment or its legend entry to **filter the device table** to that category.
- Click the same segment again to **clear the filter**.
- The center label updates to reflect the current selection.

### Device table

The device table shows all devices from connected sources, deduplicated by serial number. Columns include:

| Column | Notes |
|---|---|
| Device | Device name and serial number |
| Type | Model identifier and device category icon |
| User | Assigned user (real name if available, username as fallback) |
| Groups | Up to three static group tags; highlighted groups contain "macOS" or "iOS" in the name |
| Hardware | Model name and RAM (computers) or OS version (mobile devices) |
| Warranty | Status badge (Active / Expiring Soon / Expired / Unknown), expiry month and year, and time remaining |
| Source | Jamf Pro, Jamf School, or ASM badge |
| Last Check-in | Relative time since last MDM check-in |

**Selecting devices:**

- Click the **checkbox** on any row to select it.
- Use the **header checkbox** to select or deselect all visible (filtered) devices.
- Selected devices are highlighted and a summary bar appears at the bottom of the table showing the count. Click **Deselect All** to clear.
- Selection is used to pre-populate the Report view and static group operations.

**Row actions:**

Click the **⋯** menu at the end of any row to:
- **View Details** — opens the device detail sheet.
- **Send Remote Command** — shortcut to the remote commands section of the detail view.

### Searching and filtering

The toolbar above the device table contains:

- **Search bar** — filters by device name, serial number, or username in real time.
- **Type filter** — drop-down to show only Laptops, Desktops, Mobile devices, or Tablets.
- **Clear Filters** button — appears when any filter is active; clears all filters at once.
- **Refresh** — re-fetches all sources. Also available by clicking Refresh in the toolbar.
- **Generate Report** — opens the report sheet. If devices are selected, the button shows the count and the report opens pre-filtered to the selection.

---

## Device detail view

Click **View Details** from a row's ⋯ menu to open the device detail sheet. The sheet fetches the latest information from the device's source before displaying.

Sections shown:

| Section | Contents |
|---|---|
| General | Device name, serial number, model identifier, OS version, IP address, last enrolled date |
| Hardware | Chip/processor, memory, device type |
| Assigned User | Name and MDM source |
| MDM Status | MDM profile installed (yes/no), supervised (yes/no), last check-in |
| Warranty | Status badge, expiry date, coverage type (e.g. AppleCare+, Limited Warranty) |
| Group Memberships | All static groups the device belongs to, displayed as tag pills |
| Installed Apps | Full application inventory (see below) |
| Apple School Manager | Shown when the device has a record in ASM/ABM; includes the MDM assignment clear action |
| Remote Commands | Lock, Restart, and Erase buttons with confirmation dialogs |

### Installed apps

For **Jamf Pro computers**, Orchard View fetches the full installed application list including bundle IDs and versions. For **Jamf Pro mobile devices**, the app list is retrieved from the Classic API. For **Jamf School devices**, the app list is fetched via the Jamf School device detail endpoint (`GET /api/devices/{udid}?includeApps=true`).

Each app entry displays:
- App name
- Bundle ID
- Version

Use the **search field** at the top of the app list to filter by name or bundle ID.

> **Note:** App icons are loaded asynchronously. Jamf School devices use hosted icon URLs directly. For Jamf Pro devices, icons are resolved via the iTunes lookup API (`itunes.apple.com/lookup?bundleId=`). Built-in Apple apps (`com.apple.*`) display the appropriate SF Symbol rather than an iTunes lookup result. Icon resolution is capped at 10 concurrent requests to avoid rate limiting.

---

## Generating reports

Click **Generate Report** in the toolbar to open the report sheet. If you had devices selected in the main table, the report opens with those devices pre-selected; otherwise all devices are included.

### Filtering the report view

The left panel contains filter controls:

- **Device Type** — include or exclude Laptops, Desktops, Mobile devices, and Tablets independently.
- **Source** — filter to Jamf Pro, Jamf School, or ASM devices.
- **Warranty Status** — include Active, Expiring Soon, Expired, and/or Unknown devices.
- **Group** — filter to devices belonging to a specific static group.
- **Select All Filters / Clear** — convenience buttons to reset filters.

The table on the right supports sorting by Device, Serial, Model, Hardware, Warranty, Groups, or Source. Click any column header to sort; click again to reverse the order.

Use the **row checkboxes** to select specific devices for export or group operations. The header checkbox selects all visible (filtered) devices.

### Exporting to CSV or PDF

**Export CSV** — saves a comma-separated file containing all visible (filtered) devices. Columns include: Device Name, Serial Number, Model Identifier, Hardware, RAM/OS, Warranty Status, Warranty Expiry, Warranty Remaining, Coverage Type, Groups, Source, and Last Check-in.

**Export PDF** — saves a formatted A4 landscape PDF report. The report always renders in light mode for print legibility. The first page includes a title, generation timestamp, and device count. Subsequent pages include a continuation header and page numbers. Warranty status cells are colour-coded (green / orange / red / grey).

Both export dialogs present a standard macOS Save panel where you can choose the filename and destination. Files are named `OrchardView_Report_YYYY-MM-DD` by default.

---

## Static group management

Select one or more devices in the report view and click **Add N to Static Group** to open the group management sheet.

The sheet has two modes, selectable at the top:

### Adding devices to an existing group

**Existing Group** mode fetches all static groups from your connected Jamf sources. Groups are organised into sections:

- Jamf Pro — Computers
- Jamf Pro — Mobile Devices
- Jamf School

Use the **search field** to filter the group list by name.

Select a group from the list and click **Add to Group**. Orchard View performs a GET to retrieve the group's current membership, merges the selected devices (by serial number, deduplicating any already-members), then PUTs the updated membership back.

> **Mixed-source selections:** If your selection includes both Jamf Pro and Jamf School devices, choosing a Jamf Pro group will only add the Jamf Pro devices; Jamf School devices are unaffected, and vice versa. An informational banner appears when mixed-source selections are detected.

> **Jamf School group listing:** Some Jamf School instances do not expose a group listing endpoint. If this is the case, a notice is shown and you can use **Create New** mode to create groups for Jamf School devices instead.

### Creating a new static group

**Create New** mode creates a static group with the specified name in each MDM the selected devices belong to. Enter a name and click **Create Group**. If selected devices span both Jamf Pro and Jamf School, a group is created in each.

ASM/ABM devices cannot be added to static groups and are excluded automatically.

---

## Clearing MDM assignments in ASM/ABM

When ASM/ABM is connected, devices that have an MDM server assignment in Apple's systems show an **apple.logo** indicator in the report table. You can clear these assignments in two ways:

**From the report view:** Select the devices and click **Clear ASM Assignment (N)**. A confirmation dialog summarises the action. Once confirmed, Orchard View calls `POST /v1/orgDeviceActivities` in the ASM/ABM API with `activityType: UNASSIGN_DEVICES` for each affected MDM server.

**From the device detail view:** In the Apple School Manager section of the detail sheet, click **Clear MDM Server Assignment**.

> **Important:** Clearing an MDM assignment removes the device from its MDM server in Apple's systems. The device will appear as **Unassigned** in ASM/ABM. This does not unenroll the device from Jamf — you must handle MDM unenrollment separately if needed. Reassignment requires returning to ASM/ABM directly.

---

## Remote commands

Remote commands are available from the **device detail view** under the Remote Commands section. Three commands are supported:

| Command | Effect |
|---|---|
| **Lock Device** | Sends an MDM lock command. The user will need their PIN or password to unlock. |
| **Restart** | Sends an MDM restart command. Unsaved work may be lost. |
| **Erase Device** | Sends an MDM erase command. **This permanently erases all data and cannot be undone.** |

All commands present a confirmation dialog before sending. Erase Device is marked destructive and requires explicit confirmation. Commands are routed to the appropriate API based on the device's source (Jamf Pro MDM API or Jamf School device commands endpoint).

A status banner appears at the top of the detail sheet confirming the command was sent, or displaying an error if it failed.

---

## Appearance settings

Click the **appearance icon** (sun, moon, or half-circle) in the top-right toolbar to choose between:

- **System** — follows macOS system appearance
- **Light** — always light
- **Dark** — always dark

Your preference is saved across sessions. PDF reports always export in light mode regardless of the in-app appearance setting.

---

## Credential management and security

All credentials are stored in the **macOS Keychain** — they are never written to disk in plaintext or transmitted anywhere other than the respective API endpoints.

| Source | Stored keys |
|---|---|
| Jamf Pro | Server URL, Client ID, Client Secret |
| Jamf School | Server URL, Network ID, API Key |
| ASM/ABM | Client ID, Key ID, Organization Name, Scope, Private Key (PEM), Key Filename |

To remove credentials for a source, open **API Settings**, select the relevant tab, and click **Disconnect**. This deletes all stored keys for that source from the Keychain and removes its devices from the table on next refresh.

> **Private key security:** The ASM/ABM private key is stored in the Keychain like all other credentials. The original `.pem` file is no longer needed after import and can be stored securely offline. Orchard View never transmits the private key — it is used only to sign JWT assertions locally before the token exchange with `account.apple.com`.

---

## Logging

Orchard View writes diagnostic output to the system console via `print()` during API requests, connection tests, and remote command execution. To view logs while reproducing an issue, open **Console.app**, select your Mac under **Devices**, and search for "Orchard View" or filter by process name. Diagnostic output includes request status codes and abbreviated response bodies; no credentials, tokens, or private key material are ever logged.

---

## Troubleshooting

### No devices appear after connecting

- Check that the credentials are correct and click **Test Connection** in API Settings.
- Verify the API client (Jamf Pro) or API key (Jamf School) has read permissions for devices.
- Confirm the server URL has no trailing slash and is reachable from your Mac.

### Warranty shows "Unknown" for all devices

- ASM/ABM is either not configured or the connection test has not been run. Open API Settings, go to the ASM tab, and click Test Connection.
- Ensure the ASM/ABM API account has permission to read device and coverage information.
- Check that the private key file matches the Client ID and Key ID — a mismatch will cause JWT signing to fail silently.

### Source attribution is incorrect (devices show wrong MDM source)

Orchard View resolves MDM source by matching the ASM-assigned server name against the subdomain of your configured Jamf URLs (e.g. `blakely` from `blakely.jamfcloud.com`). If both your Jamf Pro and Jamf School URLs share the same subdomain prefix, attribution may be ambiguous. Orchard View uses a longest-match algorithm to select the most specific match. Ensure your server URLs in API Settings are set to the correct and distinct instance URLs.

### Devices appear duplicated

Deduplication matches on serial number. Devices with a missing or placeholder serial (`—`) are excluded from deduplication and will appear once per source. Check the original MDM for devices with blank serial numbers. When ASM/ABM is connected, the enriched record (which includes warranty data and group information) is preferred over a sparse summary record.

### App inventory is empty for a device

- For Jamf Pro computers: the app inventory is fetched on demand from the `computers-inventory` endpoint with the `APPLICATIONS` section. If the device has not recently checked in, the inventory may be stale in Jamf Pro itself.
- For Jamf Pro mobile devices: the app list is fetched from the Classic API (`/JSSResource/mobiledevices/serialnumber/{serial}`). Ensure your API client has Classic API read permissions.
- For Jamf School devices: app data is fetched via `GET /api/devices/{udid}?includeApps=true`. If the device UDID is missing from Jamf School's device record, the app fetch will be skipped.

### Remote command fails

- Confirm the API client (Jamf Pro) or API key (Jamf School) has remote command permissions.
- Jamf Pro requires `Send Computer Remote Commands` and/or `Send Mobile Device Remote Commands` roles.
- ASM/ABM devices do not support remote commands through Orchard View.

### ASM/ABM "Clear Assignment" fails

- Verify the ASM/ABM credentials are still valid (tokens expire; re-test the connection if needed).
- Ensure the API account has permission to manage device activities in ASM/ABM.
- Devices that are already unassigned will be silently skipped — this is expected behaviour.

### Connection test passes but fetch returns zero devices

- Jamf Pro: verify the API role includes Read Computers and Read Mobile Devices.
- Jamf School: verify the Network ID is correct (numeric, from Organisation → Details, not the URL subdomain).
- The app will display a warning banner: *"Connected but no devices were returned. Check API permissions."*

---

## Getting help

If you run into an issue that isn't covered above, open an issue in this repository or reach out in the `#team-jamf-concepts-developers` Slack channel.

---

## Dependencies

Orchard View uses one third-party dependency, pulled in via Swift Package Manager:

| Package | Version | License | Home repository | License file |
|---|---|---|---|---|
| TelemetryDeck SwiftSDK | 2.14.1 | MIT | [github.com/TelemetryDeck/SwiftSDK](https://github.com/TelemetryDeck/SwiftSDK) | [LICENSE](https://github.com/TelemetryDeck/SwiftSDK/blob/main/LICENSE) |

TelemetryDeck is used for anonymous launch analytics. It can be disabled at any time — see **Appearance settings** for the telemetry opt-out toggle.

---

## License, copyright, and privacy

Orchard View is a Jamf Concepts project. Source code and internal tooling remain private under the [Jamf Concepts Use Agreement](https://resources.jamf.com/documents/jamf-concept-projects-use-agreement.pdf).

Copyright 2026, Jamf Software LLC.

See the [Jamf Privacy Policy](https://www.jamf.com/trust-center/privacy/privacy-policy/) for information on data handling.

---

*Orchard View — authored by Eric Blakely*
