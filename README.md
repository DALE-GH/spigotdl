# spigot-dl                
```js
               =()=                                                                          Spigot go brrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrrr
           ,/'\_||_
           ( (___  `.
           `\./  `=='
                   |||
                  |||
     _____  _____  ||   ____   __
    |   __||  _  | ___ |    \ |  |
    |__   ||   __||___||  |  ||  |__
    |_____||__|        |____/ |_____| 
```
                             
I got tired of manually managing plugins for my linux-hosted minecraft network, so I made
`spigotdl` - A terminal tool for searching, downloading, tracking, and updating Minecraft server plugins from multiple plugin repositories.

It is intended for Linux-hosted Minecraft servers where the server files are accessible from the command line. It works with single-server and multi-server setups, including custom directory layouts.

 v1.3.0 adds a provider system for:
- Spigot resources through Spiget
- Modrinth projects
- Hangar projects

Examples:

```bash
spigotdl survival "https://www.spigotmc.org/resources/viaversion.19254/"
```
```bash
spigotdl search luckperms --source modrinth --api paper --mc-version 1.21.11
```
## Features

- **Multi-source plugin search**  
  Search Spigot, Modrinth, and Hangar from one command.

- **Provider-aware downloads**  
  Install by source-specific reference such as `spigot:19254`, `modrinth:luckperms`, or `hangar:ViaVersion/ViaVersion`.

- **Interactive main menu**  
  Quick access to search, install, batch, scan, update, and overview actions.

- **Batch and multi-server installs**  
  Install multiple plugins at once, including batch files that target one server or multiple servers.

- **Safe file handling**  
  Preserves the original uploaded `.jar` filename when available, validates downloads as jar/zip files, and backs up existing plugin files before replacing them.

- **Registry tracking**  
  Tracks installed plugins by provider, project ID, version ID, filename, release type, loader/API, game version, and install source.

- **Scan and update workflow**  
  Scan existing plugin folders into the registry, then update tracked plugins later.

- **Overview/status dashboard**  
  Lists detected servers, plugin counts, tracked/untracked counts, and update status with color-coded output.

- **Flexible layouts**  
  Supports custom server roots, config directories, plugin folder names, direct `--server-dir`, and direct `--plugin-dir`.

## Requirements

- An up-to-date & working Linux machine _(hopefully)_
- Python 3.8+
- Direct filesystem access to Minecraft server files
- A Bukkit/Spigot/Paper-compatible server plugin folder
- Standard CLI tools: `curl`, `unzip`, `python3`, `find`, `file`, and GNU/coreutils tools.

Debian/Ubuntu:

```bash
sudo apt update
sudo apt install -y python3
```

## Install

```bash
git clone https://github.com/DALE-GH/spigotdl
cd ./spigotdl
```
```bash
sudo install -m 0755 spigotdl /usr/local/bin/spigotdl
sudo install -m 0644 example.spigotdl.conf /etc/spigotdl.conf
```

Verify:

```bash
spigotdl --version
spigotdl list
```

## Main menu

Run without arguments:

```bash
spigotdl
```

or:

```bash
spigotdl menu
```

The main menu includes:

```text
Search plugins
Install by ID / URL / provider ref
Batch install
Scan installed plugins
Update tracked plugins
Server overview
List servers
```

## Search

Search every provider:

```bash
spigotdl search viaversion
```

Search one provider:

```bash
spigotdl search luckperms --source modrinth
spigotdl search viaversion --source spigot
spigotdl search chunky --source hangar
```

Filter by API/loader:

```bash
spigotdl search essentials --api paper
```

Filter by Minecraft version:

```bash
spigotdl search viaversion --mc-version 1.21.11
```

Filter by release channel where the provider supports it:

```bash
spigotdl search luckperms --source modrinth --release release
spigotdl search nova --source hangar --release snapshot
```

Sort:

```bash
spigotdl search essentials --sort downloads
spigotdl search bluemap --sort updated
```

Search and install the first result:

```bash
spigotdl search viaversion --source modrinth --server survival --install
```

Search and choose interactively:

```bash
spigotdl search viaversion --server survival
```

## Plugin Install

Install plugins directly by name or ID

```bash
spigotdl survival 19254
```

Provider-specific installs:

```bash
spigotdl install spigot:19254 --server survival
spigotdl install modrinth:luckperms --server survival
spigotdl install hangar:ViaVersion/ViaVersion --server survival
```

Using direct paths:

```bash
spigotdl install modrinth:luckperms --server-dir /srv/minecraft/survival
spigotdl install modrinth:luckperms --plugin-dir /srv/minecraft/survival/plugins
```

Preview without downloading:

```bash
spigotdl install modrinth:luckperms --server survival --dry-run
```

## Batch install

Simple batch download by ID:

```bash
spigotdl survival 19254 34315 6245
```

One server:

```bash
spigotdl batch examples/plugins.txt --server survival
```

Example `plugins.txt`:

```text
spigot:19254
modrinth:luckperms
hangar:ViaVersion/ViaVersion
```

Multi-server batch file:

```bash
spigotdl batch examples/fleet-plugins.txt
```

Example `fleet-plugins.txt`:

```text
survival spigot:19254
survival modrinth:luckperms
lobby hangar:ViaVersion/ViaVersion
```


## Registry v2

Registry files are stored per plugin folder:

```text
<plugins-folder>/.spigotdl/registry.tsv
```

Registry v2 columns:

```text
provider
project_id
version_id
filename
project_name
release_type
loaders
game_versions
installed_at
source
```

_Older v1 registries are migrated automatically when found._

## Scan existing plugins

Scan a server and interactively match untracked jars:

```bash
spigotdl scan --server survival
```

Non-interactive scan mode:

```bash
spigotdl scan --server survival --yes
```

Scan using one source only:

```bash
spigotdl scan --server survival --source modrinth
```

## Update tracked plugins

Update one server:

```bash
spigotdl update --server survival
```

Update selected servers:

```bash
spigotdl update --servers survival,lobby
```

Update all detected servers:

```bash
spigotdl update --all
```

Preview update checks:

```bash
spigotdl update --all --dry-run
```

## Overview/status

Show all detected servers:

```bash
spigotdl overview
```

Show details:

```bash
spigotdl overview --details
```

Skip API checks:

```bash
spigotdl overview --no-check
```

Example output:

```text
[SPDL] Server Overview

Server              Plugins Tracked Untracked Outdated Status        Directory
survival                 42      31        11        4 UPDATE        /srv/minecraft/survival
lobby                    18      18         0        0 OK            /srv/minecraft/lobby
proxy                     7       4         3        1 UPDATE        /srv/minecraft/proxy
```

## Configuration

Config files:

```text
/etc/spigotdl.conf
~/.config/spigotdl/config
```

Example:

```bash
SPDL_SERVER_ROOTS="/srv/minecraft:/opt/mc"
SPDL_CONF_DIR="/etc/mc"
SPDL_PLUGINS_DIR_NAME="plugins"
SPDL_DEFAULT_SOURCE="spigot"
SPDL_RESTART_CMD_TEMPLATE='sudo systemctl restart minecraft@{instance}'
SPDL_PLUGMAN_CMD_TEMPLATE='plugman load {filename}'
```

Important options:

```bash
SPDL_SERVER_ROOTS="/opt/mc"
SPDL_CONF_DIR="/etc/mc"
SPDL_SERVER_DIR_VAR="SERVER_DIR"
SPDL_PLUGINS_DIR_NAME="plugins"
SPDL_REGISTRY_DIR=".spigotdl"
SPDL_REGISTRY_FILE="registry.tsv"
SPDL_BACKUP_EXISTING="true"
SPDL_VALIDATE_JAR="true"
SPDL_COLOR="auto"
```

## Provider notes

### Spigot / Spiget

Spigot downloads use the Spiget API. Resource IDs and full Spigot resource URLs are supported.

```bash
spigotdl install spigot:19254 --server survival
```

### Modrinth

Modrinth supports search facets for project type, loaders/categories, and Minecraft versions. The tool uses those filters when `--api` and `--mc-version` are provided.

```bash
spigotdl search luckperms --source modrinth --api paper --mc-version 1.21.11
```

### Hangar

Hangar support is best-effort because its API has changed over time. Search and recommended download routes are implemented using the public API shape used by Hangar.

```bash
spigotdl search viaversion --source hangar
```

## Limitations

- Premium or account-gated downloads are not supported.
- Search filters vary by provider. Modrinth has the strongest structured filtering; Spigot and Hangar support fewer filters through public endpoints.
- Automatic scan matching is best-effort.
