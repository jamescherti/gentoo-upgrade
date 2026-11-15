# gentoo-upgrade - A command-line tool that updates Gentoo Linux
[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)
![Linux](https://img.shields.io/badge/Linux-FCC624?logo=linux&logoColor=black)

- Author: James Cherti <[https://www.jamescherti.com/](https://www.jamescherti.com/)>
- License: GPL 3.0 or later

The *[gentoo-upgrade](https://github.com/jamescherti/gentoo-upgrade)* is a command-line tool that updates Gentoo Linux.

It brings routine Gentoo maintenance into one repeatable command. If Portage needs an update, it upgrades `sys-apps/portage` before upgrading all packages. After a successful upgrade, it rebuilds packages that still use older shared libraries.

Optional cleanup is disabled by default. When `--clean` is specified, the tool runs scheduled cleanup tasks: `revdep-rebuild -v`, `emerge --depclean`, `eclean-dist --deep`, and `perl-cleaner --all`. It records pending work in `~/.cache/gentoo-upgrade.json`, so scheduled cleanup can be performed on a later successful run with `--clean`.

## Requirements

- Gentoo Linux
- Python 3.10 or later (`dev-lang/python`)
- psutil (`dev-python/psutil`)
- emerge (`sys-apps/portage`) and eselect (`app-admin/eselect`)
- For `--clean`: revdep-rebuild, eclean-dist (`app-portage/gentoolkit`), and perl-cleaner (`app-admin/perl-cleaner`)

Install the requirements using:

```sh
emerge --ask dev-lang/python dev-python/psutil app-admin/eselect app-portage/gentoolkit app-admin/perl-cleaner
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

To also run scheduled cleanup tasks after a successful upgrade:

```sh
gentoo-upgrade --sync --ask --clean
```

|Option         |Behavior                                                                                                                   |
|---------------|---------------------------------------------------------------------------------------------------------------------------|
|`--sync`       |Run `emerge --sync` before checking for updates. By default, use the current local repositories.                           |
|`-a`, `--ask`  |Pass `--ask` to Portage for merges. By default, the script requests no confirmation. This does not prompt for `--depclean`.|
|`-c`, `--clean`|Run scheduled cleanup tasks after a successful upgrade. Disabled by default.                                               |
|`-q`, `--quiet`|Reduce Portage output and run `eclean-dist` quietly.                                                                       |
|`-h`, `--help` |Show command-line help and exit.                                                                                           |

## How gentoo-upgrade handles updates, rebuilds, and cleanup

### Step 1: Pre-flight checks

- **Process check:** Detects running `emerge` instances before execution and prior to cleanup phases.
- **News queue verification:** Checks for unread Gentoo news before sync or upgrade actions. Read pending items with `eselect news read`.

### Step 2: Upgrade

- **Repository synchronization:** Executes `emerge --sync` when invoked with `--sync`. Defaults to local repository metadata.
- **Portage self-update:** Upgrades `sys-apps/portage` before processing the rest of `@world`.
- **Dependency resolution:** Resolves full dependency trees, processes build-time requirements, and recompiles packages impacted by USE flag modifications. Uses `--keep-going` to continue past failed builds.
- **Execution flags:** Supports `--ask` for interactive merge review and `--quiet` to reduce output from Portage and `eclean-dist`.

### Step 3: Cleanup

- **Library consistency:** Runs `emerge @preserved-rebuild` after a successful upgrade. When `--clean` is specified, it also runs `revdep-rebuild -v` if that task is scheduled.
- **Cleanup routines:** When `--clean` is specified, runs scheduled `emerge --depclean`, `eclean-dist --deep`, and `perl-cleaner --all` tasks after a successful upgrade. The Perl command runs only when a Perl task is pending. Dependency removal bypasses interactive prompts even if `--ask` was specified.
- **Perl follow-up:** Reports a pending Perl task. If `perl-cleaner` is unavailable or fails, the task remains pending.
- **State persistence:** Stores sync timestamps, pending ebuild states, and task registry in `~/.cache/gentoo-upgrade.json`. Cleanup tasks remain pending when `--clean` is omitted.

## License

Copyright (c) 2021-2026 [James Cherti](https://www.jamescherti.com)

This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with this program. If not, see <https://www.gnu.org/licenses/>.

## Links

- [gentoo-upgrade on GitHub](https://github.com/jamescherti/gentoo-upgrade)

Other projects by the same author:
- [jc-gentoo-portage](https://github.com/jamescherti/jc-gentoo-portage): An opinionated, performance-oriented Gentoo Portage (`/etc/portage`) configuration.
