# repo-maker

A Claude Code skill that sets up GitHub repos for you. You say what the project is in plain words, it shows you a plan, and after you say "go" it runs the GitHub commands itself.

I built this because I'm new to terminals, and wanted to share this for anyone else who may be new as well. Every time I wanted a new repo I was looking up the same six commands and getting one flag wrong. Now I type `/repo-maker a place to keep my client intake scripts` and it's done in about a minute.

## What it does

- Creates the repo on GitHub, private unless you say public
- Writes a small starter structure that fits the project (Python, Node, a static website, a Claude Code project, or a basic docs repo)
- Writes the README, .gitignore, and license (MIT when public) and makes the first commit and push
- Can add a starter Claude Code skill or subagent inside the new repo
- Can open a GitHub Project board and write 3 to 5 starter issues based on what you said the project is for

It never creates anything until you've seen the plan and said yes. It won't force-push, delete a repo, or flip a repo to public on its own, and before pushing an existing folder it checks for files that look like passwords or API keys.

## Who it's for

People who use Claude Code but don't live in the terminal. If you already type `gh repo create` from memory, you don't need this.

## Requirements

- [Claude Code](https://claude.com/claude-code)
- [git](https://git-scm.com/downloads)
- The [GitHub CLI](https://cli.github.com), installed and logged in (`gh auth login`)
- For project boards only: one extra permission, granted with `gh auth refresh -s project`. The skill tells you when you need it.

## Quickstart (about 60 seconds)

1. Download this repo, or clone it:
   `git clone https://github.com/humblecoachai/repo-maker.git`
2. Copy the `repo-maker` folder into your project's skills folder so the file sits at `.claude/skills/repo-maker/SKILL.md`. To use it in every project, put it at `~/.claude/skills/repo-maker/SKILL.md` instead.
3. Open Claude Code and type:
   `/repo-maker a Python tool that renames my photos by date`
4. Read the plan it shows you. Say "go," or tell it what to change.

It works from plain requests too. Saying "put this folder on GitHub" or "make me a repo for my newsletter drafts" is enough.

## What a run looks like

```
Here's what I'll set up:

  Repo:        yourname/photo-renamer (private)
  About:       Renames photos by the date they were taken
  On your PC:  ./photo-renamer
  Files:       README, .gitignore, src/, tests/, requirements.txt
  License:     none (private repo)
  Extras:      none

Say go, or tell me what to change.
```

## Limits

- It only uses the GitHub CLI. No other services or accounts are involved.
- One repo per run.
- Starter structures are deliberately small. It sets up a clean first commit and leaves the building to you.

## License

MIT. See [LICENSE](LICENSE).
