# gentoo-upgrade - A command-line tool that updates Gentoo Linux and runs cleanup tasks
[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)
![Linux](https://img.shields.io/badge/Linux-FCC624?logo=linux&logoColor=black)

- Author: James Cherti <[https://www.jamescherti.com/](https://www.jamescherti.com/)>
- License: GPL 3.0 or later

The *[gentoo-upgrade](https://github.com/jamescherti/gentoo-upgrade)* is a command-line tool that updates Gentoo Linux.

If Portage needs an update, `gentoo-upgrade` upgrades `sys-apps/portage` before updating `@world`. After a successful update, it runs pending rebuilds. When the update plan includes `dev-lang/go`, it also runs `emerge @golang-rebuild` to apply bug fixes and security fixes from the updated Go standard library (See [GLSA 202209-26](https://security.gentoo.org/glsa/202209-26)).

Optional cleanup is disabled by default. With `--clean`, the tool runs pending `emerge --depclean` and `eclean-dist --deep` tasks. It records pending work in `~/.cache/gentoo-upgrade.json`, so cleanup can run on a later successful upgrade with `--clean`.

The tool also supports optional before and after hooks, allowing custom scripts to execute before and after the upgrade process.

## Requirements

- Gentoo Linux
- Python 3.10 or later (`dev-lang/python`)
- emerge (`sys-apps/portage`) and eselect (`app-admin/eselect`)
- `app-portage/gentoolkit` for `revdep-rebuild` and `eclean-dist`.

Install the requirements using:

```sh
emerge --ask dev-lang/python app-admin/eselect app-portage/gentoolkit
```

## Installation

### Manual installation

Run these commands as root:

```sh
git clone https://github.com/jamescherti/gentoo-upgrade
cd gentoo-upgrade
install -D -m 0755 gentoo-upgrade /usr/local/bin/gentoo-upgrade
```

To verify the installation without starting an upgrade, run:

```sh
/usr/local/bin/gentoo-upgrade --help
```

### Alternative: Installation in a virtual environment

To install from GitHub into a dedicated virtual environment, run the following as root with Git installed. Use a Python 3.10 or newer interpreter for the first command.

```sh
python3 -m venv /opt/gentoo-upgrade
/opt/gentoo-upgrade/bin/python -m pip install git+https://github.com/jamescherti/gentoo-upgrade
```

## Usage

To sync repositories and review Portage's proposed merges, run as root:

```sh
gentoo-upgrade --sync --ask
```

To also run cleanup tasks after a successful upgrade:

```sh
gentoo-upgrade --sync --ask --clean
```

|Option         |Behavior                                                                                                       |
|---------------|---------------------------------------------------------------------------------------------------------------|
|`--sync`       |Run `emerge --sync` before checking for updates. By default, use the current local repositories.               |
|`-a`, `--ask`  |Ask before upgrade and cleanup operations, including depclean. By default, the script requests no confirmation.|
|`-c`, `--clean`|Run cleanup tasks after a successful upgrade. Disabled by default.                                             |
|`-q`, `--quiet`|Reduce Portage output and run `eclean-dist` quietly.                                                           |
|`-h`, `--help` |Show command-line help and exit.                                                                               |

## Before and after hooks

The tool supports two optional executable scripts that run before and after the upgrade process:

|Hook                         |Description                                       |
|-----------------------------|--------------------------------------------------|
|`~/.gentoo-upgrade-pre-hook` |Runs before the upgrade and pre-flight checks.    |
|`~/.gentoo-upgrade-post-hook`|Runs after the upgrade and state-saving operation.|

The scripts are located in the home directory resolved by the process running `gentoo-upgrade`, normally `/root` when executed as root.

If a hook does not exist, it is silently skipped. If it exists but is not executable, a warning is displayed and the hook is skipped.

Hooks execute with the same privileges as `gentoo-upgrade`, typically root. Ensure that the scripts and their directories cannot be modified by untrusted users.

### Before hook

The before hook runs before the pre-flight checks, repository synchronization, upgrades, rebuilds, and cleanup tasks.

Create `~/.gentoo-upgrade-pre-hook`:

```sh
#!/bin/sh

echo "Starting Gentoo upgrade"
```

Make it executable:

```sh
chmod 700 ~/.gentoo-upgrade-pre-hook
```

If the before hook exits with a nonzero status, the upgrade is aborted. The after hook is still attempted.

### After hook

The after hook runs when the upgrade process finishes, including when an error occurs or the upgrade is interrupted by Ctrl+C.

It receives one positional argument indicating the result:

|Argument |Description                                                        |
|---------|-------------------------------------------------------------------|
|`success`|The upgrade, cleanup, and state-saving operations completed.       |
|`failure`|The upgrade was declined, interrupted, or an operation failed.     |

Create `~/.gentoo-upgrade-post-hook`:

```sh
#!/bin/sh

status="$1"

if [ "$status" = "success" ]; then
    echo "Gentoo upgrade completed successfully"
else
    echo "Gentoo upgrade failed or was interrupted"
fi
```

Make it executable:

```sh
chmod 700 ~/.gentoo-upgrade-post-hook
```

The after hook is attempted even if the before hook, upgrade, cleanup, or state-saving operation fails.

If the after hook fails while another error is already being handled, its failure is reported without replacing the original error. Otherwise, an after hook failure causes the command to exit with an error.

Hooks cannot be guaranteed to execute if the process is forcibly terminated, such as with `SIGKILL`, or terminated without Python being able to run its cleanup handlers.

## How gentoo-upgrade handles updates, rebuilds, and cleanup

### Step 1: Before hook and pre-flight checks

- **Before hook:** Executes `~/.gentoo-upgrade-pre-hook` if installed and executable.
- **News queue verification:** Checks for unread Gentoo news before sync or upgrade actions. Read pending items with `eselect news read`.

### Step 2: Upgrade

- **Repository synchronization:** Executes `emerge --sync` when invoked with `--sync`. Defaults to local repository metadata.
- **Portage self-update:** If Portage needs an update, it upgrades `sys-apps/portage` before upgrading all packages.
- **Dependency resolution:** Updates `@world` and its dependencies. Resolves full dependency trees, processes build-time requirements, and recompiles packages impacted by USE flag modifications.

### Step 3: Rebuilds

- Runs `emerge @preserved-rebuild` after a successful world update to rebuild packages that depend on preserved shared libraries.
- Runs `revdep-rebuild -v` when scheduled to detect broken shared-library dependencies and rebuild affected packages.
- When the update plan includes `dev-lang/go`, `gentoo-upgrade` runs `emerge @golang-rebuild`. Go binaries typically include the standard library code they use. Updating `dev-lang/go` leaves that code in existing binaries, so affected packages need to be rebuilt to apply bug fixes and security fixes from the updated Go standard library.

### Step 4: Cleanup

- **Cleanup routines:** When `--clean` is specified, runs `emerge --depclean` and `eclean-dist --deep` tasks after a successful upgrade.
- **State persistence:** Stores sync timestamps, pending ebuild states, and task registry in `~/.cache/gentoo-upgrade.json`. Cleanup tasks remain pending when `--clean` is omitted.

### Step 5: After hook

- **After hook:** Executes `~/.gentoo-upgrade-post-hook` if installed and executable.
- **Result reporting:** Passes `success` or `failure` as the first argument, indicating whether the upgrade and state-saving operations completed successfully.
- **Failure handling:** Attempts the after hook even if an earlier operation fails.

## License

Copyright (c) 2021-2026 [James Cherti](https://www.jamescherti.com)

This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with this program. If not, see <https://www.gnu.org/licenses/>.

## Links

- [gentoo-upgrade on GitHub](https://github.com/jamescherti/gentoo-upgrade)

Other projects by the same author:
- [jc-gentoo-portage](https://github.com/jamescherti/jc-gentoo-portage): An opinionated, performance-oriented Gentoo Portage (`/etc/portage`) configuration.
