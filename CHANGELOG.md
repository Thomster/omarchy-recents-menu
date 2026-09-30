# Changelog

All notable changes to omarchy-recents-menu. Versions follow [Semantic Versioning](https://semver.org/).
Versions before the release of 2026-09-30 were assigned retroactively from the commit history.

## 1.0.3 – 2026-09-30

### Changed

- Author set to "Claude Code / Thomas Alt"; README: Related and Changelog sections; this CHANGELOG.
- Bar widget description aligned with the manifest description.

## 1.0.2 – 2026-09-17

### Fixed

- Apps list could stay permanently empty: fall back to a local AppLibrary when `shell.appLibrary` is null (upstream [omacom/omarchy#11788](https://github.com/omacom/omarchy/issues/11788)); README marks the issue as fixed.

## 1.0.1 – 2026-09-14

### Fixed

- Don't latch an empty apps list; README documents the upstream shell race.

## 1.0.0 – 2026-09-07

### Added

- Initial release: Omarchy menu with a recently-launched-apps row, persisted to `~/.local/state/omarchy/settings/menu-recent-apps.json`. Plugin id `omarchy_plus_recents.menu`.
- README disclaimer on how the plugin was built (2026-09-08).
