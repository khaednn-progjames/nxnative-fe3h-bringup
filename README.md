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

Use an extracted folder from a game copy you own. The Switch 2 folder workflow does not need `prod.keys`. Keep the dump and extracted files outside this repository. The expected folder layout and file requirements are in the [Mac project guide](docs/fortunes-weave-mac.md#required-game-files).

## Build the Mac app

From the repository root, pass the path to your local game folder:

```sh
python3 scripts/build_ipa_from_nsp.py "/path/to/GAME_FOLDER" --platform mac-app
```

The app and intermediate build data are created beside the game folder by default. Keep those local outputs private; they contain game data or derived assets. See the guide for output options, the launch sequence, frame pacing, and performance details.

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
