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

**Apps could come up permanently empty ("Nothing here yet"). Fixed as of
2026-09-17 — see below.** This was an upstream Omarchy shell bug, not
something in this plugin: because this manifest declares `"kinds": ["menu",
"bar-widget"]`, the shell instantiates it twice at startup (once per kind)
through `createScopedPluginShell()` in `shell.qml`. The two near-simultaneous
calls can disagree on whether `manifest.kinds` passes `Array.isArray()` (it
does on one call, doesn't on the other, despite printing identically) —
whichever call loses the race gets `appLibrary: null` baked in permanently
for that shell session, so `root.appLibrary` in `Menu.qml` was `null` and
Apps/recent-apps never populated. Restarting the shell (`omarchy restart
shell`) re-rolls the race; it does not reliably fix it. Reported upstream:
[omacom/omarchy#11788](https://github.com/omacom/omarchy/issues/11788)
(closed).

[Yacl222](https://github.com/Yacl222) commented on that issue with a data
point that complicates the race theory above: on their system (also a
`menu`+`bar-widget` clone), the failure was **deterministic**, not
intermittent — `manifestHasKind(manifest, "menu")` evaluated `true` on every
single instantiation across many restarts, with no `Array.isArray`
disagreement, and `appLibrary` still came back `null` every time. That means
there's likely a *second*, still-unidentified shell-side bug producing the
same permanent-null symptom, separate from the `Array.isArray` race
diagnosed above. Yacl222 didn't chase the shell-side cause further because
the workaround from
[omacom/omarchy#11028](https://github.com/omacom/omarchy/issues/11028)
(giving the clone its own local `AppLibrary` instance to fall back to, rather
than depending on the shell ever resolving a non-null one) resolves it
cleanly regardless of which of the two bugs is in play.

That fix is applied here: `Menu.qml` vendors a copy of the shell's own
`AppLibrary.qml`/`AppSearch.js` (see the top of `AppLibrary.qml` in this repo)
and `root.appLibrary` falls back to a local instance of it whenever
`shell.appLibrary` is null, instead of only polling and hoping the shell-side
one eventually resolves.

Separately (also upstream, also in that issue): `PluginAppLibraryApi.qml`
declares an `appsChanged` signal that `shell.qml` never actually emits, so a
plugin that depends on it to refresh once `DesktopEntries` finishes its async
scan gets no signal to act on when using the shell's proxied `appLibrary`.
`Menu.qml` here works around this locally (see `rebuildItemsFromSources()` /
`appLibraryWarmup`) by polling instead of waiting on that signal. The local
`AppLibrary` fallback above isn't affected by this — it's the real component,
not a proxy, so its own `appsChanged` fires normally — but the polling stays
in place as a safety net for the case where `shell.appLibrary` did resolve
non-null and is used instead.

With the fallback in place, Apps/recent-apps populate reliably regardless of
which upstream bug would otherwise have hit — confirmed on a live shell
restart. The upstream issue documents the shell-side root causes in case
anyone wants to fix `createScopedPluginShell()`/`manifestHasKind()` itself,
but this plugin no longer depends on that landing. This repo is currently
private; ping me if you'd like access.

## How this came to be

This is a personal customization for my own Omarchy setup, built with the
help of [Claude Code](https://claude.com/claude-code) (Anthropic's AI coding
agent). I use it daily on my own machine, but I'm not a professional plugin
developer — please read through the source before installing, especially
anything that touches system or network state, and open an issue if
something looks off.

## License

MIT
