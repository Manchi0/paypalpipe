# PaypalFlow

Fetches successful PayPal payments since a given start date, writes them to a formatted Excel file, and optionally adds each payer to a Google Workspace group.

---

## What it does

1. **Fetches transactions** from PayPal (sandbox or live) starting from `PAYPAL_START_DATE`
2. **Writes to Excel** (`transactions.xlsx`) with columns: Transaction ID, Date, Payer Name, Payer Email, Amount, Currency, Status
3. **Adds payers to a Google Group** (optional — skipped if not configured)

---

## Requirements

- Python 3.10+
- A PayPal developer account (sandbox for testing, live for production)
- _(Optional)_ A Google Workspace account with Admin SDK access

---

## Setup

### 1. Install dependencies

```bash
pip install -r requirements.txt
```

### 2. Configure credentials

Copy the example env file and fill in your values:

```bash
cp .env.example .env
```

Then edit `.env` — see the [Environment Variables](#environment-variables) section below.

### 3. Run

```bash
python main.py
```

---

## Environment Variables

Copy `.env.example` to `.env` and fill in the values below.

| Variable | Required | Default | Description |
|---|---|---|---|
| `PAYPAL_CLIENT_ID` | Yes | — | From your PayPal developer app |
| `PAYPAL_CLIENT_SECRET` | Yes | — | From your PayPal developer app |
| `PAYPAL_MODE` | No | `sandbox` | `sandbox` for testing, `live` for real transactions |
| `PAYPAL_START_DATE` | No | `2026-01-15` | Only fetch transactions on or after this date (`YYYY-MM-DD`) |
| `GOOGLE_SERVICE_ACCOUNT_FILE` | No | `service_account.json` | Path to your Google service account key file |
| `GOOGLE_ADMIN_EMAIL` | No | — | Super-admin email in your Google Workspace domain |
| `GOOGLE_GROUP_EMAIL` | No | — | Google Group email to add payers to |
| `EXCEL_OUTPUT_PATH` | No | `transactions.xlsx` | Output file path |

> The Google variables are all optional. If any are missing or `service_account.json` is not found, the Google step is silently skipped.

---

## PayPal Setup

### Getting your credentials

1. Go to [developer.paypal.com](https://developer.paypal.com)
2. Log in and navigate to **My Apps & Credentials**
3. Under **Sandbox** (for testing) or **Live** (for production), click **Create App**
4. Copy the **Client ID** and **Secret** into your `.env`

### Sandbox vs Live

- Set `PAYPAL_MODE=sandbox` while testing — no real money moves
- Set `PAYPAL_MODE=live` when you are ready to run against real transactions
- Your sandbox and live apps have separate credentials

---

## Google Workspace Setup (optional)

Only needed if you want payers automatically added to a Google Group.

### Step 1 — Create a service account

1. Go to [Google Cloud Console](https://console.cloud.google.com)
2. Create a new project (or use an existing one)
3. Navigate to **IAM & Admin → Service Accounts → Create Service Account**
4. Give it a name, click through to finish
5. Click the service account → **Keys → Add Key → Create new key → JSON**
6. Save the downloaded file as `service_account.json` in this directory

### Step 2 — Enable the Admin SDK API

1. In Google Cloud Console go to **APIs & Services → Library**
2. Search for **Admin SDK API** and enable it

### Step 3 — Enable domain-wide delegation

1. Still on the service account page, click **Edit** → check **Enable Google Workspace Domain-wide Delegation** → Save
2. Note the **Client ID** shown on the service account

### Step 4 — Grant the scope in Google Admin

1. Go to [admin.google.com](https://admin.google.com)
2. Navigate to **Security → Access and data control → API controls → Manage domain-wide delegation**
3. Click **Add new** and enter:
   - **Client ID**: the one from Step 3
   - **OAuth Scopes**: `https://www.googleapis.com/auth/admin.directory.group.member`
4. Click **Authorise**

### Step 5 — Configure `.env`

```
GOOGLE_SERVICE_ACCOUNT_FILE=service_account.json
GOOGLE_ADMIN_EMAIL=admin@yourdomain.com
GOOGLE_GROUP_EMAIL=members@yourdomain.com
```

---

## Output

The script produces `transactions.xlsx` (or the path set in `EXCEL_OUTPUT_PATH`) with the following columns:

| Column | Description |
|---|---|
| Transaction ID | PayPal transaction identifier |
| Date | Transaction initiation date/time |
| Payer Name | Full name of the payer |
| Payer Email | Email address of the payer |
| Amount | Transaction amount (stored as a number) |
| Currency | Currency code (e.g. USD) |
| Status | Transaction status (`S` = successful) |

---

## Security

- **Never commit `.env` or `service_account.json`** — both are listed in `.gitignore`
- Only share `.env.example` as a template
- If credentials are ever accidentally committed, regenerate them immediately:
  - PayPal: developer.paypal.com → your app → regenerate secret
  - Google: Cloud Console → service account → delete old key → create new one

---

## Running Tests

```bash
python -m pytest test_main.py -v
```

All tests are unit tests — no real API calls or credentials needed.
