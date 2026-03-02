---
name: create-claspo-gamified-component
description: >
  Create new Claspo gamified (prize-based gaming) web components extending BaseGamifiedComponent with prize pool integration,
  game mechanics, animations, and editor controls.
  Use when: (1) Creating a gamified/gaming component (scratch card, gift box, wheel of fortune, treasure hunt, memory cards, slot machine, etc.),
  (2) User says "create gamified component", "new game component", "add a prize game", "build a scratch card/gift box/wheel",
  (3) Any component involving prize-based gaming mechanics (spin, scratch, pick, search, match, gesture),
  (4) Extending BaseGamifiedComponent or working with connectToPrizePool/connectToForm,
  (5) Creating components with game animations, demo mode, and prize pool integration.
  Do NOT use for: regular presentation or input components without gaming mechanics — use create-claspo-component instead.
---

# Create Claspo Gamified Component

Create a new Claspo gamified (prize-based gaming) web component. All gamified components extend `BaseGamifiedComponent` (which extends `WcElement`) and integrate with the Claspo prize pool system.

**Design principle:** Gamified components should be colorful and animated by default. Game items (eggs, boxes, cards, etc.) should each have a distinct, vibrant color — never the same color for all items. Themes provide numbered asset files (`1.svg`, `2.svg`, ...) with different colors per item, cycled via `images[index % images.length]`. Animations (hover, idle jelly, ready-to-play pop, entry) are mandatory for all game items.

## Step 1: Ask Questions

Ask one at a time. Wait for each answer before proceeding.

### Q1: Game Name and Mechanics

Ask: "Describe the gamified component you want to create — give it a name and explain the game mechanics (how the player interacts and how the prize is revealed)."

**Extract:**
1. **Component name** — extract or infer PascalCase name
2. **Game mechanics** — how player interacts, what triggers prize reveal

**Name normalization** — accept any format, convert to PascalCase base name (strip "Component" suffix, split by spaces/hyphens/underscores, capitalize, join).

**Derive all names** (example for `CoinFlip`):

| Value | Rule | Result |
|---|---|---|
| Directory | `{componentsDir}/{Name}Component/` | `components/CoinFlipComponent/` |
| Class name | `{Name}Component` | `CoinFlipComponent` |
| Manifest file | `{Name}.manifest.js` | `CoinFlip.manifest.js` |
| Component file | `{Name}Component.js` | `CoinFlipComponent.js` |
| Custom element tag | kebab-case | `coin-flip` |
| Manifest `name` | `"{Name}Component"` | `"CoinFlipComponent"` |
| `metaDescription.label.en` | split PascalCase | `"Coin Flip"` |

**Classify the game mechanic:**

| Mechanic | Description | Reference Component |
|---|---|---|
| Spin | Rotates/spins to land on prize | `SysWheelOfFortuneComponent`, `SysSlotMachineComponent` |
| Scratch | Layer removed by dragging | `SysScratchCardComponent` |
| Pick | Select one item from set | `SysGiftBoxComponent` |
| Search | Search grid with limited attempts | `SysTreasureHuntComponent` |
| Match | Match pairs/groups to win | `SysMemoryCardsComponent` |
| Gesture | Physical interaction fills progress | `SysShakeChristmasTreeComponent` |
| Custom | Unique mechanic | — |

Present derived names AND inferred mechanic type for confirmation before proceeding.

### Q1b: Components Directory (auto-detect)

Auto-detect by scanning for directories containing `*Component/` subdirectories with `.manifest.js` files. Common locations: `components/` (plugin), `src/` (claspo-components repo).

If multiple candidates or none found, ask the user.

**Never add `Sys` prefix by default.** Only add if user explicitly says "internal component" / "add Sys prefix" / "system component". Do NOT ask.

### Q2: Game Configuration Options

Ask: "What configurable options should the game have in the editor?"

Suggest based on mechanic type:
- Grid-based (memory, treasure hunt) → grid size, gap, theme
- Spinning (wheel) → sector colors, pointer position, text styling
- Multi-element (gift boxes, slot) → element count, icons, colors
- Gesture-based (scratch, shake) → duration, brush size, sensitivity

Prize configuration is implicit (`PRIZE_SETTINGS` control) — do NOT list here.

### Q3: Game Interaction Details

Based on mechanic type from Q1, ask targeted follow-ups:

- **Spin**: "How should spinning work? Spins before stopping? Ease in/out? Trigger method?"
- **Scratch**: "How much surface before reveal? Fade remaining layer? Cover pattern?"
- **Pick**: "How many items? Selected item animation? Non-selected reaction?"
- **Search**: "Grid cells? Attempts? Empty vs prize cell behavior?"
- **Match**: "How many pairs? Match behavior? Mismatch behavior? Win condition?"
- **Gesture**: "What gesture? Duration? Visual progress feedback?"
- **Custom**: "Describe interaction flow step by step."

### Q4: Visual Customization

Ask: "What style properties should be customizable in the editor?"

**Always include for gamified components:**
- **Size** (width only, height auto) — PP: `SIZE`, Floating: `SIZE` with `height.hide: true`
- **Margins** — PP: `INDENTATION` MARGIN, Floating: `MARGIN`

**Suggest based on game type:**
- Prize text styling → `TEXT_PARAMS` on `content.prize.styles`
- Prize background → `BACKGROUND` on `content.prize.styles`
- Numeric config → `NUMBER_INPUT` (with `params.validators: { min, max }`, no `valueTransformers`)
- Dropdown options → `SELECT`
- Icon/image picker → `ICON_SELECT`
- Color picker → `COLOR_SELECT`

For the full controls reference, see [references/manifest-guide.md](references/manifest-guide.md).

### Q5: Has View Switcher?

Ask: "Does the component have separate views for the game and the prize/reward?"

Components with view switcher use:
- `content.activeView` prop (`"game"` / `"reward"`)
- `VIEW_SWITCHER` floating control
- `displayCondition` on prize controls to show only in reward view

If the component displays prize text as part of the game, it should have a view switcher.

### Q6: Has Theme Assets?

Ask: "Does the game use themed image assets?"

Theme components store images in `assets/themes/{themeName}/` with numbered files (`1.svg`, `2.svg`, ..., `N.svg`) — one per item variation with **different colors/styles**. Never a single image for all items.

If yes, ask: "Will you provide theme images, or should I create placeholder asset directories?"

---

## Step 2: Confirm Configuration

Present summary for confirmation:

```
Component: {Name}Component
Tag: {kebab-case-name}
Directory: components/{Name}Component/
Mechanic: {type} — {description}
View Switcher: {yes/no}
Game Config Options: {list}
Style Properties: {list}
Theme Assets: {yes/no}

Files to generate:
  - {Name}Component.js
  - {Name}.manifest.js
  - getStyleElement.js
  - assets/img/component-icon.svg
  - {Name}Utils.js          (if complex)
  - defaultPrizeStyles.js   (if displays prize label)
```

Wait for confirmation.

---

## Step 3: Read Reference Files

Before generating code, read platform interfaces AND existing component examples.

- **Platform interfaces, BaseGamifiedComponent, SDK**: See [references/platform-api.md](references/platform-api.md)
- **Code templates**: See [references/code-templates.md](references/code-templates.md)
- **Manifest template and controls**: See [references/manifest-guide.md](references/manifest-guide.md)
- **Animation and interaction patterns**: See [references/animation-patterns.md](references/animation-patterns.md)
- **Documentation guides**: Start with `https://docs.claspo.io/llms.txt` for full index

Pick the most similar existing component based on mechanic type (see table in Q1) and read it as a code pattern.

---

## Step 4: Generate Files

For file structure and all code templates, see [references/code-templates.md](references/code-templates.md).

For the manifest template and all manifest-related guidance, see [references/manifest-guide.md](references/manifest-guide.md).

For animation patterns (hover, ready-to-play, demo mode, game completion), see [references/animation-patterns.md](references/animation-patterns.md).

---

## Step 5: Auto-Registration

After generating all component files:

1. Find `claspo.config.js` in project root. Add `"{Name}Component"` to `useComponents` array.

2. Find editor components panel config (typically `quickstart/frontend/src/config/components-panel.ts`). Add `{ componentName: '{Name}Component' }` to a "Gamified" group, or create one:
   ```javascript
   {
     label: 'Gamified',
     components: [
       { componentName: '{Name}Component' }
     ]
   }
   ```

3. If either file not found, print registration instructions.

4. Tell user to restart dev server (`npm run dev`).

---

## Restrictions

### Manifest fields to NEVER include

Internal/system-managed: `children`, `focusParentOnClick`, `preventDraggable`, `recursiveRemove`, `actionsConditions`, `isExternalStartCapable`, `waitForResourcesLoad`, `resourcesPropPaths`, `events`, `syncEnabled`, `stylesImitationEnabled`.

### Sync-related control fields to NEVER use

Since we never add `syncEnabled`, these must NEVER appear: `hideSyncSelect`, `syncSelectDisplayCondition`, `syncSelectOptions`.

### Controls are conditional on Q4

Only include editor controls for properties the user selected. No selection = no control.

### Code conventions

- Write normal class methods. Babel plugin auto-transforms to arrow functions. Excluded: `constructor`, `connectedCallback`, `disconnectedCallback`, static methods, getters/setters.
- Always call `this.applyAutoAdaptiveStyles(next.adaptiveStyles)` inside `observeProps`.
- **Property pane control labels must be human-readable strings** (e.g., `"Grid Size"`, `"Gap Between Cards"`). Do NOT use `DOCUMENT_*` keys.
- For `metaDescription.label`, generate `"en"` only.
- For all available methods, see [references/platform-api.md](references/platform-api.md).
