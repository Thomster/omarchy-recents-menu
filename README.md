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

## License

MIT
