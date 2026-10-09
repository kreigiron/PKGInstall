# AGENTS.md

Agent-oriented guide to **PKGInstall** — a cross-platform Qt6 desktop GUI for
extracting and inspecting PlayStation 4 `.pkg` files and installing them into a
[shadPS4](https://github.com/shadps4-emu/shadPS4) install folder.

## Project Summary

- **Language / standard:** C++23.
- **Build system:** CMake (minimum 3.24), Ninja recommended.
- **GUI:** Qt6 Widgets + Concurrent, configured with `AUTOUIC`, `AUTOMOC`, `AUTORCC`.
- **Third-party libraries:** Crypto++ (`cryptopp`), zlib (`zlib-ng`), `fmt`, `toml11`.
- **License:** GPL-2.0-or-later. Every source file carries an SPDX header. Code is
  derived from the shadPS4 emulator project and yuzu.
- **No automated test suite.** Verification is a successful build plus a manual GUI run.

## Layout Map

```
main.cpp                 Application entry point (QApplication + MainWindow)
mainWindow.cpp/.h/.ui    Qt6 main window: PKG picker, output/DLC folders, shadPS4
                         install-folder dropdown, config loading, settings, extract
CMakeLists.txt           Top-level build; targets PKGInstall
CMakePresets.json        Qt Creator configure preset (hidden "Qt" base)
CMakeUserPresets.json    Windows/MSVC-only presets (QTDIR=C:/Qt/6.9.2/msvc2022_64)
.clang-format            Code style used by CI
.ci/clang-format.sh      Lint: clang-format-19 + trailing-whitespace check
.github/workflows/build.yml   CI: Windows, Linux, Linux AppImage, macOS
.github/linux-appimage-qt.sh  AppImage helper (linuxdeploy + Qt plugin)
dist/                    PKGIcon.icns/.png, PKGInstall.ico/.rc/.desktop, qt.conf
externals/               Vendored dependencies (git submodules) + CMakeLists.txt
src/                     Core logic (see below)
```

### `src/` modules

| Module | Responsibility |
|--------|----------------|
| `loader.cpp/.h` | File-type detection by magic; `PkgMagic = 0x544e437f` |
| `pkg.cpp/.h` | `PKGHeader`, `PKGEntry`, `PKG` (`Open`, `Extract`, `ExtractFiles`, `GetTitleID`, `GetPkgSize`, `GetPkgFlags`, `GetPkgHeader`, `GetNumberOfFiles`) |
| `pkg_type.cpp/.h` | PKG content-type helpers |
| `pfs.h` | PFS filesystem structures |
| `psf.cpp/.h` | `param.sfo` parsing |
| `crypto.cpp/.h`, `keys.h` | Crypto++-based decryption and keys |
| `io_file.cpp/.h` | Cross-platform file I/O |
| `nt_api.cpp/.h` | Windows-only NT API (guarded by `#ifdef _WIN32`) |
| `types.h`, `endian.h`, `enum.h`, `concepts.h`, `alignment.h` | Support headers |

### `externals/` submodules

Built only when the corresponding target is not already found on the system
(see `externals/CMakeLists.txt`).

| Submodule | Upstream |
|-----------|----------|
| `cryptopp` | https://github.com/Muggle345/cryptopp |
| `cryptopp-cmake` | https://github.com/Muggle345/cryptopp-cmake |
| `fmt` | https://github.com/shadps4-emu/ext-fmt |
| `zlib-ng` | https://github.com/shadps4-emu/ext-zlib-ng |
| `toml11` | https://github.com/ToruNiina/toml11 |

## Build

```bash
git submodule update --init --recursive
cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_POLICY_VERSION_MINIMUM=3.5
cmake --build build --parallel $(nproc)
./build/PKGInstall
```

The `-DCMAKE_POLICY_VERSION_MINIMUM=3.5` flag is required with CMake 4.x because
some vendored subprojects declare a minimum CMake version below 3.5. If a system
`fmt` is too old or incompatible, force the vendored copy with
`-DCMAKE_DISABLE_FIND_PACKAGE_fmt=ON`.

## Lint

```bash
bash .ci/clang-format.sh
```

The script uses `clang-format-19` and fails on trailing whitespace in `src`,
`*.yml`, `*.txt`, `*.md`, `Doxyfile`, `.gitignore`, `.gitmodules`, `.ci*`,
`dist/*.desktop`, `dist/*.svg`, and `dist/*.xml`.
A CMake `clang-format` target also exists when configured with
`-DCLANG_FORMAT=clang-format-19`.

## Conventions

- C++23; follow the existing `.clang-format` style.
- Use `#pragma once` in headers.
- Keep the SPDX header on every source file (`GPL-2.0-or-later`).
- Never introduce trailing whitespace (checked by CI).
- Do **not** modify the contents of submodules under `externals/`.
- Keep platform-specific code behind the existing guards (e.g. `#ifdef _WIN32`).

## Caveats

- **`CMakeUserPresets.json` is Windows/MSVC-only.** It hardcodes
  `QTDIR=C:/Qt/6.9.2/msvc2022_64` and must not be used on Linux or macOS.
- **Windows code is guarded by `#ifdef _WIN32`** (notably `src/nt_api.*`); it is
  not compiled on other platforms.
- **Submodules must be initialized first**, otherwise `externals/` is empty and
  configuration fails.
- **No test suite.** Do not assume tests exist; verify changes by building and
  running the GUI manually.
