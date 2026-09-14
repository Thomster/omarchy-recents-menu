# omarchy-recents-menu

A drop-in replacement for the stock [Omarchy](https://omarchy.org/) command
menu that adds a **recently launched apps** row to the root menu.

Only apps launched from the Apps submenu are recorded. The list is persisted
to `~/.local/state/omarchy/settings/menu-recent-apps.json` so it survives a
shell restart, and re-launching an app already in the list just moves it back
to the front. Everything else — search, drilldowns, icons, uninstall — behaves
exactly like the stock menu, since this is a clone of it with one feature
layered on top.

## Install

```
omarchy plugin add https://github.com/Thomster/omarchy-recents-menu.git
```

Installing switches the bar's menu widget and the `omarchy.menu` IPC target
over to this plugin in place of the stock one.

## Requirements

None beyond stock Omarchy.

## Known issues

**Apps can come up permanently empty ("Nothing here yet").** This is an
upstream Omarchy shell bug, not something in this plugin: because this
manifest declares `"kinds": ["menu", "bar-widget"]`, the shell instantiates it
twice at startup (once per kind) through `createScopedPluginShell()` in
`shell.qml`. The two near-simultaneous calls can disagree on whether
`manifest.kinds` passes `Array.isArray()` (it does on one call, doesn't on the
other, despite printing identically) — whichever call loses the race gets
`appLibrary: null` baked in permanently for that shell session, so
`root.appLibrary` in `Menu.qml` is `null` and Apps/recent-apps never populate.
Restarting the shell (`omarchy restart shell`) re-rolls the race; it does not
reliably fix it. Reported upstream:
[omacom/omarchy#11788](https://github.com/omacom/omarchy/issues/11788).

Separately (also upstream, also in that issue): `PluginAppLibraryApi.qml`
declares an `appsChanged` signal that `shell.qml` never actually emits, so a
plugin that depends on it to refresh once `DesktopEntries` finishes its async
scan gets no signal to act on. `Menu.qml` here works around this locally
(see `rebuildItemsFromSources()` / `appLibraryWarmup`) by polling instead of
waiting on that signal — but this workaround only helps when `root.appLibrary`
itself resolved non-null in the first place. It does not fix the race above.

Until the upstream race is fixed, my own daily-driver `shell.json` runs the
stock `omarchy.menu` instead of this plugin (reliable Apps, no recent-apps
row). This repo is currently private while that's the case.

## How this came to be

This is a personal customization for my own Omarchy setup, built with the
help of [Claude Code](https://claude.com/claude-code) (Anthropic's AI coding
agent). I use it daily on my own machine, but I'm not a professional plugin
developer — please read through the source before installing, especially
anything that touches system or network state, and open an issue if
something looks off.

## License

MIT
