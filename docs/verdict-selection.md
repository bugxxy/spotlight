# Verdict selection plan

Written before any code change. A tester can score the page from this file alone.

The page is `index.html`. The control is the button `#report` ("Request diplomatic immunity"). The output is `#verdict`. There are five embassy lines in the `lines` array.

---

## 1. Current behaviour

### What the page does on each click, in order

1. Visitor loads `index.html`. `#verdict` is empty. No line has been shown.
2. Visitor clicks `#report`.
3. The handler runs `lines[Math.floor(Math.random() * lines.length)]`.
4. That string is written into `#verdict`.
5. The handler ends. Nothing is stored. The next click repeats steps 3-4 against the full list.

There is no list of shown lines. There is no cycle. There is no check against the line already on screen.

### What can go wrong

- The same line can appear on click 1 and click 2.
- With five lines, a repeat inside three clicks is common.
- The visitor cannot tell a stuck button from an unlucky roll.
- After every line has appeared at least once, the page has no notion that the embassy has said everything. It just keeps rolling.
- A one-line list (if the array were ever cut to one) would look the same as a five-line list that happened to repeat: the mechanism does not special-case either.

### Refresh today

Reload clears `#verdict` because the string lives only in the DOM. The next click is another independent roll. There is no localStorage and no cookie. That part does not need to change; the defect is the roll, not the lack of memory across visits.

---

## 2. Selection rule

Treat the five lines as a deck.

Guarantee: no verdict repeats until every verdict has been shown once in the current cycle.

That guarantee is scoped to one cycle. It is not a promise about every sliding window of five clicks in the visit.

### How to pick

1. At the start of a cycle, shuffle a copy of `lines` into an order. That order is the cycle. Keep an index at 0.
2. On each click, show the line at the current index, then increment the index.
3. Do not call Math.random() against the full list on each click. Take the next card.

A tester checking a fresh visit with five lines must see five distinct verdicts on clicks 1-5. The same check applies to clicks 6-10, 11-15, and every later cycle-aligned block of five. Do not run that check on a block that starts mid-cycle.

---

## 3. After the last unshown line

Decision: start a new cycle on the next click.

When a click shows the last unused line of the current shuffle, the cycle is complete. The following click:

1. Builds a new shuffle of the full list.
2. If the list has more than one line, the first line of the new cycle must not be the last line just shown. Reshuffle or swap until that is true.
3. Shows that first line and continues through the new order.

There is no empty state and no come-back-later message. The embassy talks through the whole list, then talks through it again.

### Seam residual (explicit)

The join constraint compares the new cycle's first line against one line only: the last line shown. Every other recent line is a legal opener.

Decision: that residual is allowed.

Reason: a new shuffle has to be free to reuse lines from earlier in the previous cycle. Forbidding only the immediately previous line stops a back-to-back repeat. It does not stop a repeat inside a three-click or five-click window that sits on the join.

With N = 5, clicks 5-7 can be D, B, D. That is a conforming page. Clicks 2-6 in the walkthrough below are A, E, B, D, B. That is also conforming. A tester who fails the page for those windows is applying the cycle rule to the wrong window.

Do not tighten the join further in this plan. Killing every short-window repeat at the seam would force the next cycle to avoid several recent lines, or to replay the same order. That is a different rule. This plan keeps "new shuffle, no back-to-back."

### When there is only one line

Decision: show that line on every click.

A one-line embassy cannot avoid repeating. The single line is the whole cycle. The do-not-open-a-new-cycle-on-the-same-line constraint applies only when there are two or more lines. It exists to stop a back-to-back repeat at the seam between cycles.

---

## 4. Refresh

Decision: a refresh resets what has been shown.

Reason: simpler. Shown-state lives in page memory for this visit only. Reload, close the tab, or open a new visit and the deck is new.

Do not write the shown set to localStorage, a cookie, or a server. Repeats across visits are accepted. Repeats inside one cycle of one sitting are not. Repeats that only appear because a new cycle started are accepted, as section 3 states.

This matches what the page already does with persistence: none. Only the pick rule changes.

---

## 5. What the page will show

The live list has N = 5 lines. The rule below also states N = 1 so a tester can shrink the array and still score the page.

### First click of a visit

One of the five lines. `#verdict` was empty; it now holds that line. That line is used for this cycle.

### Clicks 2 through 5 of the first cycle

Each click shows a line that has not appeared yet in this visit's first cycle. After click 5, every line has appeared exactly once in clicks 1-5.

### Tenth click

Click 10 is the fifth click of the second cycle (clicks 1-5 first cycle, 6-10 second). Inside clicks 6-10 the five lines are each shown once. Click 10 is not the same line as click 9. After click 10 the second cycle is complete.

### Click after the last line of a cycle

That is click 6, 11, 16, and so on - the first card of a new shuffle. It is not the line just shown. It may be a line from earlier in the previous cycle.

### Every click when the list has one line

That one line. Repeating it is correct.

### After a refresh

`#verdict` is empty again. The next click is a first click. Any earlier shown-set is gone.

### Walk-through with labels A B C D E

- Click 1: say C.
- Clicks 2-5: the other four, each once, e.g. A, E, B, D.
- Click 6: new shuffle that does not start with D. Example: B, C, E, A, D.
- Clicks 1-5 = C, A, E, B, D - five distinct. Pass.
- Clicks 6-10 = B, C, E, A, D - five distinct. Pass.
- Clicks 2-6 = A, E, B, D, B - B twice. Pass. This window sits on the seam; the cycle rule does not apply to it.
- Clicks 5-7 = D, B, C - no back-to-back. If the opener had been B and click 7 were D, D, B, D would also pass.
- Refresh before click 6: memory gone. Next click is click 1 of a new visit.

### Tester checks (only these)

A tester does not need the shuffle seed. Score the page against this list and nothing else:

1. In each cycle-aligned block of N clicks (1-5, 6-10, 11-15, ...) every line appears once. Do not score a sliding window that starts mid-cycle.
2. No back-to-back repeat when N > 1, including the click that opens a new cycle.
3. When N = 1, every click is that one line.
4. After a reload, `#verdict` is empty and the next click starts a new cycle.
