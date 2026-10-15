# PROMPTS.md: Living Prompt Pack

> Module 3 · Prompt Chaining. Re-architect the build with prompt chains; capture the reusable ones here.

## How to use this pack

_Each prompt is a reusable step. Chain them: the output of one becomes the input to the next._

## Prompt chain: Onboarding Chain

### Step 1: Expand, build new screens in a strict sequence
```
Build the next phase of this app in a strict sequence:
1. Add a screen "{{Call to Action}}". Match the layout and spacing of the attached {{Notion1}} screenshot.
2. Add a screen "{{Account Status}}". Match the data density of the attached {{Notion2}} screenshot.
3. Navigation: On the Call to Action screen user executes his/her CTA and can mark it as done in which the user lands on the Account Status page

Build these in order so {{Call to Action}} is the anchor for {{Account Status}}.
```

### Step 2: Behavior, hard-code the states
```
Apply the following logic constraints to the {{screen}} flow:
- Use skeleton screens for the {{list}} loading state.
- If no data is present, show the empty state: "{{empty-state message}}".
- On fetch failure, trigger the error state: "{{error message}}".

Maintain the same design language throughout and tether all behavior strictly to these rules.
```

### Step 3: Refine, one surgical polish
```
The {{screen}} needs a professional {{style}} polish.
1. Start by listing the 3 biggest gaps in typography and spacing compared to {{reference}}.
2. Once you've identified those, resize the headers and update the {{element}} to match.

Don't change anything else in the project or touch the underlying logic.
```

## Reusable techniques learned

- The chainring makes the vibe coding more structured and intentional than coding by trial and error

## What broke (and the fix)

_Where a single mega-prompt failed and chaining fixed it._

With a mega prompt, AI sometimes interprets the instructions wrong while chaining makes it structured and more predictable and repeatable
