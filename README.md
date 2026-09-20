# DevQuest 2026 Final: ClearHouse

## What is ClearHouse?

A **clearing house** sits between the parties of a financial market. When two traders agree a deal, the clearing house records who now owes what to whom, makes sure both sides can actually pay, and settles the deal so that no one is left exposed if the other side fails. Exchanges, brokers and payment networks all depend on one. ClearHouse is a small, complete one: an HTTP API, a database and a browser dashboard.

The platform brings together the pieces a real one needs:

- **Accounts and assets.** Participants hold balances in several currencies and instruments (USD, EUR, JPY, BHD, BTC). Each asset has its own precision, so money is always an exact integer count of minor units, never a floating-point number.
- **A double-entry ledger.** Every movement of value is recorded as balanced debits and credits, so the books always add up and every balance can be explained.
- **An order book and matching engine.** Buyers and sellers place orders; the engine matches them by price and time, and supports limit, market and other order types.
- **Risk controls.** Limits on order size, open orders and position keep one participant from putting the whole market at risk.
- **Clearing and settlement.** Trades are netted down to the fewest obligations, fees are charged, and funds are held and released so a deal either completes fully or not at all.
- **Operations and security.** Signed requests, roles, idempotent retries, audit trails, metrics, reconciliation and safe data migration.
- **A live dashboard.** A browser client shows balances, the order book, risk usage and recent trades, updated in real time over WebSockets, with the whole API documented in Swagger.

## How this challenge works

You start from a working-looking codebase (TypeScript, Express, Knex, SQLite, Vitest). Much of the business logic is missing and throws `NotImplementedError`, and some of the code that is there contains defects. Your job is to implement the missing parts and fix the defects so that the tests in `tests/` pass. **The tests are the specification**: every assertion is committed in your repository, so read them, run them, and let them tell you what is expected.

Each challenge below is one area of the platform. They build on each other, so start with the foundations.

**Total available: 3195 points** across 21 challenges, plus the **Bug Fixes** category below.

## Challenge overview

| # | Challenge | Category | Points | Test file |
| --- | --- | --- | --- | --- |
| 01 | Money, Assets, Authentication and Authorization (gate) | Foundations, Security and Money | 120 | `tests/challenge01.test.ts` |
| 02 | The Double-Entry Ledger | Data Integrity | 150 | `tests/challenge02.test.ts` |
| 03 | The Matching Engine | Core Algorithms (Matching Engine) | 250 | `tests/challenge03.test.ts` |
| 04 | Settlement, Holds and Atomicity | Transactions and Concurrency | 200 | `tests/challenge04.test.ts` |
| 05 | Pre-Trade Risk and Limits | Business Rules and Limits | 140 | `tests/challenge05.test.ts` |
| 06 | Event Sourcing and Deterministic Replay | Event Sourcing and Determinism | 100 | `tests/challenge06.test.ts` |
| 07 | The Adversarial Gauntlet | Advanced Security | 200 | `tests/challenge07.test.ts` |
| 08 | Market Data and Time | Data Processing and Time | 80 | `tests/challenge08.test.ts` |
| 09 | Reconciliation, Reporting and Migration | Reconciliation and Migration | 100 | `tests/challenge09.test.ts` |
| 10 | Observability and Operations | Observability and Operations | 100 | `tests/challenge10.test.ts` |
| 11 | API Design and Performance | API Design and Performance | 90 | `tests/challenge11.test.ts` |
| 12 | Operator Dashboard | Frontend (Dashboard) | 130 | `tests/challenge12.test.ts` |
| 13 | The Extension Round: Account Closure | Requirement Change | 100 | `tests/challenge13.test.ts` |
| 14 | Netting and Multilateral Settlement | Algorithms (Netting) | 200 | `tests/challenge14.test.ts` |
| 15 | Complex Order Types | State Machines (Order Types) | 200 | `tests/challenge15.test.ts` |
| 16 | Fee and Rebate Engine | Financial Arithmetic (Fees) | 200 | `tests/challenge16.test.ts` |
| 17 | Live Dashboard Updates | Frontend (Real-time) | 150 | `tests/challenge17.test.ts` |
| 18 | The WebSocket Feed | Real-time (WebSockets) | 190 | `tests/challenge18.test.ts` |
| 19 | API Documentation (Swagger) | API Documentation (OpenAPI / Swagger) | 185 | `tests/challenge19.test.ts` |
| 20 | Dashboard and Demo Data | Frontend and Demo Data | 160 | `tests/challenge20.test.ts` |
| 21 | Integration | Integration | 150 | `tests/challenge21.test.ts` |

---

## 🐞 Bug Fixes — the Challenge 00 bug hunt — 922 points across 45 tests

**What a bug is.** The template ships with working-looking code that contains defects: wrong constants,
inverted conditions, missing validation, a migration that misbehaves, a page that does not match its script.
Each bug has a test that fails on the untouched template and passes once that one defect is fixed. Nothing
in the bug tests asks you to build a new feature. Finding the defect is the challenge; some produce
symptoms in other places before you find the real cause.

**Bug tests.** `tests/challenge00.test.ts` (the first six), `tests/challenge00b.test.ts` and `tests/challenge00c.test.ts`, plus the shipped-defect tests
inside `tests/challenge07.test.ts`, `challenge09.test.ts` and `challenge11.test.ts`.

The individual bug tests are in those test files.

---

## 🎯 Challenge Requirements

### 🔴 Challenge 01: Money, Assets, Authentication and Authorization (gate) — 120 points
**Category**: Foundations, Security and Money  
**Test File**: `tests/challenge01.test.ts`  
**The Problem**: Before anything can be traded it must be counted correctly and requested authentically. This challenge is the foundation every other challenge stands on.
**Tasks to Complete**:
- **1a: Asset registry and strict amount parsing**
  - **Task 1a-1** (6 pts): known assets round-trip through the minor-unit parser
  - **Task 1a-2** (9 pts): malformed amounts are rejected with 400, never silently coerced
  - **Task 1a-3** (5 pts): amounts supplied as JSON numbers are rejected, not coerced to strings
  - **Task 1a-4** (7 pts): decimal precision cannot exceed the asset's own exponent
  - **Task 1a-5** (6 pts): property — parse and serialise round-trip for every asset and every valid integer
  - **Task 1a-6** (6 pts): property — add/sub are inverses and asset-mismatch always throws
  - **Task 1a-7** (6 pts): property — total order is antisymmetric and transitive
- **1b: Rounding conserves value**
  - **Task 1b-1** (5 pts): half-even and half-up only disagree on an exact midpoint
  - **Task 1b-2** (6 pts): property — quotient * divisor + remainder reconstructs the dividend exactly
  - **Task 1b-3** (2 pts): division by zero is rejected, never produces Infinity or NaN
- **1c: HMAC request signing**
  - **Task 1c-1** (2 pts): a correctly signed request is accepted
  - **Task 1c-2** (2 pts): a missing signature is rejected
  - **Task 1c-3** (2 pts): a signature computed over a different body is rejected
  - **Task 1c-4** (2 pts): a timestamp outside the signing window is rejected
  - **Task 1c-5** (2 pts): a replayed nonce is rejected on the second use
  - **Task 1c-6** (2 pts): an unrecognised algorithm is rejected even with an otherwise valid signature
- **1d: Authorization**
  - **Task 1d-1** (8 pts): an operator token is rejected on an admin-only route
  - **Task 1d-2** (6 pts): an admin-only route rejects a request with no token at all
  - **Task 1d-3** (6 pts): an admin token is accepted on an admin-only route
  - **Task 1d-4** (6 pts): a token acting on its own account's ledger balance succeeds
  - **Task 1d-5** (11 pts): a valid token cannot read a different account's ledger balance
  - **Task 1d-6** (6 pts): an anonymous request (no token at all) to a balance route is unaffected by tenant isolation
  - **Task 1d-7** (7 pts): refreshing rotates the token, and the old refresh token is rejected on reuse

### 🟠 Challenge 02: The Double-Entry Ledger — 150 points
**Category**: Data Integrity  
**Test File**: `tests/challenge02.test.ts`  
**The Problem**: Every movement of value in the system is recorded as a balanced journal entry in an append-only ledger. Balances are never stored as mutable running totals that code can drift out of sync; they are derived.
**Tasks to Complete**:
- **2a: Balance rule**
  - **Task 2a-1** (12 pts): a balanced two-account entry is accepted
  - **Task 2a-2** (12 pts): an entry that does not sum to zero for its asset is rejected, and nothing is written
  - **Task 2a-3** (12 pts): an entry mixing two assets must balance each one independently
  - **Task 2a-4** (15 pts): property — a random balanced posting set is always accepted, and derived balances match the reference sum
- **2b: Append-only and reversal**
  - **Task 2b-1** (12 pts): reversing an entry restores the pre-entry balance exactly
  - **Task 2b-2** (12 pts): the trial balance across all accounts sums to zero for every asset touched
- **2c: Point-in-time balances and statement pagination**
  - **Task 2c-1** (12 pts): an as-of balance excludes entries posted after the cutoff entry
  - **Task 2c-2** (13 pts): concatenating every statement page yields each posting exactly once

### 🟡 Challenge 03: The Matching Engine — 250 points
**Category**: Core Algorithms (Matching Engine)  
**Test File**: `tests/challenge03.test.ts`  
**The Problem**: This is the heart of the system and the largest single pot of points in the competition. A limit order book with strict price-time priority.
**Tasks to Complete**:
- **3a: Price-time priority**
  - **Task 3a-1** (15 pts): a crossing order trades at the resting order's price, not its own
  - **Task 3a-2** (15 pts): within one price level, the earlier order fills first
  - **Task 3a-3** (18 pts): a large aggressor sweeps multiple price levels in ascending price order
- **3b: Order semantics**
  - **Task 3b-1** (15 pts): IOC fills what it can and never rests
  - **Task 3b-2** (18 pts): FOK rejects entirely, leaving the book untouched, when it cannot fill in full
  - **Task 3b-3** (15 pts): post-only is rejected rather than matched when it would cross on entry
  - **Task 3b-4** (18 pts): self-trade prevention cancels the resting order and the aggressor continues against the rest of the book
- **3c: Invariants**
  - **Task 3c-1** (20 pts): the book is never crossed after any sequence of orders
  - **Task 3c-2** (12 pts): total fills against an order never exceed its quantity
- **3d: Cancellation**
  - **Task 3d-1** (9 pts): cancelling a resting order removes exactly it, and a second cancel reports not found
- **3e: Stop and stop-limit orders**
  - **Task 3e-1** (8 pts): a stop order triggers as a market order once the last trade price reaches its stop price
  - **Task 3e-2** (6 pts): a stop-limit order triggers as a limit order at its own price, not a market order
  - **Task 3e-3** (6 pts): multiple stops triggered by the same trade fire in original submission order
- **3f: Amendment**
  - **Task 3f-1** (3 pts): decreasing quantity preserves queue position
  - **Task 3f-2** (4 pts): a price change loses queue position, moving the order to the back of its new level
  - **Task 3f-3** (4 pts): a quantity increase loses queue position
  - **Task 3f-4** (3 pts): amending a non-existent or already-filled order returns 404
  - **Task 3f-5** (2 pts): amending to a non-positive quantity is rejected
  - **Task 3f-6** (4 pts): decreasing the quantity of an order that is NOT at the front keeps its exact place in the queue
- **3g: Concurrency**
  - **Task 3g-1** (13 pts): a cancel racing an aggressing fill resolves to exactly one consistent outcome
  - **Task 3g-2** (12 pts): an amend racing an aggressing fill never double-counts quantity
- **3h: Determinism**
  - **Task 3h-1** (15 pts): replaying the same 10,000-operation sequence twice produces identical trades and an identical final book
- **3i: Performance**
  - **Task 3i-1** (15 pts): per-order insert cost does not grow linearly with book depth

### 🔵 Challenge 04: Settlement, Holds and Atomicity — 200 points
**Category**: Transactions and Concurrency  
**Test File**: `tests/challenge04.test.ts`  
**The Problem**: A trade that cannot settle must never have existed. Matching and settlement are one atomic act.
**Tasks to Complete**:
- **4a: Holds**
  - **Task 4a-1** (12 pts): a deposit increases available, a hold moves it to held without touching the total
  - **Task 4a-2** (12 pts): a hold exceeding available is rejected, and the balance is unchanged
  - **Task 4a-3** (14 pts): releasing exactly what was held restores available, and cannot release more than is held
  - **Task 4a-4** (22 pts): 100-way concurrent holds against 40 available succeed exactly 40 times, and the final split is exact
- **4b: Idempotency**
  - **Task 4b-1** (14 pts): replaying a deposit with the same Idempotency-Key applies it exactly once
  - **Task 4b-2** (12 pts): the same Idempotency-Key with a different body is a conflict, not a silent second effect
  - **Task 4b-3** (14 pts): withdrawal is idempotent and can only draw on available, never held
- **4c: Atomic trade settlement**
  - **Task 4c-1** (18 pts): settling a trade moves both legs atomically, and posts a balanced ledger entry
  - **Task 4c-2** (12 pts): a trade that would overdraw a hold is rejected and leaves every balance untouched

### 🟢 Challenge 05: Pre-Trade Risk and Limits — 140 points
**Category**: Business Rules and Limits  
**Test File**: `tests/challenge05.test.ts`  
**The Problem**: Risk checks that run after the trade are not risk checks.
**Tasks to Complete**:
- **5a: Individual rules**
  - **Task 5a-1** (14 pts): an order whose notional exceeds the account's limit is rejected before it ever reaches the book
  - **Task 5a-2** (14 pts): an account cannot have more resting orders open than its limit
  - **Task 5a-3** (14 pts): committed exposure beyond the position limit is rejected
  - **Task 5a-4** (12 pts): cancelling a resting order releases both its open-order slot and its exposure
- **5b: Deterministic precedence**
  - **Task 5b-1** (12 pts): when an order violates both the notional and the open-order limit, the notional violation is reported
- **5c: Concurrent joint-limit enforcement**
  - **Task 5c-1** (16 pts): of two orders each individually within the position limit but jointly over it, submitted in parallel, exactly one is accepted
- **5d: Kill switch**
  - **Task 5d-1** (8 pts): once engaged, new orders are rejected but cancellation still works

### 🟣 Challenge 06: Event Sourcing and Deterministic Replay — 100 points
**Category**: Event Sourcing and Determinism  
**Test File**: `tests/challenge06.test.ts`  
**The Problem**: The platform must be able to prove how it arrived at its current state, and to reconstruct that state from nothing but its event log.
**Tasks to Complete**:
- **6a: Chain integrity**
  - **Task 6a-1** (12 pts): every appended event chains its hash to its predecessor's
  - **Task 6a-2** (16 pts): tampering with a historical event's amount is detected, and the first broken link is reported
- **6b: Rebuild fidelity**
  - **Task 6b-1** (13 pts): rebuilding from the full event log matches the live projection after a randomised sequence of deposits
  - **Task 6b-2** (13 pts): snapshot plus tail replay equals a full replay from the log
  - **Task 6b-3** (11 pts): state-at-sequence for an arbitrary earlier point excludes later events
- **6c: Idempotent and order-tolerant replay (pure reducer)**
  - **Task 6c-1** (15 pts): property — replaying a randomly duplicated and shuffled event list yields the same balances as the deduplicated original
- **6d: Sequencing**
  - **Task 6d-1** (9 pts): event sequence numbers are strictly increasing with no gaps

### ⚡ Challenge 07: The Adversarial Gauntlet — 200 points
**Category**: Advanced Security  
**Test File**: `tests/challenge07.test.ts`  
**The Problem**: A suite of exploits runs against your platform. You can read the attack code, but each attack is **generated, not fixed**: payloads, nesting depths, amounts, orderings and target accounts are drawn from generators seeded fresh on every run, and each category fires a randomly chosen subset from a large pool of variants. Blocking the specific payload you read is worth nothing; blocking the class is worth points. Scoring is by category fully blocked, and a category counts only if every variant drawn in that run is blocked.
**Tasks to Complete**:
- **7d: Payload abuse**
  - **Task 7d-1** (9 pts): an oversized body is rejected with 413, not a hang or a 500
  - **Task 7d-2** (9 pts): a deeply nested body is rejected cleanly, not with a crash
- **7e: Security headers and disclosure**
  - **Task 7e-1** (8 pts): responses carry baseline security headers and no framework fingerprint
- **7a: Injection**
  - **Task 7a-1** (9 pts): SQL metacharacters through every string field are treated as literal data, never executed
  - **Task 7a-2** (9 pts): SQL metacharacters through a route parameter do not error and do not match a real account
- **7b: Mass assignment**
  - **Task 7b-1** (9 pts): unexpected privileged fields in a signup-shaped body are ignored, not applied
- **7c: Prototype pollution**
  - **Task 7c-1** (9 pts): a __proto__ key in the request body never reaches Object.prototype
- **7e: Security headers and disclosure**
  - **Task 7e-2** (8 pts): an internal error never leaks a stack trace or a file path in the response body

### 🌟 Challenge 08: Market Data and Time — 80 points
**Category**: Data Processing and Time  
**Test File**: `tests/challenge08.test.ts`  
**The Problem**: Candles, rollups and statements are all questions about time, and time is where generated code fails most reliably.
**Tasks to Complete**:
- **8a: OHLCV correctness**
  - **Task 8a-1** (10 pts): open and close are taken by arrival sequence, not by timestamp collision order
  - **Task 8a-2** (4 pts): property — high is always the maximum trade price and low the minimum, for any trade set in one bucket
  - **Task 8a-3** (6 pts): property — on randomised trades spanning many buckets, candles at 1m, 5m, 1h and 1d match an independent oracle
- **8b: Hierarchical consistency**
  - **Task 8b-1** (12 pts): rolling up every 1-minute candle in an hour reproduces the 1-hour candle's volume, high and low
- **8c: Empty buckets**
  - **Task 8c-1** (10 pts): a gap between trades is filled with the previous close, never a fabricated zero and never omitted
- **8d: VWAP**
  - **Task 8d-1** (10 pts): VWAP is exact, with the discarded remainder reported rather than dropped
  - **Task 8d-2** (7 pts): VWAP over an empty trade set is zero, not NaN or an error
- **8e: HTTP surface**
  - **Task 8e-1** (6 pts): the candles endpoint aggregates and serialises correctly end to end
- **8f: Late-arriving trades**
  - **Task 8f-1** (15 pts): a trade arriving late with an earlier timestamp still rebuckets correctly at every resolution

### 🔴 Challenge 09: Reconciliation, Reporting and Migration — 100 points
**Category**: Reconciliation and Migration  
**Test File**: `tests/challenge09.test.ts`  
**The Problem**: Close the books, then inherit someone else's.
**Tasks to Complete**:
- **9b: Legacy data migration**
  - **Task 9b-1** (14 pts): the normalisation backfill is idempotent and non-destructive to the original column
  - **Task 9b-2** (9 pts): rolling back removes the derived column and leaves every original value exactly as it was
- **9a: The trial balance proves the ledger**
  - **Task 9a-1** (14 pts): the trial balance sums to zero per asset after a randomised sequence of balanced entries
  - **Task 9a-2** (8 pts): an empty ledger reconciles trivially — no accounts, no imbalance

### 🟠 Challenge 10: Observability and Operations — 100 points
**Category**: Observability and Operations  
**Test File**: `tests/challenge10.test.ts`  
**The Problem**: A platform nobody can see into is a platform nobody can operate. This challenge is about making the system's own health and behaviour legible from the outside, without reading its source.
**Tasks to Complete**:
- **10a: Health and readiness**
  - **Task 10a-1** (10 pts): /health always answers 200 while the process is running
  - **Task 10a-2** (8 pts): /ready answers 200 when the database is reachable
  - **Task 10a-3** (7 pts): /ready answers 503 when the database is unreachable
- **10b: Structured logging with no secrets**
  - **Task 10b-1** (25 pts): a request never causes a password, token or full body to reach stdout
  - **Task 10b-2** (20 pts): a request produces at least one structured log line with the expected fields
- **10c: Metrics**
  - **Task 10c-1** (15 pts): /api/metrics reflects real traffic already served
- **10d: Rate limiting**
  - **Task 10d-1** (10 pts): exceeding the login rate limit returns 429 with Retry-After
  - **Task 10d-2** (5 pts): after the window resets, a request that was previously rate-limited succeeds again

### 🟡 Challenge 11: API Design and Performance — 90 points
**Category**: API Design and Performance  
**Test File**: `tests/challenge11.test.ts`  
**The Problem**: Every challenge so far has its own endpoints. This challenge asks whether they add up to one coherent API, or ten unrelated ones that happen to share a hostname.
**Tasks to Complete**:
- **11a: Response envelope consistency**
  - **Task 11a-2** (13 pts): a representative error response from every route family matches the { error: { code, details } } envelope
- **11c: API versioning**
  - **Task 11c-1** (10 pts): the same route answers identically under /api and /api/v1
  - **Task 11c-2** (10 pts): an unknown version segment returns a clean 404, not a silent fallback
- **11d: Latency budget**
  - **Task 11d-1** (10 pts): a representative read-only route stays within a documented latency budget under repeated load
- **11a: Response envelope consistency**
  - **Task 11a-1** (12 pts): a representative success response from every route family matches the { data, meta } envelope
- **11b: Cache correctness**
  - **Task 11b-1** (18 pts): a first read is a cache miss and a second identical read is a cache hit
  - **Task 11b-2** (17 pts): a write invalidates the cache — the very next read reflects it, not the stale value

### 🔵 Challenge 12: Operator Dashboard — 130 points
**Category**: Frontend (Dashboard)  
**Test File**: `tests/challenge12.test.ts`  
**The Problem**: Everything so far is an API. This challenge asks for the thinnest possible client that makes the system usable by a human: an operator dashboard showing account balances, the order book, and risk state for one account.
**Tasks to Complete**:
- **12a: Rendering real data**
  - **Task 12a-1** (10 pts): the balance view renders one row per asset with available, held and total
  - **Task 12a-2** (10 pts): the order book view highlights the best bid and best ask
  - **Task 12a-3** (10 pts): the risk view shows open order count and committed exposure
- **12b: Untrusted data never executes**
  - **Task 12b-1** (20 pts): an asset name containing markup renders as literal text, not as an element
- **12c: Distinguishable loading, empty and error states**
  - **Task 12c-1** (2 pts): an empty balance list renders a distinct empty state, not an empty table
  - **Task 12c-2** (2 pts): rendered data marks the section ready, distinct from empty or loading
  - **Task 12c-3** (2 pts): a loading state is visibly distinct from empty, error and ready
  - **Task 12c-4** (2 pts): an error state is visibly distinct from empty, loading and ready
  - **Task 12c-5** (2 pts): an empty order book renders a distinct empty state, not empty columns
- **12e: Exact money formatting**
  - **Task 12e-1** (12 pts): formatMinorUnits matches an exact BigInt oracle for any amount, sign, leading zeros and exponent
  - **Task 12e-2** (8 pts): small fractions, zero and exponent 0 format correctly, and malformed input throws RangeError
- **12f: Order book presentation**
  - **Task 12f-1** (12 pts): each side renders best price first whatever order the API returns, with only the first row marked best
  - **Task 12f-2** (8 pts): a deep book is capped at ten levels with a '+N more' row, and malformed prices never break the render
- **12g: Accessibility**
  - **Task 12g-1** (10 pts): loading sets aria-busy, errors announce themselves with role=alert, and tables have a caption and column headers

### 🟢 Challenge 13: The Extension Round: Account Closure — 100 points
**Category**: Requirement Change  
**Test File**: `tests/challenge13.test.ts`  
**The Problem**: The specification changes. It always does — and this one is published up front and part of `tests/challenge13.test.ts` from day one. There is no held-back drop for this run of the event; account closure is simply the last requirement.
**Tasks to Complete**:
- **13a: Closing requires a zero balance**
  - **Task 13a-1** (5 pts): an account with a nonzero available balance cannot be closed
  - **Task 13a-2** (5 pts): an account with a nonzero held balance cannot be closed either
  - **Task 13a-3** (6 pts): an account with every balance at exactly zero closes successfully
  - **Task 13a-4** (4 pts): a freshly opened account with no balance activity at all closes successfully
- **13b: A closed account rejects every balance-moving operation**
  - **Task 13b-1** (6 pts): deposit, withdrawal and hold are all rejected once closed, and no balance moves
  - **Task 13b-2** (6 pts): a trade touching any one of the four accounts is rejected atomically, and none of the four balances move
- **13c: A closed account cannot place new orders**
  - **Task 13c-1** (5 pts): a registered account that has been closed is rejected when it tries to place an order
  - **Task 13c-2** (3 pts): an unregistered account id — never created through the ledger — is unaffected by closure semantics and still trades normally
- **13d: Challenges 01–12 are unaffected by the amendment**
  - **Task 13d-1** (30 pts): a balanced ledger entry between two open accounts still posts, and the trial balance still proves zero
  - **Task 13d-2** (30 pts): a hold-then-release cycle on a still-open account is unaffected

### 🟣 Challenge 14: Netting and Multilateral Settlement — 200 points
**Category**: Algorithms (Netting)  
**Test File**: `tests/challenge14.test.ts`  
**The Problem**: At the end of a trading window the clearing house holds a pile of bilateral obligations ("A owes B 40 USD", "B owes C 25 USD", ...). Paying each one physically is expensive and risky. Multilateral netting replaces them with the fewest possible payments that leave every account exactly where the full pile would have left it.
**Tasks to Complete**:
- **14a: Net positions are preserved**
  - **Task 14a-1** (15 pts): applying the transfers reproduces every account's net position exactly, per asset
  - **Task 14a-2** (15 pts): transfers are well formed, and only net debtors pay and only net creditors receive
- **14b: Cycles cancel**
  - **Task 14b-1** (25 pts): a cycle of equal obligations of any length contributes no transfer at all
- **14c: The transfer count is the minimum**
  - **Task 14c-1** (60 pts): the number of transfers equals the brute-force minimum, on batches with planted zero-sum groups
- **14d: Assets and ordering**
  - **Task 14d-1** (20 pts): assets are netted independently — the combined result is the per-asset results together
  - **Task 14d-2** (20 pts): the result is identical for any input order and for any splitting of an obligation
- **14e: Scale**
  - **Task 14e-1** (35 pts): a large batch with thousands of planted pairs and triples is netted correctly within the time budget
- **14f: Input validation**
  - **Task 14f-1** (10 pts): invalid obligations throw RangeError, and an empty batch nets to nothing

### ⚡ Challenge 15: Complex Order Types — 200 points
**Category**: State Machines (Order Types)  
**Test File**: `tests/challenge15.test.ts`  
**The Problem**: Traders do not only send plain limit orders. They send *strategies*: protect this position with a take-profit and a stop (OCO), enter and exit in one instruction (bracket), show only a small slice of a big order (iceberg), or let a stop follow the market (trailing stop). Each one is a small state machine that turns market events into child orders.
**Tasks to Complete**:
- **15a: OCO (one cancels the other)**
  - **Task 15a-1** (25 pts): a triggered stop cancels the limit and sells exactly what is still unfilled, for any fill and price history
  - **Task 15a-2** (10 pts): a partially filled take-profit leaves a smaller stop, and the stop fires only once
- **15b: Bracket orders**
  - **Task 15b-1** (45 pts): exits always cover exactly the cumulative entry fill, and a stop-loss also cancels the unfilled entry
- **15c: Iceberg orders**
  - **Task 15c-1** (40 pts): only one clip is live, clips replenish only when fully filled, never exceed the total, and keep the original sequence
- **15d: Trailing stops**
  - **Task 15d-1** (35 pts): the stop only ever ratchets in the favourable direction and fires exactly once, matching an independent oracle
- **15e: Redelivery, determinism and cancellation**
  - **Task 15e-1** (20 pts): a redelivered fill changes nothing, and the same event history always yields the same actions
  - **Task 15e-2** (15 pts): cancelling a parent cancels its live limit children, disarms its stops, stops replenishing, and is idempotent
- **15f: Input validation**
  - **Task 15f-1** (10 pts): impossible orders and impossible fills throw RangeError

### 🌟 Challenge 16: Fee and Rebate Engine — 200 points
**Category**: Financial Arithmetic (Fees)  
**Test File**: `tests/challenge16.test.ts`  
**The Problem**: The clearing house earns its living from fees. Makers (who add liquidity) and takers (who remove it) pay different rates, the rate falls as an account's trailing trading volume grows, and the top tier even pays makers a rebate. Every fee must be a first-class ledger entry, exact to the minor unit.
**Tasks to Complete**:
- **16a: Exact fee arithmetic**
  - **Task 16a-1** (30 pts): fees match an exact integer oracle for any notional, including those beyond 2^53, and rebates mirror fees
  - **Task 16a-2** (25 pts): an exact half rounds to the even integer, in both directions and for rebates
- **16b: Volume tiers**
  - **Task 16b-1** (45 pts): each side's tier comes from its own trailing volume before the fill, matching an independent oracle
  - **Task 16b-2** (15 pts): a fill exactly one window old has expired from the trailing volume, one millisecond younger has not
- **16c: Fees are posted as balanced ledger entries**
  - **Task 16c-1** (30 pts): every entry balances, fees and rebates flow the right way, and the fee account nets to the fees collected
- **16d: Idempotency**
  - **Task 16d-1** (20 pts): a redelivered fill returns the original result and changes neither volume, tiers nor the timestamp clock
- **16e: Scale**
  - **Task 16e-1** (25 pts): 200,000 fills with a wide trailing window are priced exactly and quickly
- **16f: Input validation**
  - **Task 16f-1** (10 pts): invalid schedules, options and fills throw RangeError

### 🔴 Challenge 17: Live Dashboard Updates — 150 points
**Category**: Frontend (Real-time)  
**Test File**: `tests/challenge17.test.ts`  
**The Problem**: The operator dashboard from Challenge 12 polls. A trading floor needs it to *stream*: prices and the order book push to the browser as they change. The hard part is not receiving messages, it is what the screen does when the feed misbehaves — messages arrive out of order or twice, a gap opens, the connection drops, or the feed simply goes quiet. A dashboard that keeps showing frozen numbers as if they were live is worse than one that says so.
**Tasks to Complete**:
- **17a: Connection status**
  - **Task 17a-1** (20 pts): the feed reports connecting, live and stale at exactly the right moments, and never repeats a status
- **17b: Per-topic ordering**
  - **Task 17b-1** (25 pts): messages arriving shuffled and duplicated are delivered once each, in order, per topic
  - **Task 17b-2** (20 pts): a gap that never fills triggers one resync for that topic only, and deltas are ignored until the next snapshot
- **17c: Reconnecting**
  - **Task 17c-1** (25 pts): reconnect delays follow the capped exponential backoff with jitter exactly, and reset once data flows again
  - **Task 17c-2** (15 pts): a reconnect resubscribes from the last delivered seq, starts without a baseline, retries failed connects, and never reports stale while down
- **17d: Stopping**
  - **Task 17d-1** (10 pts): stop closes the socket once, cancels every timer, ignores late frames and never reconnects
- **17e: Applying order-book deltas**
  - **Task 17e-1** (15 pts): applying a delta stream matches an independent model, never mutates its input, and rejects invalid deltas
- **17f: The dashboard shows the connection state**
  - **Task 17f-1** (20 pts): stale and reconnecting keep the old data on screen with a note, live restores it, and loading/empty/error panels are left alone

### 🟠 Challenge 18: The WebSocket Feed — 190 points
**Category**: Real-time (WebSockets)  
**Test File**: `tests/challenge18.test.ts`  
**The Problem**: The dashboard should not have to poll. A WebSocket hub pushes order-book changes to authenticated clients as they happen, in order, without gaps, and without letting one stalled browser tab hurt everyone else. The tests run your hub on a real HTTP server and talk to it with a real WebSocket client.
**Tasks to Complete**:
- **18a: Handshake and subscriptions**
  - **Task 18a-1** (15 pts): only a valid token opens a connection; anything else is refused with 401 before the upgrade
  - **Task 18a-2** (20 pts): subscribing returns a snapshot per allowed topic, an error per forbidden or unknown one, and survives malformed messages
- **18b: Ordered delivery**
  - **Task 18b-1** (30 pts): each topic has its own gapless, increasing sequence, and a late subscriber's snapshot joins it seamlessly
  - **Task 18b-2** (15 pts): a resync request returns a fresh snapshot at the current sequence, and only for a subscribed topic
- **18c: Liveness and backpressure**
  - **Task 18c-1** (25 pts): idle clients receive heartbeats, and a client that stops answering pings is dropped while a healthy one stays
  - **Task 18c-2** (30 pts): a consumer that stops reading is cut loose instead of buffering without bound, and everyone else keeps receiving every message
- **18d: Shutdown**
  - **Task 18d-1** (15 pts): disconnectAll drops every client with 1012 and keeps accepting new ones; close ends everything with 1001
- **18e: Fan-out**
  - **Task 18e-1** (20 pts): thirty subscribers each receive a hundred rapid publishes complete and in order
- **18f: Book deltas**
  - **Task 18f-1** (20 pts): diffDepth returns exactly the changed levels, with quantity 0 for removed ones, so applying them rebuilds the new book

### 🟡 Challenge 19: API Documentation (Swagger) — 185 points
**Category**: API Documentation (OpenAPI / Swagger)  
**Test File**: `tests/challenge19.test.ts`  
**The Problem**: A Swagger (OpenAPI) document must be implemented for the whole API. Other teams and tools should be able to read what every endpoint takes and returns, try it from an interactive page, and trust that the document matches the running server. The tests validate the document with a real OpenAPI validator and check it against the real routes and real responses.
**Tasks to Complete**:
- **19a: A valid OpenAPI document**
  - **Task 19a-1** (10 pts): GET /api/openapi.json serves an OpenAPI 3 document with a title, a version and a server
  - **Task 19a-2** (20 pts): the document passes a real OpenAPI validator, including every $ref
- **19b: Coverage and accuracy**
  - **Task 19b-1** (25 pts): every operation the API serves is documented under its exact path template
  - **Task 19b-2** (15 pts): nothing is documented that the API does not serve
  - **Task 19b-3** (20 pts): every operation has a real summary, a tag, a unique operationId, a success response and its path parameters declared
  - **Task 19b-4** (15 pts): API operations document the standard error envelope through a shared component
- **19c: Request and response schemas**
  - **Task 19c-1** (30 pts): request bodies describe their required fields, enums and integer-string amounts, and idempotent calls declare Idempotency-Key
  - **Task 19c-2** (25 pts): the documented response schemas match what the API really returns
- **19d: Swagger UI**
  - **Task 19d-1** (10 pts): GET /api/docs serves an HTML page that loads Swagger UI against the document
  - **Task 19d-2** (15 pts): the docs page relaxes the Content-Security-Policy just enough to run, while every other route stays locked down

### 🔵 Challenge 20: Dashboard and Demo Data — 160 points
**Category**: Frontend and Demo Data  
**Test File**: `tests/challenge20.test.ts`  
**The Problem**: The dashboard should be meaningful and informative, and when it loads every account should already hold an initial amount. The demo seed funds a set of trading accounts through balanced double-entry postings, an API lists every account with its holdings, and the dashboard turns that into an account list, a portfolio summary, risk usage meters and recent trades.
**Tasks to Complete**:
- **20a: Initial data**
  - **Task 20a-1** (15 pts): running the demo seed creates at least five funded trading accounts, each holding at least two known assets
  - **Task 20a-2** (25 pts): the initial funding is double-entry: every entry balances, the ledger sums to zero per asset, and postings agree with the stored balances
  - **Task 20a-3** (15 pts): running the seed again changes nothing
- **20b: The accounts API**
  - **Task 20b-1** (25 pts): GET /api/ledger/accounts lists every account with its balances, sorted by name
- **20c: An informative dashboard**
  - **Task 20c-1** (20 pts): the account list shows every account by name with exact, exponent-aware holdings and a status badge
  - **Task 20c-2** (25 pts): the portfolio summary counts the accounts and adds each asset up exactly across all of them
  - **Task 20c-3** (20 pts): risk usage shows how much of each limit is used, with a meter that turns to warning at 80% and danger at 100%
  - **Task 20c-4** (15 pts): recent trades lists at most ten, newest first, each with its exact notional, and says so when there are none

### 🟢 Challenge 21: Integration — 150 points
**Category**: Integration  
**Test File**: `tests/challenge21.test.ts`  
**The Problem**: Individually correct parts are not a working system. These scenarios run on a real server with real sockets and only pass when the matching engine, the WebSocket hub, the live-feed client, netting, fees, the seed data and the dashboard code all agree with each other.
**Tasks to Complete**:
- **21a: The live order book**
  - **Task 21a-1** (40 pts): the book rebuilt from the WebSocket feed always equals the depth endpoint, with a gapless sequence
  - **Task 21a-2** (40 pts): when the server drops every connection the client reconnects, resynchronises from a snapshot and catches up
- **21b: From trades to settlement**
  - **Task 21b-1** (40 pts): trades taken from the API net down without changing anyone's position, and their fees balance exactly
- **21c: The dashboard on real data**
  - **Task 21c-1** (30 pts): after the demo seed and some trading, the dashboard's summary, account list and recent trades agree with the API and the database

---

## Getting started and rules

See the [DevQuest Getting Started Guide](https://udithab99x.github.io/DevQuest-2026-Guide/) for setup
