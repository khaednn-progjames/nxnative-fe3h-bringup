# Fire Emblem Fortune's Weave on Mac

This guide explains how this NxNative fork turns a legally obtained, already-extracted Switch 2 game dump into a standalone macOS app. It describes the software and build process; the repository does not include the game dump, executable modules, RomFS, Nintendo keys, save files, or machine-specific paths.

## What is in the repository

NxNative runs a game's ARM64 code as native code on Apple silicon. It supplies the operating-system and graphics interfaces the game expects; it is not a copy of the game and it does not provide Nintendo firmware or keys.

- `scripts/build_ipa_from_nsp.py` is the build entry point. Despite its filename, `--platform mac-app` also accepts an extracted Switch 2 folder and packages a Mac app.
- `dm/games/04007EF01F8AE000/` is the title profile. `game.tsv` describes its title, memory layout, modules, and packaged file rules. The README and shader tables document what has been tested and what remains open.
- `dm/tools/patch_nso.py` inspects and rewrites the game's executable modules where the host needs a compatible entry point or a known game-specific fix. Per-title changes are kept in the profile and tied to the exact executable build ID.
- `dm/host/` provides the Horizon services used by the game, including memory, filesystem, input, audio, and save access.
- `dm/gfx/nvn/` maps the game's NVN2 graphics calls to Metal. `dm/gfx/tools/` translates the game's shaders for Apple's graphics stack and builds pipeline caches.
- `dm/mac/` contains the native launcher and app packager. The launcher reads the app's bundled settings, starts the host, and presents the game window.
- `dm/third_party/` contains dependencies and their license notes. See [third-party notices](../dm/third_party/README.md).

## How this build came together

1. **Start with the Mac conversion guide.** The [AppleGamingWiki NxNative guide](https://www.applegamingwiki.com/wiki/Nxnative_Switch_to_Mac_conversion) was the starting reference for using NxNative to build a native Mac game app.
2. **Apply it to this game.** You then asked to adapt that workflow for Fire Emblem Fortune's Weave and supplied the game files. A keys file was part of the initial setup, but the final Switch 2 build uses the already-extracted folder described below and does not need `prod.keys`. No key values or key files are in this repository.
3. **Add the title-specific profile.** The build identifies the game by its title ID and executable build ID, then uses its profile for memory layout, modules, file packaging, and shader data. The host and Metal layer handle the game's system and graphics calls.
4. **Package and test on Mac.** `scripts/build_ipa_from_nsp.py --platform mac-app` builds the host and packages the title data, shaders, settings, and launcher into a local `.app`. Testing reached the opening movie, dialogue, first battle map, and battle scenes; findings and unresolved items are in the [title profile notes](../dm/games/04007EF01F8AE000/README.md).
5. **Correct the frame pacing.** Testing found that forcing one frame on every display refresh made the game's frame-counted logic run twice as fast. For this title, the Mac launch configuration follows the interval requested by the game (`DM_PRESENT_INTERVAL=game`). A 60 FPS change must also keep the game's simulation step synchronized; a display-only override is not enough.

## Required game files

Use files extracted from a copy of the game you own. The Switch 2 path accepts an extracted folder, so **this build path does not need `prod.keys`**. Keep the original dump and the extracted files outside this Git repository.

The expected structure is:

```text
GAME_FOLDER/
  base/
    exefs/       executable modules and metadata
    romfs/       the base game's read-only data
  updates/
    VERSION_NUMBER/
      exefs/     update modules
      romfs/     the resolved, updated RomFS
  dlc/
    TITLE_ID/0/romfs/
```

The `updates/` and `dlc/` directories are optional. For this game, the update directory must contain the resolved RomFS expected by the packer, not an unmerged patch layer. The build script checks the folder and reports missing or unsupported inputs.

For encrypted NSP or XCI inputs from original Switch games, a different build path may require keys that the user obtains from their own console. Those keys should remain in a private local file and must never be committed or shared. They are not required for the extracted Switch 2 folder above.

## Build a Mac app

Install the Apple build tools and Python dependencies listed in the main [README](../README.md#requirements). Then run this from the repository root, replacing the example folder with your own dump location:

```sh
python3 scripts/build_ipa_from_nsp.py "/path/to/GAME_FOLDER" --platform mac-app
```

The script identifies the title profile, validates the input layout, prepares executable modules, builds the Mac host, prepares shader translation inputs and any requested pipeline cache, and packages resources into a double-clickable `.app`. The host loads and compiles translated Metal shaders as the game uses them. By default, generated output and intermediate work stay beside the game folder. Use `--out` or `--work-dir` to choose different local locations; see the script's `--help` output for all options.

The result is specific to the game version and Mac graphics hardware used to build its pipeline archive. Keep the app and generated work folders private and outside the source repository. Do not upload them: they contain copyrighted game data or derived assets.

## What happens at launch

1. The app launcher loads its bundled settings and starts the native host.
2. The host loads the game's rewritten modules and mounts the app's bundled read-only game data.
3. The host answers the game's system-service calls and receives controller input through macOS.
4. NVN2 calls are recorded by NxNative's graphics layer. Translated shaders run through Metal; the game's DLSS temporal-scaling request is handled through Apple's MetalFX temporal scaler in this Mac implementation.
5. The host writes saves and runtime caches to the user's app data area. These are machine-local data and are not part of this repository.

## Frame rate and performance

The title profile is tied to the game executable's build ID so a patch cannot silently target a different update. A 60 FPS cheat or code patch may also affect game timing. Presentation pacing must follow the title's frame interval unless a verified frame-step fix keeps simulation time correct at the higher rate. Forcing a frame on every display refresh can make frame-counted gameplay run faster.

The game renders an internal scene and asks for temporal upscaling. Lowering output resolution may reduce some display work, but does not automatically reduce the game's internal rendering cost. Shader precompilation helps first-use stutter; it does not make a GPU-bound battle render faster once those shaders are cached. Frame generation is not implemented by this project guide or implied by MetalFX upscaling.

The tested status and title-specific rendering findings are in the [game profile notes](../dm/games/04007EF01F8AE000/README.md). Testing has reached the opening movie, dialogue, first battle map, and battle scenes; later areas have not been fully validated.

## Keep private files out of Git

The root `.gitignore` excludes common console dump formats, key files, extracted game folders, saves, logs, generated builds, and app bundles. Before publishing changes, review both staged files and history. Never add:

- `prod.keys`, title keys, device certificates, or other credentials;
- NSP, XCI, NSZ, XCZ, extracted RomFS/executable files, or DLC data;
- save data, app bundles, pipeline caches, or logs containing local paths and machine details.

The repository contains the NxNative source, game profile metadata, and documentation. A person reproducing the build supplies their own legally obtained game files locally.

## More detail

- [NxNative overview and build prerequisites](../README.md)
- [Title profile and known findings](../dm/games/04007EF01F8AE000/README.md)
- [Packing script internals](development.md#the-packing-script-in-detail)
- [Graphics and shader development](agents/04-graphics.md)
- [Debugging recipes](agents/05-debugging.md)
- [Third-party licenses](../dm/third_party/README.md)
