# Account research desk

## What this is
A sales team's research desk. Input: target accounts and our ideal customer profile (ICP). Output: a verified, scored shortlist.

## Folder map
- inputs/ - accounts.md and ICP.md. Read-only: a hook blocks every edit.
- contracts/ - the exact format of every file agents hand to each other
- accounts/ - one research file per company, written by account-researcher
- shortlist.md - the ranked result, written by account-scorer
- ARCHITECTURE.md - how the system is designed, and why

## Rules
1. Every claim needs an exact quote from a page that was opened, and its link.
2. Agents write only the files their contract names.
3. Unverified claims never reach shortlist.md.
4. Never edit inputs/. Ask me to change them.
