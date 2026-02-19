# Create Claspo Component

You are creating a new Claspo web component for a plugin project. Follow this guide step by step: ask the user questions, then generate the component files.

## Step 1: Ask Questions

Ask the following questions one at a time. Wait for each answer before proceeding.

### Q1: Component Name and Purpose

Ask: "Describe the component you want to create — give it a name and explain what it does."

The user can answer naturally, for example:
- "MySliderInput that will have slider with a knob to drag. Like I want to select numbers with a step"
- "StarRating — user clicks stars 1-5 to rate"
- "MyText — simple text display"
- "a divider line component"

**Extract from the answer:**

1. **Component name** — extract or infer a PascalCase name from the response.
2. **Purpose description** — what the component does. Use this to:
   - Determine component type (Input vs Presentation) automatically
   - Suggest visual elements in later questions
   - Guide HTML template and logic generation

**Name normalization** — accept any format, convert to PascalCase base name (without "Component" suffix):

| User input | Normalized |
|---|---|
| `MyText` | `MyText` |
| `my text` or `My Text` | `MyText` (split by spaces, capitalize each) |
| `MyTextComponent` | `MyText` (strip "Component" suffix) |
| `my-text` | `MyText` (split by hyphens, capitalize each) |
| `myText` | `MyText` (capitalize first letter) |
| `my_text` | `MyText` (split by underscores, capitalize each) |

Rules applied in order:
1. Strip `Component` suffix if present
2. Split by spaces, hyphens, or underscores
3. Capitalize first letter of each segment
4. Join into PascalCase

**Derive all names from the base name** (example for `MyText`):

| Value | Rule | Result |
|---|---|---|
| Directory | `{componentsDir}/{Name}Component/` | `components/MyTextComponent/` |
| Class name | `{Name}Component` | `MyTextComponent` |
| Manifest file | `{Name}.manifest.js` | `MyText.manifest.js` |
| Component file | `{Name}Component.js` | `MyTextComponent.js` |
| Custom element tag | kebab-case of base name | `my-text` |
| Manifest `name` field | `"{Name}Component"` | `"MyTextComponent"` |
| `metaDescription.label.en` | split PascalCase to words | `"My Text"` |
| `control.name` (input only) | camelCase of base name | `"myText"` |
| `control.integrationName` (input only) | same as control.name | `"myText"` |

**Determine component type from purpose:**
- If the component captures/submits user input (slider value, rating, text entry, selection, checkbox, etc.) → **Input** (extends `WcControlledElement`)
- If the component only displays content (text, image, divider, decorative element) → **Presentation** (extends `WcElement`)

Present the derived names AND the inferred type to the user for confirmation before proceeding.

### Q1b: Components Directory (auto-detect)

After confirming the name and type, **auto-detect the components directory** by scanning the project for directories that contain component subdirectories (folders matching `*Component/` pattern with `.js` manifest files inside).

Common locations:
- `components/` — plugin projects (docs pattern)
- `src/` — internal component libraries (claspo-components repo pattern)

If multiple candidates exist, or none are found, ask: "Which directory should the component be created in?"

**Never add a `Sys` prefix by default.** The `Sys` prefix is reserved for internal Claspo system components. Only add the `Sys` prefix if the user explicitly says the component is for internal use (e.g., "this is an internal component", "add Sys prefix", "system component"). Do NOT ask whether it's internal — just react if the user mentions it.

Examples:
- User says "MySliderInput — slider with a knob" → `MySliderInputComponent` (no prefix)
- User says "internal SysRating component" → `SysRatingComponent` (prefix because user explicitly said "internal" / included "Sys")
- Detected dir is `src/` with existing `Sys*` components → still NO prefix unless user says so

### Q2: Visual Elements

Ask: "What visual elements does your component have? (comma-separated list)"

Use the purpose description from Q1 to **suggest elements**. For example:
- "slider with a knob" → suggest: `label, sliderTrack, sliderKnob`
- "star rating" → suggest: `label, starsContainer, star`
- "simple text" → suggest: `text`

Explain:
- Every component has a `host` element implicitly — do NOT list it.
- For presentation: common elements are `text`, `button`, `image`, `container`.
- For input: common elements are `input`, `label`, `button`, `buttonsContainer`, `listContainer`.

### Q3: Inline Editable Elements

Ask: "Which of those elements should support inline text editing in the editor viewport? (comma-separated, or 'none')"

Explain: These elements will allow users to double-click to edit text directly in the editor. Each editable element maps to a `props.content.{elementName}` property.

### Q4: Customizable Style Properties

Ask: "What style properties should be customizable in the editor?"

Read `node_modules/@claspo/common/component-manifest/PropertyPaneManifest.interface.d.ts` for the full `ClPropertyPaneManifestModelName` enum (all available property pane controls) and `node_modules/@claspo/common/component-manifest/FloatingControlManifest.interface.d.ts` for the `FloatingControlManifestModelName` enum (all floating controls).

**Every suggestion must be contextual.** Do NOT dump a generic list. Instead, use the component's purpose (Q1) and visual elements (Q2) to suggest only the style properties that make sense for this specific component. For each suggestion, explain *what element* it applies to and *why* the user would want to customize it.

**Example for a slider component** (elements: label, sliderTrack, sliderFill, sliderKnob, valueDisplay):
- **size** — overall slider width so it fits different layouts (Property Pane: `SIZE`, floating: `SIZE`)
- **margins** — spacing around the slider (Property Pane: `INDENTATION` with `MARGIN` type, floating: `MARGIN`)
- **text-params** for label — font, color, alignment of the label text (Property Pane: `TEXT_PARAMS`)
- **text-params** for valueDisplay — font and color of the number readout (Property Pane: `TEXT_PARAMS`)
- **track color** — background of sliderTrack (Property Pane: `COLOR_SELECT` on sliderTrack element)
- **fill color** — color of the filled portion (Property Pane: `COLOR_SELECT` on sliderFill element)
- **knob color** — color of the draggable handle (Property Pane: `COLOR_SELECT` on sliderKnob element)
- **track height** — thickness of the track in px (Property Pane: `NUMBER_INPUT` on sliderTrack)
- **knob size** — diameter of the knob in px (Property Pane: `NUMBER_INPUT` on sliderKnob)
- **border-radius** — rounding of the track and knob (Property Pane: `BORDER_RADIUS`)

**Example for a simple text component** (elements: text):
- **size** — width/height of the text block (Property Pane: `SIZE`, floating: `SIZE`)
- **margins** — spacing around the text (Property Pane: `INDENTATION` with `MARGIN` type, floating: `MARGIN`)
- **text-params** for text — font size, color, weight, alignment, line height (Property Pane: `TEXT_PARAMS`)
- **background** — background behind the text block (Property Pane: `BACKGROUND`)

**Available Property Pane control types for style customization:**

Host-level controls (apply to the host element):
- `SIZE` (floating: `SIZE`) — width/height
- `INDENTATION` with `indentationType: "MARGIN"` (floating: `MARGIN`) — outer spacing
- `INDENTATION` with `indentationType: "PADDING"` (floating: `CONTAINER_PADDING`) — inner spacing

Element-level controls (apply to specific visual elements via `propPath` or `element`):
- `TEXT_PARAMS` — font styling for text elements
- `BACKGROUND` — background color/image/gradient
- `BORDERS` — border style/width/color
- `BORDER_RADIUS` — corner rounding
- `BOX_SHADOW` — shadow effects
- `HOVER_ANIMATION` — hover effects (only for interactive elements like buttons)
- `COLOR_SELECT` — simple color picker (lighter alternative to BACKGROUND when only a solid color is needed)
- `NUMBER_INPUT` — numeric value input. Supports `params.validators` with optional `min` and `max` to constrain the allowed range in the editor (e.g., `"validators": { "min": 1 }` to prevent zero/negative). Use `valueTransformers: [{"name": "px"}]` when the stored value is a CSS pixel string (e.g., track height, knob size, gap). Omit `valueTransformers` when the stored value is a plain number (e.g., min, max, step, count).
- `TEXT_INPUT` — text/string value input (e.g., placeholder text, custom labels). Do NOT use for numeric values — use `NUMBER_INPUT` instead.

**Choosing the right control type for the value:**
- Numeric values → `NUMBER_INPUT` (with `valueTransformers: [{"name": "px"}]` only for CSS dimension values, omit for plain numbers). Use `params.validators: { min, max }` to constrain the range when the value has logical bounds (e.g., step must be ≥ 1).
- String/text values → `TEXT_INPUT`
- Color values → `COLOR_SELECT`
- Boolean toggles → `SWITCH`

### Q5: Desktop/Mobile Style Separation

Ask: "Which of those style properties should support different values for desktop and mobile? (comma-separated, or 'all-same')"

Refer to `node_modules/@claspo/common/document/Document.interface.d.ts` for the style data structures:
- `BaseComponentAdaptiveStylesI` — `{ desktop: ClBaseComponentElementParamsI[], mobile: ClBaseComponentElementParamsI[] }`
- `ClBaseComponentElementParamsI` — `{ element, styleAttributes, hoverStyleAttributes?, hoverAnimationType?, classes? }`
- `ClComponentStyleAttributesI` — `{ [key: string]: string }` (CSS properties as key-value pairs)

Explain:
- Properties that differ per env go in `props.adaptiveStyles` (separate desktop/mobile objects).
- Properties that stay the same go in `props.styles` (environment-independent `ClBaseComponentElementParamsI[]` array).
- If ALL adaptive styles are the same for both envs, use `cloneStylesToAllEnvs()` from `ModelStyleUtils`.
- **Default to `cloneStylesToAllEnvs()`** for newly created components — start with identical desktop/mobile values and let users customize per-environment later in the editor. Only use separate desktop/mobile objects when you have a strong reason to provide different defaults upfront.

### Q6 (Input only): Input Control Type

Ask: "Is this a single-value input or a multiple-choice input?"

This can often be inferred from Q1's purpose description. If obvious, suggest the answer:
- Slider, text field, date picker → **Single value**: `componentType: "INPUT"`
- Checkbox list, radio group, choice buttons → **Multiple choice**: `componentType: "MULTIPLE_INPUT"`

For the `mappingTypes` array, read the `ManifestMappingType` enum from `node_modules/@claspo/common/component-manifest/ComponentManifest.interface.d.ts` to see all available values. Choose the appropriate type(s) based on the component's purpose. A component can support multiple mapping types (e.g., a number input could use `["TEXT", "INTEGER", "FLOAT"]`).

### Q7 (Input, multiple-choice only): Default Options Count

Ask: "How many default options should the component have? (e.g., 3)"

Generate `props.control.options` with `option_1`, `option_2`, etc.

---

## Step 2: Read Reference Files

Before generating code, read the platform interfaces AND existing component examples.

### Platform Interfaces (read these FIRST to understand available capabilities)

**Manifest structure and enums:**
- `node_modules/@claspo/common/component-manifest/ComponentManifest.interface.d.ts` — `ComponentManifestI` (all manifest fields), `ManifestMappingType` enum, `AutoContrastI`
- `node_modules/@claspo/common/component-manifest/PropertyPaneManifest.interface.d.ts` — `PropertyPaneManifestI`, `ClPropertyPaneManifestModelName` enum (all 30+ property pane control types and their params interfaces)
- `node_modules/@claspo/common/component-manifest/FloatingControlManifest.interface.d.ts` — `FloatingControlManifestModelName` enum (all floating control types: SIZE, MARGIN, CONTAINER_PADDING, ROTATION, etc.)
- `node_modules/@claspo/common/component-manifest/ContextMenuManifest.interface.d.ts` — `ContextMenuManifestModelName` enum (COMPONENT_OPERATIONS, BRING_BACK_FORWARD, FOCUS_PARENT_COMPONENT)

**Document model and component types:**
- `node_modules/@claspo/common/document/Document.interface.d.ts` — `ClComponentType` enum (for `componentType` field), `ClBaseComponentPropsI` (props structure), `ClBaseComponentElementParamsI` (style element structure), `BaseComponentAdaptiveStylesI`

**Base component classes:**
- `node_modules/@claspo/renderer/sdk/WcElement.d.ts` — `WcElement` class (all methods: `getProps()`, `getElement()`, `getRootElement()`, `getHostElement()`, `observeProps()`, `observeShared()`, `observeEnvironment()`, `applyAutoAdaptiveStyles()`, `isStaticRenderMode()`, `getEnvironment()`, `getTranslationsMap()`, `getModel()`, etc.), `ServicesI` interface (all available services)
- `node_modules/@claspo/renderer/sdk/WcControlledElement.d.ts` — `WcControlledElement` class (extends WcElement: `createControlWithValidation()`, `getRegisteredControl()`, etc.), `CreateControlConfigI`, `ValidatorFnT`

**SDK utilities:**
- `node_modules/@claspo/renderer/sdk/ManifestUtils.d.ts` — `cloneControlsToAllEnvs()`
- `node_modules/@claspo/renderer/sdk/ModelStyleUtils.d.ts` — `cloneStylesToAllEnvs()`, `getAdaptiveStylesForPlatform()`, `replaceStyleAttributes()`, `patchStyleAttributes()`
- `node_modules/@claspo/renderer/sdk/HtmlStyleUtils.d.ts` — `setStylesToElement()`, `applyInputLabelStyles()`, `setInputHostSize()`, `setFocusOutline()`
- `node_modules/@claspo/renderer/sdk/FormUtils.d.ts` — `setValidStyles()`, `setInvalidStyle()`, `setPendingStyle()`
- `node_modules/@claspo/renderer/sdk/ColorUtils.d.ts` — color manipulation utilities

**Form system:**
- `node_modules/@claspo/renderer/form/FormControl.d.ts` — `FormControl` class (setValue, getValue, validate, etc.)
- `node_modules/@claspo/renderer/form/FormControlValidator.interface.d.ts` — validator types

### Documentation Guides

Start with the docs index to discover available guides:
- `https://docs.claspo.io/llms.txt` — full index of all documentation pages

Key guides for component creation:
- `https://docs.claspo.io/docs/getting-started-components.md` — **MyText component guide**: step-by-step walkthrough of building a presentation component
- `https://docs.claspo.io/docs/clasporenderer.md` — `@claspo/renderer` SDK reference (WcElement, WcControlledElement, services, utilities)
- `https://docs.claspo.io/docs/property-pane-components.md` — property pane controls reference
- `https://docs.claspo.io/docs/custom-icons-for-components.md` — SVG icon specifications
- `https://docs.claspo.io/docs/validation.md` — custom validation rules for form components

### Existing Component Examples (read as code patterns)

Reference repository: `https://github.com/Claspo/claspo-components-public`

**For Presentation components:**
- `https://github.com/Claspo/claspo-components-public/tree/main/src/SysTextComponent` — simple presentation component (class + manifest)
- `https://github.com/Claspo/claspo-components-public/tree/main/src/SysButtonComponent` — complex presentation with button element

**For Input components:**
- `https://github.com/Claspo/claspo-components-public/tree/main/src/SysChoiceButtonsComponent` — input component with options (class, manifest, componentStyle, componentTemplate)
- `https://github.com/Claspo/claspo-components-public/tree/main/src/SysInputComponent` — single-value input component

---

## Step 3: Generate Files

### File Structure

**Presentation component:**
```
{componentsDir}/{Name}Component/
  {Name}Component.js
  {Name}.manifest.js
  assets/img/
    component-icon.svg
```

**Input component:**
```
{componentsDir}/{Name}Component/
  {Name}Component.js
  {Name}.manifest.js
  componentStyle.js
  componentTemplate.js
  assets/img/
    component-icon.svg
```

### Default Component Icon

Create `assets/img/component-icon.svg` with this default placeholder icon (20x20, `#5F5F5F`, matching existing component icon style):

```svg
<svg width="20" height="20" viewBox="0 0 20 20" fill="none" xmlns="http://www.w3.org/2000/svg">
<rect x="3" y="3" width="14" height="14" rx="2" stroke="#5F5F5F" stroke-width="2"/>
<path d="M10 7V13M7 10H13" stroke="#5F5F5F" stroke-width="1.5" stroke-linecap="round"/>
</svg>
```

---

### Presentation Component JS Template

Refer to `node_modules/@claspo/renderer/sdk/WcElement.d.ts` for all available `WcElement` methods and the `ServicesI` interface for available services (`this.services.*`).

```javascript
import WcElement from "@claspo/renderer/sdk/WcElement";
import manifest from "./{Name}.manifest";
import insertHtmlIntoElement from '@claspo/common/dom/insertHtmlIntoElement';

export default class {Name}Component extends WcElement {
  static define = {
    name: '{tag-name}',
    model: manifest.name,
    manifest: manifest,
  };
  manifest = manifest;

  constructor() {
    super();
    this.getRootElement().innerHTML = `
      <style>
        /* Base styles for each element */
      </style>

      <!-- One div per visual element, each with cl-element="{name}" -->
      <!-- Inline-editable elements also get cl-inline-edit="content, {propName}" -->
    `;
  }

  connectedCallback() {
    super.connectedCallback();

    this.observeProps((prev, next) => {
      this.applyAutoAdaptiveStyles(next.adaptiveStyles, next.styles);

      // For each inline-editable element, update content:
      // if (next.content?.{prop} && this.getElement('{element}')) {
      //   insertHtmlIntoElement({
      //     element: this.getElement('{element}'),
      //     html: next.content.{prop},
      //   });
      // }
    });
  }
}
```

### Input Component JS Template

Refer to `node_modules/@claspo/renderer/sdk/WcControlledElement.d.ts` for all available `WcControlledElement` methods (extends `WcElement`). Key methods: `createControlWithValidation(validators, options)`, `getRegisteredControl()`. See `CreateControlConfigI`, `ControlValidationOptionsI`, and `ValidatorFnT` for config shapes.

For form control methods, see `node_modules/@claspo/renderer/form/FormControl.d.ts` (`setValue`, `getValue`, `validate`, `markAsTouched`, `isValid`, `getErrorKeys`, etc.).

```javascript
import WcControlledElement from "@claspo/renderer/sdk/WcControlledElement";
import manifest from "./{Name}.manifest";
import componentStyles from "./componentStyle";
import componentTemplate from "./componentTemplate";
import insertHtmlIntoElement from '@claspo/common/dom/insertHtmlIntoElement';

export default class {Name}Component extends WcControlledElement {
  static define = {
    name: '{tag-name}',
    model: manifest.name,
    manifest: manifest,
  };
  manifest = manifest;

  constructor() {
    super();
    this.getRootElement().innerHTML = `
      <style>${componentStyles}</style>
      ${componentTemplate}
    `;
  }

  connectedCallback() {
    super.connectedCallback();

    this.registerControl();

    this.observeProps((prev, next) => {
      // Update label, create buttons/options, apply styles
      this.applyAutoAdaptiveStyles(next.adaptiveStyles, next.styles);
    });
  }

  registerControl() {
    const tooltipElement = this.getRootElement().querySelector('.tooltip');
    this.registeredControl = this.createControlWithValidation([], {
      tooltipElement
    });
    this.registeredControl.setValue(null, { silent: true, skipValidation: true });
  }
}
```

### Input componentTemplate.js Template

```javascript
export default `
 <div class="main-container">
  <div class="container-with-label">
    <div cl-element="label"
         cl-inline-edit="content, label"
         class="label">
    </div>
    <div class="container-with-tooltip">
      <!-- Component-specific content here (buttons container, input field, etc.) -->
      <div class="tooltip">
        <svg width="26" height="26" viewBox="0 0 26 26" fill="none" xmlns="http://www.w3.org/2000/svg">
          <path d="M1.5 13.0604C1.5 19.4116 6.6481 24.5605 13.0075 24.5605C19.353 24.5605 24.5 19.4107 24.5 13.0604C24.5 6.70865 19.3531 1.55909 13.0075 1.55908C6.64806 1.55908 1.5 6.7077 1.5 13.0604ZM12.9775 17.9668C12.7032 17.9668 12.4807 17.7443 12.4807 17.47C12.4807 17.1956 12.7032 16.9732 12.9775 16.9732C13.2519 16.9732 13.4743 17.1956 13.4743 17.47C13.4743 17.7443 13.2519 17.9668 12.9775 17.9668ZM12.9775 13.4764C12.7032 13.4764 12.4807 13.254 12.4807 12.9796L12.4807 8.48924C12.4807 8.21487 12.7032 7.99245 12.9775 7.99245C13.2519 7.99245 13.4743 8.21487 13.4743 8.48924L13.4743 12.9796C13.4743 13.254 13.2519 13.4764 12.9775 13.4764Z" fill="#FF0000" stroke="white" stroke-width="2"></path>
        </svg>
      </div>
    </div>
  </div>
</div>
`;
```

### Input componentStyle.js Template

```javascript
export default `
    .main-container {
        height: 100%;
    }

    .container-with-label {
      height: 100%;
      display: flex;
      flex-direction: column;
    }

    .label {
      min-height: 10px;
    }

    .label.cl-focused {
      min-height: auto;
    }

    .container-with-tooltip {
      position: relative;
      display: flex;
      height: 100%;
      width: 100%;
      cursor: pointer;
    }

    .invalid {
      border: 1px solid #ff0000 !important;
    }

    .tooltip {
      visibility: hidden;
      position: absolute;
      right: -10px;
      top: -10px;
      z-index: 1;
      border-radius: 100%;
      width: 20px;
      height: 20px;
      display: flex;
      justify-content: center;
      align-items: center;
      -webkit-touch-callout: none;
      -webkit-user-select: none;
      -khtml-user-select: none;
      -moz-user-select: none;
      -ms-user-select: none;
      user-select: none;
    }
`;
```

---

### Manifest Template

Generate a manifest based on the user's answers. The manifest must conform to `ComponentManifestI` from `node_modules/@claspo/common/component-manifest/ComponentManifest.interface.d.ts`.

For control names, params interfaces, and available options, consult:
- **Property pane**: `ClPropertyPaneManifestModelName` enum and corresponding `*PropertyPaneManifestParamsI` interfaces in `node_modules/@claspo/common/component-manifest/PropertyPaneManifest.interface.d.ts`
- **Floating controls**: `FloatingControlManifestModelName` enum and `Size*FloatingControlManifestParamsI` interfaces in `node_modules/@claspo/common/component-manifest/FloatingControlManifest.interface.d.ts`
- **Context menu**: `ContextMenuManifestModelName` enum in `node_modules/@claspo/common/component-manifest/ContextMenuManifest.interface.d.ts`
- **Props structure**: `ClBaseComponentPropsI`, `ClBaseComponentElementParamsI`, `BaseComponentAdaptiveStylesI` in `node_modules/@claspo/common/document/Document.interface.d.ts`

Use this structure:

```javascript
import {cloneControlsToAllEnvs} from '@claspo/renderer/sdk/ManifestUtils';
// If adaptiveStyles are same for both envs:
// import {cloneStylesToAllEnvs} from '@claspo/renderer/sdk/ModelStyleUtils';

export default {
  "name": "{Name}Component",
  "componentType": "{TYPE}",   // See componentType rules below
  "version": "1.0.0",

  // --- Always include: standard context menu ---
  "contextMenuModel": cloneControlsToAllEnvs([
    {
      "type": "CONTROL",
      "name": "COMPONENT_OPERATIONS"
    },
    {
      "type": "CONTROL",
      "name": "BRING_BACK_FORWARD",
      "element": "host",
      "elementProp": "styleAttributes",
      "elementSubProp": "zIndex"
    },
    {
      "type": "CONTROL",
      "name": "FOCUS_PARENT_COMPONENT"
    }
  ]),

  // --- Floating controls: ONLY include if user selected the property in Q5 ---
  "floatingControlsModel": cloneControlsToAllEnvs([
    // Include SIZE control ONLY if user selected "size"
    // Both dimensions shown by default (no params needed):
    // { "type": "CONTROL", "name": "SIZE", "elementProp": "styleAttributes", "element": "host" },
    // Hide a dimension if the component doesn't need it:
    // { "type": "CONTROL", "name": "SIZE", "elementProp": "styleAttributes", "element": "host", "params": { "height": { "hide": true } } },
    // Include MARGIN control ONLY if user selected "margins"
    // { "type": "CONTROL", "name": "MARGIN", "elementProp": "styleAttributes", "element": "host" },
    // Include CONTAINER_PADDING control ONLY if user selected "padding"
    // { "type": "CONTROL", "name": "CONTAINER_PADDING", "elementProp": "styleAttributes", "element": "host" }
  ]),

  // --- Property pane: ONLY include controls matching Q5 selections ---
  "propertyPaneModel": {
    "content": cloneControlsToAllEnvs([
      // SIZE — only if user selected "size"
      // Always explicitly define each dimension in params. Only include dimensions the component actually needs.
      // If a dimension is hidden in floatingControlsModel, omit it from Property Pane params too — keep both consistent.
      // Both dimensions needed:
      // {
      //   "type": "CONTROL", "name": "SIZE", "element": "host", "elementProp": "styleAttributes",
      //   "params": { "width": { "options": ["fixed", "fill", "hug"] }, "height": { "options": ["fixed", "hug"] } }
      // },
      // Only width needed (height is content-driven):
      // {
      //   "type": "CONTROL", "name": "SIZE", "element": "host", "elementProp": "styleAttributes",
      //   "params": { "width": { "options": ["fixed", "fill"] } }
      // },

      // TEXT_PARAMS — only if user selected "text-params"
      // {
      //   "type": "CONTROL", "name": "TEXT_PARAMS",
      //   "params": [{ "element": "{elementName}" }]
      // },

      // BACKGROUND — only if user selected "background"
      // {
      //   "type": "CONTROL", "name": "BACKGROUND",
      //   "propPath": ["styles", "[element={el}]", "styleAttributes"],
      //   "params": { "onlyColorSelection": true }
      // },

      // BORDERS — only if user selected "borders"
      // {
      //   "type": "CONTROL", "name": "BORDERS",
      //   "propPath": ["styles", "[element={el}]", "styleAttributes"],
      //   "params": { "minValue": 1, "maxValue": 5, "hideAdditionalParams": true }
      // },

      // BOX_SHADOW — only if user selected "box-shadow"
      // {
      //   "type": "CONTROL", "name": "BOX_SHADOW",
      //   "propPath": ["styles", "[element={el}]", "styleAttributes"]
      // },

      // BORDER_RADIUS — only if user selected "border-radius"
      // {
      //   "type": "CONTROL", "name": "BORDER_RADIUS",
      //   "propPath": ["styles", "[element={el}]", "styleAttributes"]
      // },

      // HOVER_ANIMATION — only if user selected "hover-animation"
      // {
      //   "type": "CONTROL", "name": "HOVER_ANIMATION",
      //   "propPath": ["styles", "[element={el}]", "hoverStyleAttributes"],
      //   "params": { "animationTypeProp": { "propPath": ["styles", "[element={el}]", "hoverAnimationType"] } }
      // },

      // INDENTATION (PADDING) — only if user selected "padding"
      // {
      //   "type": "CONTROL", "name": "INDENTATION",
      //   "elementProp": "styleAttributes", "element": "{el}",
      //   "params": { "indentationType": "PADDING" }
      // },

      // INDENTATION (MARGIN) — only if user selected "margins"
      // {
      //   "type": "CONTROL", "name": "INDENTATION",
      //   "elementProp": "styleAttributes", "element": "host",
      //   "params": { "indentationType": "MARGIN" }
      // }
    ]),

    // --- Input only: general tab ---
    // "general": [
    //   COMPONENT_OPTIONS (multiple-choice only),
    //   INTEGRATION_FIELD_MAPPING,
    //   INPUT_VALIDATION
    // ]
  },

  // --- i18n paths for inline-editable content ---
  "i18nPropPaths": [
    // "content,{elementName}" for each inline-editable element
    // "control,options,[id],label" for multiple-choice inputs
  ],

  // --- i18n property pane for translation editing ---
  // "i18nPropertyPaneModel": { "content": [...] },

  // --- Props ---
  "props": {
    "content": {
      // One entry per inline-editable element:
      // "{elementName}": "Default text"
    },

    // Environment-INDEPENDENT styles (same on desktop & mobile)
    // Based on Q6: properties user said stay the same
    // ONLY include "styles" if there are actual env-independent style entries. Do NOT add empty "styles": [].
    // "styles": [
    //   // Example for a button element with background, borders, border-radius, box-shadow, hover:
    //   {
    //     "element": "button",
    //     "styleAttributes": {
    //       "background": "rgb(0, 0, 0)",
    //       "borderTopStyle": "solid", "borderRightStyle": "solid",
    //       "borderBottomStyle": "solid", "borderLeftStyle": "solid",
    //       "borderTopWidth": "0px", "borderTopColor": "rgb(0, 0, 0)",
    //       "borderBottomWidth": "0px", "borderBottomColor": "rgb(0, 0, 0)",
    //       "borderLeftWidth": "0px", "borderLeftColor": "rgb(0, 0, 0)",
    //       "borderRightWidth": "0px", "borderRightColor": "rgb(0, 0, 0)",
    //       "borderTopLeftRadius": "0px", "borderTopRightRadius": "0px",
    //       "borderBottomLeftRadius": "0px", "borderBottomRightRadius": "0px",
    //       "boxShadow": "none"
    //     },
    //     "hoverStyleAttributes": {
    //       "background": "rgba(0, 0, 0, 0.2)"
    //     }
    //   }
    // ],

    // Environment-DEPENDENT styles (can differ per device)
    // Based on Q6: properties user said should differ
    // ONLY include "adaptiveStyles" if there are actual adaptive style entries. Do NOT add empty "adaptiveStyles": {}.
    // If desktop and mobile values are IDENTICAL, use cloneStylesToAllEnvs() instead of duplicating:
    //   "adaptiveStyles": cloneStylesToAllEnvs([ ...elements... ]),
    // If desktop and mobile values DIFFER, use separate objects:
    //   "adaptiveStyles": {
    //     "desktop": [ ...elements... ],
    //     "mobile": [ ...elements... ]
    //   },
    // "adaptiveStyles": {
    //   "desktop": [
    //     // Host element (always present):
    //     // {
    //     //   "element": "host",
    //     //   "styleAttributes": {
    //     //     "width": "100%", "minWidth": null,
    //     //     "height": "auto", "minHeight": null,
    //     //     "marginTop": "0px", "marginBottom": "0px",
    //     //     "marginLeft": "0px", "marginRight": "0px",
    //     //     "_marginEnabled": false
    //     //   }
    //     // },
    //     // Text element example:
    //     // {
    //     //   "element": "text",
    //     //   "styleAttributes": {
    //     //     "color": "rgb(50, 66, 67)", "textAlign": "start",
    //     //     "lineHeight": "120%", "fontWeight": "500",
    //     //     "fontSize": "16px", "textShadow": "none"
    //     //   },
    //     //   "classes": ""
    //     // }
    //   ],
    //   "mobile": [
    //     // Same structure, values can differ
    //   ]
    // },

    // --- Input only: control config ---
    // "control": {
    //   "name": "{camelCaseName}",
    //   "integrationName": "{camelCaseName}",
    //   "defaultValue": null,
    //   "validation": { "required": true },
    //   // Multiple-choice only:
    //   // "multipleChoice": false,
    //   // "options": { "option_1": { "exportId": "option_1", "label": "Option 1", "id": "option_1", "sort": 0 }, ... },
    //   // "optionsAlphabeticSort": { "enabled": false }
    // }
  },

  "metaDescription": {
    "icon": "/{Name}Component/assets/img/component-icon.svg",
    "label": {
      "en": "{Auto Label}"
    }
  },

  // --- Input only ---
  // "mappingTypes": [...],  // Read ManifestMappingType enum from node_modules/@claspo/common/component-manifest/ComponentManifest.interface.d.ts
  // "focusableElements": ["label", "button"],  // Elements highlightable in the editor. Should match the elements listed in TEXT_PARAMS control.
  // "showIntegrationFieldMappingPropertyPaneOnInsert": true,
  // "showComponentOptionsPropertyPaneOnInsert": true,  // multiple-choice only
  // "i18n": { "en": { "control,validation,validationErrors,REQUIRED": "Required field" } }
}
```

---

### componentType Values

Read the `ClComponentType` enum from `node_modules/@claspo/common/document/Document.interface.d.ts` for all available component types. Choose the most appropriate type for the component's purpose.

Common choices:
- **Presentation**: `TEXT`, `BUTTON`, `IMAGE`, `VIDEO`, `SLIDER`, `CONTAINER`, `SOCIAL`, etc.
- **Input**: `INPUT` (single-value), `MULTIPLE_INPUT` (multiple-choice)

The enum defines all valid values — always consult it rather than guessing.

---

### Input Component: General Tab Controls

For input components, add a `general` array to `propertyPaneModel`. Use the `ClPropertyPaneManifestModelName` enum for control names and the corresponding params interfaces (`OptionsPropertyPaneManifestParamsI`, `IntegrationFieldMappingPropertyPaneManifestParamsI`, `ValidationPropertyPaneManifestParamsI`) from `PropertyPaneManifest.interface.d.ts`:

```javascript
"general": [
  // Multiple-choice only: options editor
  {
    "type": "CONTROL",
    "name": "COMPONENT_OPTIONS",
    "params": {
      "label": "DOCUMENT_EXPORT_ID",
      "header": "DOCUMENT_OPTIONS_HEADER",
      "tooltip": "DOCUMENT_OPTIONS_TOOLTIP_PART1,DOCUMENT_OPTIONS_TOOLTIP_PART2",
      "origin": true,
      "optionsPropPath": ["control", "options"],
      "optionsAlphabeticSortPropPath": ["control", "optionsAlphabeticSort"],
      "integrationNamePropPath": ["control", "integrationName"]
    }
  },
  // Field mapping (always for input)
  {
    "type": "CONTROL",
    "name": "INTEGRATION_FIELD_MAPPING",
    "params": {
      "integrationNamePropPath": ["control", "integrationName"],
      "groupNamePropPath": ["control", "groupName"],
      "fieldNamePropPath": ["control", "fieldName"],
      "fieldTypePropPath": ["control", "fieldType"],
      "validationPropPath": ["control", "validation"],
      "placeholderPropPath": ["content", "placeholder"],
      "labelPropPath": ["content", "label"],
      "optionsPropPath": ["control", "options"],
      "optionsAlphabeticSortPropPath": ["control", "optionsAlphabeticSort"]
    }
  },
  // Validation (always for input)
  {
    "type": "CONTROL",
    "name": "INPUT_VALIDATION",
    "params": {
      "validationPropPath": ["control", "validation"],
      "fieldTypePropPath": ["control", "fieldType"],
      "required": true
    }
  }
]
```

### Input Component: i18n Translations

For input components, add `i18n` conforming to `ComponentManifestTranslationI` from `ComponentManifest.interface.d.ts` — a `{ [languageCode: string]: { [propPath: string]: string } }` map. Include at minimum the `"en"` locale:

```javascript
"i18n": {
  "en": {
    "control,validation,validationErrors,REQUIRED": "Required field",
    "content,label": "Title"
  }
}
```

For full 30-locale translations, copy from the `i18n` section of `SysChoiceButtons.manifest.js` or `SysInput.manifest.js` in the reference repository (`https://github.com/Claspo/claspo-components-public`) and adapt the `content,label` values.

### Input Component: i18nPropertyPaneModel

The `i18nPropertyPaneModel` follows the same `PropertyPaneManifestI` interface as `propertyPaneModel`. For multiple-choice inputs:
```javascript
"i18nPropertyPaneModel": {
  "content": [
    {
      "type": "CONTROL",
      "name": "COMPONENT_OPTIONS",
      "params": {
        "label": "DOCUMENT_EXPORT_ID",
        "header": "DOCUMENT_OPTIONS_HEADER",
        "tooltip": "DOCUMENT_OPTIONS_TOOLTIP_PART1,DOCUMENT_OPTIONS_TOOLTIP_PART2",
        "origin": false,
        "optionsPropPath": ["control", "options"],
        "optionsAlphabeticSortPropPath": ["control", "optionsAlphabeticSort"],
        "integrationNamePropPath": ["control", "integrationName"]
      }
    }
  ]
}
```

---

## Restrictions

### Manifest fields to NEVER include

These are internal/system-managed fields defined in `ComponentManifestI` (see `ComponentManifest.interface.d.ts`). Never add them to generated manifests:

- `children`
- `focusParentOnClick`
- `preventDraggable`
- `recursiveRemove`
- `actionsConditions`
- `isExternalStartCapable`
- `waitForResourcesLoad`
- `resourcesPropPaths`
- `events`
- `syncEnabled`
- `stylesImitationEnabled`

### Sync-related control fields to NEVER use

These fields are defined on `BasePropertyPaneManifestModelI` / `BaseFloatingControlManifestModelI` / `BaseContextMenuManifestModelI` but only make sense when `syncEnabled: true` is on the manifest. Since we never add `syncEnabled`, these must NEVER appear on any control:

- `hideSyncSelect`
- `syncSelectDisplayCondition`
- `syncSelectOptions`

### Controls are conditional on Q5

Only include editor controls (floating controls, property pane) for style properties the user selected in Q5:
- No "size" selected = no SIZE control in floatingControlsModel or propertyPaneModel
- No "margins" selected = no MARGIN control in floatingControlsModel, no INDENTATION (MARGIN) in propertyPaneModel
- No "padding" selected = no CONTAINER_PADDING control, no INDENTATION (PADDING)
- etc.

### Code conventions

- Write normal class methods. The build system's Babel plugin automatically transforms class methods to arrow functions. Excluded from transform: `constructor`, `connectedCallback`, `disconnectedCallback`, static methods, getters/setters.
- For all available component methods and properties, consult `node_modules/@claspo/renderer/sdk/WcElement.d.ts` (and `WcControlledElement.d.ts` for inputs). Key methods:
  - `this.getRootElement()` — access the ShadowRoot
  - `this.getElement('{name}')` — access elements marked with `cl-element`
  - `this.getHostElement()` — access the host element
  - `this.getProps()` — get current props
  - `this.getShared()` — get shared document settings
  - `this.getEnvironment()` — get current platform ('desktop' | 'mobile')
  - `this.isStaticRenderMode()` — check if rendering in static mode
  - `this.observeProps(cb)` — subscribe to prop changes
  - `this.observeShared(cb)` — subscribe to shared settings changes
  - `this.observeEnvironment(cb)` — subscribe to env changes
  - `this.applyAutoAdaptiveStyles(adaptiveStyles, styles)` — apply styles to elements
  - `this.getTranslationsMap(translationsMapByLanguage)` — get localized translations
  - `this.assets(path)` — resolve asset URLs
  - `this.services` — platform services (see `ServicesI` in WcElement.d.ts): `eventEmitter`, `form`, `context`, `config`, `mergeTagsProcessorFactory`, `actionFactory`, `actionRegister`
- Always call `this.applyAutoAdaptiveStyles(next.adaptiveStyles, next.styles)` inside `observeProps`.
- For SDK utilities, consult:
  - `ManifestUtils.d.ts` — `cloneControlsToAllEnvs()` for contextMenuModel, floatingControlsModel, propertyPaneModel.content
  - `ModelStyleUtils.d.ts` — `cloneStylesToAllEnvs()` for adaptiveStyles (only when desktop/mobile identical), `replaceStyleAttributes()`, `patchStyleAttributes()`
  - `HtmlStyleUtils.d.ts` — `setStylesToElement()`, `applyInputLabelStyles()`, `setInputHostSize()`, `setFocusOutline()`
  - `FormUtils.d.ts` — `setValidStyles()`, `setInvalidStyle()`, `setPendingStyle()`
- **Property pane control labels must be human-readable strings** (e.g., `"Track Color"`, `"Knob Size"`, `"Min Value"`). Do NOT use `DOCUMENT_*` keys (like `DOCUMENT_TRACK_COLOR`) — those are internal editor translation keys used only by built-in system controls (e.g., `COMPONENT_OPTIONS`, `INTEGRATION_FIELD_MAPPING`). Plugin components' custom labels are displayed as-is and are not resolved through the editor's i18n system.
- For `metaDescription.label`, generate `"en"` only. Other locales can be added later.

---

## Step 4: Auto-Registration

After generating all component files, also perform these registration steps:

1. Find `claspo.config.js` in the project root. Add the component name (e.g., `"MyTextComponent"`) to the `useComponents` array.

2. Find the editor's available components panel config (typically at `quickstart/frontend/src/config/components-panel.ts`). Add `{ componentName: '{Name}Component' }` to an appropriate group, or create a new group:
   ```javascript
   {
     label: 'My Components',
     components: [
       { componentName: '{Name}Component' }
     ]
   }
   ```

3. If either file is not found, print the registration instructions for the user to do manually.

4. Tell the user to restart the development server (`npm run dev`).
