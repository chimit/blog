---
title: Shortcut Express, or how to stop reaching for the mouse
author: Chimit
pubDatetime: 2026-09-01T20:00:00Z
featured: false
draft: false
tags:
  - shortcuts
  - hotkeys
  - productivity
  - desktop
  - macOS
  - Windows
  - Linux
  - Tauri
  - TypeScript
  - React
description: "My attempt to get the world using keyboard shortcuts."
---

I keep noticing how much people do with the mouse. The keyboard is for typing letters. Move the caret back a word, select a line, close a tab, undo something — the hand leaves the keys every time. Copy and paste are usually the only shortcuts anyone knows.

Those detours cost a second and a bit of attention each, and there are hundreds of them a day. I am no shortcut fanatic myself; I have never memorized a forty-line cheatsheet and don't intend to. But a dozen combinations is enough to make an ordinary day noticeably faster, and they are universal — the same handful works in an editor, a browser, a spreadsheet, on all three systems. Learn them once and muscle memory does the rest.

So why doesn't everyone know them? Not laziness. A list of forty combinations read once is gone by the afternoon, because none of them was ever attached to a moment when you needed it. A cheatsheet answers a question you are not asking yet.

That is why I built [Shortcut Express](https://github.com/chimit/shortcut-express), a small desktop app that makes routine work faster for ordinary computer users.

<img src="/assets/shortcut-express.gif" width="600" alt="Shortcut Express">

**It is a course, not a cheatsheet or a quiz.** Every lesson runs on something you actually operate — a document you edit, a browser with tabs and a find bar, an app that saves and prints. A keyboard at the bottom of the window lights up the keys you should press. Each chapter names a familiar problem, walks you through the faster way, then asks you to repeat it with the hints off.

**The hard part was choosing what to leave out.** Not the most shortcuts, but the smallest set that pays off every single day, in the order that makes each one stick. Three courses: text without a mouse, the browser, and the keys that mean the same thing in every application.

## Building it with AI

I wrote almost none of it by hand — it was built with Claude Code — and the interesting part is where the work moved. AI handled everything mechanical: the drawn keyboard and its per-platform legends, the CSS, the release pipeline. That is the part that would have taken weeks of evenings.

What it could not do was decide what to teach, in what order, and how many repetitions a drill needs before it stops helping and starts annoying. That is where the app lives now: in the idea, in the mechanics, and in how far they get polished. The implementation turned into the secondary question.

Which means the quality of a program is set by how stubborn its author is, not by how well they write code. It was always that way — it is just much harder to pretend otherwise now. Nothing stops you from rebuilding a lesson for the fifth time except being unwilling to.

Secondary is not the same as unimportant. I still care how the code is written and organized, so besides reading the diffs myself, I tried [Rejudge](https://rejudge.syabro.com/), a review tool that caught a fair number of real problems and inconsistencies. It was also my first serious look at the current open-source models — Kimi 3, DeepSeek v4, Qwen 3.8, GLM 5.3 — which I had not touched since Llama 3. They are far ahead of what their previous generations could do.

## Try it yourself or show it to someone who needs it

Free and open source, for Windows, macOS and Linux, built with lightweight Tauri, React and TypeScript. No backend, no accounts, no telemetry: **[github.com/chimit/shortcut-express](https://github.com/chimit/shortcut-express)**
