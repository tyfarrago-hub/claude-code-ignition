# Claude Code Setup Guide

A complete one-paste setup for Claude Code with persistent memory, session continuity, and beginner-friendly defaults. Works on **Mac** and **Windows**.

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

1. **Plain English recap on complex answers.** After any technical or multi-step answer, append a section titled `**In plain English:**` that re-states the same answer without the jargon. The point is translation, not simplification. Assume the reader is intelligent but new to the technical layer. Strip the jargon, keep the substance. Trigger this for: code, commands, errors, multi-step plans, anything with unfamiliar terms. Skip it for yes/no answers and casual chat.
2. **One clear next action per response.** Not a list of options. If I genuinely need to choose, ask me ONE question.
3. **Flag uncertainty with `[UNCLEAR]`.** Don't guess. If you don't know whether a path, command, or value is correct, mark it.
4. **Short, direct responses.** No "Great question!", no "Sure thing!", no preamble. Don't recap what I just asked.
5. **Diagnose before fixing.** When something breaks, explain what you think is wrong and why BEFORE you change anything.
6. **Verify before claiming done.** Run the check, read the output, then tell me it works. "This should work" is not allowed.
7. **No half-features.** If a step fails, fix it before moving on. Don't skip and circle back later.
8. **Ask when genuinely blocked.** Rules 4 and 7 mean "no fluff" and "don't skip," not "stay silent forever." If you've exhausted reasonable diagnostic options and need information only I have (a credential, a decision, an external state), surface the blocker in one tight sentence and ask.

## AGENT RULES

- Read `primer.md` and `MEMORY.md` at the start of every session before doing anything else.
- At the end of every session, rewrite `primer.md` completely with: active project, what was just completed, exact next step, open blockers. Keep it under 100 lines.
- Save anything long (drafts, proposals, plans, code) to a file the moment it exists. Don't let it live only in chat.
- Never ask me for context that's already in `primer.md`, `MEMORY.md`, or any imported file. Read first.
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

### Phase 7: Write `MEMORY.md` (long-term memory index)

**File path** (the same memory folder you created in Phase 4 — use the **SAFE_USER** value from Phase 4, not RAW_USER):
- Mac: `~/.claude/projects/-Users-<SAFE_USER>/memory/MEMORY.md`
- Windows: `$env:USERPROFILE\.claude\projects\-Users-<SAFE_USER>\memory\MEMORY.md`

Use my name, role, and project from the top of this guide to fill in the User Profile section. Detect my OS, shell, and architecture and fill those in too.

```
# MEMORY.md

## User Profile
- Name: (use what I gave you at the top)
- Role: (use what I gave you)
- Currently working on: (use what I gave you)
- Communication style: (will refine during onboarding — default: short, direct, autonomous)

## System
- OS: (detect — macOS / Windows / Linux + version)
- Architecture: (Apple Silicon / Intel Mac / x64 Windows / arm64 Windows)
- Shell: (zsh / PowerShell / bash)

## Key File Locations
- Memory folder: (full path you just created)
- Settings: ~/.claude/settings.json (or Windows equivalent)

## Projects
- (Claude fills this in as we work)

## Preferences (learned over time)
- (Claude adds entries here when I correct or confirm something)
```

---

### Phase 8: Write `~/.claude/settings.json` (hooks + permissions)

This is the automation layer. Two hooks fire automatically:
- **Stop hook:** when Claude finishes a response, it plays a sound and reminds Claude to update `primer.md` and `MEMORY.md`.
- **Notification hook:** when Claude needs my input, it plays a sound and shows a popup so I notice.

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
            "command": "echo 'SESSION ENDING: 1) Rewrite ~/.claude/primer.md with active project, what was just completed, exact next step, and open blockers (under 100 lines). 2) Update MEMORY.md with any new facts learned this session.'",
            "statusMessage": "Reminding Claude to update primer + memory..."
          }
        ]
      }
    ],
    "Notification": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "osascript -e 'display notification \"Claude needs your input\" with title \"Claude Code\" sound name \"Funk\"'"
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
            "command": "echo SESSION ENDING: 1) Rewrite primer.md with active project, what was just completed, exact next step, and open blockers (under 100 lines). 2) Update MEMORY.md with any new facts learned this session.",
            "statusMessage": "Reminding Claude to update primer + memory..."
          }
        ]
      }
    ],
    "Notification": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "powershell -c \"[System.Media.SystemSounds]::Exclamation.Play(); Add-Type -AssemblyName System.Windows.Forms; $n = New-Object System.Windows.Forms.NotifyIcon; $n.Icon = [System.Drawing.SystemIcons]::Information; $n.BalloonTipTitle = 'Claude Code'; $n.BalloonTipText = 'Claude needs your input'; $n.Visible = $true; $n.ShowBalloonTip(5000); Start-Sleep -Seconds 6; $n.Dispose()\""
          }
        ]
      }
    ]
  }
}
```

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
3. Update `MEMORY.md` with my answers.
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

1. **Session starts** → Claude reads `CLAUDE.md` → imports `primer.md` + `MEMORY.md`.
2. **During session** → Claude creates and updates individual memory files as it learns about you.
3. **Session ends** → Stop hook fires → Claude rewrites `primer.md` and updates `MEMORY.md`.
4. **Next session** → back to step 1, full context carried forward.

### Memory types Claude builds over time

- **user** — your role, expertise, preferences (helps Claude tailor its tone and approach).
- **feedback** — corrections and confirmations (so Claude doesn't repeat mistakes or drift).
- **project** — ongoing work, deadlines, decisions (so Claude understands the WHY behind requests).
- **reference** — pointers to external systems (Linear, Slack, dashboards, etc.).

### What carries between sessions vs what doesn't

- **Carries:** anything in `MEMORY.md`, individual memory files, `primer.md`, and `CLAUDE.md`.
- **Does NOT carry:** the chat transcript itself. If something matters, it has to land in one of those files. That's why the Stop hook exists — it forces Claude to write things down.

---

## Optional extras

### Custom skills

Skills are reusable prompt templates you call with `/`. Just ask Claude: *"Create a skill called my-skill that does X."* Claude will create the folder and file for you. Or do it manually:

**Mac:**
```
mkdir -p ~/.claude/skills/my-skill
cat > ~/.claude/skills/my-skill/my-skill.md << 'EOF'
---
name: my-skill
description: What this skill does
user_invocable: true
---

(Your prompt template here. Claude follows these instructions when you type /my-skill in chat.)
EOF
```

**Windows (PowerShell):**
```
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude\skills\my-skill"
@"
---
name: my-skill
description: What this skill does
user_invocable: true
---

(Your prompt template here. Claude follows these instructions when you type /my-skill in chat.)
"@ | Set-Content "$env:USERPROFILE\.claude\skills\my-skill\my-skill.md"
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

## Tips for using this every day

- **`primer.md` = short-term memory** (what's happening now, what's next).
- **`MEMORY.md` = long-term memory** (who you are, how you work, key facts).
- Keep `MEMORY.md` under 200 lines. It's loaded into every conversation.
- Individual memory files can be longer — they're only read when relevant.
- Say **"remember that"** when you want Claude to explicitly save something.
- If Claude forgets something between sessions, it's not in `primer.md` or `MEMORY.md`. Add it.
- The system gets smarter the more you use it. Memory grows. Trust it.
