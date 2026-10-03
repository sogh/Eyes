# Improvements

Review of the codebase (single `index.html` + `js/ui/*` modules + 19 JSON scripts), ordered by priority.

## Bugs

All seven are fixed on branch `fix/bug-batch`.

1. **Browser shortcuts hijacked** — the main keydown handler ignores modifiers and calls `preventDefault()`, so Ctrl+R (reload) narrows the gap, Ctrl+P (print) plays a script, Ctrl+D toggles the director, Ctrl+W/Q change scale. Bail out when `ctrlKey`/`metaKey`/`altKey` is held.
2. **Shortcuts fire while typing** — the main handler doesn't skip focused inputs, so typing in test-mode/debug-panel fields triggers eye actions.
3. **Fragile toast wiring / stale globals** — `initializeUIModules` showed toasts by monkey-patching `window.toggleSpiderMode`, `window.setEyeBehavior`, `window.executeScript`, `window.completeScript`. (This did work: top-level functions in a classic script are `window` properties, so the patch replaced what callers resolved — the original review wrongly said toasts never fired.) The real defects: the patching was fragile, `window.isSpiderModeActive` was a stale snapshot, and `window.activeSetIndex` (read by the debug panel) was never exposed.
4. **Director cooldown not enforced within an evaluation** — `lastScriptTime` is set inside the per-set loop but only checked before it, so several sets can start scripts in the same tick.
5. **Interaction scripts leak** — stopping/completing a script doesn't stop its `interactionInstances`. Interactions also target hardcoded set indices and silently no-op if those sets aren't visible.
6. **Persistence edge cases** — reloading mid-script can persist the scripted position/scale/gap instead of the originals; restored values aren't validated (`NaN` transforms possible); eyes moved off-screen can't be recovered without clearing storage (no reset key).
7. **Blink/squint race** — their `requestAnimationFrame` loops aren't cancelled and both write `ry`; magic numbers (`55`, `5.5`) are duplicated.

## Architecture / maintainability

8. **Dead modules** — `js/constants.js` and `js/state.js` are never imported; `index.html` has drifted inline copies. Finish the ES-module split or delete them.
9. **Replace `window.*` globals + monkey-patching** with a small event bus the UI modules subscribe to.
10. **Action registry** — generate from a `name → fn` table; flag unknown actions at load time.
11. **Script list is hardcoded** in `preloadAllScripts`, and director `scriptWeights` covers only 13 of 19 scripts (`rainbow`, `disco`, `slot_machine`, `orbit*` are never chosen). Use a `scripts/index.json` manifest with weight and solo/interaction tags.
12. **Stricter script validation** — known action names, required params, `targetSet` in range.

## Features / UX

13. **Mouse/touch tracking** — tracking mode should follow the cursor/finger; currently nothing is interactive on mobile.
14. **Pause on `visibilitychange`** — ~165 spider eyes with independent timers keep running in background tabs.
15. **Handle window resize** — focal point is initialized from `innerWidth/innerHeight` only once.
16. **Respect `prefers-reduced-motion`.**
17. **Reset key** for active set position/scale/gap (or all state).
18. **Fullscreen toggle (F)** and cursor auto-hide for kiosk/display use.
19. **Persist spider mode / director on/mode** across reloads (optional).

## Docs / housekeeping

20. **`CLAUDE.md` drift** — says 13 scripts (19 exist), "Spider Mode" vs help's "Custom eyes", missing J / N / F3 / F4 / Shift+Tab; typo "simlulate".
21. **Help overlay** is defined twice (static HTML + `innerHTML +=`); generate from one keybinding table. Remove dead `document.querySelector('script').onkeydown` line.
22. **README** with live link (eyes.doublejosh.com), screenshot, controls; trim `SCRIPTED_BEHAVIOR_DESIGN.md` into a script-format reference.
23. **Tests** — Playwright smoke test (loads, keys toggle sets, no console errors) and a JSON-schema check over `scripts/*.json`.
