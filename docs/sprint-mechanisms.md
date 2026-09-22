# Sprint write-up: mechanisms, not “fixed it”

Lars reviewed the embassy button work (plan, build, verify). This file answers every point of that feedback as a mechanism: what happens, in what order, what can go wrong. It does not re-litigate the code.

Sources: [PR #1](https://github.com/bugxxy/spotlight/pull/1), [PR #3](https://github.com/bugxxy/spotlight/pull/3) (“Make the button stop repeating verdicts”), [PR #4](https://github.com/bugxxy/spotlight/pull/4).

---

## Old behaviour and new behaviour

### Old (before the cycle rule)

1. Visitor loads `index.html`. `#verdict` is empty.
2. Visitor clicks `#report`.
3. The handler computes `lines[Math.floor(Math.random() * lines.length)]` against the **full** list.
4. That string is written into `#verdict`.
5. The handler ends. Nothing is stored. The next click repeats steps 3–4.

There is no list of shown lines, no index, no cycle. Each click is an independent roll.

**What can go wrong.** The same line can appear on click 1 and click 2. With five lines, a repeat inside three clicks is common. After every line has appeared at least once, the page has no notion that the embassy has said everything — it keeps rolling. A visitor cannot tell a stuck button from an unlucky roll. A one-line list looks the same as a five-line list that happened to repeat.

Reload clears `#verdict` because the string lives only in the DOM. That part was already correct. The defect is the roll, not the lack of memory across visits.

### New (the cycle rule now in `index.html`)

Statement order in the click handler:

1. Visitor loads `index.html`. `#verdict` is empty. `cycle` is empty, `nextIndex` is 0, `lastLine` is null. Nothing is in localStorage, a cookie, or a server.
2. Visitor clicks `#report`.
3. If `nextIndex >= cycle.length` (true on the first click, and after the last card of a cycle), the page copies `lines`, Fisher-Yates shuffles that copy, and — when there is more than one line and `lastLine` is set and the new first card equals it — swaps the first card with a later card. That permutation **is** the cycle. `nextIndex` is set to 0.
4. The handler reads `cycle[nextIndex]` into a local `line`.
5. It increments `nextIndex`.
6. It stores `lastLine = line`.
7. It writes `line` into `#verdict`.
8. The handler returns. It is synchronous, so the next click, including a rapid one, starts at step 3 with the updated index and last line. A click never writes an empty string into `#verdict`.

The DOM is updated last. Memory (`nextIndex`, `lastLine`) is already the post-click state when the visitor can see the new verdict.

**Guarantee.** Inside one cycle, no line repeats until every line has been shown once. Clicks 1–5 are five distinct lines. Clicks 6–10 are five distinct lines. The same for every later block of N that starts on a cycle boundary.

**What can still go wrong, and is allowed.** The join only forbids the immediately previous line. Clicks 5–7 can be D, B, D. Clicks 2–6 can contain a line twice. Those windows sit on the seam. The cycle rule does not apply to them. A one-line list repeats that line on every click: there is no other card. Reload throws away page memory, so the next click is a first click of a new visit; repeats across visits are accepted.

A reader can restate the old flaw without the code: **each click rolled the full list, so a punchline could come back immediately.** The new rule: **shuffle once, deal in order, reshuffle only when the deck is empty, and do not open the new deck on the card just shown.**

---

## Responses to Lars

### 1. “No duplicate inside a run of five” is not what the rule guarantees (PR #1)

**What was happening.** An early draft of the plan told testers to fail any five-click run that contained a duplicate. The pick rule, even then, only guaranteed uniqueness **inside a cycle**. After the last unshown line, a new shuffle starts. Its first card is forbidden only from matching the last card shown. Every other recent line is a legal opener.

**In order.** Click 5 finishes cycle 1 (say D). Click 6 draws the first card of a new shuffle from the other four lines. Three of those four appeared in clicks 2–5. So clicks 2–6 contain a duplicate about three times in four. Clicks 5–7 can run D, B, D: a repeat inside three clicks, which was the original symptom, surviving at the seam.

**What can go wrong.** A tester who scores a sliding window of five, or a run of three that sits on the join, fails a page that is doing exactly what the rule says. They cannot tell a broken page from a wrong criterion — the state the plan ticket was written to prevent.

**What we did with that.** We did not tighten the join. We named the residual as a decision: cycle-aligned uniqueness only; seam windows that repeat are a pass. Testers score 1–5, 6–10, 11–15, not 2–6. The walkthrough in `docs/verdict-selection.md` shows B twice in clicks 2–6 and marks it pass, with the reason.

### 2. The button implements the plan; the handler is one card per click (PR #3)

Lars approved PR #3 with no findings. The mechanism he checked, restated:

- Shuffle a **copy**, not the source array, so the embassy's list is not mutated.
- Walk it with an index. Do not call `Math.random()` on the full list on each click.
- When the index reaches the deck length, build a new shuffle. If N > 1 and the new first card equals `lastLine`, swap it off the front.
- Skip that swap when N = 1: the single line is the whole cycle, and repeating it is the only legal behaviour.
- State lives in page memory (`cycle`, `nextIndex`, `lastLine`). Reload reconstructs the script from zero, so the cycle resets.
- On each click the handler reads the card, increments the index, stores `lastLine`, then writes `#verdict`. The DOM update is last.
- The click path is synchronous, so two clicks in quick succession cannot interleave two shuffles or write an empty string.

**What can still go wrong, and is out of scope for that ticket.** Lars could not run bash in that session, so he could not inspect a raw diff or history. He judged the three files he could see (`index.html`, `docs/verdict-selection.md`, `README.md`). A change hiding in some other file would have been invisible to that review. There was no other file.

### 3. Why a deck, not a shrinking pool

Both would stop within-cycle repeats. A pool that draws uniformly from what is left is the same distribution as a random permutation.

**What a pool does, in order.** On each click, pick from the remaining unused lines, remove it, refill when empty. Randomness happens on every click. The rest of the cycle is not committed until those later clicks.

**What the deck does, in order.** At cycle start, shuffle once. Each click takes the next card. Randomness happens once per cycle. After click 1, clicks 2–5 of that visit's first cycle are already decided.

**What can go wrong if we had used a pool while testers score cycle-aligned blocks.** The uniqueness guarantee would still hold, but the cycle would not be an object you can point at. The seam rule is written as “new shuffle, first card ≠ last line.” A pool would need a special first-draw of the new urn. The plan named a deck so the tester, the join, and the click handler are the same story. Using a pool would have been a different rule than the one Lars signed off in PR #1.

### 4. Empty `#verdict` after reload cannot certify “no storage” (PR #4, first finding)

**What was happening.** The first verification sitting clicked ten times, reloaded, saw `#verdict` empty, scored Match on “refresh resets shown-state; no localStorage or cookie.”

**In order.** `index.html` always ships with an empty `#verdict`. Reload parses that file again. The paragraph is empty **before any script runs**, whether or not a previous visit wrote `localStorage`.

**What can go wrong.** A page that writes the deck to `localStorage` and never reads it back produces the same observation: empty box, then a line on the next click. The record would still have said Match. A checker who trusts the record would believe storage was verified when only reload-clears-the-DOM was observed.

**What we did with that.** We stopped scoring no-storage from that sitting. A later sitting clicked three times (mid-cycle), read `localStorage` / `sessionStorage` / `document.cookie` (all empty), reloaded, read storage again (empty), then clicked five times. Those five were a full new cycle: the second of them had already been used in the three clicks before reload, so this was not the leftover two cards of the old deck. That sitting can falsify “shown-state persisted in page memory or in those three stores.” It still cannot see a server.

### 5. Empty client storage cannot certify “no server write” (PR #4, second finding)

**What was happening.** After the storage sitting, a Match line still covered the plan clause “do not write the shown set to localStorage, a cookie, **or a server**.”

**In order.** `localStorage.getItem`, `sessionStorage`, and `document.cookie` read the browser's client stores. `fetch`, `XMLHttpRequest`, `sendBeacon`, a form post, or an image request never touch those APIs. All three reads stay empty while a write goes out on the network.

**What can go wrong.** The record certifies absence of a server write using an instrument that cannot observe a server write. Same class of error as (4): Match on a prediction the sitting could not have failed.

**What we did with that.** The storage sitting's Match for “no server write” was left **not scored**. A different sitting attached Playwright's `page.on('request')` and scored that clause only from what the listener reported.

Leaving the box empty on the storage sitting was deliberate. Scoring “yes” there would have been the same overclaim with nicer wording. The plan still names “or a server,” so the clause had to be scored somewhere an observer can see HTTP. That is the request-log sitting, not a Match smuggled through storage.

### 6. A sitting on `/tmp/embassy.html` is evidence about that file, not about `index.html` until the copy is tied (PR #4, third finding)

**What was happening.** The request-log sitting opened `file:///tmp/embassy.html`. The record listed two document GETs of that URL and scored no-server-write for the page under test.

**In order.** Evidence applies to the artifact it was collected from. If the copy was made before the page changed, or edited while testing, every conclusion lands on the wrong file. Section 5 already knew this: it named its copy and said the shipped page was not edited. Section 4's request log did not.

**What can go wrong.** A reader takes the Match as “the shipped embassy makes no network writes.” They cannot tell whether they read a sitting on the shipped blob or on a scratch file.

**What we did with that.** We named the copy step, then **observed** equivalence: `git hash-object` of `/tmp/embassy.html` and of repo `index.html` were both `d354f6da1d895524c7a33e12da291af880ac64fc`; SHA-256 of both was `dedf3a2f…`; `cmp` reported identical. The Match is on that hashed file, after that comparison.

### 7. “Every page request recorded” names no instrument (PR #4, fourth finding)

**What was happening.** The record claimed zero `http` / `https` / `websocket` requests without saying what watched the traffic.

**In order.** DevTools, a proxy, Resource Timing, and Playwright's request listener each see different subsets. A failed or blocked write can look like zero requests to a tool that does not surface it.

**What can go wrong.** An absence claim is only as strong as the observer. Without a named instrument, a checker cannot apply the blind spots.

**What we did with that.** The sitting names Chromium driven by Playwright, instrument `page.on('request')`. Only requests that listener reported are in the sitting. Traffic Chromium does not surface there is outside the evidence. Absence is only as strong as that listener.

### 8. Byte-equivalence asserted, not observed (PR #4, fifth finding)

**What was happening.** After (6) we wrote “bytes were not edited” without recording an observation. The no-server-write Match still hung on an author's statement.

**In order.** Every other claim in that sitting was scoped to a named instrument. “Not edited” was the load-bearing link and the only one with no observer.

**What can go wrong.** A truncated or corrupted copy would still be a file at `/tmp/embassy.html`. The sitting would log two document GETs and zero HTTP, and the Match would be about the wrong bytes.

**What we did with that.** Hashes and `cmp` as in (6). Equivalence is a row in the record, not a sentence of intent.

### 9. The one-line sitting's “only change” is still asserted (PR #4, remaining low finding on approve)

**What was happening.** Section 5 shrinks `lines` in a copy and clicks six times. The record says the shipped page was not edited and that only the array was reduced. No hash of “before vs after the shrink, aside from that array” was recorded.

**In order.** Same copy-vs-page chain as (6) and (8), applied to a **modified** copy. The six identical lines are evidence about whatever file was opened. They are evidence about a one-line embassy only if the rest of the handler is the shipped handler.

**What can go wrong.** If the copy also changed the click handler, six repeats could be a stub, not the N = 1 rule (skip the seam swap, the single line is the whole cycle).

**What we did with that.** We did not silently patch the verification file in this ticket. The finding stands as a scoped residual: section 5's Match is about the one-line behaviour of the copy that was run; the “only the array changed” link is an assertion. Next verification sitting that uses a mutated copy should hash both the source blob and the mutated file and record the diff, the same way the request-log sitting recorded `cmp`.

### 10. Bash denied, working tree judged directly (PR #3 and PR #4 sessions)

**What was happening.** In some review sessions Lars could not run git. He read files as checked out and could not recompute blob ids himself.

**What can go wrong.** A hash table in the record that the reviewer cannot recompute is, from that session's point of view, another assertion. That is why later reviews asked for observed hashes **in the document**, not only in our shell.

---

## Reflection

**Where decomposition helped.** SPOTLI-001 wrote the cycle rule, the seam residual, and the tester checks **before** `index.html` changed. SPOTLI-002 then had one job: implement that rule and nothing else. When Lars asked why a deck instead of a pool, the answer was already in the plan: testers score cycle-aligned blocks; the join is “new shuffle.” We did not invent a justification after the shuffle shipped. PR #3 could be checked against `docs/verdict-selection.md` line by line (copy, index, seam swap skipped at N = 1, page memory only).

**Where it slipped.** SPOTLI-003's first refresh entry scored Match on “no storage / no server” from an empty `#verdict` after reload (verification record, original §4). Plan → build → verify slipped at **verify**: we treated a sitting that cannot falsify a clause as proof of the clause. That is the same class of error Lars had already killed in the plan (sliding window of five vs cycle-aligned blocks): scoring the wrong window, then calling it a pass. The next four review rounds on PR #4 were us discovering that the verify step needs the same discipline as the plan — name the observer, then state only what that observer can see.

**Next sprint.** Before a Match line is written, name the artifact and the instrument; if the sitting cannot fail the prediction, leave the box empty or run a sitting that can. When the sitting is not the shipped file, hash both sides and record the comparison in the same document, including mutated copies (the one-line case still owes that row).
