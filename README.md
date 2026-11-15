# gentoo-upgrade
![Build Status](https://github.com/jamescherti/gentoo-upgrade/actions/workflows/ci.yml/badge.svg)
![License](https://img.shields.io/github/license/jamescherti/gentoo-upgrade)
[![PyPI Version](https://img.shields.io/pypi/v/gentoo-upgrade.svg)](https://pypi.org/project/gentoo-upgrade/)
![Linux](https://img.shields.io/badge/Linux-FCC624?logo=linux&logoColor=black)

- Author: James Cherti <[https://www.jamescherti.com/](https://www.jamescherti.com/)>
- License: GPL 3.0 or later

The `gentoo-upgrade` is a helper script that updates a Gentoo Linux installation with Portage.

It can sync repositories, update Portage itself before the rest of `@world`, and run rebuild and cleanup tasks after a successful upgrade.

## Requirements

- Gentoo Linux
- Python 3.10 or later (`dev-lang/python`)
- psutil (`dev-python/psutil`) and PyYAML (`dev-python/pyyaml`)
- emerge (`sys-apps/portage`)
- eselect (`app-admin/eselect`)
- revdep-rebuild and eclean-dist (`app-portage/gentoolkit`)

Install them using:
```sh
emerge --ask dev-python/psutil dev-python/pyyaml app-portage/gentoolkit
```

## Installation

### Manual installation

From the directory containing `gentoo-upgrade`, run these commands as root: 

```sh
install -D -m 0755 gentoo_upgrade.py /usr/local/bin/gentoo-upgrade
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

| Option | Behavior |
| --- | --- |
| `--sync` | Run `emerge --sync` before checking for updates. By default, use the current local repositories. |
| `-a`, `--ask` | Pass `--ask` to Portage for merges. By default, the script requests no confirmation. |
| `-q`, `--quiet` | Reduce Portage output and run `eclean-dist` quietly. |
| `-h`, `--help` | Show command-line help and exit. |

## Upgrade sequence

1. Before syncing or upgrading, check for a running `emerge` process and unread Gentoo news. Either condition stops the run. If news is pending, run `eselect news read` and start the upgrade again.
2. Sync if requested, identify pending ebuilds, and update `sys-apps/portage` first if needed.
3. Upgrade the world set with `emerge --update --deep --with-bdeps=y --newuse --keep-going @world`.
4. After a successful world update, run `emerge @preserved-rebuild`. If scheduled, run `revdep-rebuild -v` and the dependency cleanup described above. The script checks for a running `emerge` before starting each cleanup stage.

The script records the last sync time, pending ebuilds, and scheduled tasks in `~/.gentoo_upgrade.yaml` under the home directory used by the root process.

## License

Copyright (c) 2021-2026 [James Cherti](https://www.jamescherti.com)

This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with this program. If not, see <https://www.gnu.org/licenses/>.

## Links

- [gentoo-upgrade on GitHub](https://github.com/jamescherti/gentoo-upgrade)
