# PKGInstall

PKGInstall is a cross-platform Qt6 desktop application for extracting and
inspecting PlayStation 4 `.pkg` files and installing their contents into a
[shadPS4](https://github.com/shadps4-emu/shadPS4) install folder. Point it at a
`.pkg`, pick an output folder (and optionally a DLC folder), and it decrypts and
extracts the package into a layout that shadPS4 can use.

## Features

- Choose a PS4 `.pkg` file and extract it to a folder of your choice.
- Optional separate **DLC folder** and a **Use separate update folder** toggle.
- Dropdown of known shadPS4 install folders, with a one-click
  "Set the above folder as Output Folder" action.
- **Load shadPS4 install folders from config file** — reads a shadPS4
  `config.toml` (`GUI.installDirs` / `GUI.addonInstallDir`).
- Persistent settings (`Save Settings`) stored as TOML.
- Progress dialog and status bar during extraction.

## Supported input

- PlayStation 4 package files (`.pkg`). PKGInstall recognizes a file by its
  magic number, `0x544e437f`; unsupported files are reported as unknown.

## Building

See [INSTALL.md](INSTALL.md) for full per-OS prerequisites, build, run, and
packaging instructions. The short version on Linux:

```bash
git submodule update --init --recursive
cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_POLICY_VERSION_MINIMUM=3.5
cmake --build build --parallel $(nproc)
./build/PKGInstall
```

## Legal / usage

PKGInstall is intended only for use with **legally obtained personal dumps** of
software you own. It does not include, download, or distribute any game content
or keys. You are responsible for complying with the laws and license terms that
apply to any file you process with this tool.

## Credits

This project is built on code and knowledge from the open-source emulation
community, in particular:

- [shadPS4](https://github.com/shadps4-emu/shadPS4) — PKG/PFS parsing, crypto,
  and file I/O derive from the shadPS4 emulator project.
- yuzu — portions of the source are derived from the yuzu project.

## License

GPL-2.0-or-later. Each source file carries an SPDX header identifying the
copyright holder and the `GPL-2.0-or-later` license. See the individual file
headers for details.

Repository: https://github.com/Muggle345/PKGInstall
