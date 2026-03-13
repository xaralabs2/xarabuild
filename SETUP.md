# XaraLabs — Full Automation Setup Guide
**xarabuild.dev · From zero to fully automated in ~2 hours**

---

## What This Repo Contains

```
xarabuild/
├── index.html                        ← Full website (deploys to Cloudflare Pages)
├── make-scenario.json                ← Make.com automation blueprint (import directly)
├── .github/
│   └── workflows/
│       └── deploy.yml                ← GitHub Actions: auto-deploy on every push
├── docs/
│   └── templates/
│       ├── PRD-TEMPLATE.md           ← Copy into Google Docs as PRD template
│       └── NDA-TEMPLATE.md           ← Copy into Google Docs as NDA template
├── .gitignore
└── SETUP.md                          ← This file
```

---

## What Fires Automatically on Every Form Submission

```
Client submits brief
        │
        ▼
Make.com Webhook receives data
        │
        ├── Trello    →  Card created with full checklist + client details
        ├── Slack     →  Block message to #new-projects with Trello link
        ├── Gmail     →  Branded confirmation email → client
        ├── Gmail     →  Internal brief alert → your team
        ├── G Docs    →  PRD document created from template in Drive
        └── G Docs    →  NDA document created from template in Drive
```

---

## PHASE 1 — Accounts to Create (30 min)

Create accounts at each of these. All free tiers work.

| Service | URL | Plan Needed |
|---|---|---|
| GitHub | github.com | Free |
| Make.com | make.com | Free (1,000 ops/mo) |
| Trello | trello.com | Free |
| Slack | slack.com | Free |
| Google (Gmail + Drive + Docs) | google.com | Free |
| Cloudflare | cloudflare.com | Free |

---

## PHASE 2 — GitHub Setup (10 min)

### 2.1 Create the repo

```bash
# Install GitHub CLI if needed: brew install gh
gh auth login

# Clone this repo or init fresh
git init xarabuild
cd xarabuild
# Copy all files from this package into the folder

git add .
git commit -m "init: XaraLabs website + automation"
gh repo create xarabuild --public --push --source=.
```

### 2.2 Add GitHub Secrets

Go to: **github.com/YOUR_USERNAME/xarabuild → Settings → Secrets → Actions**

Add these two secrets:

| Secret Name | Where to get it |
|---|---|
| `CLOUDFLARE_API_TOKEN` | Cloudflare → My Profile → API Tokens → Create Token → "Edit Cloudflare Workers" template |
| `CLOUDFLARE_ACCOUNT_ID` | Cloudflare → any domain → Overview → right sidebar |

Once added, every push to `main` automatically deploys to Cloudflare Pages.

---

## PHASE 3 — Cloudflare Pages Setup (10 min)

### 3.1 Create the Pages project

1. Cloudflare Dashboard → **Pages** → Create a Project
2. Connect to Git → select your `xarabuild` repo
3. Settings:
   - **Project name:** `xarabuild`
   - **Build command:** *(leave blank)*
   - **Output directory:** `/`
4. Deploy

### 3.2 Add your custom domain

1. Pages → your project → **Custom Domains** → Add
2. Type `xarabuild.dev` → Activate domain
3. Type `www.xarabuild.dev` → Activate domain
4. Cloudflare creates DNS records automatically

### 3.3 SSL Settings

Cloudflare Dashboard → SSL/TLS:
- Mode: **Full (Strict)**
- Always Use HTTPS: **ON**
- Automatic HTTPS Rewrites: **ON**

---

## PHASE 4 — Make.com Automation (45 min)

This is the core of the automation. Work through each step in order.

### 4.1 Create Make.com account

1. Go to make.com → Sign up (free)
2. Dashboard → **Scenarios** → **Import Blueprint**
3. Upload `make-scenario.json` from this repo
4. The scenario loads with placeholders you'll replace below

### 4.2 Create Make.com Webhook

1. In your imported scenario, click the first module (Webhooks)
2. Click **Add** → name it "XaraLabs Intake"
3. Copy the webhook URL — it looks like:
   `https://hook.eu1.make.com/XXXXXXXXXXXXXXXX`
4. Open `index.html` in your repo, find line ~540:
   ```js
   const WEBHOOK_URL = 'https://YOUR_WEBHOOK_URL_HERE';
   ```
   Replace with your Make.com URL
5. Commit and push — GitHub Actions deploys automatically

### 4.3 Set up Trello

1. trello.com → Sign up → create a board: **"XaraLabs Projects"**
2. Add these lists to the board:
   - **Incoming** ← new cards land here
   - **In Review**
   - **In Progress**
   - **QA**
   - **Delivered**
3. Get your Trello credentials:
   - API Key: trello.com/app-key
   - Token: click "Generate Token" on the same page
4. Get your Board ID and List ID:
   - Open the board → add `.json` to the URL
   - Search for `"id"` at the top (board ID) and find `"idList"` for the Incoming list
5. In Make.com → your scenario → click the Trello module → **Connect** → paste API Key + Token
6. In the `make-scenario.json` replace:
   - `REPLACE_WITH_TRELLO_BOARD_ID` → your board ID
   - `REPLACE_WITH_TRELLO_INCOMING_LIST_ID` → your Incoming list ID

### 4.4 Set up Slack

1. slack.com → Create a workspace: **"XaraLabs"**
2. Create a channel: `#new-projects`
3. Get your channel ID:
   - Right-click the channel → View channel details → scroll to bottom → copy Channel ID
4. In Make.com → Slack module → **Connect** → authorize your workspace
5. Replace `REPLACE_WITH_SLACK_CHANNEL_ID` with your channel ID

### 4.5 Set up Gmail

1. Create a dedicated Gmail: `xarabuild.hello@gmail.com` (or use existing)
2. Enable 2FA → Google Account → Security → App Passwords → Generate
3. Select Mail → Other → type "Make.com" → copy the 16-char password
4. In Make.com → Gmail modules → **Connect** → use your Gmail + app password
5. In module 17 (team alert email), replace `REPLACE_WITH_YOUR_TEAM_EMAIL` with your email

### 4.6 Set up Google Docs Templates

**Create the PRD Template:**
1. docs.google.com → New Document → title it: **"[TEMPLATE] XaraLabs PRD"**
2. Copy the entire contents of `docs/templates/PRD-TEMPLATE.md` into it
3. The `{{PLACEHOLDER}}` variables are replaced by Make.com automatically
4. Get the Doc ID from the URL:
   `docs.google.com/document/d/THIS_IS_THE_ID/edit`
5. Replace `REPLACE_WITH_PRD_TEMPLATE_DOC_ID` in `make-scenario.json`

**Create the NDA Template:**
1. New Document → title: **"[TEMPLATE] XaraLabs NDA"**
2. Copy contents of `docs/templates/NDA-TEMPLATE.md`
3. Get the Doc ID and replace `REPLACE_WITH_NDA_TEMPLATE_DOC_ID`

**Create output folder in Drive:**
1. drive.google.com → New Folder → **"XaraLabs — Client Projects"**
2. Open folder → copy the folder ID from the URL
3. Replace both `REPLACE_WITH_GOOGLE_DRIVE_FOLDER_ID` entries

### 4.7 Connect Google Docs in Make.com
1. Click the Google Docs modules in Make.com → **Connect** → authorize your Google account
2. The template IDs and folder ID in the mapper fields should already match what you set above

### 4.8 Activate the Scenario
1. Make.com → your scenario → toggle **ON** (bottom left)
2. Set schedule to: **Immediately** (webhook-triggered, always on)

---

## PHASE 5 — DNS Setup (10 min, after Cloudflare is active)

Once your domain is on Cloudflare (nameservers updated at GoDaddy):

Cloudflare DNS → Add these records:

| Type | Name | Value | Proxy |
|---|---|---|---|
| CNAME | `@` | `xarabuild.pages.dev` | ✅ |
| CNAME | `www` | `xarabuild.pages.dev` | ✅ |

Cloudflare Pages adds these automatically when you set the custom domain — just verify they're there.

**Email routing** (receive at hello@xarabuild.dev):
1. Cloudflare → Email → Email Routing → Enable
2. Add address: `hello@xarabuild.dev` → Forward to your Gmail
3. Add catch-all: forward to Gmail

---

## PHASE 6 — End-to-End Test (15 min)

1. Go to `https://xarabuild.dev`
2. Fill out the form completely — use your own email
3. Submit
4. Within 60 seconds verify:
   - ✅ Trello card in Incoming list with full checklist
   - ✅ Slack message in #new-projects with Trello link button
   - ✅ Confirmation email in your inbox (client copy)
   - ✅ Internal brief email to your team address
   - ✅ PRD doc in Google Drive → XaraLabs — Client Projects folder
   - ✅ NDA doc in the same folder
5. Click through the Make.com scenario history to see each step

---

## PHASE 7 — Future: When to Upgrade Make.com

Free tier = 1,000 operations/month. Each submission uses ~12 ops (checklist items + emails + docs).
That's ~83 submissions/month before you hit the limit.

When you're ready to scale:
- Make.com Core: $9/mo → 10,000 ops/month (~833 submissions)
- Make.com Pro: $16/mo → 10,000 ops + unlimited scenarios

Or migrate to the Node.js backend in `backend/` (available on request) to handle unlimited volume at $5/mo on DigitalOcean.

---

## Quick Reference — All the IDs You'll Collect

Fill this in as you work through setup:

```
GitHub repo URL:        github.com/___________/xarabuild
Cloudflare Account ID:  ______________________________
Cloudflare API Token:   ______________________________
Make.com Webhook URL:   https://hook.__.make.com/______
Trello API Key:         ______________________________
Trello Token:           ______________________________
Trello Board ID:        ______________________________
Trello Incoming List:   ______________________________
Slack Channel ID:       ______________________________
Gmail address:          ______________________________
PRD Template Doc ID:    ______________________________
NDA Template Doc ID:    ______________________________
Drive Folder ID:        ______________________________
```

---

## Something Not Working?

| Problem | Fix |
|---|---|
| Form submits but nothing fires | Check Make.com scenario is toggled ON; check webhook URL in index.html is correct |
| Trello card missing checklist | Make.com ops may have hit limit; check scenario run history |
| Emails landing in spam | Set up SPF/DKIM in Cloudflare Email settings; use a dedicated Gmail |
| GitHub Actions failing | Check CLOUDFLARE_API_TOKEN and CLOUDFLARE_ACCOUNT_ID secrets are set correctly |
| Google Docs not creating | Re-authorize Google connection in Make.com; check template IDs are correct |
| Site not loading on xarabuild.dev | Verify Cloudflare nameservers are active at GoDaddy; check Pages custom domain is added |

---

*XaraLabs · xarabuild.dev · Senior-Led App Factory*
