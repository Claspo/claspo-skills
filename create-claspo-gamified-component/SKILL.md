---
name: create-claspo-gamified-component
description: >
  Create or edit Claspo gamified (prize-based gaming) web components extending BaseGamifiedComponent with prize pool integration,
  game mechanics, animations, and editor controls.
  Use when: (1) Creating a gamified/gaming component (scratch card, gift box, wheel of fortune, treasure hunt, memory cards, slot machine, etc.),
  (2) User says "create gamified component", "new game component", "add a prize game", "build a scratch card/gift box/wheel",
  (3) Edit/update/modify existing gamified components,
  (4) User says "edit gamified component", "update game component", "fix/change existing game", "continue working on {name}",
  (5) Any modification to an existing component with prize-based gaming mechanics,
  (6) Any component involving prize-based gaming mechanics (spin, scratch, pick, search, match, gesture),
  (7) Extending BaseGamifiedComponent or working with connectToPrizePool/connectToForm,
  (8) Creating components with game animations, demo mode, and prize pool integration.
  Do NOT use for: regular presentation or input components without gaming mechanics — use create-claspo-component instead.
---

# Create or Edit Claspo Gamified Component

Create or edit a Claspo gamified (prize-based gaming) web component. All gamified components extend `BaseGamifiedComponent` (which extends `WcElement`) and integrate with the Claspo prize pool system.

**Design principle:** Gamified components should be colorful and animated by default. Game items (eggs, boxes, cards, etc.) should each have a distinct, vibrant color — never the same color for all items. Themes provide numbered asset files (`1.svg`, `2.svg`, ...) with different colors per item, cycled via `images[index % images.length]`. Animations (hover, idle jelly, ready-to-play pop, entry) are mandatory for all game items.

---

## Step 0: Detect Intent (Create vs Edit)

Before asking any questions, determine whether the user wants to **create** a new component or **edit** an existing one.

**Detection signals:**

| Signal words | Intent |
|---|---|
| "create", "new", "build", "add", "make" | → **CREATE** — skip to Step 1 |
| "edit", "update", "modify", "fix", "change", "continue", "work on", "improve" | → **EDIT** — go to Step E1 |

If ambiguous → ask: "Are you creating a new gamified component or editing an existing one?"

---

## EDIT Flow (Steps E1–E4)

### Step E1: Identify the Component

If the user named the component explicitly → extract PascalCase base name using the same normalization rules as Q1 (strip "Component" suffix, split by spaces/hyphens/underscores, capitalize, join). Derive `{Name}Component` class name and directory.

If no name was given → auto-scan for `*Component/` directories containing a `.manifest.js` file with `componentType: "PRIZE_BASED_GAMING"`. Present the found list and ask the user to select one.

### Step E2: Locate and Read All Files

Find the component directory and read all files present:

- `{Name}Component.js`
- `{Name}.manifest.js`
- `getStyleElement.js`
- `defaultPrizeStyles.js` (if present)
- `{Name}Utils.js` (if present)
- `assets/img/component-icon.svg`

Also verify that `BaseGamifiedComponent` is present in the project. If not found, stop and tell the user:

> "BaseGamifiedComponent was not found in this project. This is a required base class for all gamified components. Please obtain it from the Claspo technical team before proceeding."

### Step E3: Validate Component

Run all checks below against the files read in E2. Group findings by severity.

**Gamified component checks:**
- [ ] Class extends `BaseGamifiedComponent`
- [ ] Manifest has `componentType: "PRIZE_BASED_GAMING"`
- [ ] Manifest has `PRIZE_SETTINGS` control in `floatingControlsModel`
- [ ] Manifest does NOT include forbidden fields: `children`, `focusParentOnClick`, `preventDraggable`, `recursiveRemove`, `actionsConditions`, `isExternalStartCapable`, `waitForResourcesLoad`, `resourcesPropPaths`, `events`, `syncEnabled`, `stylesImitationEnabled`
- [ ] Manifest controls do NOT include sync-related fields: `hideSyncSelect`, `syncSelectDisplayCondition`, `syncSelectOptions`
- [ ] `observeProps` calls `this.applyAutoAdaptiveStyles(next.adaptiveStyles)`
- [ ] `metaDescription.label` has an `"en"` key
- [ ] Property pane labels are human-readable strings (e.g., `"Grid Size"`) — not `DOCUMENT_*` keys
- [ ] Game items use distinct colors (themed assets cycled via `images[index % images.length]`)
- [ ] `getStyleElement.js` contains animation patterns (hover, idle, ready-to-play)

**Naming convention checks (from create-claspo-component):**
- [ ] Directory is named `{Name}Component/`
- [ ] Class is named `{Name}Component`
- [ ] Manifest file is named `{Name}.manifest.js`
- [ ] Custom element tag is kebab-case of the base name
- [ ] No `Sys` prefix unless explicitly intentional
- [ ] Class methods are normal (not arrow functions), except `constructor`, `connectedCallback`, `disconnectedCallback`, static methods, getters/setters

If violations found, report them and ask whether to fix:

```
Found {N} issue(s) in {Name}Component:

MUST FIX:
1. Missing PRIZE_SETTINGS control in manifest — required for all gamified components
   Suggested fix: add PRIZE_SETTINGS to floatingControlsModel

SUGGESTIONS:
2. Property pane label uses DOCUMENT_* key — should be a human-readable string
   Current: "DOCUMENT_GRID_SIZE" → Suggested: "Grid Size"
```

Ask: "Would you like me to fix these issues? (yes / no / select)"

### Step E4: Proceed with Edit Request

After validation (and any optional fixes applied), ask: "What changes would you like to make?"

Apply all requested edits following the same rules and conventions as the CREATE flow (Steps 2–5). Generate only the files that need to change.

---

## CREATE Flow

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

**Check for BaseGamifiedComponent.** After locating the components directory, verify that `BaseGamifiedComponent` is present in the project (search for `BaseGamifiedComponent.js` or its import in existing components). If not found, stop and tell the user:

> "BaseGamifiedComponent was not found in this project. This is a required base class for all gamified components. Please obtain it from the Claspo technical team before proceeding."

Do not continue with code generation until the user confirms it's available.

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
