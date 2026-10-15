# rng2 – work session log (Oct 11–15, 2026)

Roblox game **rng2** (place 82994414546943, universe 10768788764, group /breaching). Edited live in Roblox Studio through the Studio tools. This file summarises every request in this session and what was changed, so it can be loaded into a knowledge graph.

## Project conventions (carried over)
- UI text style: heavy/bold italic GothamSSm, black `UIStroke` with `StrokeSizingMode = ScaledSize` (~0.06–0.1), black "depth" shadow copy via tag `DepthText` + attribute `DepthAmount` (script `StarterPlayerScripts.TextDepth`), gradients light top → dark bottom, square red close buttons.
- Labels inside a `UIListLayout` can't use `DepthText` (the copy joins the layout) → wrap them in a holder Frame first (pattern used for `TitleHolder`, `NameHolder`).
- Stop Play before Edit changes; script edits go through `multi_edit` / `ScriptEditorService:UpdateSourceAsync`.
- Studio screenshots don't render Highlights and sometimes stop drawing the 3D view (black/blank captures).
- Rarity colours: `ReplicatedStorage.Shared.Constants.ColorsConstant.RARITY`.

---

## 1. Box Shop restyle (`GameUI.Main.StockShop`)
- Title moved into `Top.TitleHolder.Title` (holder keeps the depth copy out of `Top`'s UIListLayout); text "Box Shop" (set by `StocksController.Open` as `<stockName> Shop`).
- Title, restock timer, RESTOCK, X, item name, stock, luck, prices: heavy italic, black ScaledSize strokes, gradients, depth shadows.
- Box name + rarity text use the rarity gradient (`rarityTextGradient()` in `StocksController`): flat rarities get light/dark variants, dark-first sequences are flipped.
- Rarity label shimmer changed from diagonal sweep to a gentle vertical Sine offset tween.
- Rarity text shown in CAPS (`string.upper(itemData.Rarity)`).
- Removed a debug `print` from `StocksController`.

## 2. Upgrades
- Upgrade shop was broken: `UpgradesController` still referenced the removed `Buttons.Robux`. Robux upgrade code removed – upgrades are money-only. (`ProductsConfig.Upgrades` entries now unused.)
- Restyle: dark blue gradient container with bubble pattern (`rbxassetid://114005099282952`), blue row/icon gradients, black outlines everywhere (upgrade name stroke forced black in script), `x1` silver gradient, `x1.1` green gradient, white price.
- Locked overlay text: `REBIRTH <n>` in red gradient (was "You need X rebirth!").
- Title renamed `UPGRADES`, moved into `Top.TitleHolder` with depth shadow; upgrade names moved into `NameHolder` with depth shadow (`createUpgrade` finds `UpgradeName` recursively).

## 3. Roll animation fixes (`RollController`)
- Spam-clicking Roll / Auto started overlapping rolls → `LaunchRoll` now sets `rolling = true` before `InvokeServer` and returns if already rolling; Lucky/Ultra box use blocked while rolling or auto.
- Random flashes no longer play the idle animation (only the final result animates).
- Hidden auto-roll cards smaller (`HIDDEN_ROLL_SCALE` 0.75 → 0.6); card spacing `RollList.UIListLayout.Padding` 0.02 → 0.05.
- "More satisfying" roll: flash ticks punch in (`TICK_PUNCH` 1.42 → `REST_SCALE` 1.25), tick SFX pitch rises (`playPitched`), wind-up squeeze before reveal, results pop in from `REVEAL_FROM` 0.4 one after another, rarity-tinted glow burst behind result, character light flash, shake for Epic+.
- HUD roll button label always reads `ROLL` (was the box name).
- Removed the `ShowAll` arrow (box list stays open).
- AutoRoll frame strokes converted to ScaledSize.

## 4. Viewport / cloth bugs (`StarterPlayerScripts.Utils.SetViewport`)
- Pooled viewport clones could keep a cloth/cape pose → `resetPose()` resets Motor6D/Bone transforms and part CFrames to stored rest pose; animation tracks are destroyed when returned to the pool.
- Layered-clothing "ghost" outfits stuck on cards → clones are first moved behind the camera (`FAR_AWAY = CFrame.new(0,0,60)`) and returned to the pool 0.25 s later (`retire()`). (An earlier 1e5 offset broke viewport rendering and was fixed.)

## 5. Mobile fixes
- Auto / Hide buttons: decorative `RollCircle` images had `Active = true` and swallowed taps → set false; handlers switched to `.Activated`; buttons set `Active = true`.
- All 50 `FixedSize` UIStrokes in StarterGui converted to ScaledSize (notification "!" badges, shines, Codes, Options…). Two hidden strokes in Shop `DiceTemplate(2).Content.Template` left unchanged.
- Special reveal moved to its own ScreenGui `StarterGui.SpecialRevealGui` (IgnoreGuiInset, ScreenInsets None, no safe-area clipping) so it is truly full screen; background/flash oversized to 1.4× so screen shake never shows edges.

## 6. HUD box backings
- User-added card images behind the box buttons: renamed `Backing`, centred, 1.2× size, Fit.
- Box picture now drawn by a `BoxIcon` ImageLabel child (button image transparent) so it sits above the backing; `linkBoxIcon()` in `RollController` keeps it synced with `button.Image`.

## 7. Units inventory with stacking (new)
- Server `ServerScriptService.Manager.InventoryManager`: identical units (name + mutation + size) share ONE tool. Tool attributes `Count` and `GUID` (one real GUID of the stack); tool name `Name xN`. `AddTool`/`RemoveTool`/`RefreshInventory` manage stacks; data still stores one entry per unit.
- `PlotHandler` place: uses `RemoveTool` (removes one from the stack) instead of destroying the tool.
- `SellController` counts via `Count` (`countOwned`, `Recount`).
- New client `Controller.UnitsInventoryController` + UI `GameUI.Main.Inventory` (cloned from Index) + HUD button `HUD.Dice.InventoryButton` (left of the roll button).
- Default Roblox hotbar (Satchel) disabled permanently (`InventoryController.SetVisible` → always false). Equip Best moved to an `EQUIP BEST` button in the inventory title bar.
- Inventory features: rarity-gradient cards with animated rim, glow, name + rarity + separate `[SIZE]` / `[MUTATION]` gradient tags with strokes/depth, `xN` amount pill, HOLDING badge, search box, sort (RARITY / AMOUNT / NAME), stacks/units counter, pop-in animation, hover scale, Index idle animations (paused while closed), click to equip / click again to unequip.
- Hover shows the same tooltip as the Index (`IndexController:ShowTooltip/HideTooltip` added; name, rarity, income, chance, serial).
- Shaking grid fixed: card size now computed from `AbsoluteSize` (not `AbsoluteWindowSize`) so the scrollbar can't create a resize loop; same fix applied to `IndexController.layoutSections`; `VerticalScrollBarInset = Always`.

## 8. Escape offer popup (`StarterGui.EscapeOffer` + LocalScript `EscapeOffer`)
- Rebuilt to match other UIs: emerald gradient card, yellow-orange title bar "FREE GIFT!", cloned square close button, gradient texts with strokes/depth, CLAIM! button (text now in `Claim.Title`).
- Animations: card pop-in/out, icon pop-in, floating + rocking icon, spinning light rays, pulsing glow, breathing claim button, icon pop on claim.

## 9. Special reveal → cutscene (`RollController.PlaySpecialReveal`)
Iterations:
1. Stage cutscene: dark stage high above the map, charging orb, burst, character on pedestal, cinema bars, name in bottom bar, click to skip.
2. Sky-dive version: character dives head-first from the sky, camera chases from behind, speed lines/clouds/trail, lands on floating island, camera swings to the front.
3. **Current – "falling star" wish (based on user's reference video 0:01–0:12):**
   - Cloud tunnel fly-through → white flash.
   - Night sky over a sea of clouds (gradient backdrops via SurfaceGui on the **Back** face, LightInfluence 0); blue shooting star streaks across, turns rarity colour with ForceField glow + horizontal lens streak + spark burst + shake.
   - Four more stars rain down, screen floods with the rarity gradient.
   - Character shown as black Highlight silhouette with glowing outline in front of spinning rays + neon ring (positions parallax-corrected), then lights up; golden dust; slow push-in.
   - Name + rarity (incl. mutation) slide in bottom-left; `SKIP >>` top-right. Tap during star → jump to character; tap on character → close.
   - Hides GameUI/BackpackGui/Notification/IndexTooltip, disables controls, restores camera/UI/controls and destroys all temp objects at the end.
- Old reveal children (`BlackOverlay`, `Star`, `Reveal`) deleted; a `FitScale` UIScale is kept on `SpecialReveal` for small screens.

## Open items / ideas
- User to review the wish cutscene and request tweaks (timing, colours, path, tunnel, character entrance, text).
- Test stacked inventory with placing / selling / Lucky Box in a real session.
- `ProductsConfig.Upgrades` Robux entries are unused and can be deleted.
- Workspace asset packs ("Aura VFX Pack", "AnimePack V2") still use Highlight slots (from previous handoff).
