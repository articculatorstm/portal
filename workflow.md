This guide walks through setting up an automated workflow where non-technical club officers can update the website simply by opening an **Issue on GitHub** (e.g., *"Update meeting time to 7:30 PM"*). An AI agent automatically edits the site's code, commits the change, and deploys it live.

---

## Architecture Overview

```
[Club Officer Opens GitHub Issue] 
               │
               ▼
   [GitHub Action Triggered] 
               │
               ▼
[Claude Agent Reads Issue & Edits index.html] 
               │
               ▼
[Pushes Commit / Pull Request to main Branch] 
               │
               ▼
  [GitHub Pages Deploys Website Live]

```

---

1. **Create the GitHub Repository and Site Code:** Initial foundation.
1. Log into [github.com](https://github.com) and click **New Repository**.
2. Name it `toastmasters-club` (or similar), select **Public**, check **Add a README file**, and click **Create repository**.
3. Click **Add file** > **Create new file**, name it `index.html`, paste your initial HTML website code, and click **Commit changes**.
4. Create a second file named `CLAUDE.md`. This file serves as the AI agent's "instruction manual." Add guidelines like:

```markdown
# Site Maintenance Instructions
- The primary website file is `index.html`.
- Use official Toastmasters colors: Loyal Blue (`#004165`) and True Maroon (`#772432`).
- Keep layout clean, mobile-friendly, and preserve all Google Form embed codes.

```


2. **Enable GitHub Pages Hosting:** Making the site public.
1. In your repository, go to **Settings** > **Pages** (under Code and automation).
2. Under **Build and deployment**:
* **Source:** Deploy from a branch
* **Branch:** `main` | Folder: `/ (root)`


3. Click **Save**. Within 1–2 minutes, your live site URL will appear at the top of the page.


3. **Set Up Your Anthropic API Key:** Authenticating the AI Agent.
1. Get an API key from the [Anthropic Console](https://console.anthropic.com/).
2. In your GitHub repository, go to **Settings** > **Secrets and variables** > **Actions**.
3. Click **New repository secret**.
4. **Name:** `ANTHROPIC_API_KEY`
5. **Secret:** Paste your API key string and click **Add secret**.


4. **Configure the GitHub Action Workflow:** Automating issue processing.
1. In your repository, click **Add file** > **Create new file**.
2. Name the path `.github/workflows/ai-updater.yml`.
3. Paste the following workflow file (using the official `claude-code-action`):

```yaml
name: AI Website Updater

on:
  issues:
    types: [opened, edited]

jobs:
  update-website:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      issues: write
      pull-requests: write

    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Run Claude Code Agent
        uses: anthropics/claude-code-action@v1
        with:
          prompt: |
            You are the webmaster for this Toastmasters club website.
            An issue was opened with the following request:
            
            Title: ${{ github.event.issue.title }}
            Description: ${{ github.event.issue.body }}
            
            1. Update `index.html` according to the requested changes.
            2. Ensure you follow all branding guidelines in `CLAUDE.md`.
            3. Commit the changes directly to the main branch with a clear commit message.
            4. Reply to the issue confirming that the update is live.
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}

```

4. Click **Commit changes**.


---

## How Club Officers Maintain the Site Day-to-Day

Once set up, updating the website requires **no coding or Git knowledge**:

1. Any club officer goes to the repository's **Issues** tab and clicks **New Issue**.
2. They type a request in plain English:
* **Title:** *Update next meeting theme and date*
* **Description:** *Change the next meeting date to Oct 14th at 7 PM. The theme is 'Humorous Speaking', and Jane Doe is the Toastmaster of the Day.*


3. They click **Submit new issue**.
4. The GitHub Action runs automatically in the background. Within **60 to 90 seconds**, the AI edits `index.html`, commits the change, comments on the issue that it's complete, and GitHub Pages updates the live site.

---
