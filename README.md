# gentoo-upgrade
![License](https://img.shields.io/github/license/jamescherti/gentoo-upgrade)
![Linux](https://img.shields.io/badge/Linux-FCC624?logo=linux&logoColor=black)

- Author: James Cherti <[https://www.jamescherti.com/](https://www.jamescherti.com/)>
- License: GPL 3.0 or later

The *[gentoo-upgrade](https://github.com/jamescherti/gentoo-upgrade)* is a helper script that updates a Gentoo Linux with `emerge`.

This script automates the Gentoo `@world` upgrade lifecycle, handling execution from pre-flight checks to post-run cleanup.

* **Process locking:** Detects running `emerge` instances before execution and prior to cleanup phases.
* **News queue verification:** Checks for unread Gentoo news via `eselect news read` before sync or upgrade actions.
* **Repository synchronization:** Executes `emerge --sync` when invoked with `--sync`. Defaults to local repository metadata.
* **Portage self-update:** Prioritizes `sys-apps/portage` upgrades before processing the rest of `@world`.
* **Dependency resolution:** Resolves full dependency trees, processes build-time requirements, and recompiles packages impacted by USE flag modifications. Uses `--keep-going` to bypass failed builds.
* **Execution flags:** Supports `--ask` for interactive merge review and `--quiet` to suppress verbose output from Portage and `eclean-dist`.
* **Library consistency:** Runs `emerge @preserved-rebuild` and `revdep-rebuild -v` post-upgrade to handle ABI shifts and broken dynamic links.
* **Cleanup routines:** Executes `emerge --depclean`, `eclean-dist --deep`, and clears `/var/tmp/portage/`. Note: dependency removal bypasses interactive prompts even if `--ask` was specified.
* **State persistence:** Stores sync timestamps, pending ebuild states, and task registry in `~/.gentoo_upgrade.yaml`.

## Requirements

- Gentoo Linux
- Python 3.10 or later (`dev-lang/python`)
- psutil (`dev-python/psutil`) and PyYAML (`dev-python/pyyaml`)
- emerge (`sys-apps/portage`), eselect (`app-admin/eselect`), revdep-rebuild, and eclean-dist (`app-portage/gentoolkit`)

Install them using:
```sh
emerge --ask dev-lang/python dev-python/psutil dev-python/pyyaml app-admin/eselect app-portage/gentoolkit
```

## Installation

### Manual installation

From the directory containing `gentoo-upgrade`, run these commands as root: 

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

To install from GitHub into a dedicated virtual environment, run the following as root with Git installed. Use a Python 3.11 or newer interpreter for the first command.

```sh
python3 -m venv /opt/gentoo-upgrade
/opt/gentoo-upgrade/bin/python -m pip install git+https://github.com/jamescherti/gentoo-upgrade
```

## Usage

To sync repositories and review Portage's proposed merges, run as root:

```sh
gentoo-upgrade --sync --ask
```

**Cleanup can remove packages and files.** When dependency cleanup is pending, the script runs `emerge --depclean` without passing `--ask`, even if you requested confirmation for the upgrade. It then runs `eclean-dist --deep` and deletes the contents of `/var/tmp/portage/`.

|Option         |Behavior                                                                                        |
|---------------|------------------------------------------------------------------------------------------------|
|`--sync`       |Run `emerge --sync` before checking for updates. By default, use the current local repositories.|
|`-a`, `--ask`  |Pass `--ask` to Portage for merges. By default, the script requests no confirmation.            |
|`-q`, `--quiet`|Reduce Portage output and run `eclean-dist` quietly.                                            |
|`-h`, `--help` |Show command-line help and exit.                                                                |

## License

Copyright (c) 2021-2026 [James Cherti](https://www.jamescherti.com)

This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with this program. If not, see <https://www.gnu.org/licenses/>.

## Links

- [gentoo-upgrade on GitHub](https://github.com/jamescherti/gentoo-upgrade)
