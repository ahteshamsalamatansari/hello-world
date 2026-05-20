# Deploying Rex PER↔MJK Scraper on Render as a Cron Job

This guide explains how to set up the scraper to run automatically every day at **21:00 AWST (Perth local time)** as a scheduled **Render Cron Job** and send the resulting Excel report to your email using **Gmail SMTP with Google App Passwords**.

---

## 1. Timezone and Cron Schedule

Render's Cron scheduler evaluates all schedules in **UTC** time.
Since Perth local time (AWST) is **UTC+8**:
* **Target Perth Time**: `21:00 AWST`
* **Equivalent UTC Time**: `21:00 - 8 hours = 13:00 UTC`

Therefore, the schedule is defined in `render.yaml` as:
```yaml
schedule: "0 13 * * *"
```

---

## 2. Setting Up Google App Passwords (Gmail SMTP)

To allow the Python script to send email notifications through your Gmail account, you must generate a secure **Google App Password**:

1. Go to your **[Google Account Security settings](https://myaccount.google.com/security)**.
2. Make sure **2-Step Verification** is enabled for your account.
3. Under the "How you sign in to Google" section, click on **2-Step Verification**.
4. Scroll down to the bottom of the page and click on **App passwords**.
5. Enter a name for the app (e.g., `Rex Flight Scraper`) and click **Create**.
6. Google will display a **16-character passcode** (e.g., `abcd efgh ijkl mnop`).
7. **Copy this password** — you will use it as the `SMTP_PASSWORD` environment variable in the next step.

---

## 3. Environment Variables Configuration

To run successfully, you need to add your sensitive credentials to the **Render Dashboard**. Do not hardcode them in your repository files.

Configure the following environment variables in your Render service dashboard under **Environment**:

| Variable | Description | Example / Recommended Value |
|---|---|---|
| `BD_BROWSER_USER` | Bright Data Scraping Browser username | `brd-customer-hl_...-zone-...` |
| `BD_BROWSER_PASS` | Bright Data Scraping Browser password | `xxxxxxxxxxxx` |
| `BD_AUTH_TOKEN` | Bright Data API Authentication Token | `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` |
| `SMTP_EMAIL` | The Gmail address sending the report | `your-email@gmail.com` |
| `SMTP_PASSWORD` | The 16-character Google App Password | `abcdefghijklmnop` |
| `RECIPIENT_EMAIL` | The email address where the report is sent | `recipient-email@domain.com` |
| `SMTP_SERVER` | SMTP host (optional, defaults to Gmail) | `smtp.gmail.com` |
| `SMTP_PORT` | SMTP port (optional, defaults to TLS port) | `587` |

---

## 4. How to Deploy on Render

### Option A: Using the `render.yaml` Blueprint (Recommended)
1. Commit and push the latest changes (including your updated `render.yaml` and the scraper script) to your GitHub repository.
2. Log in to the **[Render Dashboard](https://dashboard.render.com)**.
3. Click **New +** at the top right and select **Blueprint**.
4. Connect your GitHub repository.
5. Render will detect the `render.yaml` file automatically. Click **Apply**.
6. Go to the newly created service dashboard, click on **Environment**, and manually fill in the sensitive credential environment variables listed above.

### Option B: Manual Creation in Render Dashboard
If you prefer not to use Blueprints, you can create the Cron job manually in the UI:
1. Click **New +** and select **Cron Job**.
2. Connect your GitHub repository.
3. Configure the following fields:
   * **Name**: `rex-per-mjk-scraper`
   * **Environment / Runtime**: `Python`
   * **Schedule**: `0 13 * * *` (Runs at 13:00 UTC / 21:00 Perth time daily)
   * **Build Command**: `pip install -r requirements.txt`
   * **Start Command**: `python Rex_per_mjk_Playwright_final.py --routes PER-MJK,MJK-PER --days 84 --resume`
4. Add all environment variables listed in Section 3 under the **Environment** section of the service settings.

---

## 5. Verification & Testing

To test the deployment immediately without waiting until 21:00:
1. Open the cron job service in your Render Dashboard.
2. Click the **Trigger Run** button at the top right to start a manual test execution.
3. Monitor the live console logs.
4. Verify that the scraper completes and logs:
   `📧 Sending email report to ... via SMTP...` followed by `✅ Email report sent successfully!`.
5. Check your recipient inbox for the report email with the `.xlsx` attachment.
