---
name: create-claspo-component
description: >
  Create new Claspo web components for plugin projects with full manifest, JS class, styles, and editor integration.
  Use when: (1) Creating a new Claspo component from scratch, (2) Scaffolding a presentation or input web component,
  (3) User says "create component", "new component", "add a component", or describes a UI element to build as a Claspo component,
  (4) Generating component manifest, property pane, floating controls, or context menu configuration,
  (5) Any mention of WcElement, WcControlledElement, or Claspo component creation workflow.
---

# Create Claspo Component

Create a new Claspo web component for a plugin project. Follow this guide step by step: ask questions, then generate files.

## Step 1: Ask Questions

Ask one at a time. Wait for each answer before proceeding.

### Q1: Component Name and Purpose

Ask: "Describe the component you want to create — give it a name and explain what it does."

**Extract from the answer:**
1. **Component name** — extract or infer a PascalCase name
2. **Purpose description** — use to determine type, suggest elements, guide generation

**Name normalization** — accept any format, convert to PascalCase base name (without "Component" suffix):

| User input | Normalized |
|---|---|
| `MyText` | `MyText` |
| `my text` or `My Text` | `MyText` |
| `MyTextComponent` | `MyText` (strip suffix) |
| `my-text` / `my_text` / `myText` | `MyText` |

Rules: strip `Component` suffix -> split by spaces/hyphens/underscores -> capitalize each segment -> join.

**Derive all names** (example: `MyText`):

| Value | Rule | Result |
|---|---|---|
| Directory | `{componentsDir}/{Name}Component/` | `components/MyTextComponent/` |
| Class name | `{Name}Component` | `MyTextComponent` |
| Manifest file | `{Name}.manifest.js` | `MyText.manifest.js` |
| Component file | `{Name}Component.js` | `MyTextComponent.js` |
| Custom element tag | kebab-case | `my-text` |
| Manifest `name` | `"{Name}Component"` | `"MyTextComponent"` |
| `metaDescription.label.en` | split PascalCase | `"My Text"` |
| `control.name` (input only) | camelCase | `"myText"` |
| `control.integrationName` (input only) | camelCase | `"myText"` |

**Determine component type from purpose:**
- Captures/submits user input (slider, rating, text entry, selection, checkbox) -> **Input** (extends `WcControlledElement`)
- Only displays content (text, image, divider, decorative) -> **Presentation** (extends `WcElement`)

Present derived names AND inferred type for confirmation before proceeding.

### Q1b: Components Directory (auto-detect)

Auto-detect by scanning for directories containing `*Component/` subdirectories with `.js` manifest files. Common locations: `components/` (plugin projects), `src/` (claspo-components repo).

If multiple candidates or none found, ask the user.

**Never add `Sys` prefix by default.** Only add if user explicitly says "internal component" / "add Sys prefix" / "system component". Do NOT ask.

### Q2: Visual Elements

Ask: "What visual elements does your component have? (comma-separated list)"

Use purpose from Q1 to **suggest elements** contextually:
- "slider with a knob" -> suggest: `label, sliderTrack, sliderKnob`
- "star rating" -> suggest: `label, starsContainer, star`
- "simple text" -> suggest: `text`

Every component has `host` implicitly — do NOT list it.

### Q3: Inline Editable Elements

Ask: "Which elements should support inline text editing? (comma-separated, or 'none')"

These allow double-click editing in the editor. Each maps to `props.content.{elementName}`.

### Q4: Customizable Style Properties

Ask: "What style properties should be customizable in the editor?"

Read `PropertyPaneManifest.interface.d.ts` for `ClPropertyPaneManifestModelName` enum and `FloatingControlManifest.interface.d.ts` for `FloatingControlManifestModelName` enum.

**Every suggestion must be contextual.** Use the component's purpose (Q1) and elements (Q2) to suggest only relevant properties. For each, explain *what element* it applies to and *why*.

**Example for a slider** (elements: label, sliderTrack, sliderFill, sliderKnob, valueDisplay):
- **size** — overall width for layout fit (PP: `SIZE`, floating: `SIZE`)
- **margins** — spacing around slider (PP: `INDENTATION` MARGIN, floating: `MARGIN`)
- **text-params** for label — font/color/alignment (PP: `TEXT_PARAMS`)
- **track color** — sliderTrack background (PP: `COLOR_SELECT`)
- **fill color** — filled portion color (PP: `COLOR_SELECT`)
- **knob color** — draggable handle (PP: `COLOR_SELECT`)
- **track height** — thickness in px (PP: `NUMBER_INPUT` with `valueTransformers: [{"name": "px"}]`)
- **knob size** — diameter in px (PP: `NUMBER_INPUT` with `valueTransformers: [{"name": "px"}]`)
- **border-radius** — rounding (PP: `BORDER_RADIUS`)

For the full controls reference, see [references/manifest-guide.md](references/manifest-guide.md) > "Property Pane Controls Reference".

### Q5: Desktop/Mobile Style Separation

Ask: "Which style properties should support different values for desktop and mobile? (comma-separated, or 'all-same')"

Refer to `Document.interface.d.ts` for `BaseComponentAdaptiveStylesI`, `ClBaseComponentElementParamsI`, `ClComponentStyleAttributesI`.

- Properties that differ per env -> `props.adaptiveStyles` (separate desktop/mobile)
- Properties same for both -> `props.styles` (env-independent array)
- **Default to `cloneStylesToAllEnvs()`** — start identical, let users customize per-env later

### Q6 (Input only): Input Control Type

Ask: "Is this a single-value input or a multiple-choice input?"

Infer from Q1 when obvious:
- Slider, text field, date picker -> `componentType: "INPUT"`
- Checkbox list, radio group, choice buttons -> `componentType: "MULTIPLE_INPUT"`

For `mappingTypes`, read `ManifestMappingType` enum from `ComponentManifest.interface.d.ts`.

### Q7 (Input, multiple-choice only): Default Options Count

Ask: "How many default options should the component have? (e.g., 3)"

Generate `props.control.options` with `option_1`, `option_2`, etc.

---

## Step 2: Read Reference Files

Before generating code, read platform interfaces AND existing component examples.

- **Platform interfaces and SDK**: See [references/platform-api.md](references/platform-api.md)
- **Documentation guides**: Start with `https://docs.claspo.io/llms.txt` for full index
- **Existing component examples**: See [references/platform-api.md](references/platform-api.md) > "Existing Component Examples"

---

## Step 3: Generate Files

For file structure and all code templates, see [references/code-templates.md](references/code-templates.md).

For the manifest template and all manifest-related guidance, see [references/manifest-guide.md](references/manifest-guide.md).

---

## Step 4: Auto-Registration

After generating all component files:

1. Find `claspo.config.js` in project root. Add `"{Name}Component"` to the `useComponents` array.

2. Find the editor components panel config (typically `quickstart/frontend/src/config/components-panel.ts`). Add `{ componentName: '{Name}Component' }` to an appropriate group, or create a new group:
   ```javascript
   {
     label: 'My Components',
     components: [
       { componentName: '{Name}Component' }
     ]
   }
   ```

3. If either file is not found, print registration instructions for the user.

4. Tell the user to restart the dev server (`npm run dev`).

---

## Restrictions

### Manifest fields to NEVER include

Internal/system-managed fields: `children`, `focusParentOnClick`, `preventDraggable`, `recursiveRemove`, `actionsConditions`, `isExternalStartCapable`, `waitForResourcesLoad`, `resourcesPropPaths`, `events`, `syncEnabled`, `stylesImitationEnabled`.

### Sync-related control fields to NEVER use

Since we never add `syncEnabled`, these must NEVER appear on any control: `hideSyncSelect`, `syncSelectDisplayCondition`, `syncSelectOptions`.

### Controls are conditional on Q5

Only include editor controls for style properties the user selected in Q5:
- No "size" = no SIZE in floatingControlsModel or propertyPaneModel
- No "margins" = no MARGIN floating, no INDENTATION (MARGIN) in property pane
- No "padding" = no CONTAINER_PADDING, no INDENTATION (PADDING)
- etc.

### Code conventions

- Write normal class methods. Babel plugin auto-transforms to arrow functions. Excluded: `constructor`, `connectedCallback`, `disconnectedCallback`, static methods, getters/setters.
- Always call `this.applyAutoAdaptiveStyles(next.adaptiveStyles, next.styles)` inside `observeProps`.
- **Property pane control labels must be human-readable strings** (e.g., `"Track Color"`, `"Knob Size"`). Do NOT use `DOCUMENT_*` keys — those are internal editor translation keys for built-in system controls only.
- For `metaDescription.label`, generate `"en"` only.
- For all available methods, see [references/platform-api.md](references/platform-api.md) > "Key WcElement Methods".
