# OrangeFox Universal Builder

A GitHub Actions workflow that builds **OrangeFox Recovery** for any device. You supply your own device tree at run time, so one workflow works for every device.

## Features

- Works with any device tree, public or private
- Supports any OrangeFox branch (14.1, 12.1, 11.0, ...)
- Extra trees support (common, vendor, kernel, etc.)
- Choose the build target: `vendorbootimage`, `bootimage` or `recoveryimage`
- Publishes the result to GitHub Releases, or as an artifact
- Optional pre-build command and custom lunch settings

## Setup

1. Fork or create a repository for the builder
2. Go to **Settings -> Actions -> General -> Workflow permissions** and select **Read and write permissions**.
3. If you push the workflow file from a terminal using a personal access token, the token needs the `workflow` scope.

## Usage

1. Open the **Actions** tab.
2. Select **OrangeFox Universal Builder**.
3. Click **Run workflow** and fill in the inputs.
4. Wait for the build to finish (usually around 1 hour).
5. Download the result from **Releases** (or **Artifacts** if `RELEASE` is off).

## Inputs

### Required

| Input | Description | Example |
|---|---|---|
| `MANIFEST_BRANCH` | OrangeFox branch | `14.1` |
| `DEVICE_TREE` | URL of your device tree | `https://github.com/user/recovery_device_vendor_codename` |
| `DEVICE_TREE_BRANCH` | Branch of the device tree | `fox_14.1` |
| `DEVICE_PATH` | Where the tree is placed in the source | `device/vendor/codename` |
| `DEVICE_NAME` | Device codename | `codename` |
| `LUNCH_PREFIX` | Lunch prefix, `twrp` or `omni` | `twrp` |
| `LUNCH_VARIANT` | Lunch variant. `auto` uses `ap2a-eng` for 14.x and `eng` for older branches | `auto` |
| `BUILD_TARGET` | Make target | `vendorbootimage` |
| `JAVA_VERSION` | JDK version | `17` |
| `RELEASE` | Upload to Releases instead of artifact | `true` |

### Optional

| Input | Description |
|---|---|
| `EXTRA_TREES` | Additional trees, one per line, in the format `URL BRANCH PATH` |
| `PRE_BUILD_CMD` | Shell command run in the source root before building |

### Choosing the right values

| Android base | `MANIFEST_BRANCH` | `JAVA_VERSION` |
|---|---|---|
| Android 14 | `14.1` | `17` |
| Android 12 | `12.1` | `11` |
| Android 11 | `11.0` | `11` |

| Device partition layout | `BUILD_TARGET` |
|---|---|
| Uses `vendor_boot` for recovery | `vendorbootimage` |
| Recovery inside `boot` | `bootimage` |
| Has a dedicated `recovery` partition | `recoveryimage` |

Check your `AndroidProducts.mk` for the product name. If it is `omni_codename`, set `LUNCH_PREFIX` to `omni`.

## Private repositories

1. Create a personal access token with `repo` access.
2. Add it as a repository secret named `TREE_TOKEN` in **Settings -> Secrets and variables -> Actions**.

The workflow uses it automatically for the device tree and every entry in `EXTRA_TREES`. Only `https://` URLs are supported.

An optional `SSH_KEY` secret is also supported. When present, it is loaded into the SSH agent before syncing.

## Output

Files copied from `out/target/product/<codename>/`:

- `OrangeFox*.zip`
- `OrangeFox*.img`
- `vendor_boot.img`, `boot.img`, `recovery.img` (whichever the build produced)

The release tag is `fox-<codename>-<run number>`.

## Flashing

Your bootloader must be unlocked. Back up the original partition first.

```bash
# vendor_boot target
fastboot flash vendor_boot vendor_boot.img

# boot target
fastboot flash boot boot.img

# recovery target
fastboot flash recovery recovery.img

fastboot reboot recovery
```

Flashing steps can differ per device, so follow your device's own instructions when they exist.

## Troubleshooting

| Problem | Fix |
|---|---|
| `lunch` fails with an unknown product | Check the product name in `AndroidProducts.mk`, then adjust `LUNCH_PREFIX` and `DEVICE_NAME` |
| Clone fails with an authentication error | The tree is private: add the `TREE_TOKEN` secret |
| Release upload fails with a 403 error | Enable **Read and write permissions** for workflows |
| Push of the workflow file is rejected | Your token needs the `workflow` scope |
| Build fails with a Java error | Match `JAVA_VERSION` to the branch (17 for 14.1, 11 for older) |
| "Tidak ada output" at the output step | The build failed earlier: read the **Building recovery** log |
| Build runs out of disk or memory | Remove unneeded trees and check that the swap step ran |

## Notes

- Builds run on GitHub-hosted `ubuntu-22.04` runners with a 6 hour limit.
- Use at your own risk. The author is not responsible for bricked devices or data loss.

## Credits

- [OrangeFox Recovery Project](https://orangefox.download)
- [TeamWin Recovery Project](https://github.com/TeamWin)
