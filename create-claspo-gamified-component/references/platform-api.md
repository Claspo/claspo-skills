# Platform API Reference

## Platform Interfaces (read FIRST)

**Manifest structure and enums:**
- `node_modules/@claspo/common/component-manifest/ComponentManifest.interface.d.ts` — `ComponentManifestI`, `ManifestMappingType` enum
- `node_modules/@claspo/common/component-manifest/PropertyPaneManifest.interface.d.ts` — `PropertyPaneManifestI`, `ClPropertyPaneManifestModelName` enum
- `node_modules/@claspo/common/component-manifest/FloatingControlManifest.interface.d.ts` — `FloatingControlManifestModelName` enum
- `node_modules/@claspo/common/component-manifest/ContextMenuManifest.interface.d.ts` — `ContextMenuManifestModelName` enum

**Document model and component types:**
- `node_modules/@claspo/common/document/Document.interface.d.ts` — `ClComponentType` enum, `ClBaseComponentPropsI`, `ClBaseComponentElementParamsI`, `BaseComponentAdaptiveStylesI`

**Base component classes:**
- `node_modules/@claspo/renderer/sdk/WcElement.d.ts` — `WcElement` class, `ServicesI` interface

**SDK utilities:**
- `node_modules/@claspo/renderer/sdk/ManifestUtils.d.ts` — `cloneControlsToAllEnvs()`
- `node_modules/@claspo/renderer/sdk/ModelStyleUtils.d.ts` — `cloneStylesToAllEnvs()`
- `node_modules/@claspo/renderer/sdk/HtmlStyleUtils.d.ts` — `setStylesToElement()`, `resizeElementTextToFitContainer()`

## BaseGamifiedComponent

Read `components/BaseGamifiedComponent.js` for inherited behavior:

- `connectToPrizePool(prizeLoadedCb, prizeUpdatedCb)` — prize pool connection with two callbacks
- `connectToForm(allControlsValidCb, emptyFormDelayMs)` — form validation monitoring
- `setLastPrizeToContext()` / `setGameDataToContext()` — context record management
- `savePrizeOptionToStorage()` — persists selected prize
- `getPrizeOption()` — returns current prize data
- `deleteDataFromContext()` — cleanup on disconnect
- `this.prize` — current prize object (set by connectToPrizePool)
- `this.prizePool` — prize pool instance
- `this.prizePool.getModel().options` — array of all prize options
- `this.prizePool.mapPrizeOption(option)` — map raw option to `{ id, label, value }`
- `this.readyToPlay` — flag set by connectToForm
- `this.isStaticEntryModule` — `true` in live widget, `false` in editor canvas

## Existing Gamified Components

Pick the most similar one as a code pattern:

| Pattern | Component | When to reference |
|---|---|---|
| Click-to-reveal items | `components/SysGiftBoxComponent/` | Simple click games with multiple items |
| Click grid with limited attempts | `components/SysTreasureHuntComponent/` | Grid games with limited tries, themed assets |
| Card matching | `components/SysMemoryCardsComponent/` | Multi-step matching/pairing games |
| Canvas scratch/draw | `components/SysScratchCardComponent/` | Touch/drag canvas interaction |
| Canvas spinning | `components/SysWheelOfFortuneComponent/` | Rotation/spinning animations |
| Multi-reel animation | `components/SysSlotMachineComponent/` | Reel/column animations, helper classes |
| Gesture + external lib | `components/SysShakeChristmasTreeComponent/` | Physics/gesture games, Lottie |

## Key WcElement Methods

- `this.getRootElement()` — access ShadowRoot
- `this.getElement('{name}')` — access elements marked with `cl-element`
- `this.getHostElement()` — access host element
- `this.getProps()` — get current props
- `this.getShared()` — get shared document settings
- `this.getEnvironment()` — get current platform (`'desktop'` | `'mobile'`)
- `this.isStaticRenderMode()` — check static mode
- `this.observeProps(cb)` — subscribe to prop changes
- `this.observeShared(cb)` — subscribe to shared settings changes
- `this.observeEnvironment(cb)` — subscribe to env changes
- `this.applyAutoAdaptiveStyles(adaptiveStyles)` — apply styles to elements
- `this.getTranslationsMap(translationsMapByLanguage)` — get localized translations
- `this.assets(path)` — resolve asset URLs
- `this.services` — platform services (`eventEmitter`, `form`, `context`, `config`, `mergeTagsProcessorFactory`, `actionFactory`, `actionRegister`)
- `this.services.config.getConfig(PreviewMode.DEMO)` — check demo mode
- `this.services.config.getConfig(PreviewMode.PREVIEW)` — check preview mode
- `this.services.form.isValid()` — check form validity
- `this.services.form.markAsTouched()` — show validation errors
