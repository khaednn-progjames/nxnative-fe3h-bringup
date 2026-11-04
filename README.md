# NxNative: Fire Emblem Fortune's Weave for Mac

This fork documents and builds a standalone macOS app for Fire Emblem Fortune's Weave using NxNative. The build uses a game dump that you provide locally; this repository contains the host software, game profile, and build instructions, not the game itself.

NxNative runs the game's ARM64 code on Apple silicon and implements the system and graphics interfaces it needs. The host maps NVN2 graphics calls to Metal and handles input, audio, storage, and other services. See [the Mac project guide](docs/fortunes-weave-mac.md) for the full architecture and build flow.

## Requirements

- An Apple silicon Mac with macOS 26.6 or later
- Xcode 27.2 with its Metal toolchain
- Homebrew and Python 3.12 or newer (arm64)

Select Xcode, then install the build tools and Python packages:

```sh
sudo xcode-select -s /Applications/Xcode.app
xcodebuild -downloadComponent MetalToolchain
brew install xcodegen zstd glslang
brew install --cask dotnet-sdk
python3 -m pip install lz4 "capstone>=5,<6" fonttools nsz zbic "backports.zstd; python_version < '3.14'"
```

## Game files

You need a legally obtained dump of the game you want to build. Original Switch NSP/XCI workflows can require the matching update and your own local `prod.keys`. Fortune's Weave uses an extracted Switch 2 folder, so this build path does not need keys. Keep the dump, extracted files, and any keys outside this repository. The expected folder layout and file requirements are in the [Mac project guide](docs/fortunes-weave-mac.md#required-game-files).

## Build the Mac app

From the repository root, pass the path to your local game folder:

```sh
python3 scripts/build_ipa_from_nsp.py "/path/to/GAME_FOLDER" --platform mac-app
```

The app and intermediate build data are created beside the game folder by default. Keep those local outputs private; they contain game data or derived assets. See the guide for output options, the launch sequence, frame pacing, and performance details.

## How I used Codex

Codex is free on the ChatGPT Free plan. I used it as a coding assistant: I provided the [NxNative source](https://github.com/seedds/nxnative), the [AppleGamingWiki Mac conversion guide](https://www.applegamingwiki.com/wiki/Nxnative_Switch_to_Mac_conversion), and my game files locally, then asked it to adapt NxNative for Fortune's Weave. We iterated through the title profile, app build, launch errors, graphics issues, and frame pacing. The app itself was built and tested locally on the Mac; no game dump or keys came from Codex or went into this repository. [OpenAI plan details](https://learn.chatgpt.com/docs/pricing).

Here is a cleaned-up version of the prompt I used:

```text
Use the NxNative source and the AppleGamingWiki Mac conversion guide below as references. I have my own legally obtained dump of [GAME TITLE]. Check whether this title needs NSP/XCI files or an already-extracted folder, then adapt NxNative to build a native macOS app for it. Inspect the title and executable build IDs, make only the game-specific changes needed, build the app locally, and help diagnose problems from its logs and screenshots. Investigate separate 60 FPS and 120 FPS options, and verify that gameplay speed and simulation timing stay correct before calling either one working. Keep game files, keys, saves, private paths, and credentials out of GitHub. Explain the steps and identify anything that still needs testing.

NxNative source: https://github.com/seedds/nxnative
Mac conversion guide: https://www.applegamingwiki.com/wiki/Nxnative_Switch_to_Mac_conversion
Game files: [provide locally]
```

During this project's testing, I requested both 60 FPS and 120 FPS options, then removed 120 FPS and kept the 60 FPS patch. Forcing a frame on every display refresh made frame-counted gameplay run twice as fast; the Mac launch configuration now follows the game's own present interval. A high-refresh patch must preserve the game's simulation timing too.

## Project files

- `scripts/build_ipa_from_nsp.py` — validates the game folder and coordinates the build
- `dm/games/04007EF01F8AE000/` — title profile, shader map, and tested behavior
- `dm/tools/patch_nso.py` — prepares game modules for the host
- `dm/host/` — system services, input, audio, and storage
- `dm/gfx/nvn/` and `dm/gfx/tools/` — NVN2-to-Metal graphics and shader translation
- `dm/mac/` — macOS launcher and app packager
- `dm/third_party/` — dependencies and license notices

## Tested status

Mac testing has reached the opening movie, dialogue, first battle map, and battle scenes. Later areas have not been fully validated. See the [title profile notes](dm/games/04007EF01F8AE000/README.md) for graphics behavior and known limitations.

The repository does not contain the game dump, executable modules, RomFS, keys, or save data. NxNative is an independent project and is not affiliated with Nintendo or the publisher.
