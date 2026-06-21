# spigotdl

I got tired of manually managing plugins for my linux-hosted minecraft network, so I made
`spigotdl` - A small Bash CLI tool for downloading and updating plugin`.jar` files directly from the terminal.

It is intended for Linux-hosted Minecraft servers where the server files are accessible from the command line. It works with single-server and multi-server setups, including custom directory layouts.

## What it does

`spigotdl` accepts Spigot resource IDs or full Spigot resource URLs, resolves downloads through the Spigot API, saves plugins into a server's plugin directory, validates that each result is a real `.jar` file, and records downloaded plugins in a local registry so they can be updated later.

Example:

```bash
spigotdl survival "https://www.spigotmc.org/resources/viaversion.19254/"
```

## Features

- **Simple plugin downloads**  
  Download Spigot plugins using either a resource ID or full resource URL. Resource IDs are extracted automatically from supported URLs.

- **Interactive and script-friendly usage**  
  Use the interactive server picker for manual installs, or direct commands for scripts, automation, and repeatable workflows.

- **Batch and multi-server installs**  
  Install multiple plugins at once, including batch files that target one server or multiple servers.

- **Safe file handling**  
  Preserves the original uploaded `.jar` filename when available, validates downloads with `unzip -t`, and backs up existing plugin files before replacing them.

- **Plugin tracking and updates**  
  Maintains a local plugin registry, can scan existing plugin folders, and can update tracked plugins for one server, selected servers, or all detected servers.

- **Flexible server layouts**  
  Supports custom server roots, config directories, plugin folder names, and direct `--server-dir` or `--plugin-dir` paths for non-standard setups.

- **Configurable behavior**  
  Supports system-wide config, per-user config, environment-variable overrides, and optional post-install restart or PlugMan command suggestions.

## Requirements

- 5 minutes and
- A working Linux server
- Standard CLI tools: `curl`, `unzip`, `python3`, `find`, `file`, and GNU/coreutils tools.
- Minnecraft server(s) running on bare-metal or of which the files can be accessed directly via CLI

Install dependencies:

Debian/Ubuntu:

```bash
sudo apt update
sudo apt install -y bash curl unzip python3 coreutils findutils file
```

RHEL/Fedora:

```bash
sudo dnf install -y bash curl unzip python3 coreutils findutils file
```

Arch Linux:

```bash
sudo pacman -S --needed bash curl unzip python coreutils findutils file
```

## Install

Clone the repository or download the files, then install the script:

```bash
sudo install -m 0755 spigotdl /usr/local/bin/spigotdl
```

Optional system config:

```bash
sudo install -m 0644 example.spigotdl.conf /etc/spigotdl.conf
```

Verify:

```bash
spigotdl --version
spigotdl --help
```

## Quick start

Interactive mode:

```bash
spigotdl
```

Direct mode:

```bash
spigotdl survival 19254
```

Using a full Spigot resource URL:

```bash
spigotdl survival "https://www.spigotmc.org/resources/viaversion.19254/"
```

Using flags:

```bash
spigotdl --server survival --resource 19254
```

Using a direct server directory:

```bash
spigotdl --server-dir /srv/minecraft/survival --resource 19254
```

Using a direct plugins directory:

```bash
spigotdl --plugin-dir /srv/minecraft/survival/plugins --resource 19254
```

List detected servers:

```bash
spigotdl --list
```

Disable color:

```bash
spigotdl --no-color survival 19254
```

## Batch downloading

You can download multiple plugins in one command:

```bash
spigotdl survival 19254 34315 6245
```

Or by repeating `--resource`:

```bash
spigotdl --server survival --resource 19254 --resource 34315 --resource 6245
```

### Batch file for one server

Create `plugins.txt`:

```text
# resource-id-or-url [optional-output-name]
19254
https://www.spigotmc.org/resources/viaversion.19254/
34315 SomePlugin.jar
```

Run:

```bash
spigotdl --server survival --batch plugins.txt
```

### Batch file for multiple servers

Create `fleet-plugins.txt`:

```text
# server resource-id-or-url [optional-output-name]
survival 19254
lobby https://www.spigotmc.org/resources/viaversion.19254/
hub 34315 SomePlugin.jar
```

Run:

```bash
spigotdl --batch fleet-plugins.txt
```

By default, batch mode continues after a failed download. Change that with:

```bash
SPIGOTDL_BATCH_CONTINUE_ON_ERROR=false spigotdl --server survival --batch plugins.txt
```

## Updating plugins

`spigotdl` tracks plugins it downloads in a local registry:

```text
<plugins-folder>/.spigotdl/registry.tsv
```

You can update all tracked plugins for one server:

```bash
spigotdl --server survival --update
```

Update selected servers:

```bash
spigotdl --servers survival,lobby --update
```

Update all detected servers:

```bash
spigotdl --all --update
```

Preview updates without changing files:

```bash
spigotdl --server survival --update --dry-run
```

### How update tracking works

When `spigotdl` downloads a plugin, it records:

```text
resource_id
filename
resource_name
version_id
installed_at
source
```

During `--update`, it checks the latest metadata for each tracked resource. If the latest version ID differs from the stored version ID, it downloads the newest jar, backs up the old file, and updates the registry.

## Scanning existing plugin folders

If you already have plugins installed, use `--scan` to build the registry:

```bash
spigotdl --server survival --scan
```

The scanner:

1. Finds `.jar` files in the plugin directory.
2. Guesses a search term from the filename.
3. Searches Spiget for matching resources.
4. Asks you which result to track.
5. Adds the selected resource to `.spigotdl/registry.tsv`.

Then update tracked plugins:

```bash
spigotdl --server survival --update
```

You can scan and update in one command:

```bash
spigotdl --server survival --scan --update
```

For multiple servers:

```bash
spigotdl --servers survival,lobby --scan --update
```

For all detected servers:

```bash
spigotdl --all --scan --update
```

### Non-interactive scan mode

Use `--yes` to avoid prompts:

```bash
spigotdl --server survival --scan --yes
```

In `--yes` scan mode, `spigotdl` only auto-tracks exact non-premium matches. Unclear matches are skipped.

## Supported server layouts

`spigotdl` is not limited to one directory structure.

By default, it checks:

```text
/opt/mc/<server>
/etc/mc/<server>.conf
```

You can configure a different root:

```bash
SPIGOTDL_SERVER_ROOTS="/srv/minecraft" spigotdl --list
```

You can configure multiple roots:

```bash
SPIGOTDL_SERVER_ROOTS="/srv/minecraft:/opt/servers:/home/minecraft/servers" spigotdl --list
```

You can bypass auto-detection entirely:

```bash
spigotdl --server-dir /custom/path/to/server --resource 19254
```

or:

```bash
spigotdl --plugin-dir /custom/path/to/server/plugins --resource 19254
```

## Per-server config files

If your setup stores server paths in config files, use:

```text
/etc/mc/<server>.conf
```

Example:

```bash
SERVER_DIR="/srv/minecraft/survival"
```

Then this:

```bash
spigotdl survival 19254
```

installs to:

```text
/srv/minecraft/survival/plugins
```

The variable name is configurable:

```bash
SPIGOTDL_SERVER_DIR_VAR="MINECRAFT_SERVER_DIR"
```

## Example output

```text
Done.
Installed:
  /srv/minecraft/survival/plugins/ViaVersion-5.4.2.jar

Optional live-load command:
  plugman load ViaVersion-5.4.2.jar
```

If you configure a restart command, the output can also include:

```text
Restart command:
  sudo systemctl restart minecraft@survival
```

## Configuration

`spigotdl` loads configuration from:

```text
/etc/spigotdl.conf
~/.config/spigotdl/config
```

The user config overrides the system config.

Environment variables can override settings for one run:

```bash
SPIGOTDL_FILENAME_MODE=resource spigotdl survival 19254
```

## Important config options

### Server discovery

```bash
SPIGOTDL_SERVER_ROOTS="/opt/mc"
```

One or more base directories containing Minecraft server folders. Use `:` to separate multiple roots.

```bash
SPIGOTDL_SERVER_ROOTS="/srv/minecraft:/opt/mc:/home/minecraft/servers"
```

```bash
SPIGOTDL_CONF_DIR="/etc/mc"
```

Directory containing optional per-server config files such as `survival.conf`.

```bash
SPIGOTDL_SERVER_DIR_VAR="SERVER_DIR"
```

Variable name inside each per-server config file that points to the actual server directory.

```bash
SPIGOTDL_PLUGINS_DIR_NAME="plugins"
```

Plugin folder name under each server directory.

```bash
SPIGOTDL_EXCLUDE_REGEX='(^$|^\..*|^backups?$|^backup$|^cache$|^logs?$|^plugins?$|^old$|^archive$|^tmp$|^temp$)'
```

Folders matching this regex are hidden from the interactive server picker.

### Filename behavior

```bash
SPIGOTDL_FILENAME_MODE="file"
```

Filename mode.

Options:

```text
file       Use the original uploaded jar filename when available.
resource   Use the Spigot resource/plugin name.
resourceid Use resource-name-resource-id.jar.
```

### Safety behavior

```bash
SPIGOTDL_BACKUP_EXISTING="true"
```

Back up an existing jar before replacing it.

```bash
SPIGOTDL_REQUIRE_JAR="true"
```

Validate the downloaded file with `unzip -t`.

```bash
SPIGOTDL_AUTO_CHOWN="true"
```

Try to set the installed jar owner/group to match the plugin folder.

```bash
SPIGOTDL_INSTALL_MODE="0644"
```

File mode for installed jars.

```bash
SPIGOTDL_BATCH_CONTINUE_ON_ERROR="true"
```

Continue processing a batch file if one download fails.

### Registry and update behavior

```bash
SPIGOTDL_REGISTRY_DIR_NAME=".spigotdl"
SPIGOTDL_REGISTRY_FILE_NAME="registry.tsv"
```

Registry location under the plugin folder.

```bash
SPIGOTDL_SCAN_MAX_RESULTS="5"
```

Maximum search results shown for each untracked plugin during `--scan`.

```bash
SPIGOTDL_SCAN_AUTO_EXACT="true"
```

Reserved for scan behavior. Current `--yes` mode only auto-tracks exact non-premium matches.

### Output behavior

```bash
SPIGOTDL_COLOR="auto"
```

Color mode.

Options:

```text
auto
always
never
```

```bash
SPIGOTDL_RESTART_CMD_TEMPLATE=''
```

Optional restart command printed after install. Empty by default because service names vary by setup.

Example:

```bash
SPIGOTDL_RESTART_CMD_TEMPLATE='sudo systemctl restart minecraft@{instance}'
```

```bash
SPIGOTDL_PLUGMAN_CMD_TEMPLATE='plugman load {filename}'
```

Optional PlugMan command printed after install.

Available placeholders:

```text
{instance}
{filename}
{dest}
{server_dir}
{plugin_dir}
```

Examples:

```bash
SPIGOTDL_RESTART_CMD_TEMPLATE='sudo systemctl restart minecraft@{instance}'
SPIGOTDL_PLUGMAN_CMD_TEMPLATE='plugman load {filename}'
```

```bash
SPIGOTDL_RESTART_CMD_TEMPLATE='docker restart mc-{instance}'
SPIGOTDL_PLUGMAN_CMD_TEMPLATE=''
```

```bash
SPIGOTDL_RESTART_CMD_TEMPLATE=''
SPIGOTDL_PLUGMAN_CMD_TEMPLATE='screen -S {instance} -X stuff "plugman load {filename}\015"'
```

## Example config

See [`example.spigotdl.conf`](example.spigotdl.conf).

Minimal example:

```bash
SPIGOTDL_SERVER_ROOTS="/srv/minecraft"
SPIGOTDL_CONF_DIR="/etc/minecraft/servers"
SPIGOTDL_FILENAME_MODE="file"
SPIGOTDL_RESTART_CMD_TEMPLATE='sudo systemctl restart minecraft@{instance}'
SPIGOTDL_PLUGMAN_CMD_TEMPLATE='plugman load {filename}'
```

## Running without sudo

`spigotdl` only needs write access to the target plugin directory.

A simple group-based setup:

```bash
sudo groupadd -f minecraft-admins
sudo usermod -aG minecraft-admins "$USER"

sudo chgrp -R minecraft-admins /srv/minecraft/*/plugins
sudo chmod -R g+rwX /srv/minecraft/*/plugins
sudo find /srv/minecraft/*/plugins -type d -exec chmod g+s {} \;
```

Log out and back in, then test:

```bash
spigotdl survival 19254
```

If parent directories block traversal, allow group execute on those directories:

```bash
sudo chgrp minecraft-admins /srv/minecraft /srv/minecraft/*
sudo chmod g+x /srv/minecraft /srv/minecraft/*
```

Adjust `/srv/minecraft` to match your server root.

## Limitations

- This tool is intended for free Spigot resources.
- Premium resources may require downloading through your Spigot account or another vendor-supported method.
- Some resources use external download URLs. `spigotdl` warns before continuing and validates that the final download is a real `.jar`.
- Automatic scanning is best-effort. Filenames do not always map cleanly to Spigot resource IDs.
- `--update` is safest for plugins originally installed by `spigotdl` or manually confirmed through `--scan`.
- This tool downloads plugin files. It does not guarantee plugin compatibility with your Minecraft, Java, Paper, Spigot, or proxy version.
- Live-loading plugins with tools such as PlugMan may not be safe for every plugin. Restarting the server is usually the safer option.

## License

MIT
