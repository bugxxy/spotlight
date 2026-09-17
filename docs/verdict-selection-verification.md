# Verdict selection — verification record

Checked against [`docs/verdict-selection.md`](verdict-selection.md) on 17 September 2026. Page: `index.html` on `main` (blob `d354f6da`). Control: `#report`. Output: `#verdict`.

This is a record of one sitting. A later visit will shuffle a different order. Do not treat these exact strings as the only legal sequence.

Live list (N = 5):

1. Verdict: the sock is on holiday in a dimension made of static cling.
2. Verdict: your washing machine ate it. It will not apologize.
3. Verdict: the pair is fine. It simply prefers to live separately now.
4. Verdict: classified. Also, check inside the duvet. We never said that.
5. Verdict: granted. You may wear mismatching socks in public with pride.

**Mismatches: none.** Every case below matched the plan. Nothing was changed on the page.

---

## 1. First click

**Plan:** `#verdict` starts empty. The first click shows one of the five lines. That line is used for this cycle.

**Actual:** `#verdict` was empty before the click. First click showed:

> Verdict: the sock is on holiday in a dimension made of static cling.

**Match:** yes. A verdict appeared on the first click. It is line 1 of the live list.

---

## 2. Ten consecutive clicks (two cycles of five)

**Plan:** Clicks 1–5 are five distinct lines (every line once). Clicks 6–10 are five distinct lines (every line once). Do not score a window that starts mid-cycle. Click 10 is not the same line as click 9.

**Actual:**

| Click | Cycle | `#verdict` |
|------:|------:|---|
| 1 | 1 | Verdict: the sock is on holiday in a dimension made of static cling. |
| 2 | 1 | Verdict: your washing machine ate it. It will not apologize. |
| 3 | 1 | Verdict: classified. Also, check inside the duvet. We never said that. |
| 4 | 1 | Verdict: the pair is fine. It simply prefers to live separately now. |
| 5 | 1 | Verdict: granted. You may wear mismatching socks in public with pride. |
| 6 | 2 | Verdict: classified. Also, check inside the duvet. We never said that. |
| 7 | 2 | Verdict: the sock is on holiday in a dimension made of static cling. |
| 8 | 2 | Verdict: the pair is fine. It simply prefers to live separately now. |
| 9 | 2 | Verdict: granted. You may wear mismatching socks in public with pride. |
| 10 | 2 | Verdict: your washing machine ate it. It will not apologize. |

Clicks 1–5: five different lines, the full list, no repeat.  
Clicks 6–10: five different lines, the full list, no repeat.  
Click 10 ≠ click 9.

**Match:** yes.

Clicks 3 and 6 are the same line. That window sits on the seam. The plan allows it.

---

## 3. Click after the last unshown line

**Plan:** Click 5 finishes cycle 1. Click 6 starts a new shuffle. It is not the line just shown. It may be a line from earlier in the previous cycle. A verdict is shown (no empty state).

**Actual:** Click 5 was “granted…”. Click 6 was “classified…”. That is not the line just shown. It had already appeared as click 3. `#verdict` was not blank.

The same check on the next join: click 10 was “washing machine…”. The following click (11) was “the pair is fine…”, not the line just shown, and not blank.

**Match:** yes. A new cycle began as specified.

---

## 4. Refresh

**Plan:** Reload clears `#verdict`. Shown-state is gone. The next click is a first click of a new visit. No localStorage or cookie.

**Actual:** After the ten clicks, the page was reloaded. `#verdict` was empty. The next click showed:

> Verdict: the sock is on holiday in a dimension made of static cling.

That is a legal first-click line. It matching click 1 of the previous visit is allowed: repeats across visits are accepted.

**Match:** yes.

---

## 5. One-line list

**Plan:** If the array is cut to one line, every click shows that line. Repeating it is correct. The seam constraint applies only when there are two or more lines.

**How this was run:** `lines` in a copy of `index.html` was reduced to the static-cling line only. The shipped page was not edited.

**Actual:** `#verdict` started empty. Six clicks each showed:

> Verdict: the sock is on holiday in a dimension made of static cling.

**Match:** yes.
