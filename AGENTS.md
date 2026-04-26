# ssfconv — AGENTS.md

Single-file Python3 CLI tool that converts Sogou `.ssf` skin files to fcitx/fcitx5 format.

## Entrypoint

- `/ssfconv` — both the source code and the executable (shebang `#!/usr/bin/env python3`)
- `main(args)` called at bottom via argparse; `type` choices: `fcitx` (default), `fcitx5`, `dir`, `encrypted`, `zip`

## Dependencies

```sh
pip install pycryptodome pillow numpy
```

No package manager / lockfile; install directly with pip.

## Key Architecture

- AES key and IV hardcoded at `ssfconv:29-34` for decrypting `.ssf` files (header magic `b'Skin'`)
- `extractSsf()` → `ssf2fcitx()` or `ssf2fcitx5()` — both read `skin.ini` (UTF-16), write `fcitx_skin.conf` or `theme.conf`
- Uses `CaseSensitiveConfigParser` subclass to preserve key casing for fcitx/fcitx5 config format
- Generates symlinks to `/usr/share/fcitx/skin/default/active.png` and `inactive.png` — fcitx install required at that path

## No Tests / No Lint / No CI

No test framework, no linter config, no CI workflows. Run manually:
```sh
python3 ssfconv <input.ssf> <output_dir>
python3 ssfconv -t fcitx5 <input.ssf> <output_dir>
```

## Gotchas

- `skin.ini` uses **UTF-16** encoding (not UTF-8) — ConfigParser must read with `encoding='utf-16'`
- Output config files are written in UTF-8
- Temp directory cleanup: when source is a file, `tmp_dir` is created via `tempfile.mkdtemp()` and removed via `shutil.rmtree()` after conversion
- `encrypted` output type is stubbed with `assert False` — not implemented
