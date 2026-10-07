# Events

Subscribe to events with `EL.events.on(name, fn)` or `api.events.on`. Every event carries `type` set to the event name (except for `graphics:shader` where instead of `type` it's `'vertex'`/`'fragment'`, and `graphics:texture` with no `type` for obvious reasons).

Rewrite/block events (`game:load`, `net:message`, `net:send`) return `false` from a handler to block. To manipulate it, return a replacement value, or set `payload.data` to a new value of the same type. Handlers act one after another in a stack-ish architecture. `api.events.on` forwards return values to the game's native events so mods can do all that too.

## API and mods

| event | payload | notes |
|---|---|---|
| `el:ready` | `{ version, mods }` | fires when the API is finished initializing; previously imported mods from previous sessions register before this.
| `el:menu` | `{ open }` | whether the mod menu is open or not |
| `mod:registered` | `{ id, name }` | registered mod(s) and their name(s) |
| `mod:enabled` | `{ id }` | whether or not mod is enabled |
| `mod:disabled` | `{ id }` | whether or not mod is disabled |
| `mod:error` | `{ id, error }` | self explanatory |
| `mod:config` | `{ id, key, value }` | `key` is `null` when set to defaults |

## Game lifecycle

| event | payload | notes |
|---|---|---|
| `game:screen` | `{ name, prev, width, height, scale }` | the GUI was changed, `null` when no GUI is open |
| `game:title` | `{}` | title screen (`GuiMainMenu`) opened |
| `game:world-enter` | `{}` | player enters a world |
| `game:world-exit` | `{}` | player was in a world, now at title/world list |
| `game:death` | `{ screen }` | `GuiGameOver` opened |
| `game:chat-open` | `{ screen }` | `GuiChat` opened |
| `game:inventory` | `{ screen }` | `GuiInventory`/`GuiContainer` opened |
| `game:pause` | `{ screen }` | `GuiIngameMenu` opened |
| `game:save` | `{ key, value }` | the game wrote profile/settings stuff to localStorage |
| `game:load` | `{ key, value, data }` | (Rewritable and Blockable) the game read a key, `value`/`data` are the raw stored string. Returning `false` makes the load return `null`, returning a string replaces the loaded value with whatever your string was |
| `game:crash` | `{ report }` | the game died :( |
| `game:fps` | `{ fps }` | current fps, returned once per second |
| `game:tick` | `{ n }` | returned every 50 ms, incrementing counter used to measure TPS (if game is slower relative to this = less than 20 TPS, if in-game stuff happens faster relative to this = more than 20 TPS) |

The frame event is `graphics:frame` if you were confused, and there actually isn't a `game:frame`.

## Input

| event | payload | notes |
|---|---|---|
| `input:key` | `{ code, key, keyCode, down, repeat, ctrl, shift, alt }` | window key-capture phase |
| `input:mouse` | `{ x, y, button, down }` | player coordinates in-game |

## Network

| event | payload | notes |
|---|---|---|
| `net:open` | `{ url, socket }` | tunnel constructed not yet connected, `socket` is the live instance |
| `net:connected` | `{ url }` | the client is currently connected to `url` |
| `net:close` | `{ url, code }` | the connection closed with `code` |
| `net:error` | `{ url }` | connection error with `url` |
| `net:message` | `{ url, data }` | (Rewritable and Blockable) inbound network frame; `data` is the raw frame (string or ArrayBuffer); blocking stops the game from seeing it while rewriting it redefines `event.data` |
| `net:send` | `{ url, data }` | (Rewritable and Blockable) outbound network frame; blocking stops it from sending and rewriting replaces the raw frame (string or ArrayBuffer) |
| `net:blocked` | `{ url, dir }` | `dir: 'out'` happens when a blocked URL was refused at open (`EL.net.block`), `dir: 'in'` is when a `net:send` handler dropped a frame on purpose |

## Graphics

| event | payload | notes |
|---|---|---|
| `graphics:context` | `{ gl, canvas, type }` | fires once when the game creates its WebGL context. `gl` is the same context `EL.graphics.gl()` hands out afterwards |
| `graphics:shader` | `{ source, type, rewritten }` | (Rewritable) the shader type and source. Return a string or set `ev.source` to replace the shader source; `type` is `'vertex'`/`'fragment'` |
| `graphics:texture` | `{ target, level, source, width, height }` | (Rewritable) the current texture pack/texture. You can return a replacement source or set `ev.source` but it must be an image/canvas/video/ImageBitmap/ArrayBuffer/typed array |
| `graphics:frame` | `{ t }` | once per animation frame |

## Storage

| event | payload | notes |
|---|---|---|
| `storage:change` | `{ key, value, prefix }` | something was stored using `set`; `prefix` identifies the store |
| `storage:remove` | `{ key, prefix }` | a storage value was removed |

## HUD

| event | payload | notes |
|---|---|---|
| `hud:layout` | `{ id, x, y }` | a HUD item moved; `x`/`y` are `null` after a reset to default |
| `hud:arrange` | `{ on }` | arrange mode toggled (the user is arranging the HUD stuff) |
