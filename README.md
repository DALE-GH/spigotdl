# spigotdl

I got tired of using the -L flag with wget when downloading plugins via Spigot for my linux-hosted minecraft network, so I made
`spigotdl` - A small Bash CLI tool for downloading Spigot`.jar` files directly into a selected Minecraft server's `plugins/` folder.

It is designed for multi-server Minecraft hosts on linux using a layout such as:

```text
/opt/mc/server1
/opt/mc/server2
/opt/mc/server3
/etc/mc/server1.conf
/etc/mc/server2.conf
/etc/mc/server3.conf
```

If `/etc/mc/<server>.conf` contains `SERVER_DIR=/some/path`, `spigotdl` uses that path. Otherwise it falls back to `/opt/mc/<server>`.

## Features

- Interactive server picker.
- Direct mode for automation.
- Accepts either a Spigot resource ID or full resource URL.
- Extracts the resource ID automatically from URLs.
- Downloads through the Spigot API.
- Keeps the original uploaded `.jar` filename when available.
- Validates that the result is a real `.jar`/ZIP file.
- Backs up existing plugin files before replacing them.
- Supports config files and environment overrides.

## Requirements

- 5 minutes
- A working Linux server
- Minnecraft server(s) running on bare-metal or of which the files can be accessed directly via CLI

Install dependencies:

```bash
sudo apt update
sudo apt install -y bash curl unzip python3 coreutils findutils file
```

## Install

Clone or copy this project, then install the script:

```bash
sudo install -m 0755 spigotdl /usr/local/bin/spigotdl
```

Optional config file:

```bash
sudo cp example.spigotdl.conf /etc/spigotdl.conf
```

## Usage

Interactive mode:

```bash
spigotdl
```

Direct mode:

```bash
spigotdl server1 19254
```

Using a full Spigot resource URL:

```bash
spigotdl server1 "https://www.spigotmc.org/resources/viaversion.19254/"
```

Using flags:

```bash
spigotdl --server server1 --resource 19254
```

Override the output filename:

```bash
spigotdl --server server1 --resource 19254 --name ViaVersion.jar
```

List detected servers:

```bash
spigotdl --list
```

Skip external-download confirmation prompts:

```bash
spigotdl --yes server1 19254
```

## Example output

```text
Done.
Installed:
  /opt/mc/server1/plugins/ViaVersion-5.4.2.jar

Restart the server when ready:
  sudo systemctl restart mc@server1

OR..
connect to the console & manage via PlugMan:
  plugman load ViaVersion-5.4.2.jar
```

## Configuration

`spigotdl` loads config from:

```text
/etc/spigotdl.conf
~/.config/spigotdl/config
```

The user config overrides the system config.

Environment variables can also override settings for one run:

```bash
SPIGOTDL_FILENAME_MODE=resource spigotdl server1 19254
```

## Important config options

```bash
SPIGOTDL_MC_ROOT="/opt/mc"
```

Base directory where Minecraft server folders are stored.

```bash
SPIGOTDL_CONF_DIR="/etc/mc"
```

Directory containing instance config files such as `server1.conf`.

```bash
SPIGOTDL_SERVER_DIR_VAR="SERVER_DIR"
```

Variable name inside `/etc/mc/<server>.conf` that points to the server directory.

```bash
SPIGOTDL_EXCLUDE_REGEX='(^$|^\..*|^backups?$|^backup$|^cache$|^logs?$|^plugins?$|^old$|^archive$)'
```

Folders matching this regex are hidden from the interactive server picker.

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
SPIGOTDL_RESTART_CMD_TEMPLATE='sudo systemctl restart mc@{instance}'
```

Command printed for restarting the server.

```bash
SPIGOTDL_PLUGMAN_CMD_TEMPLATE='plugman load {filename}'
```

Command printed for PlugMan.

Available placeholders:

```text
{instance}
{filename}
{dest}
{server_dir}
{plugin_dir}
```

## Example config

See [`example.spigotdl.conf`](example.spigotdl.conf).

## Running without sudo

The script only needs write access to the target `plugins/` folder.

A simple group-based setup:

```bash
sudo groupadd -f mcadmin
sudo usermod -aG mcadmin "$USER"

sudo chgrp -R mcadmin /opt/mc/*/plugins
sudo chmod -R g+rwX /opt/mc/*/plugins
sudo find /opt/mc/*/plugins -type d -exec chmod g+s {} \;
```

Log out and back in, then test:

```bash
spigotdl server1 19254
```

If parent directories block traversal, allow group execute:

```bash
sudo chgrp mcadmin /opt/mc /opt/mc/*
sudo chmod g+x /opt/mc /opt/mc/*
```

## Notes

This tool is intended for free Spigot resources. Premium resources may require downloading through your Spigot account or using browser cookies

Some resources use external download URLs. `spigotdl` warns before continuing and validates that the final download is a real `.jar`.
