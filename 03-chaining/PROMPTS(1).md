# PROMPTS.md: Living Prompt Pack

> Module 3 · Prompt Chaining. Re-architect the build with prompt chains; capture the reusable ones here.

## How to use this pack

_Each prompt is a reusable step. Chain them: the output of one becomes the input to the next._

## Prompt chain: Start your day Chain

### Step 1: Expand, build new screens in a strict sequence
```
Build the next phase of this app in a strict sequence:
1. Add a screen "Start your day". Match the layout and spacing of the attached {{reference A}} screenshot.
2. Add a screen "Who's Next?". Match the data density of the attached {{Zoho}} screenshot.
3. Navigation: write the logic so {{Start Your Day}} links to {{Who's Next?}}.
4. Add a screen "Active Deals". Match the data density of the attached {{Zoho464}}
5. Navigation: Write the logic so {{Start Your Day}} links to {{Active Deals}}

Build these in order so {{Start Your Day}} is the anchor for {{Who's Next?}}.
```

### Step 2: Behavior, hard-code the states
```
Apply the following logic constraints to the {{Who's Next?}} flow:
- Use skeleton screens for the {{list}} loading state.
- If no data is present, show the empty state: "{{Nothing on deck!}}".
- On fetch failure, trigger the error state: "{{Failure to load, please refresh.}}".

Apply the following logic constraints to the {{Active Deals}} flow:
- Use skeleton screens for the {{list}} loading state.
- If no data is present, show the empty state: "{{Nothing on deck!}}".
- On fetch failure, trigger the error state: "{{Failure to load, please refresh.}}".
Maintain the same design language throughout and tether all behavior strictly to these rules.
```

### Step 3: Refine, one surgical polish
```
The {{Start your day}} needs a professional {{Zoho-style}} polish.
1. Start by listing the 3 biggest gaps in typography and spacing compared to {{Zoho-style}}.
2. Once you've identified those, resize the headers and update the {{note cards}} to match.

Don't change anything else in the project or touch the underlying logic.
```

## Reusable techniques learned

- How to construct a chained prompt to have the page built the way I want and move through the navigation as desired.
- Creating an error handling prompt was not something that I had thought about previously.

## What broke (and the fix)

_Where a single mega-prompt failed and chaining fixed it._

I have not had issues with a prompt failing and this did not solve that issue since it did not exist.
