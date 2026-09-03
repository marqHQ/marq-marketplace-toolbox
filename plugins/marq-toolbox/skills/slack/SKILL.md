---
name: slack
description: Resolve a Marq coworker's name to their Slack user ID (from a per-user cached directory, falling back to live search), then draft and send Slack messages/DMs. Use whenever the task is to Slack, DM, message, or ping a specific person at Marq by name — e.g. "slack Yue", "DM Aria the summary", "message Brad about the deploy". Also use to look up someone's Slack ID/email, or to build or refresh the cached Slack directory.
---

# Slack directory + send

Eliminates the per-request cost of figuring out a coworker's Slack user ID. Each
operator keeps a private roster cache on their own machine; this skill ships no
employee data. The cache starts empty, fills itself as people are resolved, and
can be built out in full on request.

Slack user IDs are **permanent** — a person's ID never changes once assigned, so
a cached ID is never wrong for anyone still in the workspace. The only drift is
membership (new hires missing, leavers lingering), handled by the fallback and
the refresh procedure below.

## Where the cache lives

| OS | Cache file |
|---|---|
| macOS / Linux | `~/.config/marq-slack/directory.csv` |
| Windows | `%USERPROFILE%\.config\marq-slack\directory.csv` |

Create the directory and file on first use. Format:

```
# refreshed: <YYYY-MM-DD> · <N> accounts · mode: lazy|full
user_id,full_name,email,title,account_type,notes
```

`account_type` is `member`, `guest`, or `bot`. `notes` is free text — use it for
`PREFERRED` on people who have duplicate accounts. The first line is a comment
the freshness check reads; keep it in that exact shape.

## Who the operator is

The operator is the person you are working for: the logged-in Slack user. The
Slack MCP search tool's description states the current user's `user_id`. The
operator is never a send target unless they explicitly ask you to message
themselves.

## Resolve a recipient (do this first, always)

1. Grep the cache for the name — first name, last name, or fragment,
   case-insensitive:

   ```bash
   grep -i "yue" "$HOME/.config/marq-slack/directory.csv"
   ```

   Take `user_id` from the matching row.
2. **One match** → use it.
3. **Multiple matches** (two "Brad", or a member + guest duplicate) → prefer the
   row marked `PREFERRED` or `account_type=member`; if still ambiguous, ask the
   operator which one.
4. **No match, or no cache yet** → live lookup:

   ```
   mcp__claude_ai_Slack__slack_search_users  query="<name>"  response_format="concise"
   ```

   Then `slack_read_user_profile user_id=<id>` for the real name, email, title,
   and whether the account is restricted (guest) or a bot. **Append the row to the
   cache** so the next lookup is free, and update the `<N> accounts` count in the
   comment line. If it's a bot, record it as `bot` and do not DM it.

Never message automation or bot accounts. If the profile says bot, or the name
is obviously a bot ("Marketing Bot", "Zapier"), stop and tell the operator.

## Freshness check — after the request is served, never before

**Serve the operator's request first, always.** A cached ID is still correct even
if the roster is old. Then, in one line, offer a refresh if either holds:

- **Stale** — the `refreshed:` date in the comment line is 90+ days old.
- **Sparse** — mode is `lazy` and this session had a cache miss. A miss means
  the roster is probably missing other people too.

Say why ("your Slack roster is 90+ days old" / "<name> wasn't cached") and ask
whether to run **Build or refresh the full directory**. Only run it on a yes.
Otherwise say nothing more.

## Prepare the text — three things that silently break a message

Do this pass on **every** message, especially one the operator wrote themselves.
All three fail silently: the message sends, looks fine in the tool call, and is
wrong in Slack.

**1. Mentions must be `<@USERID>` — `@Name` is just text.**
A literal `@Jane Doe` produces no ping, no highlight, no notification. When the
operator types `@Name` in a draft they mean "mention them" — convert it using the
resolved ID. Channels are `<#CHANNELID>`. This is the difference between the
message reaching someone and sitting unread.

**2. Backtick every identifier — Slack markdown eats underscores.**
Slack renders `_italic_` and `*bold*`, so any name containing `_` or `*` gets
mangled when the characters pair up:

| Sent raw | Arrives as |
|---|---|
| `max(_fivetran_synced)` | max(*fivetran* synced) |
| `PROD_DOCUMENT_DATA_C2_DOCUMENTS_DATA` | PROD*DOCUMENT*DATA*C2*DOCUMENTS_DATA |

Wrap every table, column, file, function, and env-var name in backticks. This
bites hardest on technical messages — exactly where precision matters most and
where a mangled identifier makes the sender look careless.

**3. Fix obvious typos, then report every change.**
When sending text the operator wrote, silently fixing a clear slip (`ehlp` →
`help`, doubled spaces) is doing them a favor. Fixing their word choice, tone,
sentence order, or structure is not — that's their voice, leave it. After
sending, list back **every** change made, mechanical ones included, so nothing
was altered without their knowledge. If a change is more than mechanical, ask
before sending rather than after.

## Send the message

Use the Slack MCP send tool with the resolved ID as the channel/recipient. The
parameter is `message`, not `text`:

```
mcp__claude_ai_Slack__slack_send_message   channel_id="<user_id>"   message="<text>"
```

Sending is outward-facing and hard to unsend. **Draft the text and confirm the
recipient + message with the operator before sending**, unless they've explicitly
said to send without confirming. For a DM, the recipient is the user_id; for a
channel, resolve it with `slack_search_channels` (this skill only caches people)
and confirm the channel name back to the operator — posting to the wrong channel
is public and permanent.

Return the `message_link` from the response so the operator can jump to it.

## Drafting in the operator's voice

**When the operator supplies the text, send their words** — mechanical pass only.
These rules are for drafting from scratch.

- **Look before you write.** Pull a handful of the operator's recent sent messages
  (`slack_search_public_and_private` with `from:<@THEIR_ID>`) and match what you see:
  greeting or no greeting, bullets or prose, emoji or none, how they open and close.
- **Default to no runway.** Start with the mention and the substance on the same
  line. End on the operative point. Skip "thanks in advance" and "let me know if
  you have questions" unless the operator's own messages use them.
- **State the shape up front** for anything with more than two parts: "2 questions:",
  then bullets.
- **Keep technical specifics exact** — full table names, dates, counts — in backticks.
- **Flag uncertainty plainly** rather than hedging around it.
- **Learn from edits.** When the operator changes your draft before sending, treat
  phrasing, structure, and tone changes as standing preferences for the rest of the
  session. Content corrections (a wrong number, a different channel) are not voice.

## Build or refresh the full directory

Run only when the operator says yes to the freshness offer, or asks for it
("build my Slack directory", "refresh the roster"). Tell them it takes a couple of
minutes of tool calls. There is no list-all-members endpoint on the connector, so
the roster is built by exhausting search + reading profiles:

1. Paginate `slack_search_users query="@marq.com" response_format="concise"` until
   "End of results". The pagination cursor is base64 `CURRENT_PAGE:N`, so pages can
   be fetched in parallel.
2. Probe other email domains to catch guests/contractors: `gmail`, `marqapp.com`,
   and a couple of role words (`Engineer`, `Designer`). Collect any new IDs.
3. For each unique ID, `slack_read_user_profile user_id=<id>` to get the true
   `Real Name`, `Email`, `Title`, and the `Restricted` flag (Restricted=Yes → guest)
   or bot flag. The search results alone only give display names, which are often
   just first names.
4. Rewrite the cache file with the same columns. Preserve any existing `notes`
   values, especially `PREFERRED` markers on duplicate-account people.
5. Rewrite the comment line: today's date, the new account count, `mode: full`.
   This resets the 90-day clock.

The roster is a floor, not a guaranteed-complete list — search-only discovery can
miss anyone matching none of the probes. Cache misses after a full build still
append lazily.
