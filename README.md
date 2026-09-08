# Spotlight: The Lost Sock Embassy

A tiny, slightly unhinged website for socks that vanished mid-cycle and now demand diplomatic recognition.

## What this is

- `index.html` — the embassy homepage
- A red official-looking button that issues fake verdicts
- Zero backend, because the dryer already ate the database

## Verdict selection (plan, not yet coded)

The button still picks a line at random from the full list on every click. That can repeat a punchline inside three clicks.

The written rule for the next change lives in [`docs/verdict-selection.md`](docs/verdict-selection.md). Review that file on its own. It states:

- what the page does today, click by click
- the no-repeat-until-exhausted cycle rule
- what happens after the last unshown line
- what happens when there is only one line
- that a refresh resets what has been shown, and why

No code in `index.html` changes until that plan is accepted.

## Open it

Open `index.html` in a browser. That is the entire deployment plan.

## Why

Because every laundry load deserves a conspiracy theory.
