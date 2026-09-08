---
name: gws-setup
description: One-time, self-service setup of the Google Workspace CLI (`gws`) for a Marq employee, so their AI agent (Codex, Claude Code, etc.) can work in their Google Drive, Docs, Sheets, Slides, and Calendar as them. Installs gws, fetches Marq's shared OAuth client file from the company Google Drive through the agent's Drive connector (manual download only as a fallback), runs the browser login with the approved scopes, verifies it, then installs Google's official gws usage skills. Safe to invoke on an already-configured machine (it detects that and exits). Use when someone invokes /gws-setup, or says "set up gws", "install the Google Workspace CLI", "connect my Google Drive to Codex/Claude", or "connect Google Workspace". Do NOT use for day-to-day gws usage after setup — Google's `gws-*` skills cover that.
---

# gws setup for Marq (one-time)

You are setting up `gws` for a **non-technical colleague**. They will not know
what a terminal, a hidden folder, or an OAuth scope is, and they should never
need to. **You do every step.** The only things they do themselves:

1. Click **Allow** in the browser once (step 4).
2. Possibly type their Mac password if Homebrew asks (rare).
3. Only if you have no Google Drive tool: download one file from a link (step 3 fallback).

Talk to them in plain language. Never ask them to run a command, open a
folder, or edit a file. If something needs a human, tell them exactly what to
click, and nothing more.

Bundled with this skill:

| Path (relative to this SKILL.md) | What it is |
|---|---|
| `reference/troubleshooting.md` | Known failures and the fix for each. Read it when a step fails. |

Marq's shared OAuth client file is **not** bundled (this plugin's source is
public). It lives in the company Google Drive, readable by every `@marq.com`
account:

| | |
|---|---|
| Location | Shared drives → **Marq Company Wide** → `ai_plugin_assets` → `client_secret.json` |
| File ID | `1bxqqg1i2CLPCgmT7RpzwAzNaAKZdFFJx` |
| Folder ID | `1lX5rlThTe2AfJnWC_L8r8sU_YgrGAYdv` |
| Link | https://drive.google.com/file/d/1bxqqg1i2CLPCgmT7RpzwAzNaAKZdFFJx/view |

You fetch it yourself with the Google Drive connector (step 3). The user only
downloads it by hand if no Drive tool is available to you.

## Non-negotiables

- **Never run `gws auth setup`.** It needs `gcloud`, project-creation rights,
  and creates a personal "unverified" Google Cloud project. Marq already has
  the project. The Drive file *is* the output of that step.
- **Never install `gcloud`, never create or touch a Google Cloud project.**
- **Never run `gws auth login` bare, with `--full`, or with any Gmail scope.**
  The bare command silently requests mailbox read/send access. The only
  approved login is the exact command in step 4.
- **Never overwrite an existing login without asking** (step 3 explains how
  to tell).
- Treat the sign-in as the user's, not yours: never paste their tokens,
  `credentials.enc`, or `gws auth export` output anywhere.
- Never commit, upload, or share the client file anywhere else. Copy it into
  the config dir, then delete the download.

## Before anything — is this machine already set up?

This skill may be invoked on a machine where setup already happened (a
second `/gws-setup`, a stray mention of gws, a re-run after a crash). Check
first, so a finished user gets a one-line answer instead of a re-login:

```bash
gws auth status
```

If **all** of these hold, the user is done:

- the command runs (gws is installed),
- `"has_refresh_token": true`,
- the account is an `@marq.com` address,
- `"project_id": "marq-gws-cli"` (Marq's shared client; `client_id` is shown
  truncated, so check the project instead),
- scopes include `https://www.googleapis.com/auth/drive` and
  `https://www.googleapis.com/auth/calendar`.

Then say so in one sentence ("You're already set up — your AI can work in
your Google Drive, Docs, Sheets, Slides, and Calendar."), run step 6 only if
Google's `gws-*` skills aren't installed yet, and stop. Otherwise continue
from step 0.

## Step 0 — Preflight (no user interaction)

Figure out two things and keep them in mind:

- **OS**: macOS or Windows. (Linux: treat like macOS with `~/.config/gws`.)
- **Which agent you are** (Codex, Claude Code, Cursor, …). Needed in step 6.

Config directory (create it if missing):

| OS | gws config dir | Downloads dir |
|---|---|---|
| macOS / Linux | `~/.config/gws` | `~/Downloads` |
| Windows | `%USERPROFILE%\.config\gws` (e.g. `C:\Users\<name>\.config\gws`) | `%USERPROFILE%\Downloads` |

## Step 1 — Is gws already installed?

```bash
gws --version
```

- Prints `gws 0.22.x` or newer → skip to step 3.
- Prints an older version → upgrade with whichever method installed it
  (`brew upgrade googleworkspace-cli`, `npm install -g @googleworkspace/cli@latest`,
  or re-download the binary), then continue.
- Command not found → step 2.

## Step 2 — Install gws

Try in this order and stop at the first that works. Do not ask the user to
choose; just pick.

**macOS**

1. Homebrew present (`brew --version` works):
   ```bash
   brew install googleworkspace-cli
   ```
2. Else Node present (`npm --version` works):
   ```bash
   npm install -g @googleworkspace/cli
   ```
3. Else prebuilt binary, no admin rights needed:
   ```bash
   ARCH=$(uname -m); [ "$ARCH" = "arm64" ] && T=aarch64-apple-darwin || T=x86_64-apple-darwin
   mkdir -p ~/.local/bin && cd /tmp
   curl -sL "https://github.com/googleworkspace/cli/releases/latest/download/google-workspace-cli-$T.tar.gz" | tar xz
   mv $(find . -maxdepth 3 -type f -name gws | head -1) ~/.local/bin/gws && chmod +x ~/.local/bin/gws
   grep -q '.local/bin' ~/.zshrc || echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
   export PATH="$HOME/.local/bin:$PATH"
   ```

**Windows** (PowerShell)

1. Node present:
   ```powershell
   npm install -g @googleworkspace/cli
   ```
2. Else prebuilt binary:
   ```powershell
   $d = "$env:LOCALAPPDATA\Programs\gws"; New-Item -ItemType Directory -Force $d | Out-Null
   Invoke-WebRequest "https://github.com/googleworkspace/cli/releases/latest/download/google-workspace-cli-x86_64-pc-windows-msvc.zip" -OutFile "$env:TEMP\gws.zip"
   Expand-Archive "$env:TEMP\gws.zip" -DestinationPath $d -Force
   $exe = Get-ChildItem $d -Recurse -Filter gws.exe | Select-Object -First 1; Move-Item $exe.FullName "$d\gws.exe" -Force
   [Environment]::SetEnvironmentVariable("Path", [Environment]::GetEnvironmentVariable("Path","User") + ";$d", "User")
   $env:Path += ";$d"
   ```

Confirm with `gws --version` before moving on. If it still isn't found, see
troubleshooting → "gws: command not found after install".

## Step 3 — Get the client file in place

1. Create the config dir (table in step 0).
2. Look for an existing `client_secret.json` there.
   - **Exists and its `client_id` starts with `623463698939-`** → it is already
     Marq's client. Leave it and skip to step 4. (The "Before anything" check
     already established the login itself is missing or incomplete.)
   - **Exists with a different `client_id`** → someone set up gws another way.
     Tell the user in one sentence ("You have an older Google login for this
     tool — I'll replace it with Marq's shared one, which is what we want") and
     wait for a yes. Then rename the old file to `client_secret.json.bak` and
     continue.
   - **None** → continue.
3. **Fetch it yourself with the Google Drive connector** (default path, no
   user interaction). Use whatever Drive tool your harness exposes:
   - **Claude (claude.ai Google Drive connector):**
     `mcp__claude_ai_Google_Drive__download_file_content` with
     `fileId="1bxqqg1i2CLPCgmT7RpzwAzNaAKZdFFJx"`. It returns the file as a
     base64 string in `content` — decode it to get the JSON text. (The
     `read_file_content` tool does not support JSON files; use download.)
     If the ID ever fails, `search_files` with
     `title = 'client_secret.json' and parentId = '1lX5rlThTe2AfJnWC_L8r8sU_YgrGAYdv'`
     and download the result.
   - **ChatGPT / Codex (Google Drive app or connector):** open the file by its
     link or search for `client_secret.json` in the `ai_plugin_assets` folder of
     the **Marq Company Wide** shared drive, and read its contents. If the tool
     returns base64, decode it; if it returns text, use it as-is.
   - Write the JSON text to `<config dir>/client_secret.json`. Never echo the
     contents into the conversation; the file holds a client secret.
4. **Fallback — only if no Drive tool exists or every fetch attempt fails.**
   Ask the user to download it. Say this, or close to it:

   > I need one small settings file from Marq's Google Drive. Please open this
   > link, make sure you're signed in with your **@marq.com** account, and click
   > the **download** icon at the top right. Then tell me when it's done.
   > https://drive.google.com/file/d/1bxqqg1i2CLPCgmT7RpzwAzNaAKZdFFJx/view

   When they say it's downloaded, take the newest file matching
   `client_secret*.json` in the Downloads dir (browsers may add ` (1)`), copy it
   to `<config dir>/client_secret.json`, then **delete the copy in Downloads**
   so the client file isn't lying around twice. If Drive says they need access,
   they are signed in with the wrong Google account — see troubleshooting.
5. Sanity check the installed file either way: it must contain
   `"project_id":"marq-gws-cli"` and a `client_id` starting with `623463698939-`.
   If it doesn't, it's the wrong file — retry the fetch.

## Step 4 — Log in (the one thing the user does)

Before running it, tell the user what is about to happen, in these words or
close to them:

> A browser window is about to open asking you to sign in to Google and
> approve access. Use your **@marq.com** account. Click **Allow**. Then come
> back here.

Then run exactly this. Nothing more, nothing less:

```bash
gws auth login -s drive,sheets,docs,slides,calendar
```

Notes for you:

- The command **blocks until the user finishes in the browser**. If your
  tool has a short command timeout, run it in the background or with the
  longest timeout you have. If it dies before the user finishes, just run
  it again — it is safe to repeat.
- It prints a `https://accounts.google.com/...` URL. On most machines the
  browser opens by itself. If the user says nothing opened, give them the
  URL to click.
- On macOS the first run can pop a **Keychain** dialog. Tell the user to
  click **Always Allow**.
- Success looks like a JSON block with `"status": "success"` and the user's
  `@marq.com` address. See troubleshooting for "Access blocked" and other
  failures.

## Step 5 — Verify (no user interaction)

```bash
gws auth status
```

Must show `"has_refresh_token": true`, the user's `@marq.com` account, and
scopes including `https://www.googleapis.com/auth/drive` and
`https://www.googleapis.com/auth/calendar`.

Then a real call:

```bash
gws drive files list --params '{"pageSize":3,"fields":"files(name)","q":"trashed=false"}'
```

Three file names back → authentication works end to end.

If that call fails, **do not improvise and do not re-run step 4.** Match the
error in `reference/troubleshooting.md` first. A failing step 5 never means the
install or the client file was wrong — by this point both are verified — so
redoing earlier steps only wastes the user's time.

The most likely failure here is
`Caller does not have required permission to use project marq-gws-cli`
(or a bare `403`). That is an account permission on Marq's Cloud project, is
fixed by Chandler and no one else, and is not something any step in this skill
can resolve. Read the troubleshooting entry before saying anything to the user.

## Step 6 — Install Google's gws usage skills (default, do it)

Google publishes official agent skills for using gws. Install the Marq-relevant
subset so future sessions know how to drive Drive, Docs, Sheets, Slides,
Calendar, and Tasks. Pick `AGENT` from the list you identified in step 0
(`codex`, `claude-code`, `cursor`, `gemini-cli`, …). Requires Node; if
`npx` is missing, skip this step and say so in the wrap-up.

```bash
npx -y skills add https://github.com/googleworkspace/cli -g -y --copy -a AGENT -s gws-shared,gws-drive,gws-drive-upload,gws-docs,gws-docs-write,gws-sheets,gws-sheets-read,gws-sheets-append,gws-slides,gws-calendar,gws-calendar-insert,gws-calendar-agenda,gws-tasks
```

Deliberately **not** installed: `gws-gmail*` (Gmail is excluded from Marq's
login on purpose), `gws-chat*`, `gws-admin-reports`, `gws-events*`,
`gws-modelarmor*`, `gws-script*`, personas, and recipes. Don't add them.

One caveat to remember: Google's `gws-shared` skill shows `gws auth login`
with no flags as its example. The user is already logged in, so **never run
that** in a later session. If credentials ever need refreshing, the step 4
command is the only approved one.

## Step 7 — Tell the user they're done

Plain language, three or four sentences, no jargon. Cover:

- It's done, and they won't have to do this again.
- What it means: their AI can now read and edit their Google Drive, Docs,
  Sheets, Slides, and Calendar, as them, with the same access they have.
- One concrete thing to try: *"Ask me to list your five most recent Google
  Docs, or to add a row to a sheet."*
- If step 6 was skipped, say a follow-up is needed for the usage skills.

Never delete anything inside the config dir.
