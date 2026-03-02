# Platform API Reference

## Table of Contents

- [Platform Interfaces](#platform-interfaces)
- [Documentation Guides](#documentation-guides)
- [Existing Component Examples](#existing-component-examples)
- [SDK Utilities](#sdk-utilities)
- [Key WcElement Methods](#key-wcelement-methods)

---

## Platform Interfaces

Read these FIRST before generating code to understand available capabilities.

**Manifest structure and enums:**
- `node_modules/@claspo/common/component-manifest/ComponentManifest.interface.d.ts` — `ComponentManifestI`, `ManifestMappingType` enum, `AutoContrastI`
- `node_modules/@claspo/common/component-manifest/PropertyPaneManifest.interface.d.ts` — `PropertyPaneManifestI`, `ClPropertyPaneManifestModelName` enum (all 30+ property pane control types and params interfaces)
- `node_modules/@claspo/common/component-manifest/FloatingControlManifest.interface.d.ts` — `FloatingControlManifestModelName` enum (SIZE, MARGIN, CONTAINER_PADDING, ROTATION, etc.)
- `node_modules/@claspo/common/component-manifest/ContextMenuManifest.interface.d.ts` — `ContextMenuManifestModelName` enum (COMPONENT_OPERATIONS, BRING_BACK_FORWARD, FOCUS_PARENT_COMPONENT)

**Document model and component types:**
- `node_modules/@claspo/common/document/Document.interface.d.ts` — `ClComponentType` enum, `ClBaseComponentPropsI`, `ClBaseComponentElementParamsI`, `BaseComponentAdaptiveStylesI`

**Base component classes:**
- `node_modules/@claspo/renderer/sdk/WcElement.d.ts` — `WcElement` class, `ServicesI` interface
- `node_modules/@claspo/renderer/sdk/WcControlledElement.d.ts` — `WcControlledElement` class, `CreateControlConfigI`, `ValidatorFnT`

**Form system:**
- `node_modules/@claspo/renderer/form/FormControl.d.ts` — `FormControl` class (setValue, getValue, validate, etc.)
- `node_modules/@claspo/renderer/form/FormControlValidator.interface.d.ts` — validator types

## Documentation Guides

Start with the docs index: `https://docs.claspo.io/llms.txt`

Key guides:
- `https://docs.claspo.io/docs/getting-started-components.md` — MyText component guide (presentation walkthrough)
- `https://docs.claspo.io/docs/clasporenderer.md` — `@claspo/renderer` SDK reference
- `https://docs.claspo.io/docs/property-pane-components.md` — property pane controls reference
- `https://docs.claspo.io/docs/custom-icons-for-components.md` — SVG icon specifications
- `https://docs.claspo.io/docs/validation.md` — custom validation rules for form components

## Existing Component Examples

Reference repository: `https://github.com/Claspo/claspo-components-public`

**Presentation components:**
- `src/SysTextComponent` — simple presentation (class + manifest)
- `src/SysButtonComponent` — complex presentation with button element

**Input components:**
- `src/SysChoiceButtonsComponent` — input with options (class, manifest, componentStyle, componentTemplate)
- `src/SysInputComponent` — single-value input

## SDK Utilities

- `ManifestUtils.d.ts` — `cloneControlsToAllEnvs()` for contextMenuModel, floatingControlsModel, propertyPaneModel.content
- `ModelStyleUtils.d.ts` — `cloneStylesToAllEnvs()` for adaptiveStyles (only when desktop/mobile identical), `replaceStyleAttributes()`, `patchStyleAttributes()`
- `HtmlStyleUtils.d.ts` — `setStylesToElement()`, `applyInputLabelStyles()`, `setInputHostSize()`, `setFocusOutline()`
- `FormUtils.d.ts` — `setValidStyles()`, `setInvalidStyle()`, `setPendingStyle()`
- `ColorUtils.d.ts` — color manipulation utilities

## Key WcElement Methods

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
- `this.services` — platform services (`eventEmitter`, `form`, `context`, `config`, `mergeTagsProcessorFactory`, `actionFactory`, `actionRegister`)
