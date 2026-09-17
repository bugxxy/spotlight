# Spotlight: The Lost Sock Embassy

A tiny, slightly unhinged website for socks that vanished mid-cycle and now demand diplomatic recognition.

## What this is

- `index.html` — the embassy homepage
- A red official-looking button that issues fake verdicts
- Zero backend, because the dryer already ate the database

## Verdict selection

The button deals from a shuffled copy of the five lines. No line repeats until every line in that shuffle has been shown. Then a new shuffle starts, and (when there is more than one line) it does not open on the line just shown. A refresh clears the deck; nothing is stored.

The written rule is [`docs/verdict-selection.md`](docs/verdict-selection.md).

## Open it

Open `index.html` in a browser. That is the entire deployment plan.

## Why

Because every laundry load deserves a conspiracy theory.
