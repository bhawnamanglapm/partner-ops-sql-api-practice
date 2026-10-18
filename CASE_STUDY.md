# Partner Ops — SQL & API Testing Practice

*⭐⭐⭐⭐ Built · self-directed practice, not client work · relational schema, multi-table SQL, REST API test suite*

> Exported from the portfolio page on 2026-10-19. The live page is the canonical version: https://bhawnamanglapm.github.io/travel-tech-more.html#case-partner-ops-practice

- 18 partners, 90 payouts, 160 activity logs — 5 issues seeded on purpose, found by query
- 13 SQL queries, basic → advanced, verified live on SQL Server
- 10-request Postman collection + mock server, incl. a chained POST→PATCH workflow
- Found and fixed 2 real bugs along the way, not just a clean first pass

**Tools used:** SQL Server 2022 Express (T-SQL), VS Code with the SQL Server (mssql) extension, Postman (collection + a self-hosted mock server).

### Why This Project

A self-directed practice project, not client work — built to turn "I know SQL and APIs conceptually" into something demonstrable, in the same operational world I already know from owning PCMS at Bajaj Finserv: partners, e-agreements, payouts, and activity logs. Five real data issues (duplicate payouts, payments made after an agreement had expired, agreements stuck unsigned for 90+ days, activity logged after an account was suspended) were seeded into the raw data on purpose, not pre-labeled — the exercise was writing the SQL and API checks that actually find them.

### Two Real Bugs, Found and Fixed

Nothing here was a clean first pass, which is arguably the more useful part to show. A regional-average SQL query came back with the right leading region but the wrong number — tracing it back showed the payouts data was correct and the problem was 10 of 18 partners carrying the wrong region tag in a dimension table, found with a JOIN against a known-correct reference set and fixed with one set-based `UPDATE ... FROM ... JOIN`. Separately, a Postman mock server was silently returning the wrong saved example for two requests that shared a path (`/payouts`) but differed only by query string — two conditions in the generated server code both matched on path alone, so the first one always fired and the second was dead code. The fix was adding a check on the actual query parameter the two examples were supposed to be distinguished by.

### Key Findings

- **3 duplicate payout pairs** surfaced with a single `GROUP BY ... HAVING COUNT(*) > 1` — the classic signature of a non-idempotent retry or a batch job re-run, not a data-entry typo.
- **4 payouts moved after their governing agreement had already expired**, across 2 partners — the sharpest finding in the dataset, surfaced by a 3-table JOIN comparing payout date against agreement expiry.
- **2 payout requests logged after the partner's account was suspended**, found by joining the activity log to account status — the same "cross-reference what someone did against what their account should allow" pattern, just on behavioral data instead of transactional data.
- **The Postman side mirrors the SQL side exactly** — every example response in the API collection was pulled directly from the same database, so testing "fetch a partner's activity logs" via API returns the identical numbers already verified in SQL.

### What This Means, Applied

The methodology matters more than any single query: start with a structural check (does every payout have a valid, currently-active agreement behind it), then a duplication check, then an activity-consistency check, then consolidate everything into one queryable report instead of three separate spreadsheets. That's a real discipline, not a one-off script — the same instinct behind [the research above](https://bhawnamanglapm.github.io/index.html#research), applied to payments and partner operations instead.

### Walkthrough — Input → Output → Understanding

Six of the real query/request pairs from this project, each with what was sent, what actually came back, and what it means — the same way I'd walk an interviewer through it verbally rather than just showing a result.

**1. Duplicate payouts — Input**

SELECT partner_id, amount, payout_date, COUNT(*) AS cnt FROM payouts GROUP BY partner_id, amount, payout_date HAVING COUNT(*) > 1;

**Output**

| partner_id | amount | payout_date | cnt |
|---|---|---|---|
| 1 | 50935.88 | 2026-03-26 | 2 |
| 1 | 62587.81 | 2025-03-26 | 2 |
| 1 | 13526.91 | 2026-06-23 | 2 |

**Understanding** — 3 pairs, same partner/amount/date, each appearing exactly twice. That's the signature of a retry that wasn't idempotent or a batch job that re-ran — not a typo. A running total or reconciliation report built on this data without deduping first would silently overstate what that partner was actually owed.

**2. Payouts after agreement expiry — Input**

SELECT p.payout_id, pt.partner_name, p.payout_date, e.expiry_date FROM payouts p JOIN partners pt ON pt.partner_id = p.partner_id JOIN e_agreements e ON e.partner_id = p.partner_id WHERE e.status = 'expired' AND p.payout_date > e.expiry_date AND p.status = 'completed';

**Output**

| payout_id | partner_name | payout_date | expiry_date |
|---|---|---|---|
| 177 | Bluepeak Finserv | 2026-04-19 | 2026-02-15 |
| 178 | Bluepeak Finserv | 2026-04-14 | 2026-02-15 |
| 179 | Ganges Retail Finance | 2026-03-19 | 2026-02-15 |
| 180 | Ganges Retail Finance | 2026-04-04 | 2026-02-15 |

**Understanding** — 4 payouts, 2 partners, all moved 1–2 months after the governing agreement had already lapsed. This is the sharpest finding in the dataset: money moved on a contract that wasn't valid anymore. The 3-table JOIN is what makes it visible — payouts alone has no idea an agreement even exists.

**3. Activity after suspension — Input**

SELECT pt.partner_name, a.activity_ts FROM activity_logs a JOIN partners pt ON pt.partner_id = a.partner_id WHERE a.activity_type = 'payout_request' AND pt.status = 'suspended';

**Output**

| partner_name | activity_ts |
|---|---|
| Lakeside Capital | 2026-08-15 10:22:00 |
| Everest Capital Partners | 2026-08-15 10:22:00 |

**Understanding** — 2 partners requested a payout after their account had already been suspended. This is the same instinct as finding #2, applied to behavior instead of money: cross-reference what someone did against what their account status should have allowed.

**4. Fetch a partner's aggregate summary via API — Input**

GET {{baseUrl}}/partners/1/summary

**Output** — 200 OK

{ "partner_id": 1, "partner_name": "Aravali Traders", "status": "active", "total_payouts": 6, "total_amount_paid": 254101.2, "agreement_status": "signed", "flagged_duplicate_payouts": 6 }

**Understanding** — one API call returning the same numbers that took several SQL queries to assemble (6 payouts, ₹254,101.20 total, 6 duplicate-flagged rows — matching finding #1 exactly, since all 3 duplicate pairs happen to belong to partner 1). This is the API equivalent of a JOIN + aggregate: the server does the assembling so the client doesn't have to make 4 separate calls.

**5. Create a discrepancy record — Input**

POST {{baseUrl}}/discrepancies Body: { "partnerId": 1, "issueType": "manual_review", "note": "Flagged during interview practice run" }

**Output** — 201 Created

{ "discrepancy_id": 999, "partnerId": 1, "issueType": "manual_review", "note": "Flagged during interview practice run", "detected_date": "2026-09-09", "resolved": false }

**Understanding** — 201, not 200: this request *created* something new, and the ID it returned (999) gets saved automatically into a collection variable by a test script, ready for the next request to reuse — no copy-pasting an ID by hand.

**6. Resolve it — the chained request — Input**

PATCH {{baseUrl}}/discrepancies/{{createdDiscrepancyId}} Body: { "resolved": true }

**Output** — 200 OK

{ "discrepancy_id": 999, "resolved": true }

**Understanding** — `{{createdDiscrepancyId}}` in the URL was never typed by hand — it's the exact ID request #5 saved a moment earlier. That's a real multi-step workflow (create → confirm → resolve), the same shape as create-an-order → pay-for-it → ship-it in any real transactional API, not three disconnected calls that happen to run one after another.

### Quick recall

| Section | Summary |
|---|---|
| **Problem** | Turn "I know SQL and APIs conceptually" into something demonstrable, in the same operational world already known from owning PCMS at Bajaj Finserv — partners, e-agreements, payouts, activity logs. |
| **Research steps** | Seeded 5 real data issues into raw data on purpose, unlabeled (duplicate payouts, post-expiry payments, stuck-unsigned agreements, activity after suspension) → wrote 13 SQL queries from basic to advanced to find them, not pre-knowing which query would surface which issue → built a 10-request Postman collection mirroring the same database, including a chained POST→PATCH workflow → found and fixed 2 real bugs along the way: a wrong region tag in a dimension table (traced via JOIN against a known-correct reference), and a Postman mock server silently returning the wrong example for two requests sharing a path. |
| **Solution** | 13 verified SQL queries plus a 10-request Postman collection against a self-hosted mock server, with every API example response traceable back to the same database the SQL queries already verified. |
| **Conclusion** | The methodology matters more than any single query: structural check → duplication check → activity-consistency check → consolidate into one queryable report instead of three separate spreadsheets. The sharpest single finding: 4 payouts moved after their governing agreement had already expired — invisible without a 3-table JOIN. |

Guide[SQL practice guide →](https://bhawnamanglapm.github.io/projects/partner-ops-practice/SQL_GUIDE.md) Collection[Postman collection →](https://bhawnamanglapm.github.io/projects/partner-ops-practice/Partner-Ops-API.postman_collection.json)
