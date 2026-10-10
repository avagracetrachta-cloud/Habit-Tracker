# AI Email Sorter

Automatically sorts your Gmail inbox into **high** and **low** priority by applying Gmail labels
(`AI-High-Priority` and `AI-Low-Priority`). Uses Claude to read each email and decide, with a simple
keyword fallback if you don't have an API key.

## How it works

1. Connects to Gmail through the official Gmail API (OAuth, so no password is stored).
2. Fetches recent inbox emails that haven't been labeled yet.
3. Sends each one (sender, subject, first ~2,000 characters) to Claude, which returns `high` or `low` plus a short reason.
4. Applies the matching Gmail label. Optionally archives low-priority mail.

It never deletes mail. If the AI errors or is unsure, the email defaults to **high** so nothing important gets buried.

## Setup

### 1. Install

```bash
cd email-sorter
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

### 2. Get Gmail API credentials (one time, about 5 minutes)

1. Go to [console.cloud.google.com](https://console.cloud.google.com) and create a new project.
2. **APIs & Services → Library** → search **Gmail API** → **Enable**.
3. **APIs & Services → OAuth consent screen** → choose **External**, fill in the app name and your email, and add your own Gmail address under **Test users**.
4. **APIs & Services → Credentials → Create Credentials → OAuth client ID** → application type **Desktop app**.
5. Download the JSON, rename it `credentials.json`, and put it in this folder.

### 3. (Optional) Turn on AI mode

Get a key from [console.anthropic.com](https://console.anthropic.com), then:

```bash
export ANTHROPIC_API_KEY="your-key-here"     # Windows: set ANTHROPIC_API_KEY=your-key-here
```

Without a key the tool runs in offline mode using keyword rules.

## Usage

```bash
python main.py --dry-run          # preview only, changes nothing (start here!)
python main.py                    # label the 25 most recent unsorted emails
python main.py --max 50 --days 3  # bigger batch, shorter window
python main.py --archive-low      # also move low-priority mail out of the inbox
```

The first run opens a browser window for you to approve access. After that it saves `token.json` and won't ask again.

## Customize it

Open `classifier.py` and edit `USER_CONTEXT` to describe what *you* consider important. That one paragraph is what makes the sorting personal.

## Privacy

- Email content is sent to the Anthropic API for classification when AI mode is on. Skip the API key to keep everything local.
- `credentials.json` and `token.json` give access to your inbox. They're in `.gitignore`, so **never commit them**.

## Ideas for next steps

- Add a medium priority tier or per-category labels (jobs, school, bills)
- Run automatically on a schedule (cron or GitHub Actions)
- Learn from corrections: log emails you move between labels and use them as examples in the prompt
- Add a small dashboard showing how many emails were sorted each week
