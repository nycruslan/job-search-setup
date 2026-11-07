---
name: job-search-setup
description: Sets up a complete AI-assisted job search in Claude Code from the user's CV. Installs the career-ops plugin, builds their profile, finds and verifies job boards for their target companies, connects job-search and pay-data tools, runs a first scan and schedules a free daily one. Use when someone asks to set up their job search, install or configure career-ops, or "set this up for me with my CV".
---

# Job search setup

Set up [career-ops](https://github.com/career-ops-hq/career-ops) for the user, personalized from their CV, in one session. Do the work yourself and ask only for what the CV can't tell you. Say in one line what each step does as you go.

**Rules**
- The CV and everything about the job search are private. Keep them on this machine; never put them in URLs, commits, issues or messages.
- Never submit an application, send a message, or sign in to anything for the user.
- Never invent CV facts or metrics.
- If a permission rule blocks a command, give the user the exact command to run as `! <command>` and carry on with the rest.
- If `~/career-ops-data/cv.md` already exists, career-ops is already set up. Show what's there and ask before changing or overwriting anything.

## 1. Ask once

In a single message, ask for:
1. Their CV: a file path (PDF, DOCX or Markdown; LinkedIn's "Save to PDF" works) or pasted text.
2. Target roles, or "infer them from my CV".
3. Where: a city for office or hybrid work, and/or countries for remote work.
4. Target total compensation and currency (optional).
5. Work authorization: where they can work without sponsorship.
6. Companies they especially want, and any to skip (optional).

Read the CV, then work out seniority, years of experience, role archetypes, strengths and proof points yourself.

## 2. Prerequisites

- `node --version` must be 22.13 or newer. If it isn't, ask, then install: macOS `brew install node`, Windows `winget install OpenJS.NodeJS.LTS`, Linux via nvm or the distro.
- `claude --version` must work; the plugin commands below use it.

## 3. Install career-ops and the tools

Everything is enabled only in the data folder `~/career-ops-data`, so the user's other projects stay untouched.

```sh
claude plugin marketplace add anthropics/claude-plugins-official   # skip if it already exists
claude plugin marketplace add career-ops-hq/career-ops
claude plugin marketplace add Levels-fyi/levels-fyi-plugin
mkdir -p ~/career-ops-data && cd ~/career-ops-data
claude plugin install career-ops@career-ops --scope project
claude plugin install levels-fyi@levels-fyi --scope project
claude plugin install playwright@claude-plugins-official --scope project
```

For US tech roles, also add Dice: `claude mcp add --scope project --transport http dice https://mcp.dice.com/mcp`.

career-ops normally finishes installing on the next Claude Code start. Finish it now instead:

```sh
P=$(ls -d ~/.claude/plugins/cache/career-ops/career-ops/*/ | sort -V | tail -1)
D=~/.claude/plugins/data/career-ops-career-ops
mkdir -p "$D" && CLAUDE_PLUGIN_ROOT="$P" CLAUDE_PLUGIN_DATA="$D" node "$P/scripts/claude-plugin-session-start.mjs"
cd "$P" && npx --no-install playwright install chromium
node doctor.mjs --json --init-templates
```

Run every career-ops script below from `$P`. They find `~/career-ops-data` on their own.

## 4. Profile

1. **CV:** write `~/career-ops-data/cv.md` in the format of `$P/examples/cv-example.md`. Keep their wording and numbers.
2. **Settings:** build the answers JSON (the shape is in `node onboard-write.mjs --help`) and pipe it to `node onboard-write.mjs --answers - --only profile`. Include 3–4 archetypes (primary, secondary, adjacent), a headline, superpowers and proof points from the CV, and work authorization (`authorized_in`, `needs_sponsorship`).
3. Append to `~/career-ops-data/config/profile.yml`:
   ```yaml
   scan:
     extractor: cli   # compact page reader, fewer tokens
   ```
4. **Pitch files:** rewrite every `## Your …` section of `modes/_profile.md` (target roles, adaptive framing, exit narrative, cross-cutting advantage, portfolio, comp targets, negotiation scripts, location policy). Fill every placeholder in `modes/_brief.md` (identity, archetypes, proof points, comp, location scoring, hard disqualifiers, priority companies). Use only CV facts. In `_profile.md`, say that pay research uses the Levels.fyi tools first.
5. `node remember-cli.mjs claude`

## 5. Companies and filters

Keep `~/career-ops-data/portals.yml` short and readable: their filters plus 40–70 companies, not career-ops's full template. Write this skeleton, filled in for them:

```yaml
# What the daily scan looks at. After editing, check it with:
#   node validate-portals.mjs   (from the career-ops plugin folder)

location_filter:
  allow: [<their city and its boroughs/suburbs>, <remote countries>, "Remote"]
  # Office/hybrid roles only in their city; remote roles only where they can work.
  workplace_policy:
    onsite_locations: [<city names>]
    remote_countries: [<countries>]
    remote_regions: [<e.g. "North America", "EMEA", "Anywhere">]

max_posting_age_days: 45
experience_filter: {enabled: true, stretch_years: 1, action: label}

title_filter:
  positive: [<role keywords from their targets>]
  negative: [<junior levels>, <functions and stacks they don't do>]   # never exclude an industry they target
  seniority_boost: [<their level words>]

tracked_companies: []
job_boards: []
```

Then add companies:

1. Pick companies that fit their CV and goals: the best payers in their field plus their wishlist. Write them to `~/career-ops-data/data/companies.yml` as `companies: [{name: ...}, ...]`.
2. Preview: `node discover-ats.mjs --in ~/career-ops-data/data/companies.yml --summary`. For each board marked small or unresolved, confirm it belongs to the right company. Try slug variants (for example `runway-ml` instead of `runway`) and set `slug:` in the YAML. Re-run until the list is right.
3. Write: the same command with `--write` instead of `--summary` appends the boards to `portals.yml`.
4. Big companies need a special provider. Add the relevant ones by hand:
   ```yaml
     - name: Amazon
       careers_url: https://www.amazon.jobs
       provider: amazon
       amazon: {loc_query: "<City, ST or country>", category: [software-development]}
       enabled: true
     - name: Apple
       careers_url: https://jobs.apple.com/en-us/search
       provider: apple
       apple: {queries: ["<role keywords>"]}   # Apple has no default search
       max_pages: 2
       enabled: true
     - name: Microsoft
       careers_url: https://apply.careers.microsoft.com/careers?domain=microsoft.com
       provider: eightfold
       eightfold_host: apply.careers.microsoft.com
       enabled: true
     - name: Netflix
       careers_url: https://explore.jobs.netflix.net/careers?domain=netflix.com
       provider: eightfold
       eightfold_host: explore.jobs.netflix.net
       enabled: true
     - name: NVIDIA
       careers_url: https://nvidia.wd5.myworkdayjobs.com/NVIDIAExternalCareerSite
       enabled: true
     - name: OpenAI
       careers_url: https://jobs.ashbyhq.com/openai
       ashby: {embed: true}   # the full board is over the scanner's 10 MB limit
       enabled: true
   ```
5. A company with no readable board gets `scan_method: websearch` and a `scan_query`. Only the AI scan (`/career-ops scan`) covers those.
6. Don't add Google or Meta: their robots.txt forbids automated job search. Use the Indeed connector for their postings. Never scrape LinkedIn.
7. Boards that cover many employers go under `job_boards`: `provider: builtin` for Built In city sites (`builtin: {host: www.builtinnyc.com, queries: [...], max_pages: 2}`; other cities include builtinsf, builtinla, builtinboston, builtinaustin, builtinseattle, builtincolorado) and `provider: hackernews` for tech roles. `$P/templates/portals.example.yml` documents every provider.
8. `node validate-portals.mjs` must report 0 errors.

## 6. Optional imports

- **Existing tracker** (Teal, Huntr, Simplify or a spreadsheet saved as CSV): `node import-tracker.mjs <file.csv>` previews; add `--confirm` to import.
- **LinkedIn connections**, for warm intros: LinkedIn → Settings → Data privacy → Get a copy of your data → Connections. Save the file as `~/career-ops-data/data/Connections.csv`, then run `node linkedin-join.mjs --summary`.

## 7. First scan and daily schedule

1. Run `node scan.mjs --quiet` in the background; a first scan takes 5–15 minutes. Fix every line under "Errors" in its summary, re-testing with `node scan.mjs --company "<name>" --dry-run`.
2. Write `~/career-ops-data/bin/daily-scan.sh` and make it executable. Replace `<NODE_DIR>` with the folder of `command -v node`:
   ```sh
   #!/bin/sh
   # Daily job scan. Free: it only reads public job boards. Log: ~/career-ops-data/data/scan.log
   set -eu
   export PATH="<NODE_DIR>:/usr/local/bin:/usr/bin:/bin"
   # Newest installed career-ops, so plugin updates need no edit here.
   PLUGIN=$(ls -d "$HOME"/.claude/plugins/cache/career-ops/career-ops/*/ | sort -V | tail -1)
   echo "== $(date '+%Y-%m-%d %H:%M') career-ops $(basename "$PLUGIN")"
   cd "$PLUGIN"
   status=0
   node scan.mjs --quiet || status=$?
   pending=$(grep -c '^- \[ \]' "$HOME/career-ops-data/data/pipeline.md" 2>/dev/null || true)
   msg="${pending:-0} roles waiting. Run /career-ops:career-ops pipeline in ~/career-ops-data."
   command -v osascript >/dev/null && osascript -e "display notification \"$msg\" with title \"career-ops scan\"" || true
   command -v notify-send >/dev/null && notify-send "career-ops scan" "$msg" || true
   exit "$status"
   ```
3. Schedule it every day at 8:00:
   - **macOS:** write `~/career-ops-data/bin/io.career-ops.scan.plist` (label `io.career-ops.scan`, ProgramArguments = the script's absolute path, StartCalendarInterval Hour 8 Minute 0, StandardOutPath and StandardErrorPath = `~/career-ops-data/data/scan.log` as absolute paths). Check it with `plutil -lint`, then install it:
     `cp ~/career-ops-data/bin/io.career-ops.scan.plist ~/Library/LaunchAgents/ && launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/io.career-ops.scan.plist`
   - **Linux:** add the crontab line `0 8 * * * $HOME/career-ops-data/bin/daily-scan.sh >> $HOME/career-ops-data/data/scan.log 2>&1`
   - **Windows:** register a daily 8:00 Task Scheduler task that runs `node scan.mjs --quiet` from the newest folder under `%USERPROFILE%\.claude\plugins\cache\career-ops\career-ops\`, appending output to `%USERPROFILE%\career-ops-data\data\scan.log`.

   Installing a scheduled job often needs the user's approval. If it's blocked, give them the install command.

## 8. Connectors (the user clicks)

- **Indeed**, and optionally **Gmail** for recruiter replies: claude.ai → Settings → Connectors → Connect. Claude Code picks them up automatically.
- Levels.fyi, Playwright and Dice are already installed in the data folder.

## 9. Finish

1. From `~/career-ops-data`, run `node "$P/doctor.mjs"`. Fix anything it flags until all checks pass.
2. Write `~/career-ops-data/README.md`: a one-screen guide covering how to start, a table of common requests, what each file is for, the schedule commands, and how to update the plugin (`/plugin` → Marketplaces → career-ops → auto-update).
3. Tell the user in a few lines what was set up, how many roles the first scan queued, and how to start:
   `cd ~/career-ops-data && claude`, then `/career-ops:career-ops triage` for a quick pass/fail on the backlog and `/career-ops:career-ops pipeline` to evaluate the best ones. On the first launch Claude Code asks them to trust the folder and approve the Dice server.
