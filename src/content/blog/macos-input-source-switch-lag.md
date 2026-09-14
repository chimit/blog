---
title: How I finally fixed the keyboard language switching delay on macOS
author: Chimit
pubDatetime: 2026-09-13T16:00:00Z
featured: false
draft: false
tags:
  - macOS
  - keyboard
  - productivity
description: "For years, switching the input source with Caps Lock or Cmd+Space lagged on my Mac. It turned out to be the little language popup next to the cursor."
---

If you type in more than one language on a Mac, you may know this: you switch the input source, start typing, and the first word comes out in the wrong language. The switch does happen, just a fraction of a second too late.

For me it lasted four or five years. First it was Caps Lock: once I set it to switch languages, the switch started to lag. Complaints about exactly that go back at least to [Big Sur 11.5 in August 2021](https://discussions.apple.com/thread/253035209). At some point I moved to Cmd+Space, which was fine for a while, until it started to lag too, around the time [Sonoma](https://discussions.apple.com/thread/255161577) came out in 2023.

## Everything that didn't help

I searched for a fix every few months. A typical example is [this Reddit thread](https://www.reddit.com/r/mac/comments/r2odrh/), started in 2021 and still getting replies five years later, along with its [sibling in r/MacOS](https://www.reddit.com/r/MacOS/comments/15qviy4/). Between them they collect pretty much every known remedy:

- switch with Caps Lock instead of the Globe key, or the other way round;
- swap "Select the previous input source" (Ctrl+Space) and "Select next source in Input menu" (Ctrl+Option+Space), since the first one supposedly waits on purpose;
- uncheck the Quick Note shortcut (Globe+Q), so macOS stops waiting for a second key, or set "Press Globe key to" to "Do Nothing";
- disable and re-enable the Siri keyboard shortcut, or turn Siri off entirely;
- remove the Caps Lock delay with `hidutil property --set '{"CapsLockDelayOverride":0}'`;
- try a new user account or safe mode, as Apple Support suggests.

Some of these do help people, mostly with the Globe key. I tried all of them, and none made a difference for Caps Lock or Cmd+Space.

The rest of the advice comes down to installing something: Caramba Switcher, Karabiner, CommandShift, FnLangSwitch, CapsLockNoDelay. I didn't go there on principle. Switching keyboard languages is a basic OS function, and fixing it shouldn't mean running third-party software in the background.

## The actual cause

The culprit turned out to be the language indicator, the small bubble with the current input source that pops up next to the text cursor every time you switch. Showing it is what slows the switch down. Someone in the r/MacOS thread came close in 2023: after Sonoma, the language changes only once the popup disappears, while in Ventura and Monterey it changed as soon as the popup appeared. Turn it off, and switching is instant again, with both Caps Lock and Cmd+Space.

There is no checkbox for it in System Settings, but there is a hidden preference:

```bash
defaults write kCFPreferencesAnyApplication TSMLanguageIndicatorEnabled -int 0
```

Log out or restart after running it. The current language is still shown in the menu bar, so you lose nothing useful.

If you ever want the indicator back:

```bash
defaults delete kCFPreferencesAnyApplication TSMLanguageIndicatorEnabled
```

I've also posted the fix as [a comment in that Reddit thread](https://www.reddit.com/r/mac/comments/r2odrh/comment/p9p7qd3/). If it helped you, give it an upvote there, so it rises above years of Siri and Quick Note tips and the next person with the same problem finds it right away.
