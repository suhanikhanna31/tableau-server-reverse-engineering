# Tableau Server Data Extraction Client

A Python client that programmatically pulls live dashboard data from Tableau Server by working directly against its internal REST endpoints — replacing a 6-hour weekly manual export process with automated, real-time extraction.

---

## 📌 Overview

Tableau Server's public REST API doesn't expose everything the web UI itself uses under the hood — session handling, workbook metadata, and embedded data source queries are all driven by internal endpoints not covered in official docs. This project reverse-engineers those endpoints by observing the traffic the browser actually sends, then wraps them in a clean Python client so dashboard data can be pulled on demand instead of exported by hand every week.

---

## ✨ Highlights

| Area | Result |
|---|---|
| API discovery | Mapped undocumented internal Tableau Server endpoints by intercepting XHR traffic via **Chrome DevTools** and **Charles Proxy** |
| Session handling | Reverse-engineered the session token lifecycle (issue, refresh, expiry) to keep the client authenticated without manual re-login |
| Metadata mapping | Documented workbook metadata schema and embedded data source query parameters |
| Client | Built a **Python (`requests`)** client to extract live dashboard data on demand |
| Pipeline | Fed extracted data directly into a **Pandas** transformation pipeline for downstream reporting |
| Impact | Eliminated a **6-hour/week manual data pull**, enabling real-time report automation |
| Adoption | Packaged as a reusable internal library, later adopted by **3 teams** |

---

## 🏗️ How It Works

```
   Browser (Tableau Server UI)
            │
            │  observed via DevTools + Charles Proxy
            ▼
   Internal/undocumented endpoints
     • session token issuance
     • workbook metadata
     • embedded data source queries
            │
            │  reverse-engineered request/response shapes
            ▼
   ┌─────────────────────────┐
   │  Python requests client  │
   │  (auth, retries, parsing)│
   └────────────┬─────────────┘
                │
                ▼
   ┌─────────────────────────┐
   │   Pandas transformation  │
   │        pipeline          │
   └────────────┬─────────────┘
                │
                ▼
     Automated / real-time reports
```

---

## 🔍 Reverse-Engineering Process

1. **Traffic capture** — Used Chrome DevTools' Network tab and Charles Proxy to intercept XHR requests made by the Tableau Server web UI while navigating dashboards, opening workbooks, and triggering data refreshes.
2. **Session lifecycle mapping** — Identified how session tokens are issued at login, attached to subsequent requests, and refreshed/expired, so the client could replicate authenticated sessions programmatically.
3. **Schema mapping** — Catalogued the JSON shape of workbook metadata responses (workbook IDs, view IDs, project hierarchy) and the query parameters used by embedded data source calls.
4. **Client implementation** — Translated the mapped endpoints into a `requests`-based Python client with typed request builders, response parsers, and error handling for auth expiry/retries.

---

## 📁 Suggested Repo Structure

```
.
├── README.md
├── requirements.txt
├── .env.example
├── .gitignore
├── LICENSE
├── src/
│   └── tableau_client/
│       ├── __init__.py
│       ├── auth.py              # session token lifecycle handling
│       ├── client.py            # core requests-based API client
│       ├── endpoints.py         # mapped internal endpoint definitions
│       ├── schemas.py           # workbook / data source metadata schemas
│       └── transform.py         # Pandas transformation pipeline
├── examples/
│   └── extract_dashboard.py     # sample script: pull + transform a live dashboard
└── tests/
    └── test_client.py
```

---

## ⚙️ Setup

```bash
git clone https://github.com/<your-username>/<repo-name>.git
cd <repo-name>
python -m venv venv && source venv/bin/activate
pip install -r requirements.txt
cp .env.example .env      # fill in your Tableau Server host + credentials
```

### Example usage

```python
from tableau_client import TableauClient

client = TableauClient(server_url="https://tableau.yourcompany.com")
client.login(username="user", password="pass")

data = client.get_dashboard_data(workbook_id="wb-123", view_id="view-456")
df = client.transform_to_dataframe(data)

df.to_csv("weekly_report.csv", index=False)
```

---

## ⚠️ Notes & Caveats

- This client depends on **internal, undocumented Tableau Server behavior**, not the officially supported REST API — it can break on server upgrades and should be re-validated against traffic captures after any Tableau version change.
- Intended for use against servers you or your organization own/administer, with credentials you're authorized to use.
- Because it isn't Tableau's public API, there's no compatibility guarantee across versions — pin your Tableau Server version in the README of any real deployment and note it here.

---

## 📝 License

MIT — see [LICENSE](./LICENSE).
