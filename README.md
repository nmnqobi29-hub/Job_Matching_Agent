# AI Job-Matching Agent

A semi-autonomous job search assistant built with **n8n**, **Claude**, **PostgreSQL** and **Streamlit**. It reads my job alert emails, scores each listing against my resume, drafts a cover note for the strong matches, and saves them to a review dashboard where I approve or reject them before anything is sent.

> **Status:** in active development and use.

## The problem

Job alerts pile up in the inbox from several platforms. Reading each one, deciding if it fits, and writing a tailored note for every application takes hours every week, and good matches get buried.

## The solution

1. Job alert emails are labelled automatically in Gmail.
2. The workflow picks up the labelled emails and extracts each listing.
3. **Claude scores** every listing against my resume and background.
4. For strong matches, Claude **drafts a cover note**.
5. Strong matches are saved to PostgreSQL.
6. I review them in a **Streamlit dashboard** and approve or reject each one.

## Human in the loop, by design

This is deliberately **not** a fully automatic application bot. Nothing is submitted without my approval. Automated applying on job platforms risks breaking their terms of service, and generic auto-applications are low quality anyway. The agent does the sorting and drafting, and I make the decisions.

## How it works

```
Gmail job alerts --> Auto-Label workflow (every 10 min)
                              |
                              v
              Job Application Agent workflow (n8n)
                 extract listings --> Claude scores vs resume
                              |
                    strong match? --> Claude drafts cover note
                              |
                              v
                  PostgreSQL (job_matches table)
                              |
                              v
             Streamlit review dashboard (approve / reject)
```

### Components

| Part | What it does |
|------|--------------|
| **Auto-Label Job Alerts** | A safety-net workflow that runs every 10 minutes and labels incoming job alert emails so none are missed. |
| **Job Application Agent** | The main n8n pipeline: extraction, scoring, drafting and saving. |
| **Claude (Anthropic API)** | Extracts listing details, scores the fit against my background, and drafts cover notes. |
| **PostgreSQL** | A `job_matches` table stores scored matches and their status. |
| **Streamlit dashboard** | Lets me read each match and its draft note, then approve or reject. Deployed on Streamlit Community Cloud with the database connection kept in secrets. |
| **Run-status logging** | After each run, the workflow logs how many jobs it processed, how many strong matches it saved and the last run time to a shared status table used by my other agent project. |

## Tech stack

- n8n (workflow automation and scheduling)
- Claude via the Anthropic API (extraction, scoring, drafting)
- PostgreSQL (storage)
- Streamlit (review dashboard)
- Gmail (source of job alerts)
- Railway (hosting for n8n and the database)

## Results

- Jobs processed so far: **[100+]**
- Strong matches saved: **[58+]**
- Applications sent from approved matches: **[58+]**

## Lessons learned

- **Check that it's actually running.** I found the main workflow was still inactive in n8n even though I believed it was live. Now I verify the active status and the logged run time, not just my assumption.
- **Monitor API credits.** A depleted API balance blocked the scoring step. Failures like this should be visible, which is why the workflow reports its run status.
- **Verify the prompt inputs.** I confirmed the scoring step uses my real background text and not a placeholder, since a scorer is only as good as what it compares against.
- **Keep a human in the loop.** Review before sending gives better applications and avoids platform policy problems.

## Privacy and security

- No credentials, API keys or database URLs are stored in this repository. They live in n8n credentials and in Streamlit secrets.
- The email account and resume content stay private. Only the code and workflow design are shared here.

## Possible improvements

- Learn from my approvals and rejections to improve scoring.
- Add a daily summary by email or Telegram.
- Track application outcomes to see which listing types lead to replies.

## Author

**Mnqobi**, automation engineer based in Cape Town, South Africa.
GitHub: https://github.com/nmnqobi29-hub
