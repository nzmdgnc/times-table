# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Single-file German "Kleines Einmaleins" (small times-table) practice app. 15 problems in 60 seconds, factors 2–10, mobile-first UI with on-screen numpad. UI strings are in German.

## Running

No build, no dependencies, no package manager. Open `einmaleins-test.html` directly in a browser, or serve the directory (e.g. `python3 -m http.server`) and load the file. Loads Google Fonts (Fraunces, DM Sans) over the network — needs internet on first load.

## Architecture

Everything lives in `einmaleins-test.html`: HTML, CSS (in `<style>`), and JS (in `<script>`) in one file. Keep it that way unless explicitly asked to split — there is no toolchain to bundle anything.

Three screens swapped via the `hidden` attribute on `<div class="screen">` elements (`#start`, `#game`, `#results`). `showScreen(name)` is the single switcher.

Game state is a single module-level `state` object created in `startGame()` and consulted/mutated by `pressKey`, `submitAnswer`, `updateTimer`, `renderProblem`, `finishGame`. Key fields: `problems[]` (each `{a, b, answer, userAnswer, correct}`), `currentIdx`, `input` (string buffer for the numpad), `startTime` (`performance.now()` baseline), `timerInterval`, `finished`, `locked` (blocks input during the post-answer flash animation).

Timer is wall-clock based (`performance.now()` diffed against `startTime`), not a decremented counter — pausing/backgrounding the tab will still advance it. The 100ms interval only drives the UI.

Input rules in `pressKey`: max 3 digits, no leading zero, `back` deletes one char, `ok` submits. Submitted answers trigger a CSS flash animation (`.flash-good` / `.flash-bad`) plus `navigator.vibrate` on supported devices, then auto-advance after 280ms (correct) or 480ms (wrong).

Constants `TOTAL_TIME` (60) and `TOTAL_PROBLEMS` (15) at the top of the script — change these if the user wants different difficulty/length. Problem range is hardcoded inline in `generateProblems()` as `2..10`.

## Conventions

- Plain ES (no modules, no transpile). `const $ = sel => document.querySelector(sel)` is the only helper.
- Styling uses CSS custom properties on `:root` (`--bg`, `--ink`, `--accent`, etc.) and a "neo-brutalist" hard-shadow look (`--shadow: 0 4px 0 var(--ink)`). Reuse the variables rather than hardcoding colors.
- All user-facing strings are German; keep them German unless asked otherwise.
