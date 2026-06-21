# Changelog

## [1.3.0] - Multi-Source Search, Menus, Overview, and Registry v2

### Added

- Added provider interface for multiple plugin sources.
- Added Spigot provider through Spiget.
- Added Modrinth provider.
- Added Hangar provider.
- Added `search` command with provider, API/loader, Minecraft version, release type, and sort options.
- Added provider references:
  - `spigot:<resource_id>`
  - `modrinth:<project_slug_or_id>`
  - `hangar:<namespace>/<project>`
- Added interactive main menu.
- Added SP-DL ASCII branding with faucet art.
- Added provider-specific color tags:
  - Spigot: yellow
  - Modrinth: green
  - Hangar: cyan
- Added `overview` command for server/plugin/update status.
- Added registry v2 with provider-aware tracking.
- Added automatic registry migration from v1-style Spigot registry data.
- Added overview detail mode for outdated plugins and API check failures.

### Changed

- Reworked the project from a single-source downloader into a provider-based plugin manager.
- Kept legacy command compatibility for `spigotdl <server> <spigot-resource-id>`.
- Updated README for the new v1.3.0 command structure.
- Updated example config to use `SPDL_*` names while keeping compatibility with older `SPIGOTDL_*` environment variables.

### Notes

- Modrinth has the strongest structured search filters.
- Hangar support is implemented as best-effort because public API routes have changed over time.
- Premium/account-gated resources remain unsupported.
