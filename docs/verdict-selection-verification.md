# Verdict selection — verification record

Checked against [`docs/verdict-selection.md`](verdict-selection.md) on 17 September 2026. Page: `index.html` on `main` (blob `d354f6da`). Control: `#report`. Output: `#verdict`.

This is a record of sittings on that page. A later visit will shuffle a different order. Do not treat these exact strings as the only legal sequence.

Live list (N = 5):

1. Verdict: the sock is on holiday in a dimension made of static cling.
2. Verdict: your washing machine ate it. It will not apologize.
3. Verdict: the pair is fine. It simply prefers to live separately now.
4. Verdict: classified. Also, check inside the duvet. We never said that.
5. Verdict: granted. You may wear mismatching socks in public with pride.

**Mismatches: none.** Nothing was changed on the page.

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

**Plan:** Reload clears `#verdict`. Shown-state is gone. The next click is a first click of a new visit. Do not write the shown set to localStorage, a cookie, or a server.

### What the first sitting could not test

After the ten clicks above, the page was reloaded. `#verdict` was empty, then one click showed the static-cling line.

That empty `#verdict` is also what a first load of `index.html` always shows. It does not prove shown-state was discarded. It also does not prove there is no localStorage, cookie, or server write.

### Follow-up sitting (mid-cycle reload, client storage)

Three clicks, then reload, then five clicks. Storage was read after the three clicks, after reload, and after the five clicks.

**Actual, before reload:**

1. Verdict: your washing machine ate it. It will not apologize.
2. Verdict: granted. You may wear mismatching socks in public with pride.
3. Verdict: classified. Also, check inside the duvet. We never said that.

`localStorage` keys: none. `sessionStorage` keys: none. `document.cookie`: empty.

**Actual, after reload:** `#verdict` empty. Same empty storage.

**Actual, five clicks after reload:**

1. Verdict: the pair is fine. It simply prefers to live separately now.
2. Verdict: classified. Also, check inside the duvet. We never said that.
3. Verdict: granted. You may wear mismatching socks in public with pride.
4. Verdict: the sock is on holiday in a dimension made of static cling.
5. Verdict: your washing machine ate it. It will not apologize.

Those five are the full list, each once. The second of them (“classified…”) was already used in the three clicks before reload, so this is not the remainder of the old cycle. Storage was still empty after the five clicks.

Client storage reads cannot observe a network write. Empty `localStorage`, `sessionStorage`, and `document.cookie` are not evidence that the shown set was not sent to a server.

**Match, shown-state after reload:** yes, on this sitting. The first sitting’s empty `#verdict` after reload is not used as proof of reset.

**Match, no localStorage / sessionStorage / cookie:** yes, on the storage reads above.

**Match, no server write:** not scored from this sitting.

### Follow-up sitting (request log)

Three clicks, reload, five clicks, with every page request recorded.

**Actual requests:**

| Method | URL | Type |
|---|---|---|
| GET | `file:///tmp/embassy.html` | document (first load) |
| GET | `file:///tmp/embassy.html` | document (reload) |

Zero `http`, `https`, or `websocket` requests. Clicks did not add requests.

**Match, no server write:** yes, on this sitting. Storage reads are not used as proof of it.

---

## 5. One-line list

**Plan:** If the array is cut to one line, every click shows that line. Repeating it is correct. The seam constraint applies only when there are two or more lines.

**How this was run:** `lines` in a copy of `index.html` was reduced to the static-cling line only. The shipped page was not edited.

**Actual:** `#verdict` started empty. Six clicks each showed:

> Verdict: the sock is on holiday in a dimension made of static cling.

**Match:** yes.
