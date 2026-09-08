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

### How to pick

1. At the start of a cycle, shuffle a copy of `lines` into an order. That order is the cycle. Keep an index at 0.
2. On each click, show the line at the current index, then increment the index.
3. Do not call Math.random() against the full list on each click. Take the next card.

A tester checking a fresh visit with five lines must see five distinct verdicts on the first five clicks.

---

## 3. After the last unshown line

Decision: start a new cycle on the next click.

When a click shows the last unused line of the current shuffle, the cycle is complete. The following click:

1. Builds a new shuffle of the full list.
2. If the list has more than one line, the first line of the new cycle must not be the last line just shown. Reshuffle or swap until that is true.
3. Shows that first line and continues through the new order.

There is no empty state and no come-back-later message. The embassy talks through the whole list, then talks through it again.

### When there is only one line

Decision: show that line on every click.

A one-line embassy cannot avoid repeating. The single line is the whole cycle. The do-not-open-a-new-cycle-on-the-same-line constraint applies only when there are two or more lines. It exists to stop a back-to-back repeat at the seam between cycles.

---

## 4. Refresh

Decision: a refresh resets what has been shown.

Reason: simpler. Shown-state lives in page memory for this visit only. Reload, close the tab, or open a new visit and the deck is new.

Do not write the shown set to localStorage, a cookie, or a server. The defect named in the ticket is a repeat inside one sitting, within three clicks - not a repeat across days.

This matches what the page already does with persistence: none. Only the pick rule changes.

---

## 5. What the page will show

The live list has N = 5 lines. The rule below also states N = 1 so a tester can shrink the array and still score the page.

### First click of a visit

One of the five lines. `#verdict` was empty; it now holds that line. That line is used for this cycle.

### Clicks 2 through 5 of the first cycle

Each click shows a line that has not appeared yet in this visit's cycle. After click 5, every line has appeared exactly once.

### Tenth click

Click 10 is the fifth click of the second cycle (clicks 1-5 first cycle, 6-10 second). It is a line that has not appeared in the second cycle yet, and it is not the same line as click 9. After click 10 the second cycle is complete.

### Click after the last line of a cycle

That is click 6, 11, 16, and so on - the first card of a new shuffle. It is not the line just shown.

### Every click when the list has one line

That one line. Repeating it is correct.

### After a refresh

`#verdict` is empty again. The next click is a first click. Any earlier shown-set is gone.

### Walk-through with labels A B C D E

- Click 1: say C.
- Clicks 2-5: the other four, each once, e.g. A, E, B, D.
- Click 6: new shuffle that does not start with D. Example: B, C, E, A, D.
- Refresh before click 6: memory gone. Next click is click 1 of a new visit.

A tester does not need the shuffle seed. They only check: no duplicate inside a run of five; no back-to-back repeat when there is more than one line; a one-line list stays that line; reload starts clean.
