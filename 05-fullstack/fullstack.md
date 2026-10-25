# Full-Stack: Data, Access Rules, Edge Cases, Deploy

> Module 5 · Full-Stack. Add data schemas, access rules, and edge cases; stress-test and deploy.

## Deployed link

_The working, shareable link that survives real users._

_____

## Data schema

| Entity | Key fields | Notes |
|---|---|---|
| onboarding_outcomes | id text primary key (risk, onboard, report), title text, description text, headline text, position smallint | Replaces OUTCOMES; test_sessions.outcome_picked references it |
| session_assignments | session_id uuid → test_sessions (on delete cascade), account_id uuid → accounts, owner_id uuid → teammates, note text (≤ 500 chars), assigned_at timestamptz, marked_done_at timestamptz, via text (guided_step, account_status, explore); primary key (session_id, account_id) | Each participant's owner picks and "mark done", kept apart from the real assignments table |

## Access rules

_Who can see / do what? Where are the auth boundaries?_

Since Oct 24, every result row belongs to one account: a signed-in user reads and writes only rows whose user_id is their own, and signed-out participants can only add.

## Edge cases hardened

| Case | Before | After |
|---|---|---|
| Empty / first-run state | Whenever the page had no sessions in hand: still loading, failed to load, or truly none. A failed load looked like “you have no data”. | Only after a successful load that found no sessions. Loading shows the skeleton; a failure shows the error with Retry. |
| Bad / malicious input | The same email address twice, in any capitalization	Nothing was stored at all | Saved once per session |
| Failure / offline | Results can’t load sessions	If none were loaded yet, it showed “No sessions yet”, as if you had no data | “Couldn’t load your results. Check your internet connection, then retry.” + Retry |

## Stress test results

_What you threw at it, and what held / broke._

_____
