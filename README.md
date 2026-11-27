# ESKANOR VOID

Single-file Windows build. Drop `ESKANOR_VOID.exe` anywhere and run it.

## Build from source

Requirements: Python 3.10+, internet access.

```
python -m pip install customtkinter pillow pyaesm urllib3 pyinstaller
python build_tool.py
```

`build_tool.py` cleans `build/` and `dist/`, bundles `Components/` and `Extras/`
as zip archives, and produces `ESKANOR_VOID.exe` next to it.

The script fails loudly if no interpreter is found or PyInstaller exits non-zero.

### Two things it does that matter

- **Stages the source under a short temp path.** PyInstaller writes a corrupt
  one-file archive when the project sits under a path with spaces in it; the
  bootloader then dies on launch with `decompression resulted in return code -3`.
  `StageSource()` copies the tree to `%TEMP%\eskanor_src_*` and builds from there,
  so the project can live anywhere, `C:\Users\...\eskanor main\ESKANOR VOID`
  included.
- **Excludes the AVIF plugin.** `PIL.AvifImagePlugin` pulls in a 7.5 MB `_avif.pyd`
  that nothing here reads. Dropping it takes the build from 21.5 MB to 17.4 MB
  and shortens the cold start.

## Release

| file | size | sha256 |
|---|---|---|
| `ESKANOR_VOID.exe` | 18211613 bytes | `C616316B31AD5D7E52A43D8AF987CEE484962B124402AEC261C90FE53CBCD618` |

Built with Python 3.12.10, PyInstaller 6.22.3, `--onefile --noconsole`.
Cold start measured at 1.31–1.41 s, ~61 MB working set.

## Consumers

`loader_online.py` pulls this file from the `main` branch of this repository:

```
PAYLOAD_URL = "https://github.com/yyyfff0003-ai/ESKANOR/raw/refs/heads/main/ESKANOR_VOID.exe"
```

Replace `PAYLOAD_URL` if the file moves. Anything over 1 MiB passes the
loader's size gate; a non-200 status retries three times.

## Notes

- Password for the builder is read from `config.json` (`"Password"`).
- License keys live in `licenses.json`.
- KeyAuth credentials live in `keyauth.json`.
- Runtime state goes to `%LOCALAPPDATA%\Microsoft\Windows\Caches\e6a1`
  (`eskanor_error.log` next to the extracted bundle).
