# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Single-file Python3 CLI tool (`ssfconv`) that converts Sogou input method `.ssf` skin files to fcitx/fcitx5 theme format. The main script is executable (shebang `#!/usr/bin/env python3`) with no file extension.

## Development Commands

```bash
# Install dependencies
pip install -r requirements.txt

# Run conversion (batch all .ssf in sougou/ directory)
./ssfconv -i

# Convert single file
./ssfconv -i 花未央.ssf

# Convert to fcitx format (default is fcitx5)
./ssfconv -t fcitx <input.ssf>

# Output as directory instead of zip
./ssfconv --no-zip <input.ssf>
```

No test framework, linter, or CI is configured.

## Architecture

Single file `ssfconv` (1271 lines) with this pipeline:

```
main(args)
  ├── extractSsf(src, dest)    # Decrypt AES-encrypted .sf or unzip plain .sf
  ├── ssf2fcitx(skin_dir)      # Generate fcitx_skin.conf from skin.ini
  └── ssf2fcitx5(skin_dir)     # Generate theme.conf from skin.ini
```

### Key Functions

- `extractSsf()` (line 18): Handles two .ssf formats — AES-CBC encrypted (header `b'Skin'`) and plain zip. AES key/IV hardcoded at lines 29-34.
- `ssf2fcitx()` (line 181): Reads `skin.ini` (UTF-16), writes `fcitx_skin.conf` (UTF-8). Uses `CaseSensitiveConfigParser` subclass to preserve key casing.
- `ssf2fcitx5()` (line 710): Same pattern, writes `theme.conf` for fcitx5.
- `main()` (line 1159): CLI entrypoint via argparse. Handles batch mode (no `src` arg → process all `.ssf` in `sougou/`), auto-generates output paths, optional zip packaging and fcitx5 install.
- Helper functions: `getImageAvg()` (pixel average with numpy), `rgbDist()`/`rgbDistMax()` (color distance), `savePolygon()` (generate arrow images), `getImageSize()`.

### Output Formats (`-t` flag)

| Format   | Output                    |
|----------|---------------------------|
| `fcitx5` | `theme.conf` (default)    |
| `fcitx`  | `fcitx_skin.conf`         |
| `dir`    | Raw unpacked directory    |

## Critical Technical Details

- **skin.ini encoding**: Always UTF-16 (not UTF-8). ConfigParser must read with `encoding='utf-16'`.
- **Output encoding**: All config files written in UTF-8.
- **Color format**: skin.ini stores colors as little-endian hex integers (e.g., `0x00BBGGRR`). Conversion at `colorConv()` function. Output uses `r g b` space-separated format for fcitx, `#rrggbb` hex for fcitx5.
- **CaseSensitiveConfigParser**: Subclass of `configparser.ConfigParser` that overrides `optionxform()` to preserve key casing — required because fcitx/fcitx5 config keys are case-sensitive.
- **Symlinks**: fcitx conversion creates symlinks to `/usr/share/fcitx/skin/default/active.png` and `inactive.png` — requires fcitx installed at that path.
- **Temp cleanup**: When source is a file, `tempfile.mkdtemp()` is used and cleaned up via `shutil.rmtree()` after conversion.
- **`encrypted` output type**: Stubbed with `assert False` at line 1216 — not implemented.
- **Dead code**: Lines 1269-1271 after `exit(main(args))` are unreachable legacy code.

## Directory Structure

- `sougou/` — Input directory for .sf files (gitignored)
- `output/` — Output directory for converted themes (gitignored)
- `venv/` — Python virtual environment (gitignored)
