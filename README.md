# EaglerLite Mod API :D

### **DISCLAIMER**
The EaglerLite Client and Mod API are **completely separate projects!** While the Mod API is included as a default patch for the EaglerLite client itself, the two projects are entirely separately maintained.

## What it is
The EaglerLite API is a patch for Eaglercraft 1.12.2 that adds extensive API hooks that mods can interact with for everything from gameplay to networking.

New in v1.0.2: there is a small packet guard baked into the netcode now. Servers running 1.21 (with ViaBackwards) sometimes send the 1.12.2 client packets it can't really deal with - the classic one is withers sitting in water spamming attribute updates, which used to get players kicked. The guard clamps what it can (bad attribute values get fixed up instead of crashing the parser) and drops the frames that can't be saved. Dropped frames fire a `net:guard` event if a mod wants to know how often that happens. Mods don't need to do anything different, it all happens below the hooks.

Also the mod menu is easier to read now - dark list, white text, like the multiplayer screen.

## What this repo has

- `patcher.html` - injects the mod loader into any standalone Eaglercraft 1.12.2 HTML file. No mods are bundled; import them in-game (F6 or the Mods button).
- `docs/` - self explanatory

## How to use the patcher
1. Open `patcher.html`
2. Upload a full Eaglercraft 1.12.2 HTML
3. Download and open the patched HTML
4. In-game, press F6 or use the Mods button on the title/pause menus to upload and manage mods

## Mods
- A mod is a single `.js` file containing `EL.registerMod`.
- The in-game mod window imports `.js` mod files and `.json` mod bundles.
- Imports stay imported through the browser's localStorage.
