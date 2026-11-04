# NxNative: Fire Emblem Fortune's Weave for Mac

This fork documents how I used Codex and NxNative to build a native Mac app for Fire Emblem Fortune's Weave.

## What you need

- Codex. It is free on the ChatGPT Free plan ([plan details](https://learn.chatgpt.com/docs/pricing)).
- The [NxNative source repository](https://github.com/seedds/nxnative).
- The [AppleGamingWiki Mac conversion guide](https://www.applegamingwiki.com/wiki/Nxnative_Switch_to_Mac_conversion).
- A legally obtained dump of the game, provided locally to Codex.

For Fortune's Weave, use the extracted Switch 2 game folder. That build path does not need `prod.keys`. Other encrypted NSP/XCI inputs may require your own keys kept locally. The game files and keys are not included in this repository.

## The simple workflow

1. Give Codex the NxNative repository link and the Mac conversion guide.
2. Provide access to your own game dump on the Mac.
3. Give Codex the prompt below and ask it to build the Mac app.

Codex can inspect the project, check the Mac's build setup, find the right build steps, and work through errors with you. You can describe the goal in plain language instead of typing build commands. The NxNative page is a guide; it does not host the build or bundle Xcode and other tools. If something needed is missing from the Mac, Codex can identify it and explain what to install.

## Prompt

This is a cleaned-up version of the request I used:

```text
Use the NxNative repository and the AppleGamingWiki Mac conversion guide below as references. I have my own legally obtained dump of [GAME TITLE] available on this Mac. Check what format this game needs, adapt NxNative for it, and build a native Mac app. Check that the app launches and help fix issues using its logs. Investigate separate 60 FPS and 120 FPS options, and verify that gameplay speed stays correct before saying either works. Keep game files, keys, saves, private paths, and credentials out of GitHub. Explain the steps in plain language and tell me if anything is missing.

NxNative: https://github.com/seedds/nxnative
Mac guide: https://www.applegamingwiki.com/wiki/Nxnative_Switch_to_Mac_conversion
Game dump: [provide locally]
```

In my testing, I later removed the 120 FPS option and kept 60 FPS. Forcing one game frame per display refresh had made gameplay run twice as fast, so the Mac launch configuration follows the game's own frame interval. A higher frame rate needs the game's simulation timing to stay synchronized too.

## Tested status

Mac testing reached the opening movie, dialogue, first battle map, and battle scenes. Later areas have not been fully validated. See the [game profile notes](dm/games/04007EF01F8AE000/README.md).

This repository contains NxNative code and game profile metadata, not the game dump, executable modules, RomFS, keys, or save data. NxNative is independent and is not affiliated with Nintendo or the publisher.
