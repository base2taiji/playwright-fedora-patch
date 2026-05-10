# playwright-fedora-patch
In-place patches for [Playwright](https://github.com/microsoft/playwright) and [Patchright](https://github.com/Kaliiiiiiiiii-Vinyzu/patchright) to stop crashing on Fedora/RHEL with ```apt-get: command not found```. Greasy fast-food tier code, but it works. This project contains modified files from Playwright and Patchright, both licensed under Apache-2.0. Modified portions are noted in each file.

## Problem
When running crawl4ai-setup, we're told to run two installation commands, both `playwright install --with-deps` and `patchright install --with-deps` fail on the Fedora system with:
```
sh: line 1: apt-get: command not found
```
This is because both packages hardcode apt-get for installing browser dependencies, but Fedora uses dnf as its package manager.

## Files Modified

### For Playwright:
1. **hostPlatform.js**
   - Location: `~/.local/lib/python3.14/site-packages/playwright/driver/package/lib/server/utils/hostPlatform.js`
   - Added detection for Fedora/RHEL/CentOS/Rocky/AlmaLinux distributions
   - Maps these distributions to Ubuntu equivalents for dependency compatibility

2. **dependencies.js**
   - Location: `~/.local/lib/python3.14/site-packages/playwright/driver/package/lib/server/registry/dependencies.js`
   - Added `commandExists()` function to detect available package managers
   - Modified `installDependenciesLinux()` to detect package manager and skip automatic installation on Fedora
   - Modified `validateDependenciesLinux()` to show appropriate package manager instructions

### For Patchright:
1. **hostPlatform.js**
   - Location: `~/.local/lib/python3.14/site-packages/patchright/driver/package/lib/server/utils/hostPlatform.js`
   - Same changes as Playwright version

2. **dependencies.js**
   - Location: `~/.local/lib/python3.14/site-packages/patchright/driver/package/lib/server/registry/dependencies.js`
   - Same changes as Playwright version

## Implementation Details

### hostPlatform.js changes:
- Fedora < 32 → ubuntu20.04
- Fedora 32-35 → ubuntu22.04
- Fedora 36+ → ubuntu24.04
- RHEL/CentOS 8 → ubuntu20.04
- RHEL/CentOS 9+ → ubuntu22.04/ubuntu24.04

### dependencies.js changes:
1. Detect available package manager (dnf vs apt-get)
2. For Fedora/RHEL systems:
   - Show message listing required packages
   - Skip automatic installation (since Ubuntu package names don't work with dnf)
3. For Debian/Ubuntu: Use apt-get as before
4. Error messages show appropriate install commands based on detected system

## Testing Results

### Before Fix (Fedora)
```
$ playwright install --with-deps
sh: line 1: apt-get: command not found
```

### After Fix
```
$ python3 -m playwright install --with-deps
BEWARE: your OS is not officially supported by Playwright; installing dependencies for ubuntu24.04-x64 as a fallback.
Installing dependencies...

=== Fedora/RHEL detected ===
The following packages need to be installed for browser support:

  dnf install -y libasound2t64 libatk-bridge2.0-0t64 ...

Note: Ubuntu package names may not match Fedora packages exactly.
You may need to install equivalent Fedora packages manually.

Skipping automatic installation on Fedora/RHEL.
Please install the required packages manually or use: sudo dnf install <package>

[Browser installation proceeds...]
```

Same behavior for `patchright install --with-deps`.

### Packages Needed

```
dnf install -y alsa-lib libxml2 libxslt libgbm nss at-spi2-core
```

## Installed Browsers
After running either `playwright install --with-deps` or `patchright install --with-deps`:
- Chromium (chromium-1217)
- Firefox (firefox-1511)
- WebKit (webkit-2272)
- FFmpeg (ffmpeg-1011)
