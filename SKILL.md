---
name: repo-maker
description: Use when someone wants a new GitHub repo created or set up from inside Claude Code. Triggers on "make a repo", "create a GitHub repo", "new repo for X", "set up a repo", "put this on GitHub", "start a project on GitHub", "scaffold a repo", or any request to spin up a repository, its starter files, a project board, or starter issues, even if they never say "repo." Runs the GitHub CLI (gh) so the person only describes what they want in plain words. Always confirms the plan before anything is created.
argument-hint: "[what the repo is for, even one messy sentence works]"
---

# Repo Maker

You create and set up GitHub repos for someone who may never have used a terminal. They describe what they want in plain words. You work out the details, show them one plan, and run the `gh` commands only after they say yes.

Nothing gets created on GitHub silently. Ever.

---

## Step 0: Preflight (run it yourself, say nothing unless it fails)

Run these checks before asking a single question:

1. `gh --version`: is the GitHub CLI installed?
2. `gh auth status`: is it logged in, and as which account? Note the token scopes it prints.
3. `git --version`: is git installed?
4. `git config user.name` and `git config user.email`: will commits have an author?

If everything passes, move on without narrating it. If something fails, stop and give the fix as click-by-click steps, never a bare command:

- **gh missing:** point them to https://cli.github.com, tell them to run the installer with the default options, then close and reopen their terminal (or VS Code).
- **Not logged in:** have them type `! gh auth login` in the Claude Code prompt, then walk them through the answers: GitHub.com, HTTPS, "Login with a web browser," copy the 8-character code, paste it in the browser page that opens, click Authorize.
- **git author missing:** ask for the name and email they want on commits, then set them with `git config --global user.name "..."` and `git config --global user.email "..."` yourself.

Save the logged-in username. Every repo goes under that account unless they name an organization.

---

## Step 1: Understand the ask (draft first, never a form)

Read what they said and fill in everything you reasonably can. Only ask about what you genuinely can't infer. Most requests need one or two questions, not ten.

You need these answers, whether stated or inferred:

| Decision | Default if they don't say |
|---|---|
| Repo name | Lowercase, hyphens, taken from what they described (e.g. "a tool that renames photos" becomes `photo-renamer`) |
| Public or private | **Private.** Public only when they say public, open source, or share with everyone |
| One-line description | Written by you from their words, plain and specific |
| License | MIT for public repos. None for private unless they ask |
| Starter structure | Picked from the table in Step 2 based on what the project is |
| Where it lives on their computer | A new folder named after the repo, inside the current working directory. If the current folder is already a git repo, ask where instead, since a repo inside a repo causes trouble. On Windows, keep the full path short (under about 150 characters) or git fails with "Filename too long" |
| Scaffold a skill or agent inside it? | No, unless they ask |
| Project board and starter issues? | No, unless they ask |

Before proposing, check the name is free: `gh repo view OWNER/NAME`. If it exists, suggest a different name in the plan.

If they're turning an existing folder into a repo ("put this on GitHub"), that's a different path: skip the scaffold and use `gh repo create NAME --source . --push` after confirming what's in the folder. Check for secrets first (see Guardrails).

---

## Step 2: Pick the starter structure

Keep it small. A starter structure is a clean first commit, not a framework.

| Project type | Files created |
|---|---|
| **Basic** (notes, docs, anything unclear) | `README.md`, `.gitignore`, `docs/` |
| **Python** | `README.md`, `.gitignore` (Python), `src/<package_name>/__init__.py`, `tests/`, `requirements.txt` |
| **Node / JavaScript** | `README.md`, `.gitignore` (Node), `package.json`, `src/index.js`, `tests/` |
| **Website (static)** | `README.md`, `.gitignore`, `index.html`, `styles.css`, `assets/` |
| **Claude Code project** (skills, agents, an AI workspace) | `README.md`, `.gitignore`, `CLAUDE.md`, `.claude/skills/`, `.claude/agents/` |

Add `LICENSE` whenever a license was chosen. Empty folders need a `.gitkeep` file or git won't keep them.

The README you write for the first commit has four short parts: what it is (one sentence), why it exists (one or two sentences, from what they told you), how to use it (a placeholder line they'll fill later is fine here), and the license. No badges, no hype.

---

## Step 3: Show the plan and wait

Show one short plan in plain words. Example:

```
Here's what I'll set up:

  Repo:        yourname/photo-renamer (private)
  About:       Renames photos by the date they were taken
  On your PC:  ./photo-renamer
  Files:       README, .gitignore, src/, tests/, requirements.txt
  License:     none (private repo)
  Extras:      a project board called "photo-renamer" with 3 starter issues

Say go, or tell me what to change.
```

Proceed only on a clear yes ("go," "yes," "do it"). "Looks about right" or silence is not a yes; ask once more. If they change anything, show the updated plan again before running.

---

## Step 4: Build it

Run in this order. Stop at the first failure and explain it plainly.

1. **Create an empty repo on GitHub:**
   `gh repo create OWNER/NAME --private --description "..."` (or `--public`)
   Don't pass `--add-readme`, `--gitignore`, or `--license`. Those make GitHub write a commit of its own, and then your first push collides with it. You write all three locally instead.
2. **Make the local folder and files.** Write the starter structure with your file-writing tool, not shell tricks, so it works the same on Windows, Mac, and Linux. If a license was chosen, write the LICENSE file too (MIT text, current year, the account's display name from `gh api user --jq .name`, falling back to the username).
3. **First commit and push:**
   `git init -b main`
   `git add .`
   `git commit -m "Initial commit: starter structure"`
   `git remote add origin https://github.com/OWNER/NAME.git`
   `git push -u origin main`
4. **Optional: scaffold a skill or agent** (Step 5).
5. **Optional: project board and starter issues** (Step 6).
6. **Verify.** Run `gh repo view OWNER/NAME --json name,visibility,url,defaultBranchRef` and `git status`. Confirm the visibility matches the plan and the working tree is clean.

---

## Step 5: Scaffold a skill or agent inside the repo (optional)

If the project already has a skill-building or agent-building skill (look for `skill-builder` or `agent-builder` in the available skills), hand off to it for the design interview and have it write into the new repo. Don't run a second, competing interview here.

If none exists, write a minimal working scaffold and tell them it's a starting point:

- **Skill:** `.claude/skills/<skill-name>/SKILL.md` with frontmatter (`name`, a `description` that says when to use it, `argument-hint` if it takes input) and a body of numbered steps drawn from what they described.
- **Subagent:** `.claude/agents/<agent-name>.md` with frontmatter (`name`, `description`, `tools` limited to what the job needs, `model`) and a short system prompt stating the one job, its inputs, and what it returns.

Commit and push it as its own commit: `Add <name> skill scaffold` or `Add <name> agent scaffold`.

---

## Step 6: Project board and starter issues (optional)

Boards need the `project` permission on the GitHub login. Check the scopes from Step 0. If `project` isn't there, stop and have them grant it:

> Type `! gh auth refresh -s project` in the Claude Code prompt and press Enter. It shows an 8-character code and opens your browser. Paste the code, click Authorize, and come back. It takes about 30 seconds and you only do it once.

Then:

1. **Create the board:** `gh project create --owner OWNER --title "NAME"`. Read the project number from the URL it prints.
2. **Link it to the repo:** `gh project link NUMBER --owner OWNER --repo OWNER/NAME`.
3. **Create starter issues.** Three to five, drawn from what they told you the project is for. Each one is a real next step with a one-line body saying what "done" looks like. Never generic filler like "Add tests" or "Improve docs" unless that's genuinely the next step.
   `gh issue create --repo OWNER/NAME --title "..." --body "..."`
4. **Add each issue to the board:** `gh project item-add NUMBER --owner OWNER --url ISSUE_URL`.

Show the proposed issue titles as part of the Step 3 plan so they approve them with everything else.

---

## Step 7: Report back

Keep it to a few lines: the repo link, the board link if one was made, where the folder is on their computer, and the one thing they'd most likely do next ("open the folder in VS Code and start on the first issue"). No command recap unless they ask.

---

## Guardrails

- **Never create, rename, delete, or change the visibility of a repo without an explicit yes on the exact plan.** This includes test repos.
- **Private by default.** Making something public is a deliberate choice they state, never your inference.
- **Secrets check before any push from an existing folder.** Look for `.env` files, files with `key`, `secret`, `token`, or `credential` in the name, and anything that looks like an API key inside text files. If you find one, stop, name the file, and add it to `.gitignore` before the first commit. Once a secret is pushed to a public repo, assume it's compromised.
- **No force-push, no history rewrites, no deleting branches.** If something is out of sync, explain it and ask.
- **Never delete a repo on your own.** If they ask to delete one, confirm the exact `OWNER/NAME` back to them first. Deleting needs the `delete_repo` permission (`! gh auth refresh -s delete_repo`), and it can't be undone.
- **GitHub CLI only.** Don't install extra tools or services to do this job.
- **One repo per run** unless they asked for several by name.

## Common failures

| What they see | What it means | What you do |
|---|---|---|
| `Name already exists on this account` | The repo name is taken | Suggest a variant and re-confirm |
| `failed to push some refs` | GitHub already has a commit (usually a license or README made there) | Explain it, then `git pull --rebase origin main` and push again. Never force-push |
| `your authentication token is missing required scopes [project]` | Board permission not granted yet | Give the `gh auth refresh -s project` steps from Step 6 |
| `Author identity unknown` | git has no name or email set | Run the Step 0 author fix |
| `Filename too long` (Windows) | The folder path plus git's internal file names passed Windows' 260-character limit | Delete the half-made `.git` folder, move the project to a short path (e.g. `C:\Users\NAME\repos\NAME`), and commit again. Deep folders like Temp or OneDrive are the usual cause |
| `gh: command not found` | The CLI isn't installed, or the terminal was open before install | Install it, then fully close and reopen the terminal |
