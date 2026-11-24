# Smash Ultimate local wireless test on Mac

This is an experimental test of nxnative pull request [#846](https://github.com/seedds/nxnative/pull/846). The networking tests pass, but a Smash local wireless match has not yet been verified. This is a standalone nxnative build, not an NX2Apple feature. Do not send anyone your game files or keys.

Both players need an Apple silicon Mac, their own game files and console keys, and the same Smash update. The profile tested here is Smash Ultimate 13.0.5 (`v2031616`). The two Macs must be on the same local network and able to reach each other. Local wireless supports two to four players, but start with two.

## 1. Install build tools

The upstream build requires macOS 26.6 or later, Xcode 27.2, its Metal toolchain, Homebrew, Python 3.12 or newer and .NET 10. Follow the [upstream setup](README.md#build-from-the-command-line) if these are not already installed. In Terminal:

```sh
xcodebuild -version
xcodebuild -downloadComponent MetalToolchain
brew install python@3.12 glslang zstd xcodegen
brew install --cask dotnet-sdk
```

## 2. Check out the experimental engine

Once the fork owner has pushed the `codex/ldn-smash-guide` branch, replace `GITHUB_USERNAME` with that owner's GitHub username:

```sh
git clone --branch codex/ldn-smash-guide https://github.com/GITHUB_USERNAME/nxnative.git nxnative-smash-ldn
cd nxnative-smash-ldn
git config core.hooksPath .githooks
```

This branch is based on PR revision `63fffb940cdb752c7895cd8e3d1548be15b8f85b`. Both players should use the same branch revision. This guide cannot be used from the fork until its owner creates the fork and pushes the branch.

## 3. Build your game

Replace the two example paths with the locations on your Mac. Put your own base game and 13.0.5 update in the game folder. The build can use NSP, NSZ, XCI or XCZ packages. If you want DLC, follow the upstream build documentation to include your own DLC files. Do not put game files, keys or extracted assets in the repository.

```sh
GAME_DIR="$HOME/Games/MySmashFiles"
KEYS_FILE="$HOME/Games/prod.keys"
TEST_DIR="$GAME_DIR/smash-ldn-test"
mkdir -p "$TEST_DIR"
python3.12 -m venv "$TEST_DIR/venv"
"$TEST_DIR/venv/bin/python" -m pip install lz4 'capstone>=5,<6' fonttools nsz zbic backports.zstd

"$TEST_DIR/venv/bin/python" scripts/build_ipa_from_nsp.py "$GAME_DIR" \
  -k "$KEYS_FILE" --platform mac-app \
  --bundle-id org.nxnative.smash-ldn-test \
  --work-dir "$TEST_DIR/work" \
  -o "$TEST_DIR/Smash LDN Test.app" --progress lines
```

The first build can take a long time and substantial disk space. The output is separate from any existing Smash app. Both Macs must complete their own builds using their own files.

## 4. Launch with local wireless enabled

From Terminal, set `GAME_DIR` and `TEST_DIR` as above if this is a new session. The save, temporary files and engine data below are separate test directories. The upstream app still uses the normal title-ID log location and may rotate a previous log.

```sh
mkdir -p "$TEST_DIR/run/save" "$TEST_DIR/run/temp" "$TEST_DIR/run/root"
DM_NETWORK=ldn DM_SAVE="$TEST_DIR/run/save" DM_TEMP="$TEST_DIR/run/temp" \
DM_ROOT="$TEST_DIR/run/root" DM_LDN_TRACE=1 DM_NET_TRACE=1 \
  "$TEST_DIR/Smash LDN Test.app/Contents/MacOS/nxnative"
```

Allow local network or firewall access if macOS requests it. Repeat on both Macs. In Smash, open **Local Wireless**, create a room on one Mac and try to join from the other. Record whether you can select fighters, finish a match and return to the lobby. If it fails, note which step failed and share the relevant log only after reviewing it for private paths and network addresses. The default log is `~/Library/Logs/nxnative/01006A800016E000.log`.

This PR enables local IP networking, not Nintendo's official online service or a promise of compatibility with other emulators. A successful build or room listing does not prove that gameplay works.
