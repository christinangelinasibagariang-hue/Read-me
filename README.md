# SCAS AI Email Operations Bot — safe pilot v1

## What it does
- Reads Hostinger IMAP inbox via SSL without marking messages read. First run establishes a baseline (no drafts or alerts for existing mail).
- Uses Cloudflare Workers AI REST API to detect **candidate** freight inquiries and missing shipment details.
- Saves a **draft** reply in Hostinger's Drafts mailbox where available (should sync into Outlook via IMAP).
- Emails an **internal alert** only (to `ALERT_EMAIL`) for AI-classified complete inquiries that have no later sent email, if a Sent mailbox is detectable.
- Stores only processed `Message-ID` references/timestamps in `state.json`. The GitHub repository **must remain private**.
- **Never sends a customer email automatically. Never reads or writes Supabase. Does not change existing CRM data.**

## Limitations to understand before enabling
- This is a pilot. AI may miss information, misclassify threads, or generate unnecessary drafts. Review all drafts.
- Follow-up checks a *later sent message addressed to the sender*, not an actual quotation in SCAS CRM. It might flag false positives or miss sent quotations. Do not label its alerts as confirmed 'unoffered'.
- It does not yet create CRM inquiry records or check actual inquiry/quotation status in Supabase: table schemas, RLS permissions and mapping are unknown. Integrate later with a **read-only Supabase view** after schema review and database backup.
- Draft folder names depend on mailbox; when no Drafts folder is detected, the bot does not attempt to create it.
- GitHub Actions cron may be delayed, and GitHub Actions may disable inactivity schedules. This is *not* guaranteed real-time delivery.
- Secret must be in **GitHub Actions secrets**, not in config files or chat. Prefer a dedicated mailbox or mailbox-specific password if Hostinger supports it. Limit access to the private repo.
- Cloudflare Workers AI usage/limits/billing are subject to account configuration.
- Incoming email is untrusted. Never instruct the bot to take arbitrary actions based on its text.

## Deployment from a phone
1. Create a **private** GitHub repository (prefer dedicated `scas-email-ops-bot`).
2. Upload `bot.py`, `state.json`, `README.md`, and `.github/workflows/scas-email-bot.yml` preserving the directory.
3. Repository -> Settings -> Secrets and variables -> Actions -> New repository secret. Add:
   - `HOSTINGER_EMAIL`: your mailbox address, such as `sales@scaslogisticspacking.com`
   - `HOSTINGER_PASSWORD`: Hostinger mailbox password. **Do not share this with ChatGPT.**
   - `ALERT_EMAIL`: internal staff email (your choice; must not be a customer address)
   - `CF_ACCOUNT_ID`: Cloudflare account ID
   - `CF_AI_TOKEN`: Cloudflare token specifically scoped to Workers AI REST API
4. Open Actions -> SCAS AI Email Operations -> Run workflow.
5. Inspect Actions logs (ensure no sensitive email contents printed), Outlook/Hostinger **Drafts** and internal notification mailbox.
6. If pilot works as expected, the scheduled workflow runs hourly. Disable the workflow to stop.

## Hostinger connection
Standard Hostinger Email: `imap.hostinger.com:993` (SSL), `smtp.hostinger.com:465` (SSL). Check your actual mailbox provider in hPanel: Titan/cPanel may differ.

## Extra caution
- First test with a dedicated test inbox or controlled messages. Avoid enabling this on your live customer inbox until reviewed.
- To avoid losing data, this package contains **no SQL migration**, and no website `index.html`, `config.js` or CRM code replacements.
- A newer deployment must not include service-role keys in a client-facing application.
