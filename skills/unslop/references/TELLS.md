# The Tell Taxonomy

Every edit in the CHANGES log cites one of these class IDs. No tag, no edit.
Each entry: what it is, how to detect it, and a before/after pair.

## Prose tells (T1–T9)

### T1: Filler opener
Throat-clearing before the point. The first sentence exists to warm up, not
to inform.

Detect: "In today's fast-paced world", "In the world of X", "It's worth
noting that", "When it comes to", "In an era where", "It goes without
saying". Also any opener that would survive being pasted onto a different
article.

Before:
```
In today's fast-paced development landscape, it's worth noting that caching
can significantly improve performance.
```
After:
```
Caching cut our p95 from 800ms to 90ms.
```
Fix: delete the opener entirely. Never replace it with a smaller opener.

### T2: Hype adjective / AI-tell verb
Words that assert quality without evidence. They do no work; if the thing is
fast, show the number.

Detect: robust, seamless, powerful, comprehensive, cutting-edge, world-class,
best-in-class, blazingly fast, revolutionary, game-changing, sophisticated,
elegant, effortless. Verbs: leverage, elevate, unlock, delve, empower,
supercharge, streamline, harness, navigate (the landscape), embark.

Before:
```
Leverage our robust API to seamlessly unlock powerful integrations.
```
After:
```
The API covers all 14 resource types; auth is one header.
```
Fix: replace with evidence or delete. "Leverage" → "use". "Delve into" →
"read" or "cover".

### T3: Empty tricolon
The rule-of-three list where at least one item exists for rhythm, not truth.
Models reach for triples reflexively.

Detect: any "X, Y, and Z" where Z restates X, or where the three can't each
survive the question "what specifically?".

Before:
```
The CLI is fast, reliable, and efficient.
```
After:
```
The CLI resolves a 500-package lockfile in under a second.
```
Fix: keep the one item that's true and demonstrable. A genuine triple of
three distinct, load-bearing facts is fine; that's the test.

### T4: Hedge
Refusing to commit to the claim. Either it does the thing or it doesn't.

Detect: "can help to", "may potentially", "is designed to", "aims to",
"tends to", "in many cases", "it could be argued", "generally speaking".

Before:
```
Enabling compression can help to potentially reduce bandwidth usage.
```
After:
```
Compression cuts bandwidth roughly 70% on JSON payloads.
```
Fix: commit (with the real number or condition) or cut the sentence. If the
truth is genuinely conditional, state the condition: "Reduces bandwidth when
payloads exceed 1KB."

### T5: Rhythm tells (em-dash overuse, uniform cadence)
Distribution, not vocabulary. An em-dash every third sentence; every sentence
12–18 words; every paragraph three sentences ending on a flourish.

Detect: count em-dashes per paragraph (more than one is suspect, decoration
vs. structure is the test). Read three consecutive sentences aloud; if they
scan identically, it's a tell.

Before:
```
The watcher detects changes — instantly. It rebuilds the affected modules —
nothing more. The browser refreshes — automatically.
```
After:
```
The watcher rebuilds only the modules that changed, then refreshes the
browser. Total round trip is under 200ms.
```
Fix: vary sentence length on purpose. Short one. Then a longer one that
carries the actual mechanics. Replace decorative em-dashes with commas,
periods, or parentheses; keep an em-dash only where it carries structure.

### T6: False balance
"Not only X but also Y" and the "it's not just X, it's Y" formula. Inflation
dressed as nuance.

Detect: "not only... but also", "not just a X — it's a Y", "more than just",
"while X, it also Y" where the contrast is fake.

Before:
```
It's not just a linter — it's a complete code-quality platform.
```
After:
```
It lints, formats, and runs your test suite from one config file.
```
Fix: say what it is. If both halves are real, list them as facts without the
seesaw construction.

### T7: Restating close
The conclusion that summarizes what the reader just read. "In conclusion,"
"To sum up," or an unmarked final paragraph that re-walks the page.

Detect: could the last paragraph be deleted with zero information loss?

Before:
```
In conclusion, we've seen how caching, compression, and connection pooling
can work together to improve your application's performance.
```
After:
```
(deleted; the doc ends on the last instruction)
```
Fix: delete. If there's a real next action, end on that instead: "Start with
the cache; it's the 80% win."

### T8: Emoji / bold-bullet decoration
Decoration standing in for structure: emoji headers, every bullet opening
with a **Bold label:** that adds nothing, checkmark lists.

Detect: ✨🚀📦🎯 in headers; bullets where the bold lead-in restates the
sentence that follows; ✅ used as a bullet glyph.

Before:
```
## 🚀 Features
- **Blazing speed:** Processes files at blazing speed
- **Easy setup:** Setup is easy
```
After:
```
## Features
- Processes a 2GB log file in 4 seconds (`bench/` has the harness)
- Setup is one command: `tool init`
```
Fix: words for headers, facts for bullets. Bold lead-ins only when the label
genuinely differs from the body.

### T9: Passive-corporate voice
Nobody does anything; functionality "is provided" and users "are empowered."

Detect: "users are able to", "functionality is provided", "it is recommended
that", "support is offered for", "X is empowered to".

Before:
```
Users are empowered to customize their workflow through the settings that
are provided.
```
After:
```
You can change any of this in Settings.
```
Fix: active voice, second person, a named actor.

## UI-copy tells (U1–U5)

The same disease on the product surface, where every user reads every word.
Budgets: buttons ≤ 3 words, toasts ≤ 8, error messages = what happened +
what to do.

### U1: Button fluff
The label sells instead of naming the action.

Before → After:
```
"Get Started Now!"        → "Start"
"Unlock Premium Features" → "Upgrade"
"Begin Your Free Trial"   → "Start free trial"
"Yes, I'm Ready!"         → "Continue"
```
Test: the label is the verb the click performs. Exclamation marks never ship
on a button.

### U2: Empty-state hype
The empty state pitches the feature at someone who already bought it, instead
of telling them what to do.

Before:
```
✨ Unlock the power of automated invoicing! Our seamless invoice engine
helps you get paid faster than ever before.
```
After:
```
No invoices yet. Create your first one with the button above.
```
Test: orient (what is this, why empty, what's next). Never sell.

### U3: Vague-cheerful error
Apologizes with personality, says nothing actionable. The worst slop in the
product: it spends trust at the exact moment the product is failing.

Before:
```
Oops! Something went wrong 😞 Please try again later!
```
After:
```
Couldn't save: server didn't respond. Your draft is kept locally; retry in
a minute.
```
Test: what happened + what the user should do. No "Oops", no emoji, no
exclamation marks. If the cause is unknown, say what's safe ("Nothing was
charged") and the next step.

### U4: Cheerleader toast
Confirmation inflated into celebration.

Before → After:
```
"Awesome! Your changes have been successfully saved! 🎉" → "Saved"
"Woohoo! Email sent successfully!"                        → "Email sent"
"Success! Your profile has been updated!"                 → "Profile updated"
```
Test: ≤ 8 words, past-tense fact, no adverbs ("successfully" is the tell:
saving IS succeeding).

### U5: Onboarding journey
Tour copy that narrates an adventure instead of giving the next instruction.

Before:
```
Welcome aboard! Let's embark on your journey to productivity mastery. First,
we'll explore the magical world of workspaces!
```
After:
```
Create a workspace to keep projects separate. Most teams start with one.
```
Test: each step is an instruction plus the reason it matters. Nobody is
aboard anything.

## What is NOT a tell

- A genuine em-dash carrying structure (one per stretch, doing work commas
  can't).
- A real triple of three distinct facts.
- An intentional brand catchphrase or running joke is voice. When a
  span pattern-matches a tell but might be deliberate, FLAG it, don't cut it.
- Hedging that encodes a true condition ("requires Node 20+ on ARM"). The
  hedge tell is about dodging commitment, not about stating limits.
