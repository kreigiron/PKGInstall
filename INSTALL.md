# Building and Installing PKGInstall

This guide covers prerequisites, building, running, and packaging PKGInstall on
Linux, Windows, and macOS.

## Prerequisites

PKGInstall needs a C++23 compiler, CMake 3.24 or newer, Git, and Qt6
(Widgets + Concurrent). All other dependencies (`cryptopp`, `zlib-ng`, `fmt`,
`toml11`) are vendored as git submodules under `externals/` and built
automatically when a compatible system copy is not found.

### Ubuntu / Debian

```bash
sudo apt install build-essential cmake ninja-build git qt6-base-dev qt6-tools-dev
```

For AppImage builds, CI additionally installs:

```bash
sudo apt install libx11-dev libxext-dev libwayland-dev libdecor-0-dev \
  libxkbcommon-dev libglfw3-dev libgles2-mesa-dev libfuse2 \
  qt6-multimedia-dev libasound2-dev libpulse-dev libopenal-dev libudev-dev
```

### Fedora

```bash
sudo dnf install qt6-qtbase-devel qt6-qtbase-private-devel qt6-qttools-devel
```

### Arch Linux

```bash
sudo pacman -S base-devel cmake ninja git qt6-base qt6-tools
```

### Windows

Install Visual Studio (MSVC toolchain) and Qt 6.9.3 with the `qtbase` and
`qttools` archives (`win64_msvc2022_64`). The provided
`CMakeUserPresets.json` assumes Qt is installed at the hardcoded path
`C:/Qt/6.9.2/msvc2022_64`. If your Qt is elsewhere, edit the `QTDIR` value in
that file or set `CMAKE_PREFIX_PATH` to your Qt installation.

### macOS

Install the Xcode command-line tools and Qt6. Configure with
`-DCMAKE_OSX_ARCHITECTURES=x86_64` (or `arm64` for Apple silicon).

## 1. Initialize submodules

The vendored dependencies live in git submodules. They must be fetched before
configuring, otherwise `externals/` is empty and configuration fails.

```bash
git submodule update --init --recursive
```

## 2. Configure

### Arch Linux / generic Linux

This is the verified local build path:

```bash
cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_POLICY_VERSION_MINIMUM=3.5
```

The `-DCMAKE_POLICY_VERSION_MINIMUM=3.5` flag is required when using **CMake 4.x**
because some vendored subprojects declare a minimum CMake version below 3.5, and
CMake 4 removed compatibility with those older version ranges.

If your system `fmt` is too old or incompatible, force the vendored copy:

```bash
cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_POLICY_VERSION_MINIMUM=3.5 -DCMAKE_DISABLE_FIND_PACKAGE_fmt=ON
```

### Windows

Using the provided presets (MSVC + Qt 6.9.2 — this is the path hardcoded in
`CMakeUserPresets.json`):

```powershell
cmake --preset Qt-Release
```

> **Note:** CI uses Qt 6.9.3 (`.github/workflows/build.yml`), while the committed
> `CMakeUserPresets.json` targets Qt 6.9.2. If you installed Qt 6.9.3, update the
> `QTDIR` value in the preset to match your installation.

Or configure directly, ensuring CMake can find your Qt installation via
`CMAKE_PREFIX_PATH`.

### macOS

```bash
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release -DCMAKE_OSX_ARCHITECTURES=x86_64
```

## 3. Build

```bash
cmake --build build --parallel $(nproc)
```

On Windows with a multi-config generator, add `--config Release`.

## 4. Run

```bash
./build/PKGInstall
```

On Windows the executable is under `build/Release/PKGInstall.exe`; on macOS it is
`build/PKGInstall.app`.

## Packaging

### Windows

Copy the executable, add `dist/qt.conf`, and deploy the Qt runtime with
`windeployqt`:

```bat
mkdir upload
move build\Release\PKGInstall.exe upload
copy dist\qt.conf upload\qt.conf
windeployqt --plugindir upload\qtplugins --no-compiler-runtime ^
  --no-system-d3d-compiler --no-system-dxc-compiler --dir upload upload\PKGInstall.exe
```

### macOS

```bash
mv build/PKGInstall.app upload
macdeployqt upload/PKGInstall.app
```

### Linux AppImage

The CI workflow builds the AppImage with `linuxdeploy` and
`linuxdeploy-plugin-qt`, using `dist/PKGInstall.desktop` and
`dist/PKGIcon.png`:

```bash
# Download the linuxdeploy and linuxdeploy-plugin-qt AppImages (see the
# script/CI for the exact release URLs), then:
chmod +x linuxdeploy*.AppImage
./linuxdeploy-x86_64.AppImage --appdir AppDir -d dist/PKGInstall.desktop \
  -e build/PKGInstall -i dist/PKGIcon.png --plugin qt
./linuxdeploy-x86_64.AppImage --appdir AppDir --output appimage
```

See `.github/linux-appimage-qt.sh` and `.github/workflows/build.yml` for the
exact CI steps, including the download URLs.

## Troubleshooting

### `externals/` is empty / submodule errors

The vendored dependencies were not initialized:

```bash
git submodule update --init --recursive
```

If that still fails, resync the URLs first:

```bash
git submodule sync --recursive
git submodule update --init --recursive
```

### CMake 4 policy error

Errors mentioning "Compatibility with CMake < 3.5 has been removed" come from
vendored subprojects with old `cmake_minimum_required` values. Add
`-DCMAKE_POLICY_VERSION_MINIMUM=3.5` to your configure command.

### System `fmt` incompatibility

If configuration or compilation fails on `fmt`, the system copy is likely too old
(CMake requests `fmt 10.2.0`). Force the vendored copy:

```bash
-DCMAKE_DISABLE_FIND_PACKAGE_fmt=ON
```

### Missing Qt6 tools / `uic`

`AUTOUIC` requires Qt's `uic`. Install the Qt tools package
(`qt6-tools-dev` on Ubuntu/Debian, `qt6-qttools-devel` on Fedora, `qt6-tools`
on Arch) and make sure CMake can find Qt, e.g. by setting `CMAKE_PREFIX_PATH` or
`Qt6_DIR` to your Qt installation.

### `CMakeUserPresets.json` fails on Linux or macOS

`CMakeUserPresets.json` is **Windows/MSVC-only** and hardcodes
`QTDIR=C:/Qt/6.9.2/msvc2022_64`. Do not use it on Linux or macOS; configure with
plain `cmake -S . -B build ...` commands instead (see above).

### No tests to run

PKGInstall has no automated test suite. Verification is a successful build plus a
manual GUI run.
