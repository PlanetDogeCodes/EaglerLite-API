# API reference

The API is currently v1.0. Most stuff is reachable from `window.EL` (window.EaglerLite) or per-mod `api`.

## Global

- `EL.version` - string, currently `'1.0.0'`.
- `EL.boot` - boot config the loader was constructed with: `{ ver, opts, cfg, mods?, store? }`. `opts` may carry `relays`, `servers`, `joinServer`, `joinCode`, `lang`; `cfg` may carry `menuKey`, `inMenuMods`, `debug`.
- `EL.mods` - currently registered mods (not on `api`).
- `EL.log(msg)` - prints a timestamped line to a ring buffer, and mirrors to console when debug is on.
- `EL.dump()` - copy of the current log buffer.
- `EL.toast(msg, ms?, kind?)` - bottom-right toast; `ms` default 3400; `kind: 'err'` for the red border. Same function as `EL.ui.toast` because yes.
- `EL.registerMod(meta, init)` - see mod-format.md for more info about this.
- `window.EL` / `window.EaglerLite` - self explanatory

## EL.events (see events.md)

- `on(name, fn)` - subscribes to events until unsubscribed. returns an unsubscribe function.
- `once(name, fn)` - fires once, then unsubscribes. returns an unsubscribe function.
- `off(name, fn)` - remove a handler by its reference.
- `names()` - the event names that have handlers.


## EL.game

- `screen()` - current screen's Java class name, or `null` when no GUI is open or before the first screen event.
- `screenState()` - `{ name, width, height, scale }` in GUI units.
- `screens()` - recent screen history `[{ name, t }]` from oldest to newest (newest last), max amount 60.
- `inWorld()` - is true when no screen is open and a world-related screen has been seen this session.
- `onTitle()` - is true when the current screen matches `GuiMainMenu`.
- `click(xPct, yPct)` - do a mouse click (returns bool).
- `key(code, key?, keyCode?)` - do a fake keydown+keyup on `window` (returns bool)
- `type(text)` - keydown/keypress/keyup per character, for text fields.
- `escape()` - fake press `key('Escape', 'Escape', 27)`.
- `canvas()` - the game canvas element.
- `chat(text)` - opens chat, types text, presses Enter (returns false when a screen is open).
- `screenshot(cb)` - after the next rendered frame, takes a screenshot (`cb` receives a PNG dataURL or `null`, returns false when `cb` is not a function).
- `disconnect()` - opens the pause menu (Esc) and clicks Save and Quit (returns false when already in a GUI screen).

## EL.movement

- `isDown(code)` - is a key currently held.
- `tap(code)` - synthetic key press.
- `press(code, ms?)` - synthetic hold, default 250 ms unless changed.
- `walk(dir, seconds?)` - `dir` is `'forward'`, `'back'`, `'left'` or `'right'`; holds the key for `seconds` (default 1) (returns false for an unknown direction).
- `jump()` - presses space for ~280 ms.
- `sprint(on)` / `sneak(on)` - hold/release ControlLeft / ShiftLeft (returns bool).
- `look(dx, dy)` - synthetic mousemove with `movementX`/`movementY` at the canvas center (returns bool).

## EL.input

- `isDown(code)` - boolean, tracked from real key events.
- `pressed()` - all currently-held keys by their codes.
- `mouseAt()` - `{ x, y }` last mouse position.
- `bind(combo, fn)` - bind a combo like `'ctrl+shift+k'` which fires on a set keydown (returns an unsubscribe function).

## EL.net

- `sockets()` - last 40 `[{ url, opened, rx, tx, t, blocked }]` from the sockjet registry (rx/tx in bytes).
- `recent(n?)` - last `n` entries, default 5.
- `block(match)` - block new sockets of a specific  url. `match` is a substring string with a `.test(url)` method (e.g. RegExp), or a `fn(url)` (returns an unblock function).
- `stats()` - `{ open, total, rx, tx }` over the registry.

Returning `false` blocks an operation, returning a value rewrites it.

## EL.graphics

- `onShader(fn)` - subscribe to `graphics:shader` (returns an unsubscribe function). Handler receives `{ source, type, rewritten }`, return a string (or assign `ev.source`) to replace the shader source (only sources longer than 40 characters pass through).
- `onTexture(fn)` - subscribe to `graphics:texture` (returns an unsubscribe function, hooks to `texImage2D`/`texSubImage2D`). Handler receives `{ target, level, source, width?, height? }`, return a replacement (or assign `ev.source`) - it must be valid for textures, otherwise its ignored.
- `onFrame(fn)` - subscribe to `graphics:frame` (returns an unsubscribe function), fires once per animation frame once the WebGL context exists.
- `onContext(fn)` - runs `fn({ canvas, type, attrs })` right before the game creates its WebGL context (returns an unsubscribe function). Mutating `attrs` changes how the context gets made (`attrs.antialias = false`, `attrs.powerPreference = 'high-performance'`, that kind of thing). The game only makes its context once, so this has to be registered while the game hasn't booted yet for it to apply.
- `gl()` - the captured WebGL context, or `null`, depending on what's happening onscreen.
- `canvas()` - game canvas (fallback `EL.game.canvas()`).
- `resolution()` - `{ width, height, scale, px }`: GUI dimensions (width and height), approximated scale (scale), exact pixel compared to GUI ratio (px).
- `fullscreen()` - toggles fullscreen (returns bool).
- `addOverlay(el)` - overlay content.
- `removeOverlay(el)` - remove the overlayed contetn.

## EL.storage / EL.gameStorage / api.storage

`elmod:global:`, `elmod:game:`, and `elmod:<mod id>:` in `api`.

- `get(key, def?)` - supposed to be a JSON-parsed value, raw string when not JSON and `def` when missing.
- `set(key, value)` - stores JSON, fires `storage:change` (returns bool).
- `remove(key)` - remove stuff
- `keys()` - store stuff using your own keys
- `clear()` - remove the stuff from `keys()`
- `watch(fn)` - `fn({ key, value })` for stuff written (returns an unsubscribe function).
- `exportAll()` - exports a JSON string in format `{ schema, prefix, data }`.
- `importAll(json, merge?)` - number of imported keys or `-1` on bad input, clears the store first unless `merge`.
- `size()` - approximated amound of keys stored.

## EL.hud

- `register(id, el, opts?)` - register something from DOM as a HUD item. `opts`: `{ anchor, left, top }`, anchor can be `'tl'` (top-left), `'tr'` (top-right), `'bl'` (bottom-left) or `'br'` (bottom-right). Any stored position is favored, then `left`/`top`, otherwise the item goes to its `anchor` value (default is `tl`). Returns a handle `{ id, el, move(x, y), reset(), save(), pos() }` or `null`.
- `unregister(id)` - removes the HUD item (returns bool).
- `list()` - lists the current HUD stuff in format `[{ id, x, y, w, h, anchor, fixed }]`.
- `move(id, x, y)` - repositions HUD stuff (returns bool). Call `save()` to make sure it stays that way.
- `reset(id)` - clears the current HUD position, returns the item to its anchor (returns bool).
- `resetAll()` - resets every HUD item.
- `onLayout(fn)` - subscribe to `hud:layout`.
- `arrange(on?)` - toggle drag-arrange mode for the HUD widgets (apps, widgets, whatever). While active, items are draggable with snapping to edges/centers of the canvas and other HUD apps, and the positions save automatically.

for `api`, `api.hud.register`/`unregister` must prefix ids with `<mod id>:`.

## EL.ui

- `toast(msg, ms?, kind?)` - see `EL.toast`.
- `panel(opts?)` - HUD panel with `opts`: `{ title, anchor, left, top, visible }`. Returns `{ el, set(text), show(), hide(), destroy(), hud }` where `hud` is the generated ID for an HUD app, and `destroy()` unregisters the HUD item.
- `overlay(el)` - does the same exact thing as `EL.graphics.addOverlay`. Don't even know why I made this to be honest.

## EL.performance

- `fps()` - frames the game actually drew in the last second (counted off the game's own frame deliveries, not a side timer).
- `frameMs()` - last frame difference from its previous frame.
- `drawCalls()` - GL draw calls in the last frame.
- `drawCallsTotal()` - every GL draw call since the context got made.
- `uptime()` - how many ms since the API booted.
- `every(ms, fn)` - frame timing, `ms` can't be lower than 50 (returns a stop function).

## EL.util

- `wrap(obj, name, fn)` - replace `obj[name]`. `fn(ret, orig)` receives `ret = { skip, args, result }` and the original function, and you can set `ret.skip = true` to return `ret.result` (returns an unwrap function).
- `debounce(fn, ms)` - debounced `fn` (AI made this one beacause it was unreasonably hard to make for some reason).
- `findGlobal(predicate, limit?)` - scan the `window` properties, return `[{ name, value }]` where `predicate(value, name)` (default limit 10).
- `path(obj, 'a.b.c')` - trace a path (`undefined` if it misses).

## EL.source 

DO NOT USE ANY OF THESE UNLESS YOU KNOW **EXACTLY** WHAT YOU'RE DOING.

- `addPatch(fn)` - register stuff over the client script. `fn(code)` returning a string replaces the source.
- `patches()` - number of registered patches.
- `opts()` - the default `eaglercraftXOpts` the game reads (default relays, servers, hooks, etc). Change `window.eaglercraftXOpts` from the API, do not change it here unless you want something to break.

## api.config

Per-mod config access and config panel stuff.

- `get(key)` - current key value stored value or the descriptor's `def`.
- `all()` - object of all values.
- `set(key, value)` - change a mod config using `mod:config`.
- `reset()` - clears stored values back to defaults, fires `{ key: null }`.
- `describe()` - copy of the normalized config descriptors.

## Mod window

The mod window lists mods with a search box, an enable option per mod, config options (if you spent the time to make them), and an Export button when the mod's source is stored. A mod must contain an `EL.registerMod` call, and new mods get applied to the game almost instantly.

The window is styled like the game's own GUI now, bevels and all, and the config controls are vanilla-style too: ON/OFF buttons for bools, little `<` `>` steppers for numbers and choices.

There's also an Arrange HUD button in the footer now. It closes the window and puts every HUD item into drag-arrange mode at once, so arranging your HUD doesn't need its own mod anymore.

Enable/disable state persists between launches. In the EaglerLite client it goes through the client's mod store, in patched games it's plain localStorage.
