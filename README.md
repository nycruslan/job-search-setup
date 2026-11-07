# Job search setup for Claude Code

One skill that turns Claude Code into your job-search assistant, built from your CV.
Tell it "set up my job search", hand it your CV, and in about 20–30 minutes (mostly
Claude working) you get:

- **[career-ops](https://github.com/career-ops-hq/career-ops)**, the open-source job-search
  agent: scores each job against your CV, writes tailored CVs and cover letters, helps fill
  applications (never submits), and tracks everything.
- **Your profile**: CV, target roles, pitch per role type, comp and location rules.
- **A curated company list** with verified job boards, plus Built In and Hacker News boards.
- **Job and pay tools**: Indeed, Dice (US) and Levels.fyi.
- **A daily scan**, your choice of:
  - **On your computer**: free, at 8:00, with a desktop notification of new matches.
  - **Phone mode**: runs in Claude's cloud with your computer off. Weekday mornings it
    pushes the best matches to the Claude app; reply `apply 1` and about 10 minutes later
    you get a tailored CV and copy-ready answers. You open the link and press Submit.

## Install (pick one)

**Plugin** (Claude Code in the terminal or the desktop app):

```
/plugin marketplace add nycruslan/job-search-setup
/plugin install job-search@job-search-setup
```

Then say **"set up my job search"**, or run `/job-search:job-search-setup`.
Restart Claude Code if the skill doesn't show up right away.

**No install:** copy all of [`skills/job-search-setup/SKILL.md`](skills/job-search-setup/SKILL.md)
and paste it into Claude Code as your message.

**Drop-in skill:** save that file as `~/.claude/skills/job-search-setup/SKILL.md`, then run
`/job-search-setup`.

## What you need

- Claude Code on a paid Claude plan.
- Your CV (PDF, Word or text; LinkedIn's "Save to PDF" works).
- Node.js 22.13 or newer. Claude offers to install it if it's missing.
- macOS, Linux or Windows.
- For phone mode: a GitHub account and the Claude app on your phone. Each morning run
  uses some of your Claude plan.

## Privacy and safety

- Your CV and job data stay in `~/career-ops-data` on your computer. Phone mode also
  keeps them in a private GitHub repo under your account, so the cloud run can use them.
- It never submits applications, sends messages, or signs in for you. You press Submit:
  job sites' bot checks silently drop automated applications.
- It only reads public job boards. It skips sites that forbid automated access
  (Google, Meta, LinkedIn); use the Indeed connector for those companies.

## After setup

```sh
cd ~/career-ops-data && claude
```

Then `/career-ops:career-ops triage` for a quick pass over new matches, and
`/career-ops:career-ops pipeline` to evaluate the best ones. `~/career-ops-data/README.md`
has the full guide, including how to pause and resume the daily scan.

## License

MIT. career-ops, Levels.fyi, Dice and Indeed are separate projects with their own terms.
