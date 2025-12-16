# AdvHunt-001 — User-reported phish: blast-radius + mailbox targeting (Sentinel/Defender XDR)

## What this query does
When a user reports an email as phishing/malware in Outlook, this query correlates the alert to the underlying message and then **expands scope to everyone who received the same email**, even if they did **not** report it.

The result is a **message-level view** that shows:
- who reported it (reporters)
- who else received it (all recipients)
- how many mailboxes are impacted (recipient count)

This supports rapid triage and **mailbox remediation** (helpdesk knows exactly which mailboxes to pull the message from).

## Why it matters operationally
User reports are a strong early signal, but they only represent one mailbox. This hunt quickly answers:
- Is this a single-user event or a broader campaign?
- Which specific mailboxes need the email removed?
- What identifiers should we pivot on (NetworkMessageId / ReportId / AlertIds)?

## Data sources
- `AlertInfo` — identifies the “Email reported by user as malware or phish” alert
- `AlertEvidence` (EntityType = `MailMessage`) — extracts message identifiers + sender/subject and the reporting recipient
- `EmailEvents` — enumerates **all recipients** for the same `NetworkMessageId`

## Output (one row per message)
- **NetworkMessageId** (primary correlation key)
- **Sender** / **EmailSubject**
- **Reporter** (set of users who reported it)
- **AllRecipients** (set of all mailboxes that received it)
- **TotalRecipients** (count of distinct recipients)
- **ReportId** and **AlertIds** (pivot points back into Defender/Sentinel)

## How I use the results
1. Confirm the message is a true phish/malware (subject/sender context + pivots as needed).
2. Use **AllRecipients** as the authoritative mailbox list for cleanup/remediation.
3. Use **TotalRecipients** to size the blast radius and prioritize response.
4. Pivot via **ReportId / AlertIds / NetworkMessageId** for deeper investigation if needed.

## Notes / assumptions
- Default lookback is the last 1 hour (adjust for ingestion timing and queue volume).
- This repo contains only sanitized content (no tenant identifiers or real user/mailbox names).
