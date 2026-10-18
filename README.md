# Partner Ops: SQL and API testing practice

**Practice guide and Postman collection. Not client work.**

## In one minute

A self-directed project in a domain the author knows from payout operations: partners, e-agreements, payouts and activity logs. The page says five data issues were seeded on purpose (duplicate payouts, payments after an agreement expired, agreements unsigned for 90+ days, activity after suspension) and found with SQL, and that a 10-request Postman collection, including a chained POST then PATCH workflow, was tested against a mock server.

## Links

- Portfolio case study: https://bhawnamanglapm.github.io/travel-tech-more.html#case-partner-ops-practice

## What is in this repo

| File | What it is |
|---|---|
| SQL_GUIDE.md | Query guide, basic to advanced |
| Partner-Ops-API.postman_collection.json | The Postman collection |
| CASE_STUDY.md | The write-up, exported from the portfolio page |

## What is real, modeled or not verified

| Topic | Status |
|---|---|
| Files missing from this repo | SQL_GUIDE.md refers to generate_data.py, schema.sql and partner_ops.db. The Postman work refers to a self-hosted mock server. None of these are in the portfolio's project folder, so the dataset and mock server cannot be rebuilt from this repo. If the originals exist elsewhere, the owner can add them. |
| Version mismatch to confirm | SQL_GUIDE.md says the queries ran on SQLite, while the portfolio card says SQL Server 2022. Which is accurate for the final version is for the owner to confirm. |
| Fictional data | The partners, payouts and issues are invented for the exercise. |

## Provenance and credit

- Product owner, built with AI (Claude).
- Copied on 2026-10-19 from the `projects/partner-ops-practice` folder of [bhawnamanglapm.github.io](https://github.com/bhawnamanglapm/bhawnamanglapm.github.io). The portfolio repo's commit history is not carried over, and the portfolio copy is unchanged.
- The portfolio repo is MIT-licensed. No separate licence file has been added here; that is the owner's decision.
