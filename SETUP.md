# Environment Setup Guide

Step-by-step instructions for setting up your `.env` file and credentials before running PaypalFlow.

---

## Step 1 — Copy the env template

In your terminal, from the project folder:

```bash
cp .env.example .env
```

Open `.env` in any text editor. You will fill in each section below.

---

## Step 2 — PayPal credentials

### 2a. Create a PayPal developer account

If you don't have one already:

1. Go to [developer.paypal.com](https://developer.paypal.com)
2. Click **Log in to Dashboard** and sign in with your PayPal account

### 2b. Create an app

1. In the dashboard, click **My Apps & Credentials**
2. Choose **Sandbox** (for testing) or **Live** (for real transactions)
3. Click **Create App**
4. Give it a name (e.g. `PaypalFlow`) and click **Create App**

### 2c. Copy credentials into `.env`

On the app page you will see your **Client ID** and **Secret**:

```
PAYPAL_CLIENT_ID=paste_your_client_id_here
PAYPAL_CLIENT_SECRET=paste_your_secret_here
```

### 2d. Set the mode

```
PAYPAL_MODE=sandbox
```

Change to `live` only when you are ready to run against real transactions.

### 2e. Set the start date

Only transactions on or after this date will be fetched:

```
PAYPAL_START_DATE=2026-01-15
```

Change to any date in `YYYY-MM-DD` format.

---

## Step 3 — Google Workspace credentials (optional)

Skip this entire section if you do not need the Google Group feature. The script runs fine without it.

### 3a. Create a Google Cloud project

1. Go to [console.cloud.google.com](https://console.cloud.google.com)
2. Click the project dropdown at the top → **New Project**
3. Give it a name and click **Create**

### 3b. Enable the Admin SDK API

1. In your new project, go to **APIs & Services → Library**
2. Search for **Admin SDK API**
3. Click it and press **Enable**

### 3c. Create a service account

1. Go to **IAM & Admin → Service Accounts**
2. Click **Create Service Account**
3. Enter a name (e.g. `paypalflow-bot`) and click **Create and Continue**
4. Skip the optional role/access steps and click **Done**

### 3d. Enable domain-wide delegation

1. Click on the service account you just created
2. Go to the **Details** tab → click **Edit**
3. Check **Enable Google Workspace Domain-wide Delegation**
4. Click **Save**
5. Note the **Client ID** shown — you will need it in Step 3f

### 3e. Download the JSON key

1. Still on the service account page, go to the **Keys** tab
2. Click **Add Key → Create new key → JSON**
3. The file downloads automatically
4. Rename it to `service_account.json` and place it in the project folder

### 3f. Authorise the scope in Google Admin

1. Go to [admin.google.com](https://admin.google.com) and sign in as a super-admin
2. Navigate to **Security → Access and data control → API controls**
3. Click **Manage domain-wide delegation**
4. Click **Add new** and enter:
   - **Client ID**: the one from Step 3d
   - **OAuth Scopes**: `https://www.googleapis.com/auth/admin.directory.group.member`
5. Click **Authorise**

### 3g. Fill in `.env`

```
GOOGLE_SERVICE_ACCOUNT_FILE=service_account.json
GOOGLE_ADMIN_EMAIL=admin@yourdomain.com
GOOGLE_GROUP_EMAIL=members@yourdomain.com
```

- `GOOGLE_ADMIN_EMAIL` — a super-admin account in your Google Workspace domain
- `GOOGLE_GROUP_EMAIL` — the group that payers will be added to (must already exist)

---

## Step 4 — (Optional) Change the output file path

By default the Excel file is saved as `transactions.xlsx` in the project folder. To change it:

```
EXCEL_OUTPUT_PATH=C:/Users/YourName/Desktop/transactions.xlsx
```

---

## Step 5 — Install Python dependencies

```bash
pip install -r requirements.txt
```

---

## Step 6 — Run the script

```bash
python main.py
```

You should see log output showing transactions being fetched and the Excel file being written.

---

## Checklist

- [ ] `.env` file created from `.env.example`
- [ ] `PAYPAL_CLIENT_ID` and `PAYPAL_CLIENT_SECRET` filled in
- [ ] `PAYPAL_MODE` set to `sandbox` or `live`
- [ ] `PAYPAL_START_DATE` set to desired date
- [ ] _(Optional)_ `service_account.json` placed in project folder
- [ ] _(Optional)_ Google Admin SDK enabled and domain-wide delegation configured
- [ ] _(Optional)_ Google `.env` variables filled in
- [ ] `pip install -r requirements.txt` run successfully
- [ ] `python main.py` runs without errors

---

## Troubleshooting

**`KeyError: PAYPAL_CLIENT_ID`**
Your `.env` file is missing or the variable is not set. Make sure you copied `.env.example` to `.env` and filled it in.

**`401 Unauthorized` from PayPal**
Your Client ID or Secret is wrong, or you are using sandbox credentials with `PAYPAL_MODE=live` (or vice versa).

**Google step is skipped silently**
Check that `service_account.json` exists in the project folder and that all three Google variables are set in `.env`.

**`HttpError 403` from Google**
Domain-wide delegation is not set up correctly. Re-check Steps 3d and 3f. Make sure the correct Client ID and scope are authorised.

**`Member already exists` errors**
These are handled automatically and can be safely ignored — the script skips duplicates.
