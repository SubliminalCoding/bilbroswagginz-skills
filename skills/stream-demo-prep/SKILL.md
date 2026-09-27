---
name: stream-demo-prep
description: Pre-stream checklist for live demos, covering screen hygiene, rehearsal discipline, fallback plan when the demo breaks on air, chat triage, post-stream capture. Use when prepping to demo a feature on a live stream, planning a launch stream, choosing what to show and what to hide, or reviewing what went wrong after a stream where the demo blew up.
---

# Stream Demo Prep

A live demo is a release with an audience. It fails the same way releases fail: the path wasn't rehearsed, the screen wasn't scrubbed, the fallback didn't exist, or the feature wasn't actually done. This skill is the checklist that gets run before OBS starts recording.

## The four non-negotiables

Every live demo clears these four bars before going on air. No exceptions.

1. **The happy path has been rehearsed end-to-end at least three times today.** Not "I've done this before." Three times today, on the machine being streamed from, on the network being streamed on, with the exact cadence the audience will see. If the rehearsal count is below three, delay the demo.
2. **The screen is scrubbed.** No secrets, no client data, no personal DMs, no open tabs that reveal private work. Scrub before OBS is live, not during.
3. **The acceptance test is something the audience can see.** If only the streamer can tell whether it worked, the demo is a monologue. The audience should see the phone ring, the email arrive, the output appear, the number go up.
4. **The fallback exists.** A plan for what happens if the live demo breaks: a prerecorded clip, a code walkthrough, a "here's what it looks like when it works" screenshot. If there is no fallback, the live demo is betting the whole stream on a single try.

## Pre-stream checklist

Run this top-to-bottom before going live. Not "glance at it." Execute each item.

**Environment**
- Close every tab that isn't part of the demo. Bookmarks bar hidden.
- Terminal scrollback cleared. Shell history considered for the current session only.
- Notifications off on the OS. iMessage, Slack, email, calendar pings. All muted for the stream window.
- Desktop wallpaper is neutral. Desktop icons are hidden or not the streamed monitor.

**Secrets and data**
- `.env` files and `~/.aws/credentials`-class files are not visible, not in terminal history, not in any open editor pane.
- Client names, email addresses, phone numbers that aren't the demo's own. Replace with fixtures. If the demo phone number for the AI receptionist is being shown, that's fine; that's the demo. Other client numbers are not fine.
- Git log for the repo being shown: scan for commit messages that leak anything.

**The demo path**
- Written down, step by step, where the streamer can see it off-camera. Not "I'll remember." On paper or in a second monitor.
- Every external dependency the demo touches has been poked in the last hour. Phone line answers, API returns 200, webhook fires, whatever.
- The machine has not been rebooted between rehearsal and stream. If it has, rehearse again. Process state matters.

**The fallback**
- Prerecorded clip of the demo working, ready to drop into OBS as a scene.
- Or: a code walkthrough prepared: the narrative that works even if nothing runs.
- Or: a screenshot pack showing the demo's output at each stage.
- The switch to fallback is rehearsed. The streamer knows how to pivot without dead air.

## What not to show

Keep a running "do not show" list for every demo. Typical entries:

- **Production data from clients.** Even "just to show a realistic volume." No. Use fixtures or anonymized samples.
- **Unfinished features adjacent to the demo path.** A half-wired admin page the audience spots in the sidebar becomes "when's that shipping?" questions for the rest of the stream.
- **Experimental branches not meant for public yet.** If the repo is being streamed, checkout main (or the actual demo branch) and stay there.
- **The cost side of the economics.** "This costs $X per call" is a spec the streamer hasn't committed to publicly. Don't name a number on air unless it's the real public number.
- **Any screen that shows the streamer's own personal accounts.** Banking, health portal, therapist's calendar invite. One glance and it's out there.

## During the stream

- **One dedicated chat-check pause per demo segment.** Not constant monitoring. The demo flow breaks if the streamer is reading chat mid-demo.
- **Narrate the path.** The audience doesn't know what the streamer is about to click. Every action gets a sentence: "I'm about to call the demo number. Watch the terminal on the right for the incoming call event."
- **If something breaks: acknowledge it once, pivot to the fallback, keep the energy.** Do not spend ten minutes debugging on air for a crowd that's there to see the thing work. Fix the bug after the stream.

## Post-stream capture

The stream's second product is the list of questions it generated. Capture them while they're fresh:

- **Chat questions that surfaced real product friction.** These are candidate backlog items, but run them through `build-in-public-scope-guard` before committing to build them.
- **The moments that landed.** What got the most reactions? Those are the beats for the next demo.
- **The moments that fell flat.** Usually because the audience didn't see the payoff. Note it and fix the visibility next time.
- **Any breakage.** What broke, how long the recovery took, what the fallback looked like to the audience. If the breakage was interesting enough, it becomes its own post-mortem content.

Write this down within an hour of the stream ending, while memory is current.

## When this skill should say no

- **Before going live without three rehearsals of the demo path.** Stop. Reschedule or cut the demo from the stream.
- **Before demoing a feature that isn't finished end-to-end.** Stop. Show the code or the plan instead; don't fake the demo.
- **Before going live with notifications on.** Stop. Kill them first.
- **Before going live without a fallback.** Stop. Build the fallback, even a thin one.
- **Before showing any screen that hasn't been scrubbed.** Stop. Scrub first.

A stream that doesn't happen tonight because the demo wasn't ready is a better outcome than a stream where the demo leaks a client's data or breaks in front of an audience with no recovery. The bar is shipping on air, not shipping on schedule.
