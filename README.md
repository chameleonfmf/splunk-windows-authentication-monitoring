
# Windows Authentication Monitoring with Splunk

## Overview

This project simulates a SOC analyst workflow using **Splunk** to monitor Windows authentication activity.

You:
- Ingested sample Windows Security logs into Splunk
- Built searches to analyze failed and successful logons
- Created a **Windows Authentication Monitoring** dashboard
- Implemented a scheduled alert to detect multiple failed logons from the same source (brute-force style behavior)
- Validated the alert via triggered events

This repository is meant to be shared with recruiters and hiring managers as a **SOC analyst portfolio project**.

---

## Repository Structure

```text
.
├── README.md                          # Project documentation (this file)
├── sample_windows_security_logs.csv   # Sample dataset used in Splunk
├── spl_queries.spl                    # SPL searches used in the project
└── images/                            # (You add) Screenshots exported from Splunk
    ├── dashboard_overview.png
    ├── failed_logons_search.png
    ├── alert_config.png
    └── alert_triggered.png
```

> **Note:** The `images/` folder is just a suggestion. Create it in your repo and upload your own screenshots with those names, or update the paths in this README.

---

## Dataset

File: `sample_windows_security_logs.csv`

Columns:
- `TimeCreated` – timestamp of the event
- `EventCode` – Windows Security event ID (4624 = successful logon, 4625 = failed logon)
- `Account_Name` – account used for authentication
- `IpAddress` – source IP address of the logon attempt
- `Workstation` – workstation name
- `Message` – generic event message

This CSV was generated to simulate realistic Windows Security logon activity.

---

## How to Recreate the Lab in Splunk

### 1. Start Splunk

- Launch **Splunk Enterprise** on your local machine (free trial/developer license is fine).
- Open **Splunk Web** in your browser (usually http://localhost:8000).

### 2. Ingest the Sample Logs

1. Go to **Settings → Add Data → Upload**.
2. Select `sample_windows_security_logs.csv`.
3. Click **Next** until you reach **Set Source Type**.
4. Choose source type **`csv`** (or any structured CSV type).
5. Set the index to `main` (or your preferred index).
6. Click **Review → Submit**, then **Start Searching**.

### 3. Run the Core SPL Searches

All SPL used in this project is stored in `spl_queries.spl` and documented below.

You can paste each search into **Search & Reporting** in Splunk to validate the results.

---

## SPL Queries

### 1. Failed Logons (EventCode 4625)

**All failed logon events:**

```spl
index=* EventCode=4625
```

**Failed logons over time:**

```spl
index=* EventCode=4625
| timechart span=1h count
```

**Failed logons by account name:**

```spl
index=* EventCode=4625
| stats count by Account_Name
| sort -count
```

**Failed logons by IP address:**

```spl
index=* EventCode=4625
| stats count by IpAddress
| sort -count
```

---

### 2. Successful Logons (EventCode 4624)

**All successful logon events:**

```spl
index=* EventCode=4624
```

**Successful logons over time:**

```spl
index=* EventCode=4624
| timechart span=1h count
```

**Successful logons by account name:**

```spl
index=* EventCode=4624
| stats count by Account_Name
| sort -count
```

---

### 3. Correlation: Authentication Success vs Failure

This search compares failed vs successful logons over time.

```spl
index=* (EventCode=4624 OR EventCode=4625)
| eval Status = if(EventCode=4625, "Failed", "Successful")
| timechart span=1h count by Status
```

Use this search as a **time-series panel** in the dashboard to visually compare normal vs suspicious behavior.

---

### 4. Detection Logic: Multiple Failed Logons per Account & IP

This query aggregates failed logons and highlights possible brute-force style activity.

```spl
index=* EventCode=4625
| stats count by Account_Name, IpAddress
| where count >= 2
```

- In a real environment you might add a time window and tune the threshold higher (e.g., `count >= 10`), but `>= 2` works well with smaller lab datasets.

---

## Dashboard: *Windows Authentication Monitoring*

In Splunk, create a dashboard named:

> **Windows Authentication Monitoring**

Add panels based on the searches above. Suggested layout:

1. **Row 1**
   - *Failed logons over time* (`timechart` for EventCode=4625)
   - *Successful logons over time* (`timechart` for EventCode=4624)

2. **Row 2**
   - *Failed logons by account name*
   - *Failed logons by IP address*

3. **Row 3**
   - *Successful logons by account name*
   - *Authentication Success vs Failure (Time Comparison)*

Take a screenshot of this dashboard and save it as:

- `images/dashboard_overview.png`

---

## Alert: Multiple Failed Logons per Account & IP

**Search used for alert:**

```spl
index=* EventCode=4625
| stats count by Account_Name, IpAddress
| where count >= 2
```

### Alert Configuration (example)

- **Alert type:** Scheduled
- **Schedule:** Every 5 minutes (or a cron schedule such as `*/5 * * * *`)
- **Trigger condition:** `Number of Results > 0`
- **Trigger action:** Add to Triggered Alerts (optionally email, webhook, etc.)

This alert represents a basic brute-force detection used by a SOC analyst to monitor suspicious authentication behavior.

Take screenshots of:

- Alert configuration page → `images/alert_config.png`
- Triggered alerts showing this alert firing → `images/alert_triggered.png`

---

## How to Talk About This Project (Resume / Interviews)

**Suggested resume bullet:**

> Built a Splunk-based *Windows Authentication Monitoring* solution by ingesting Windows Security logs, creating dashboards for failed and successful logons (EventCodes 4624 & 4625), and implementing a scheduled alert to detect multiple failed logons from the same account and IP, simulating brute-force detection in a SOC environment.

**Key concepts you can discuss:**
- Difference between EventCode 4624 (successful) and 4625 (failed) logons
- How to use `stats` and `timechart` in SPL
- Why multiple failed logons from the same source may indicate brute-force attempts
- How dashboards and alerts help a Tier 1 SOC analyst triage authentication incidents

---

## How Recruiters / Reviewers Can Reproduce

1. Install Splunk Enterprise (free) locally.
2. Upload `sample_windows_security_logs.csv` as a CSV source type.
3. Run the SPL queries from `spl_queries.spl`.
4. Create the **Windows Authentication Monitoring** dashboard and add the panels listed above.
5. Configure the **Multiple Failed Logins per Account & IP** alert using the SPL and settings in this README.
6. Use the dashboard and alert history to review authentication activity.

This repository gives them everything needed to reproduce the core of your lab environment.
