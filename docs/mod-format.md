# Mod format

A mod is one `.js` file that calls `EL.registerMod` when it loads.

## EL.registerMod(meta, init)

`meta` fields:

| field | type | notes |
|---|---|---|
| `id` | string | required. restricted to letters, digits, `-`, and `_` |
| `name` | string | displayed in the mod window |
| `version` | string | displayed in the mod window, default `'1.0.0'` |
| `author` | string | displayed in the mod window, optional |
| `description` | string | displayed in the mod window on one line under the mod name |
| `config` | array | config descriptors to be changed from the mod menu, see below |
| `onConfig` | function | actions for config changes, see below |
| `configScreen` | function | for the config UI, see below |
| `enabled` | bool | default state of the mod (`false` = start disabled) |

## init(api)

`init` runs once, when the mod is first enabled. A mod registered as disabled initializes on its first enable.

You can return `{ onDisable }` (optional) to do stuff in response being switched off. Switching the mod back on re-runs `init`, so no `onEnable` is needed.

## config

Descriptors:

```js
config: [
  { key: 'rows', label: 'Rows shown', type: 'number', def: 4, min: 1, max: 10, step: 1 },
  { key: 'theme', label: 'Theme', type: 'choice', def: 'dark',
    options: [{ value: 'dark', label: 'Dark' }, { value: 'light', label: 'Light' }] },
  { key: 'color', label: 'Accent', type: 'color', def: '#7ee48c' }
]
```

| field | notes |
|---|---|
| `key` | required |
| `label` | shown in the mod window's Config panel; defaults to `key` |
| `type` | `bool`, `number`, `string`, `choice`, `color`, or `key` (configurable via user input in the menu) |
| `def` | default value for stuff |
| `min`, `max`, `step` | only slightly janky number sliders |
| `options` | `choice` options: `[{ value, label }]` or plain values |

Values are read through `api.config` (see api-reference.md) and shown in the mod window, with a Reset to defaults button.

## onConfig(changed)

Called after every config change: `changed` is `{ key, value }`, or `{ key: null }` for default values.

## configScreen(api, container, close)

Replaces the regular Config controls with your own UI. `container` is a `div` and `close()` calls collapse the Config panel.

## Per-mod `api`

`init` receives `api`, which is built for the mod and not not `window.EL` itself:

- `api.storage` - namespaced storage with prefix `elmod:<id>:` (global `EL.storage` uses `elmod:global:`)
- `api.config` - the mod's config (get/all/set/reset/describe)
- `api.hud.register(id, ...)` / `api.hud.unregister(id)` - ids are prefixed with `<mod id>:`
- `api.log(msg)` - prefixes the mod id
- `api.id` - the mod id
- `api.enabled()` - current mod enabled state
- `api.events.on` / `once` - skipped while the mod is disabled (values stay registered). Values from these handlers are forwarded from other in-game values, so most in-game events can be handled through `api.events` too.
- `api.graphics.onShader` / `onTexture` / `onFrame` - rendering hooks that only work when the mod is enabled.

Everything else is shared with `window.EL`

## Hello World example

```js
EL.registerMod({
  id: 'hello', name: 'Hello', version: '1.0.0', author: 'you',
  description: 'hello world'
}, function (api) {
  api.events.on('game:world-enter', function () { api.toast('hello world'); });
});
```
