# Engineering Handoff Note

> Module 4 · Production Specs. Open the black box, make the build legible to an engineer.

## What this is

_One paragraph an engineer can read in 60 seconds._

The Retention Engine prototype is a no-build web page (plain HTML, CSS and JavaScript) that runs a moderated test of guided versus exploratory onboarding. Participants click through fictional sample data. The only real backend is a small shared store, attached to the published page, that keeps one record per test session and drives the kill switch. Version 15, a single 1,500-line index.html, is live at the published link and is what moderators use. A refactor into 49 files grouped by feature sits on disk, unpublished: it parses and every original line is accounted for, but it has not been click-tested in a browser yet. Your first job is to run the checklist in "How to run it" against the refactor, then publish it to the same link. Also check the saved results: P02–P05 look like design walkthroughs and are counted toward the kill switch.

## Architecture (plain language)

- **Frontend:** One global state object drives everything: every click changes state, and the router redraws the whole screen from it. 1) Load 1. Load. index.html loads the stylesheets, the Supabase JS client (v2.45.4, from a CDN), then about 45 plain <script> files in a fixed order. There are no modules and no bundler. Scripts share one global scope, so each file can use what earlier files defined.; 2) State, app/state.js holds one object: which screen is showing, the live session, each arm's in-progress UI state, the moderator's sign-in status, saved sessions, and the preview settings.; 3) Click, Buttons carry data-action and data-v attributes. app/router.js listens once on the document and calls the handler that a feature registered for that action name (registerActions in app/actions.js). Text inputs work the same way through registerInputs.; 4) 1. Render. A handler updates state and calls render(). The router picks a screen function (for example AccountStatusScreen()), which returns an HTML string written into <main>. The moderator bar redraws alongside it.; 5) 1. Sign in and save. Moderators sign in with Supabase Auth (email and password) from the panel in features/sign-in/. On the Debrief, saveSession() in experiment/session-store.js inserts one row into the test_sessions table. The page reloads the table after every save and every 30 seconds on Session start and Results, and Results recomputes the kill switch with evaluateKillSwitch(). session-store.js is the only file that knows the table's column names.
- **Backend / data:** Supabase schema and access rules: Row level security on all 10 tables; signed-out reads return nothing and writes are refused; signed-in save, read and delete tested; security advisor clean
- **Key flows:** 1. Click-to-render loop (every interaction)
1.1. Click. A button carries a data-action name.
1.2. Router. app/router.js hands the click to the feature that registered that action.
1.3. Feature action. The handler in that screen’s *.actions.js updates state.
1.4. State. The one object in app/state.js changes.
1.5. Render. render() calls the screen function for the current state, such as AccountPage(), which returns HTML.
1.6. Page. The HTML is written into <main>, ready for the next click.

Everything in this loop stays in the browser’s memory. Nothing is saved until the Debrief.
2. Sign-in and save branch (the only part that leaves the browser)
2.1. Sign in. A moderator signs in with Supabase Auth from the sign-in panel.
2.2. Save. On the Debrief, Save calls saveSession() in experiment/session-store.js, which inserts one row into Supabase’s test_sessions table.
2.3. Reload. The page re-reads the table after each save and every 30 seconds on Session start and Results.
2.4. Recompute. Results recalculates the kill switch from those rows.

## What's solid vs. what's duct tape

| Area | State | Notes |
|---|---|---|
| Kill-switch and rubric logic (experiment/rules.js, kill_switch view) | solid | Pure functions, checked at the 30% and 50% boundaries, the 8-session gate, and exploratory sessions excluded; the database view uses the same rule |
| Sample accounts, teammates, risk signals | rough | Fictional data in data/sample-accounts.js; the same data sits in Supabase's product tables, but the page doesn't read them |

## Risks & assumptions for the team

The biggest risks are a wrong go/no-go and exposed participant answers, not a crash: split or bad records move the kill switch directly, and today anyone can create an account.

#
Risk
Impact
Mitigation
1
Open sign-ups
Supabase lets anyone create an account by default, and the page's key is public by design; a stranger who finds the page could sign up and read participants' answers
Once moderators have accounts, turn off new sign-ups in Supabase (Authentication → Sign In / Providers), or limit access rules to a list of moderator emails
2
Results split across two stores
The old claude.ai link still saves to its own store; sessions saved there never reach Supabase or its kill switch
Move every moderator to the new page and retire the claude.ai link
3
Sign-in and save untested end to end
The first real save fails during a session
Sign up, confirm, sign in and save one real (not practice) test session before the first participant; delete it afterwards
4
Walkthrough rows counted as participants
The kill switch moves on test data (P02–P05 did this in the old store)
Use Practice run for any walkthrough; P02–P05 were not copied to Supabase
5
Page reloaded mid-session
The in-progress session lives in memory and is lost
Finish and save before reloading; serve with caching off while developing
6
Treatment drift
The guided arm differs in more than guidance (outcome pick, teammate preview, account queue), so a result can't isolate "guidance"
Freeze the design before counting sessions; treat P01 as a pilot
7
Small sample
At 8 guided sessions, one session moves the pass rate 12.5 points
Read the result as a direction, not a measurement
8
Moderator inconsistency
Prompting or loose scoring inflates "Correct"
Exact question on screen; rubric ticks and overrides saved for audit
9
Unescaped text
A value inserted into HTML without esc() could run script
Review any new template field for esc()

## How to run it

```
Serve the retention-engine-prototype folder from any local web server and open it in a browser; there is nothing to install or build. Opening index.html directly as a file no longer works, because the browser won't load the 45 separate script and style files that way.

Run it locally
• From the project root, run python3 -m http.server 8765 --bind 127.0.0.1 --directory retention-engine-prototype, then open http://localhost:8765. The same command is saved as retention-engine in .claude/launch.json for the Claude desktop app's preview.
• Needs an internet connection: the page loads its fonts and the Supabase JS client from CDNs.
• After changing files, hard-refresh (Shift + reload). The browser caches the script files, and a stale one looks like a bug.

Supabase
• Project: ocsxyjygatbnfwepyinl (us-east-1). The URL and public anon key are in experiment/supabase-config.js; they are safe to ship, because access rules protect the data.
• Schema lives in supabase/migrations/ (two migrations, both applied), and sample data in supabase/seed.sql. Apply new migrations with the Supabase CLI (supabase db push) or Claude's Supabase connector, and check the database advisors afterwards.
• Moderators create an account from the sign-in panel on Session start, confirm by email, then sign in. The confirmation link lands on localhost:3000, Supabase's default; that error page is expected, and the email is confirmed regardless. For a prototype you can turn off "Confirm email".
• Results: the Results screen, or the test_sessions table and kill_switch view in Supabase's Table Editor.

Before the first real session
[x] Browser click-through of the refactor: both arms, Account status loop, Debrief, practice run, Screen states, Results, phone width, no console errors
[ ] Sign up, confirm and sign in as a moderator; the chip reads "Supabase connected · 1 saved"
[ ] Save one real (not practice) session; it appears in Results and in test_sessions; then delete it
[ ] Turn off new sign-ups once every moderator has an account
[ ] Host the folder somewhere moderators can reach (e.g. Netlify, Supabase Storage) and add that address to Supabase's allowed redirect URLs
[ ] Retire the claude.ai link so all sessions land in Supabase
The project folder isn't under version control, and the pre-refactor single file is only backed up in a temporary folder. Commit the current folder, including supabase/, to a repository before the next change.
```
