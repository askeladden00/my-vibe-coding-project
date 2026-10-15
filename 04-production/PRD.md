# Living PRD

> Module 4 · Production Specs. Refactor for readability; extract a living PRD that stays true as the build evolves.

## Problem

_What user problem does this solve? Tie to the validated hypothesis._

Replacing exploratory onboarding with a guided, outcome-based activation flow, one that reaches the aha action and prompts the user to involve their team, will help new accounts understand the team value and reach first value faster.

## Users & jobs

- **Primary user:** a new account's buyer or admin at a B2B SaaS company, such as an ops lead, head of customer success or eng manager. They own retention outcomes but rarely work every at-risk account themselves.
- **Job to be done:** "When I've just signed up, help me see which customers are at risk and get each one to the right person on my team, so the tool earns its place before the team writes it off."

## Scope

- **In:** Guided flow: 4 steps (choose an outcome, review 6 at-risk accounts, assign an owner, see the teammate's view and invite)
Account status page: a queue of the remaining 5 accounts, each opened to assign and mark done
Per-account status page: owner, risk signals, alerts, plan and billing, contacts, activity
Loading, empty and error states for the account lists
Exploratory arm (today's onboarding) as the comparison
Moderator tools: session start, arm assignment, timer, debrief rubric, practice runs
Results and kill switch from saved sessions
- **Out (explicitly):** Sign-up, authentication, workspace creation
Real data sources (CRM, billing, product usage) and real risk scoring
Sending invites, notifications or emails to teammates
Teammate's own experience after assignment
Live A/B rollout, week-one activation and 60-day churn measurement

## Requirements

| # | Requirement | Priority | Acceptance criteria |
|---|---|---|---|
| 1 | Guided flow reaches the aha action in 4 steps | Must | Step 3's Assign logs aha_reached with seconds since start; the shortest path is 4 clicks (verified). |
| 2 | Account status queue after setup | Should | Lists the 5 remaining accounts with a "Needs an owner" chip and Open; progress reads "x of 6 done"; item ticks at 6 of 6 (verified). Mocked: sample accounts. |

## Data & events

_What gets stored, what gets tracked._

One session record per participant, in the prototype's shared store (sessions collection). Events are logged in the moderator's browser for that page load only and are not stored. All account, teammate and billing data is fictional sample data in the page.

## Open questions

[ ] The guided arm now differs from exploratory in more than guidance (outcome choice, teammate preview, account queue). Is that the treatment we mean to test?
