# Architecture: account research desk

Three accounts (Lenskart, boAt, Licious), scored against `inputs/ICP.md`. Four criteria, maximum score 30.
Score = Size x2 + Support load x3 + Trigger x3 + Support pain x2, each criterion 0 to 3.

## 1. Agents
- **account-researcher** (one per account). Job: find evidence for the 4 criteria and write `accounts/<name>.md`. Tools: WebSearch, WebFetch, Write. Model: sonnet.
- **claim-verifier-A** and **claim-verifier-B** (two per account). Job: re-open each link and confirm the quote is really there. Tools: WebFetch, Read, Write. A is sonnet; B is a different model (haiku or opus) so their mistakes are less likely to match.
- **account-scorer** (one). Job: score and rank using only confirmed claims, and write `shortlist.md`. Tools: Read, Write. No web tools. Model: sonnet.
- The main session orchestrates. It spawns the agents and does no research itself.
- **Verifier input:** only the claim, the link and the quote. No researcher reasoning, and never the other verifier's file.
- **Verifier task:** copy the passage containing the quote from the fetched page, then judge. The default verdict is FAIL. PASS needs the exact string on the page, a page that supports the claim as worded, and a date inside the ICP window for Trigger claims (last 6 months).
- **Canary check:** the orchestrator adds one planted false claim per account, with a made-up quote. A verifier that passes it has its whole verdict file discarded and is rerun.

## 2. Handovers
- `accounts/<name>.md`: researcher to both verifiers and the scorer. Format in `contracts/account-file.md`.
- `accounts/<name>.verdict-a.md` and `accounts/<name>.verdict-b.md`: each verifier writes only its own. Needs `contracts/verdict-file.md`.
- `accounts/<name>.status.md`: one line per account, written by the orchestrator: `ok`, `research-failed` or `unverified`. Needs a small contract.
- `shortlist.md`: scorer to the user. Needs `contracts/shortlist-file.md`: rank, company, score out of 30, per-criterion score with link, dropped claims, and Not found gaps.
- `inputs/ICP.md` and `inputs/accounts.md` are read-only for everyone.

## 3. Checks
- **Two-verifier rule:** a claim counts only if both verifiers PASS it. A disagreement is treated as FAIL and listed as "disputed" for the user to look at.
- **A hook, not an agent, does the merge.** A script compares the two verdict files and writes the list of confirmed claims. The scorer reads that list and cannot override it.
- A validator hook checks every researcher file against the contract: valid criteria, no empty Evidence or Link cells, a quote from a page that was opened.
- The scorer counts only confirmed claims. A criterion with no confirmed claim scores 0. The total must equal the weighted sum.
- A hook on `shortlist.md` blocks any cited link that is not in a confirmed claim.
- The user spot-checks the top-ranked account's links.

## 4. Order
- Step 1, parallel: 3 researchers.
- Step 2, per account as each researcher finishes: validator hook, then verifier A and verifier B in parallel.
- Step 3: the merge script.
- Step 4, waits until all 3 accounts are resolved: the scorer.
- Step 5: the user reviews the shortlist.

## 5. Failure handling
- A researcher failure means an error, a timeout, no file, or a file that fails the validator.
- The orchestrator retries once, passing the validator's error message.
- If the retry fails, the status becomes `research-failed`. The account skips verification and the other accounts carry on.
- The scorer does not score a failed account. `shortlist.md` shows "Not ranked: research failed", never a 0, because a 0 would rank it as a weak fit when the fit is simply unknown.
- A verifier failure gets the same single retry. If it still fails, the status is `unverified` and no claim from that account counts.
- Every failure is reported to the user at the end of the run, with the reason.

## 6. Never
- **Editing `inputs/`.** Enforced by a PreToolUse hook that blocks Write, Edit and Bash writes to that path.
- **A verifier reading the other's file.** Enforced by a permission denying each verifier read access to the other's path. The orchestrator also never passes one verdict to the other.
- **A claim counting on one verifier's word.** Enforced by the merge script.
- **Writing outside your contract.** Enforced by path-scoped permissions.
- **An unconfirmed claim in `shortlist.md`.** Enforced by the hook on that file, backed by the scorer's prompt rule.
- **A claim without an exact quote and link.** Enforced by the validator hook and the verifiers' FAIL default.
- **Invented or paraphrased quotes.** Enforced by the verifiers' fresh fetch and the canary check.
- **The scorer doing new research.** Enforced by permissions: no web tools.
- **Agents running git commits or shell commands.** Enforced by permissions: Bash is not granted.
- **A failed account being silently dropped or scored 0.** Enforced by the status file and the scorer's contract.

## Decisions
