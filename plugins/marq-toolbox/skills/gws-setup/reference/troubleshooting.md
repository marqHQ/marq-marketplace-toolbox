# gws setup — troubleshooting

Each entry: what the user or you will see → what it means → what to do.
Fix it yourself where possible; involve the user only for the browser.

## "Error 403: org_internal" / "This client is restricted to users within its organization"

**Means:** the user signed in with a non-Marq Google account (personal Gmail,
a client's account). The consent screen is internal-only by design.

**Do:** run the step 4 login command again and tell the user to pick their
`@marq.com` account on the account chooser. The `prompt=select_account` in
the URL forces the chooser, so they can switch.

## "Error 403: access_denied" / "This app is blocked"

**Means:** a Workspace admin policy is blocking the client, or the user's
account is suspended.

**Do:** stop. Tell the user to message Chandler Shipley with the exact error
text. Do not try alternative clients, `gws auth setup`, or other scopes.

## Browser never opened / user sees only a long URL in the terminal

**Do:** copy the `https://accounts.google.com/o/oauth2/auth?...` URL from
the command output and give it to the user to click. The command is still
waiting; it will finish when they click Allow.

## Login command timed out or was killed before the user clicked Allow

**Do:** run it again. Each run creates a fresh local callback port; nothing
is left half-done. If your tool's command timeout is short, run it in the
background and poll, or ask the user to click Allow promptly.

## "address already in use" on login

**Means:** a previous login attempt is still listening on the callback port.

**Do:** wait 30 seconds and retry. If it persists, find and kill the stale
`gws` process (`pkill gws` on macOS; `Stop-Process -Name gws` on Windows),
then retry.

## macOS Keychain dialog: "gws wants to use your confidential information"

**Means:** gws is storing the encryption key for its credentials file in the
login keychain. Normal.

**Do:** tell the user to click **Always Allow** (not "Allow", which prompts
every time). If they clicked Deny, run `gws auth logout` then step 4 again.

## "gws: command not found" after install

- **npm install:** the npm global bin dir isn't on PATH. `npm bin -g` (or
  `npm prefix -g` + `/bin`) shows it. Add it to `~/.zshrc` (macOS) or the
  user PATH (Windows), then open a fresh shell or `export PATH=...` in the
  current one.
- **Prebuilt binary:** confirm `~/.local/bin/gws` (macOS) or
  `%LOCALAPPDATA%\Programs\gws\gws.exe` (Windows) exists and is executable,
  and that the PATH line was added. In the current shell, export the PATH
  yourself; a new terminal will pick up the rc file.
- **Homebrew on Apple Silicon:** brew lives at `/opt/homebrew/bin`; if it's
  not on PATH, `eval "$(/opt/homebrew/bin/brew shellenv)"`.

## `gws auth status` shows `has_refresh_token: false` after a "success" login

**Means:** credentials were written but the keyring step failed (usually a
denied Keychain prompt, or a headless/locked session).

**Do:** `gws auth logout`, then step 4 again and make sure the user clicks
**Always Allow**. If it still fails and the machine is headless, set
`GOOGLE_WORKSPACE_CLI_KEYRING_BACKEND=file` for the login and for every later
gws call (put it in the shell rc). Mention this to the user in the wrap-up.

## "insufficient authentication scopes" on a Sheets/Docs/Slides/Calendar call

**Means:** the token was created with fewer scopes than step 4 requests —
usually because someone ran a bare `gws auth login` earlier or filtered to
`-s drive` only.

**Do:** run the exact step 4 command again. It replaces the token with the
full approved set. Do not add scopes beyond the step 4 list.

## Gmail calls fail with "Gmail API has not been used in project marq-gws-cli"

**Means:** working as intended. Gmail is deliberately disabled on Marq's
project. Tell the user gws isn't set up for email at Marq and move on. Do
not enable it, do not request Gmail scopes, do not suggest another client.

## Existing `client_secret.json` with a different client_id

Covered in step 3: ask, back up as `client_secret.json.bak`, copy ours in,
then step 4. The old refresh token belongs to the old client and is
replaced by the new login.

## Codex/agent sandbox refuses to write to `~/.config` or `~/Downloads`

**Means:** the agent's filesystem sandbox is limited to the workspace.

**Do:** request elevated/unsandboxed permission for the specific commands
(the agent's usual approval prompt). Explain to the user in one sentence
that you need permission to save a settings file in their home folder. Do
not work around it by putting the file somewhere else and pointing
`GOOGLE_WORKSPACE_CLI_CONFIG_DIR` at it — that breaks the next session.

## Step 6 skill install fails or `npx` missing

Not blocking. Auth is done. Tell the user in the wrap-up that the "how to
use gws" skills still need installing and that Chandler can help; give
Chandler the exact error.

## Drive link says "You need access" / asks to request access

**Means:** the browser is signed in to a personal or client Google account,
not the user's `@marq.com` account. The client file is shared with the Marq
domain only.

**Do:** tell the user to switch accounts in the top-right avatar menu on the
Drive page (or open the link in an incognito window and sign in with
`@marq.com`), then download again. Never request access on their behalf and
never look for the file anywhere else.

## Downloaded file is named `client_secret (1).json` or similar

**Means:** the browser found an earlier download with the same name.

**Do:** use the newest file whose name starts with `client_secret` and ends
in `.json`, verify the `project_id` as in step 3, copy it, then delete every
`client_secret*.json` left in Downloads.
