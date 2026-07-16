# Claude Code Ignition

A complete one-paste setup for Claude Code with persistent memory, session continuity, and beginner-friendly defaults. Works on **Mac** and **Windows**.

Ignition is the first message you send Claude Code: one paste sets up memory, rules, and hooks. After setup, level up with [compound](https://github.com/tyfarrago-hub/compound) (7 workflow skills for daily use), [taste](https://github.com/tyfarrago-hub/taste) (frontend design skills), and [soul](https://github.com/tyfarrago-hub/soul) (agent personality template).

---

## How to use this guide

1. Download the **Claude Code desktop app** and sign in with your Anthropic account.
2. Open it. You'll see a chat box.
3. **Fill in the 3 blanks** at the top of the block below with your info (name, role, project).
4. Copy **everything** below the line that says `PASTE EVERYTHING BELOW THIS LINE`.
5. Paste it as your very first message to Claude Code.
6. From there, Claude installs the tools, builds your memory system, and walks you through the rest.

You don't need to know terminal commands. Claude runs them. You'll need to:
- Click "Install" once when a system popup appears (Mac only, for Xcode tools).
- Type your password if Homebrew or an installer asks (Mac only).
- Click "Run" or "Accept" on any installer dialog that opens.

If a step fails, Claude consults the Troubleshooting section at the bottom and fixes it before moving on. If something genuinely can't be fixed, Claude tells you exactly what's blocked.

---

## PASTE EVERYTHING BELOW THIS LINE INTO YOUR FIRST CLAUDE CODE MESSAGE

---

My name is **[your name]**.
I'm a **[your role — developer, creator, entrepreneur, student, parent, etc.]**.
I'm currently working on **[your main project, business, or goal]**.

You are setting up my Claude Code environment from scratch. Use my info above to seed my memory files. Detect my OS (Mac or Windows) and run the OS-appropriate commands.

Run every phase below sequentially. Verify each phase worked before moving to the next. If something fails, consult the Troubleshooting section at the bottom and fix it before moving on. Don't skip steps. Don't assume something worked without checking.

You don't need to confirm with me between phases. Only stop and ask me when:
- A system popup needs my click (Xcode CLI tools, installer dialogs).
- A command needs my password (Homebrew install, sudo).
- A failure can't be auto-resolved from Troubleshooting.

When you write commands, run them yourself. Don't ask me to type them.

---

### Phase 1: Detect my OS

Run the right command and tell me which OS you detected:

- **Mac:** `uname -s` returns `Darwin`.
- **Windows:** `$env:OS` (in PowerShell) returns `Windows_NT`.

Branch every command from here on based on what you found.

---

### Phase 2: Install prerequisites — Git, Homebrew (Mac only), Node.js

Each tool gets a check, an install, and a verify. Don't move past one until it returns a version number.

#### On Mac

**1. Xcode Command Line Tools** (this bundles Git on Mac)
- Check: `xcode-select -p`
- If missing: `xcode-select --install`. **Important:** this command returns instantly while the GUI installer keeps running in the background. Don't trust the exit code as "done." After triggering the install, tell me to click "Install" on the popup, then poll with `pkgutil --pkg-info=com.apple.pkg.CLTools_Executables` every 30 seconds. Only continue once that command returns package info (not "No receipt found"). This usually takes 5–15 minutes.

**2. Homebrew**
- Check: `which brew`
- If missing: the Homebrew installer requires an interactive sudo password prompt. Your shell tool can't reliably handle that. Instead:
  1. Tell me to open the **Terminal app** myself (Applications > Utilities > Terminal, or Spotlight: "Terminal").
  2. Give me this command to paste into Terminal and run:
     ```
     /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
     ```
  3. Tell me to type my Mac password when it asks, and press Return when prompted.
  4. Wait for me to confirm "Homebrew install complete" before continuing.
- After install, detect architecture: `uname -m` returns `arm64` for Apple Silicon (M1/M2/M3/M4), `x86_64` for Intel.
- **On Apple Silicon**, `brew` is at `/opt/homebrew/bin/brew` and NOT yet on this shell's PATH. Add it to ~/.zshrc AND use the full path for the rest of Phase 2:
  ```
  echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zshrc
  eval "$(/opt/homebrew/bin/brew shellenv)"
  ```
  Each new Bash subprocess starts fresh, so chain the eval inline for any subsequent brew command in this phase, e.g.:
  ```
  eval "$(/opt/homebrew/bin/brew shellenv)" && brew install node
  ```
- **On Intel Macs**, brew is at `/usr/local/bin/brew` and on PATH automatically. No eval needed.
- Verify: `brew --version` returns a version.

**3. Git**
- Check: `git --version` (should already work from Xcode CLI tools).
- If missing on Apple Silicon: `eval "$(/opt/homebrew/bin/brew shellenv)" && brew install git`. On Intel: `brew install git`.
- Verify: `git --version` returns a version.

**4. Node.js**
- Check: `node --version && npm --version`
- If either is missing on Apple Silicon: `eval "$(/opt/homebrew/bin/brew shellenv)" && brew install node`. On Intel: `brew install node`.
- Verify: both return version numbers.

#### On Windows (PowerShell)

**1. Git for Windows**
- Check: `git --version`
- If missing, try in this order:
  - `winget install --id Git.Git -e --source winget`
  - or `choco install git -y`
  - or download installer: `https://git-scm.com/download/win`
- After install, close PowerShell and reopen it. Verify: `git --version`.

**2. Node.js LTS**
- Check: `node --version`
- If missing, try in this order:
  - `winget install OpenJS.NodeJS.LTS`
  - or `choco install nodejs-lts -y`
  - or download installer: `https://nodejs.org`
- After install, close PowerShell and reopen it. Verify: `node --version && npm --version` both return numbers.

**3. Homebrew is NOT used on Windows.** Skip it. winget, choco, and direct installers handle everything.

**4. Refresh PATH inside the current session.** Windows installers update the system PATH, but already-running shells (including the one Claude Code is using) keep the OLD PATH until they're restarted. After installing Git or Node, run this in PowerShell to pick up the new PATH without restarting Claude Code:
```
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")
```
Then re-run `git --version` or `node --version` to confirm. If they're still missing, tell me to fully quit Claude Code and reopen it.

**Note:** if any `npm install -g` later returns EPERM, tell me to reopen PowerShell as Administrator.

---

### Phase 3: Install Claude Code CLI

Run:
```
npm install -g @anthropic-ai/claude-code
```

Verify with `claude --version`.

If `claude` isn't found:
- **Mac:** add the npm global bin to PATH:
  ```
  export PATH="$(npm prefix -g)/bin:$PATH"
  echo 'export PATH="$(npm prefix -g)/bin:$PATH"' >> ~/.zshrc
  ```
- **Windows:** close and reopen PowerShell. If still not found, run `npm prefix -g` — on Windows this prints the folder that directly contains `claude.cmd` (typically something like `C:\Users\<name>\AppData\Roaming\npm`). Add **that exact folder** to your User PATH environment variable. Do NOT append `\bin` — there is no `bin` subfolder on Windows npm-global installs. Reopen PowerShell after editing PATH.

---

### Phase 4: Create the directory structure

First, capture the username and produce a SAFE folder-name version of it. **Spaces in usernames break `@` imports later** — sanitize by replacing any whitespace with underscores. Use the sanitized version everywhere you write folder paths or `@` import paths from this point on.

#### Mac
```
RAW_USER=$(whoami)
SAFE_USER=$(echo "$RAW_USER" | tr ' ' '_')
mkdir -p ~/.claude/projects/-Users-${SAFE_USER}/memory
mkdir -p ~/.claude/skills
mkdir -p ~/.claude/commands
echo "RAW_USER=$RAW_USER  SAFE_USER=$SAFE_USER"
```
Write down BOTH values. `RAW_USER` is for system commands. `SAFE_USER` is for `@` import paths and folder names you author.

#### Windows (PowerShell)
```
$RAW_USER = $env:USERNAME
$SAFE_USER = $RAW_USER -replace '\s','_'
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude\projects\-Users-$SAFE_USER\memory"
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude\skills"
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude\commands"
Write-Host "RAW_USER=$RAW_USER  SAFE_USER=$SAFE_USER"
```
Write down BOTH values.

Verify the directories exist before continuing:
- Mac: `ls -la ~/.claude/projects/-Users-${SAFE_USER}/memory`
- Windows: `Get-ChildItem "$env:USERPROFILE\.claude\projects\-Users-$SAFE_USER\memory"`

---

### Phase 5: Write `~/.claude/CLAUDE.md` (the master rule book, loaded every session)

This file gets read at the start of EVERY conversation. It tells Claude how to behave and points to my memory files.

**File path:**
- Mac: `~/.claude/CLAUDE.md`
- Windows: `$env:USERPROFILE\.claude\CLAUDE.md`

**Important — write OS-native absolute paths in the `@` imports.** Don't rely on `~` expanding inside `@` imports on Windows; behavior varies. Use the actual full path of the file you just created in Phase 4. Replace `SAFE_USER_FROM_PHASE_4` below with the sanitized username you captured.

**On Mac**, write:
```
@/Users/SAFE_USER_FROM_PHASE_4/.claude/primer.md
@/Users/SAFE_USER_FROM_PHASE_4/.claude/projects/-Users-SAFE_USER_FROM_PHASE_4/memory/MEMORY.md
```

**On Windows**, the user-profile root is whatever `$env:USERPROFILE` resolves to — often `C:\Users\<name>` but NOT guaranteed (domain accounts, non-C drives, shortened profile names). Resolve it at runtime first, then build the imports. Run:
```
echo $env:USERPROFILE
```
Substitute that exact value below as `<USERPROFILE>`. Use SAFE_USER (sanitized username) only inside the `-Users-` project subfolder, not in the root.

```
@<USERPROFILE>\.claude\primer.md
@<USERPROFILE>\.claude\projects\-Users-SAFE_USER_FROM_PHASE_4\memory\MEMORY.md
```

Example for a typical account: if `$env:USERPROFILE` is `C:\Users\Tyler Smith` and SAFE_USER is `Tyler_Smith`, the imports become:
```
@C:\Users\Tyler Smith\.claude\primer.md
@C:\Users\Tyler Smith\.claude\projects\-Users-Tyler_Smith\memory\MEMORY.md
```

Then below the imports, append the rules block (the same on both OSes):

```

## PREFERENCES

1. **Plain English recap on complex answers.** After any technical or multi-step answer, append a section titled `**In plain English:**` that restates the same answer without the big words. The point is translation. Assume the reader is intelligent but new to the technical layer. Keep the substance, drop the insider vocabulary. Trigger this for: code, commands, errors, multi-step plans, anything with unfamiliar terms. Skip it for yes/no answers and casual chat.
2. **One clear next action per response.** Not a list of options. If I genuinely need to choose, ask me ONE question.
3. **Flag uncertainty with `[UNCLEAR]`.** Don't guess. If you don't know whether a path, command, or value is correct, mark it.
4. **Short, direct responses.** No "Great question!", no "Sure thing!", no preamble. Don't OPEN by recapping my question. Get straight to the answer.
5. **Diagnose before fixing.** When something breaks, explain what you think is wrong and why BEFORE you change anything.
6. **No half-features.** If a step fails, fix it before moving on. Don't skip and circle back later.
7. **Ask when genuinely blocked.** "No fluff" and "don't skip" don't mean "stay silent forever." If you've exhausted reasonable diagnostic options and need information only I have (a credential, a decision, an external state), surface the blocker in one tight sentence and ask.
8. **RESPONSE BOOKEND (fires on every substantive response).** End the response with a horizontal rule (`---`) followed by exactly two items:
   - A status, exactly one of: `✅ **DONE**` / `🟨 **PARTIAL**` / `🟥 **BLOCKED**`. Only one ever shows. Red means it genuinely did not fully land. Never sugarcoat the status.
   - `▪ **SIMPLE ANALOGY**` with the core idea explained in third-grade language, 1 to 2 lines. A plain, friendly comparison that makes it click.
   Skip the bookend only for trivial back-and-forth where the ask is self-evident.

## GUARDRAILS (hard rules — these prevent the most common failure loops)

- **Verify before claiming done.** Never say "fixed", "done", "live", or "working" unless you reproduced my exact action this turn and saw evidence: command output, a test result, a screenshot, an HTTP status. "The code is in place" and "the outcome is proven" are two different claims. Always say which one you have.
- **Two-strike rule.** After 2 failed fixes of the same symptom, stop patching. Re-diagnose from scratch, explain the root cause with evidence, and only then touch code again. A third attempt at the same theory is banned.
- **Stale file guard.** If an edit fails with "string not found," the file changed since you last read it. Re-read it, then edit. Never retry blind.
- **Scope gate.** An idea I mention in passing is input. Anything over ~30 minutes of building gets one confirming line first ("Building X, scoped to Y. Starting.") so I can redirect before you invest the hour.
- **Constraints stick.** When I correct course or restrict scope mid-task ("just X, nothing else", "keep it simple", "skip that part"), that constraint stays binding for the rest of the session. Re-check it before every deliverable.
- **Keep it simple.** Minimum code that solves the problem. No extra features I didn't ask for, no speculative flexibility, no abstractions for single-use code.
- **Surgical changes.** Touch only what the task needs. Don't reformat, refactor, or "improve" nearby code that wasn't part of the ask.
- **State assumptions.** If my request could mean two different things, say which interpretation you picked, or ask me ONE question before starting.

## AGENT RULES

- Read `primer.md` and `MEMORY.md` at the start of every session before doing anything else.
- At the end of every session, rewrite `primer.md` completely with: active project, what was just completed, exact next step, open blockers. Keep it under 100 lines.
- Save anything long (drafts, proposals, plans, code) to a file the moment it exists. Don't let it live only in chat.
- Never ask me for context that's already in `primer.md`, `MEMORY.md`, or any imported file. Read first.
- Before saying "I don't know" about past work, search the memory folder and past session transcripts under `~/.claude/projects/` first.

## MEMORY RULES

- One memory = one file in the memory folder, named `<type>_<slug>.md` where type is `user`, `feedback`, `project`, or `reference`.
- Every memory file starts with frontmatter (`name`, `description`, `type`), then the fact itself. For `feedback` memories, also write a `**Why:**` line and a `**How to apply:**` line so future sessions apply the lesson instead of just knowing it.
- After writing a memory file, add ONE line to `MEMORY.md` pointing at it: `- [Short title](filename.md) — one-line hook`. `MEMORY.md` is an index. The detail lives in the individual files.
- Before saving a new memory, check whether an existing file already covers it. Update that file instead of creating a near-duplicate. Delete memories that turn out to be wrong.
- When I say "remember that," save it immediately. Don't wait for the end of the session.
- When I correct you, that is a `feedback` memory. Save the correction AND the reason, so the mistake never repeats.
```

After writing, verify the file exists, the imports use absolute paths (no `~`), and the username segments are filled in correctly:
- Mac: `head -5 ~/.claude/CLAUDE.md`
- Windows: `Get-Content $env:USERPROFILE\.claude\CLAUDE.md -TotalCount 5`

Then verify each `@` import path resolves to a real file (the next phases will create them; this check happens AFTER Phases 6 and 7).

---

### Phase 6: Write `~/.claude/primer.md` (the session sticky note)

This file gets rewritten at the end of every session so the next session picks up exactly where we left off.

```
# Session Primer

## Last Session: (none yet — this is the first one)

### What Was Completed
- (first session — nothing yet)

### Active Projects
- (will fill in during onboarding)

### Next Steps
1. (will fill in during onboarding)

### Key File Paths
- (Claude will add paths here as we work)

### Open Blockers
- (none)
```

---

### Phase 7: Write the memory system (`MEMORY.md` index + first memory files)

Memory is a folder of small files plus one index. Each durable fact lives in its own file. `MEMORY.md` holds one line per file and gets loaded into every session, so the index stays small while the memory itself can grow forever. This split is the single biggest upgrade over a flat notes file: the always-loaded part stays cheap, and the detail is there when it's relevant.

**Folder** (created in Phase 4, use the **SAFE_USER** value, not RAW_USER):
- Mac: `~/.claude/projects/-Users-<SAFE_USER>/memory/`
- Windows: `$env:USERPROFILE\.claude\projects\-Users-<SAFE_USER>\memory\`

**Memory file format.** Every memory file follows this shape:

```
---
name: short-kebab-slug
description: one-line summary, used to decide whether this memory is relevant
metadata:
  type: user | feedback | project | reference
---

The fact itself, in a few lines.

(For feedback memories, also include:)
**Why:** the reason this matters.
**How to apply:** what to do differently next time.
```

The four types:
- **user** — who I am: role, expertise, preferences.
- **feedback** — corrections and confirmed approaches, with the why attached.
- **project** — ongoing work, goals, decisions, constraints.
- **reference** — pointers to external things: URLs, dashboards, tools, file locations.

**Now create the first two memory files** in that folder, using my name, role, and project from the top of this guide, plus the OS/architecture/shell you detected:

1. `user_profile.md` — type `user`. Body: my name, role, what I'm currently working on, and communication style (default: short, direct, autonomous; refine during onboarding).
2. `reference_system.md` — type `reference`. Body: OS + version, architecture (Apple Silicon / Intel / x64), shell, full path of the memory folder, and the settings.json path.

**Then write `MEMORY.md`** in the same folder, as an index with one line per memory:

```
# MEMORY.md

## User Profile
- [Who I am](user_profile.md) — name, role, current focus

## System
- [This machine](reference_system.md) — OS, architecture, shell, key paths

## Preferences (learned over time)
- (one line added here per feedback memory)

## Projects
- (one line added here per project memory)
```

Verify: both memory files and `MEMORY.md` exist in the folder, and every line in `MEMORY.md` points at a file that exists.

---

### Phase 8: Write `~/.claude/settings.json` (hooks + permissions)

This is the automation layer. One hook fires automatically:
- **Stop hook:** when Claude tries to end its turn, it plays a soft glass chime (so you can look away while Claude works and still know when it's done) and sends Claude a blocking reminder to update `primer.md` and memory. Claude reads the reminder, writes the updates, then stops for real. The script checks `stop_hook_active` in the hook input and lets that second stop through, so it never loops.

#### Mac version

```json
{
  "permissions": {
    "defaultMode": "default"
  },
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "afplay /System/Library/Sounds/Glass.aiff"
          },
          {
            "type": "command",
            "command": "python3 -c 'import json,sys; d=json.load(sys.stdin); print(json.dumps({\"decision\": \"block\", \"reason\": \"Before ending: if meaningful work happened this session, rewrite primer.md, save new durable facts as memory files, index them in MEMORY.md, then stop again.\"})) if not d.get(\"stop_hook_active\") else None'",
            "statusMessage": "Reminding Claude to update primer + memory..."
          }
        ]
      }
    ]
  }
}
```

#### Windows version

```json
{
  "permissions": {
    "defaultMode": "default"
  },
  "hooks": {
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "powershell -c \"[System.Media.SystemSounds]::Asterisk.Play()\""
          },
          {
            "type": "command",
            "command": "powershell -c \"$d=[Console]::In.ReadToEnd()|ConvertFrom-Json; if(-not $d.stop_hook_active){@{decision='block';reason='Before ending: if meaningful work happened this session, rewrite primer.md, save new durable facts as memory files, index them in MEMORY.md, then stop again.'}|ConvertTo-Json -Compress}\"",
            "statusMessage": "Reminding Claude to update primer + memory..."
          }
        ]
      }
    ]
  }
}
```

#### How the Stop hook works

A plain `echo` in a Stop hook only prints to the transcript. Claude never sees it. To actually reach Claude, the hook prints JSON with `"decision": "block"` and a `reason`. Claude Code hands that reason to Claude, Claude updates the memory files, then tries to stop again. On that second stop the hook's input has `"stop_hook_active": true`, so the script prints nothing and exits 0, and the session ends cleanly. That guard is what keeps it from looping forever.

#### About permission mode

- `"defaultMode": "default"` means Claude **asks before running shell commands**. This is the safe starting point.
- Once you've used Claude Code for a few sessions and trust how it works, you can change `"defaultMode": "default"` to `"defaultMode": "bypassPermissions"`. That lets Claude run commands without asking, which is faster but riskier — one bad command runs unchecked.
- Start safe. Graduate to bypass when ready.

---

### Phase 9: Verify everything works

Run these checks. ALL must pass before moving on:

1. `claude --version` returns a version number.
2. `CLAUDE.md` exists and has the `@` import lines:
   - Mac: `test -f ~/.claude/CLAUDE.md && head -3 ~/.claude/CLAUDE.md`
   - Windows: `Test-Path $env:USERPROFILE\.claude\CLAUDE.md; Get-Content $env:USERPROFILE\.claude\CLAUDE.md -TotalCount 3`
3. `primer.md` exists:
   - Mac: `test -f ~/.claude/primer.md && echo OK`
   - Windows: `Test-Path $env:USERPROFILE\.claude\primer.md`
4. `MEMORY.md` exists at the memory folder path from Phase 4.
5. `settings.json` exists and is valid JSON. Try in this order:
   - Mac: `python3 -m json.tool < ~/.claude/settings.json` — if `python3` isn't found, try `node -e "JSON.parse(require('fs').readFileSync(require('os').homedir()+'/.claude/settings.json','utf8'))" && echo OK`.
   - Windows: `Get-Content $env:USERPROFILE\.claude\settings.json -Raw | ConvertFrom-Json; if ($?) { "OK" }`
6. The `@` import paths in `CLAUDE.md` point to files that actually exist (`test -f` each path on Mac, `Test-Path` each on Windows).

If anything fails, go to Troubleshooting at the bottom and fix it. Don't move on with broken parts.

---

### Phase 10: First real session

Once everything is verified:

1. Tell me setup is complete. Echo my name, role, and project back so I know context loaded.
2. Ask me 4 short onboarding questions, **one at a time** (don't list them all at once — wait for my answer to each before asking the next):
   - How do you like me to communicate? (autonomous and terse / collaborative and detailed / somewhere between)
   - What tools or platforms do you use most? (so I can suggest integrations later)
   - How often will you use Claude Code? (daily / weekly / for specific projects)
   - What's the biggest thing you hope this unlocks for you?
3. Save each answer that reveals something durable as a memory file (type `user` or `feedback`) and add its line to `MEMORY.md`.
4. Rewrite `primer.md` with my current project state and a concrete next step.
5. Suggest one specific thing I can ask you next to test the setup.

---

## Troubleshooting (Claude reads this if a step fails)

**`command not found: claude` after npm install:**
- Mac: npm global bin not on PATH. Run `npm prefix -g`, then add `<that-path>/bin` to `~/.zshrc` and reload.
- Windows: close ALL terminal windows and reopen. If still missing, run `npm prefix -g` and add the parent folder to system PATH.

**`command not found: git` after Xcode CLI tools install (Mac):**
- The popup wasn't completed. Re-run `xcode-select --install` and have me click Install on the dialog.

**`command not found: brew` after Homebrew install (Apple Silicon Mac):**
- The eval line wasn't run. Run `eval "$(/opt/homebrew/bin/brew shellenv)"` and add it to `~/.zshrc`.

**npm EACCES on Mac:**
- Don't sudo. Run `sudo chown -R $(whoami) $(npm config get prefix)/{lib/node_modules,bin,share}` once, then retry.

**npm EPERM on Windows:**
- Reopen PowerShell as Administrator. If that still fails, run `npm config set prefix "$env:APPDATA\npm"`.

**Node / npm / git missing right after install:**
- PATH only updates in NEW terminal windows. The Claude Code session itself may also need a restart.
- **Mac:** open a fresh shell or `source ~/.zshrc`. Then re-check.
- **Windows:** run this in PowerShell to refresh PATH without restarting:
  ```
  $env:Path = [System.Environment]::GetEnvironmentVariable("Path","Machine") + ";" + [System.Environment]::GetEnvironmentVariable("Path","User")
  ```
  If it's still missing after that, tell me to fully quit Claude Code and reopen it, then resume from the failed phase.

**Claude Code can't see new PATH after install (subprocess inherited old PATH):**
- Same fix as above. Refresh PATH in the current shell, or quit and reopen Claude Code. This is the most common silent failure on Windows.

**PowerShell blocks scripts on Windows:**
- Run `Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser` (this is safe; it only allows local scripts).

**winget or choco not available on Windows:**
- Skip them. Use the direct installers from `https://git-scm.com/download/win` and `https://nodejs.org`.

**`@` imports in CLAUDE.md not loading:**
- The paths must be exact. The username in the path must match `whoami` on Mac or `$env:USERNAME` on Windows. The `-Users-` prefix is literal.

**Windows Defender blocking Node:**
- Add the Node install folder to exclusions: Settings > Virus & Threat Protection > Manage Settings > Add Exclusion > Folder.

**Claude takes one extra turn when a session ends:**
- That's the Stop hook working as designed. It blocks the first stop so Claude updates `primer.md` and `MEMORY.md`, then the `stop_hook_active` guard lets the next stop through. If Claude keeps stopping and restarting forever, the guard check got mangled: re-copy the Stop hook command from Phase 8 exactly.

**settings.json hooks not firing:**
- The JSON is probably malformed. Validate it:
  - Mac: `python3 -m json.tool < ~/.claude/settings.json` (or `node -e "JSON.parse(require('fs').readFileSync('$HOME/.claude/settings.json'))"`)
  - Windows: `Get-Content $env:USERPROFILE\.claude\settings.json -Raw | ConvertFrom-Json`
- Common causes: trailing commas, unescaped quotes inside command strings, missing closing braces.
- After fixing JSON, fully quit and reopen Claude Code so settings reload.

**Claude can't find primer.md or MEMORY.md mid-session:**
- The path in `CLAUDE.md` is wrong. Re-run Phase 5 using the SAFE_USER value from Phase 4 (spaces replaced with underscores), and absolute paths (`/Users/...` on Mac, `C:\Users\...` on Windows — never `~`).
- Confirm with `test -f` (Mac) / `Test-Path` (Windows) on each `@` path in `CLAUDE.md`.

**Homebrew installer asks for Return/password and seems frozen:**
- It's not frozen. Homebrew is genuinely waiting for input. Tell me what it's asking for so I can respond.

**`xcode-select --install` popup didn't appear (Mac):**
- Run `softwareupdate --list` to confirm CLI Tools are available, then re-run `xcode-select --install`. If still nothing, install manually from `https://developer.apple.com/download/all/` (search for "Command Line Tools for Xcode").

**`xcode-select` returned 0 but `git --version` still fails:**
- The GUI installer is still running. Poll `pkgutil --pkg-info=com.apple.pkg.CLTools_Executables` until it returns "version: ..." instead of "No receipt found." Don't proceed before then.

**Username has a space (Mac or Windows):**
- The `@` import line treats whitespace as a delimiter. Use the SAFE_USER value from Phase 4 (spaces replaced with underscores) for the folder path. The folder physically lives at e.g. `~/.claude/projects/-Users-Tyler_Smith/`, and the `@` import path matches.

---

## How it all works (read this once setup is done)

On Mac your `.claude` folder lives at `~/.claude/`. On Windows it lives at `C:\Users\YOURUSERNAME\.claude\`. Same structure either way:

```
.claude/
  CLAUDE.md                          <-- Master rule book (loaded every session via @ imports)
  primer.md                          <-- Session sticky note (rewritten each session)
  settings.json                      <-- Hooks + permissions
  commands/                          <-- Custom slash commands (optional)
  skills/                            <-- Custom skills (optional)
  projects/
    -Users-YOURUSERNAME/
      memory/
        MEMORY.md                    <-- Long-term memory index (always loaded via @)
        feedback_*.md                <-- How you want Claude to work
        project_*.md                 <-- Project context + decisions
        reference_*.md               <-- Where to find things in external systems
        user_*.md                    <-- Who you are, your expertise
```

### The session loop

1. **Session starts** → Claude reads `CLAUDE.md` → imports `primer.md` + `MEMORY.md` (the index of everything it knows about you).
2. **During session** → when Claude learns a durable fact, it writes a memory file and adds one line to the `MEMORY.md` index. When you correct it, that becomes a `feedback` memory with the why attached.
3. **Session ends** → Stop hook blocks the first stop and hands Claude the reminder → Claude rewrites `primer.md` and indexes any new memories → the `stop_hook_active` guard lets the next stop through.
4. **Next session** → back to step 1, full context carried forward.

### Memory types Claude builds over time

- **user** — your role, expertise, preferences (helps Claude tailor its tone and approach).
- **feedback** — corrections and confirmations, each with a "Why" and "How to apply" (so Claude doesn't repeat mistakes or drift). These are the highest-value memories in the whole system.
- **project** — ongoing work, deadlines, decisions (so Claude understands the WHY behind requests).
- **reference** — pointers to external systems (Linear, Slack, dashboards, etc.).

One fact per file, one line per file in the index. `MEMORY.md` never holds the full content, and individual files never go unindexed.

### What carries between sessions vs what doesn't

- **Carries:** anything in `MEMORY.md`, individual memory files, `primer.md`, and `CLAUDE.md`.
- **Does NOT carry:** the chat transcript itself. If something matters, it has to land in one of those files. That's why the Stop hook exists — it forces Claude to write things down.

---

## Optional extras

### Custom skills

Skills are reusable instruction files Claude loads when a task matches their description. Just ask Claude: *"Create a skill called my-skill that does X."* Claude will create the folder and file for you. Or do it manually. Two rules: the file must be named `SKILL.md` inside the skill's folder (any other filename won't register), and the `description` should be written as trigger text ("Use when...") so Claude knows when to load it.

**Mac:**
```
mkdir -p ~/.claude/skills/my-skill
cat > ~/.claude/skills/my-skill/SKILL.md << 'EOF'
---
name: my-skill
description: Use when I ask for X, mention Y, or need help doing Z.
---

(Your prompt template here. Claude follows these instructions when the skill triggers or when you type /my-skill in chat.)
EOF
```

**Windows (PowerShell):**
```
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude\skills\my-skill"
@"
---
name: my-skill
description: Use when I ask for X, mention Y, or need help doing Z.
---

(Your prompt template here. Claude follows these instructions when the skill triggers or when you type /my-skill in chat.)
"@ | Set-Content "$env:USERPROFILE\.claude\skills\my-skill\SKILL.md"
```

### Custom slash commands

Simpler than skills — just a markdown file with a prompt.

**Mac:**
```
cat > ~/.claude/commands/hello.md << 'EOF'
Say hello and summarize what we worked on last session based on primer.md.
EOF
```

**Windows (PowerShell):**
```
"Say hello and summarize what we worked on last session based on primer.md." | Set-Content "$env:USERPROFILE\.claude\commands\hello.md"
```

Invoke with `/hello`.

---

## Leveling up (your first month)

The setup above makes Claude Code useful on day one. These habits are what make it powerful. Take them roughly in order; each one builds on the last.

### Week 1: build the memory habit

Say **"remember that"** whenever Claude learns something worth keeping. Correct it when it does something you don't like, and tell it when it nails something so it saves the confirmed approach. Every correction becomes a `feedback` memory, and within a week Claude stops making the same mistake twice. This habit is 80% of what separates people who love Claude Code from people who feel like they're starting over every session.

### Week 2: plan before big builds

Press **Shift+Tab** to cycle into **Plan Mode** before anything bigger than a small edit. Claude researches your files and proposes a plan you approve BEFORE it touches anything. A bad plan costs one message to fix. A bad build costs an afternoon. Rule of thumb: if the task would take you more than 30 minutes by hand, plan it first.

### Week 3: turn repeat work into skills

The second time you ask for the same kind of thing (a weekly report, a code review checklist, formatted meeting notes, a content template), say: *"Create a skill called X that does this, based on what we just did."* From then on it's one slash command. And when a skill's output disappoints you, say *"update the skill so this doesn't happen again."* Skills that learn from every run are how your setup compounds.

### Week 4: longer leashes

- **Permission graduation.** You started with `"defaultMode": "default"` (Claude asks before running commands). Once you trust the workflow, move to `"acceptEdits"` (file edits auto-approved, commands still ask), and later `"bypassPermissions"` for maximum speed. Graduate one level at a time.
- **Goals.** In the terminal version of Claude Code, `/goal` gives Claude a finish line and lets it keep working turn after turn until the finish line is verifiably true (for example: "every task in docs/roadmap.md is checked off and the app builds cleanly"). The skill is writing a condition a machine can check. Ask Claude: *"Help me structure a goal for this before I start."*
- **Sub-agents.** For big independent chunks (research three options, audit every page, migrate many files), ask Claude to *"use sub-agents in parallel."* Each one gets its own fresh focus while your main conversation stays clean.

### Anytime: let Claude improve its own setup

Once a week, ask: *"Look at how we've been working. What's slowing us down, and what should we add to CLAUDE.md, memory, or skills to fix it?"* Your setup should be a living system that gets sharper the more you use it. Everything in this guide came from exactly that loop.

---

## Tips for using this every day

- **`primer.md` = short-term memory** (what's happening now, what's next).
- **`MEMORY.md` = long-term memory index** (one line per fact Claude knows about you).
- Keep `MEMORY.md` under 200 lines. It's loaded into every conversation. The detail belongs in the individual memory files, which are only read when relevant.
- Say **"remember that"** when you want Claude to explicitly save something.
- If Claude forgets something between sessions, it's missing from `primer.md` or the memory folder. Add it.
- Full transcripts of past sessions live under `~/.claude/projects/`. If Claude claims it doesn't know something you've discussed before, tell it to search those transcripts before you re-explain.
- The system gets smarter the more you use it. Memory grows. Trust it.
