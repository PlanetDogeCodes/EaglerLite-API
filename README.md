# EaglerLite Mod API :D

### **DISCLAIMER**
The EaglerLite Client and Mod API are **completely separate projects!** While the Mod API is included as a default patch for the EaglerLite client itself, the two projects are entirely iindependently maintained.

The EaglerLite API is a patch for Eaglercraft 1.12.2 that adds extensive API hooks that mods can interact with for everything from gameplay to networking.

## What this repo has

- `patcher.html` - injects the mod loader into any standalone Eaglercraft 1.12.2 offline HTML file. No mods are bundled; import them in-game (F6 or the Mods button).
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

