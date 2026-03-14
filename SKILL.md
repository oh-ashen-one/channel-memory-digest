---
name: channel-memory-digest
description: Reads all active Discord channels every 6 hours and consolidates what happened into a rolling memory digest. Gives *** and *** cross-channel context without loading every memory file at startup.
private: true
---

# Channel Memory Digest

## Purpose
Run every 6 hours. Read the last N messages from every active Discord channel. Write a rolling 7-day digest to disk. Archive everything permanently. Post one confirmation to #announcements.

## Schedule
Cron: `0 8,14,20,2 * * *` (America/Chicago)
Model: `anthropic/claude-haiku-3-5`
Timeout: 300s

## Channels
| Channel ID | Name | Messages to Read |
|---|---|---|
| ***REMOVED*** | #general | 50 |
| ***REMOVED*** | #*** | 50 |
| ***REMOVED*** | #posting | 30 |
| ***REMOVED*** | #*** | 30 |
| ***REMOVED*** | #*** | 30 |
| ***REMOVED*** | #ideas | 30 |
| ***REMOVED*** | #announcements | 30 |
| ***REMOVED*** | #*** | 20 |
| ***REMOVED*** | #*** | 20 |
| ***REMOVED*** | #*** | 20 |

## Steps

### 1. Read channels
For each channel, use the message tool:
```
action=read, channel=discord, target=<channel_id>, limit=<N>
```
Skip any channel with no messages in the last 12 hours.

Extract per channel (3-5 bullets max):
- What was built, decided, or shipped
- What's in progress
- Any open questions or blockers
- Skip small talk and reactions

### 2. Write files

**Rolling file** (7-day window):
`memory/channels/digest.md`
- Prepend new entry at the TOP
- Trim entries older than 7 days from the bottom
- Never grows beyond ~1 week of history

**Archive file** (permanent, never delete):
`memory/channels/digest-archive.md`
- Append every entry here too
- Never trimmed

Entry format:
```
## Digest — <YYYY-MM-DD HH:MM CT>
### #general
- <bullet>
### #***
- <bullet>
[active channels only]
---
```

### 3. Post to #announcements
Use message tool (action=send, target=***REMOVED***):
```
<@***REMOVED***> 🧠 Memory digest — read [total] messages across [X] active channels. Snapshot saved.
```

No other output. No narration. Files + one Discord message only.

## How Memory Is Used
- `digest.md` is read ON-DEMAND — when *** asks about something from another channel
- It is NOT loaded at every session start (would waste tokens)
- HEARTBEAT.md contains the rule: check digest.md before saying "I don't know"
- `digest-archive.md` exists for deep historical lookups only

## Cron Setup (***'s machine)
```
openclaw cron add \
  --name "channel-memory-digest" \
  --cron "0 8,14,20,2 * * *" \
  --tz "America/Chicago" \
  --session isolated \
  --model anthropic/claude-haiku-3-5 \
  --timeout-seconds 300 \
  --announce \
  --channel discord \
  --to "channel:***REMOVED***" \
  --best-effort-deliver \
  --message "<paste cron message from above>"
```

## ***'s Setup
*** runs an identical cron on his machine. Writes to:
- `/Users/YOUR_USERNAME/.openclaw/workspace/memory/channels/digest.md`
- `/Users/YOUR_USERNAME/.openclaw/workspace/memory/channels/digest-archive.md`

Same schedule, same channels, same format. Both agents tag <@***REMOVED***> in #announcements on each run.
