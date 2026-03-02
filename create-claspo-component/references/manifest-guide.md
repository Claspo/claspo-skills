# Manifest Guide

## Table of Contents

- [Manifest Template](#manifest-template)
- [componentType Values](#componenttype-values)
- [Property Pane Controls Reference](#property-pane-controls-reference)
- [General Tab Controls (Input only)](#general-tab-controls-input-only)
- [i18n Translations (Input only)](#i18n-translations-input-only)
- [i18nPropertyPaneModel (Multiple-choice Input)](#i18npropertypanemodel-multiple-choice-input)

---

## Manifest Template

Must conform to `ComponentManifestI` from `node_modules/@claspo/common/component-manifest/ComponentManifest.interface.d.ts`.

For control names and params interfaces, consult:
- **Property pane**: `ClPropertyPaneManifestModelName` enum in `PropertyPaneManifest.interface.d.ts`
- **Floating controls**: `FloatingControlManifestModelName` enum in `FloatingControlManifest.interface.d.ts`
- **Context menu**: `ContextMenuManifestModelName` enum in `ContextMenuManifest.interface.d.ts`
- **Props structure**: `ClBaseComponentPropsI`, `ClBaseComponentElementParamsI`, `BaseComponentAdaptiveStylesI` in `Document.interface.d.ts`

```javascript
import {cloneControlsToAllEnvs} from '@claspo/renderer/sdk/ManifestUtils';
// If adaptiveStyles are same for both envs:
// import {cloneStylesToAllEnvs} from '@claspo/renderer/sdk/ModelStyleUtils';

export default {
  "name": "{Name}Component",
  "componentType": "{TYPE}",   // See componentType values below
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
    // ONLY include "adaptiveStyles" if there are actual adaptive style entries. Do NOT add empty "adaptiveStyles": {}.
    // If desktop and mobile values are IDENTICAL, use cloneStylesToAllEnvs():
    //   "adaptiveStyles": cloneStylesToAllEnvs([ ...elements... ]),
    // If desktop and mobile values DIFFER, use separate objects:
    //   "adaptiveStyles": {
    //     "desktop": [
    //       {
    //         "element": "host",
    //         "styleAttributes": {
    //           "width": "100%", "minWidth": null,
    //           "height": "auto", "minHeight": null,
    //           "marginTop": "0px", "marginBottom": "0px",
    //           "marginLeft": "0px", "marginRight": "0px",
    //           "_marginEnabled": false
    //         }
    //       },
    //       {
    //         "element": "text",
    //         "styleAttributes": {
    //           "color": "rgb(50, 66, 67)", "textAlign": "start",
    //           "lineHeight": "120%", "fontWeight": "500",
    //           "fontSize": "16px", "textShadow": "none"
    //         },
    //         "classes": ""
    //       }
    //     ],
    //     "mobile": [
    //       // Same structure, values can differ
    //     ]
    //   },

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
  // "mappingTypes": [...],  // Read ManifestMappingType enum from ComponentManifest.interface.d.ts
  // "focusableElements": ["label", "button"],  // Elements highlightable in the editor. Match TEXT_PARAMS elements.
  // "showIntegrationFieldMappingPropertyPaneOnInsert": true,
  // "showComponentOptionsPropertyPaneOnInsert": true,  // multiple-choice only
  // "i18n": { "en": { "control,validation,validationErrors,REQUIRED": "Required field" } }
}
```

## componentType Values

Read the `ClComponentType` enum from `node_modules/@claspo/common/document/Document.interface.d.ts` for all available types.

Common choices:
- **Presentation**: `TEXT`, `BUTTON`, `IMAGE`, `VIDEO`, `SLIDER`, `CONTAINER`, `SOCIAL`, etc.
- **Input**: `INPUT` (single-value), `MULTIPLE_INPUT` (multiple-choice)

Always consult the enum rather than guessing.

## Property Pane Controls Reference

**Host-level controls:**
- `SIZE` (floating: `SIZE`) — width/height
- `INDENTATION` with `indentationType: "MARGIN"` (floating: `MARGIN`) — outer spacing
- `INDENTATION` with `indentationType: "PADDING"` (floating: `CONTAINER_PADDING`) — inner spacing

**Element-level controls** (via `propPath` or `element`):
- `TEXT_PARAMS` — font styling for text elements
- `BACKGROUND` — background color/image/gradient
- `BORDERS` — border style/width/color
- `BORDER_RADIUS` — corner rounding
- `BOX_SHADOW` — shadow effects
- `HOVER_ANIMATION` — hover effects (interactive elements only)
- `COLOR_SELECT` — simple color picker (solid color alternative to BACKGROUND)
- `NUMBER_INPUT` — numeric value. Use `params.validators: { min, max }` to constrain range. Use `valueTransformers: [{"name": "px"}]` for CSS dimension values only, omit for plain numbers.
- `TEXT_INPUT` — text/string value (placeholder text, custom labels). NOT for numeric values.
- `SWITCH` — boolean toggles

**Choosing the right control type:**
- Numeric values -> `NUMBER_INPUT`
- String/text values -> `TEXT_INPUT`
- Color values -> `COLOR_SELECT`
- Boolean toggles -> `SWITCH`

## General Tab Controls (Input only)

Add a `general` array to `propertyPaneModel`. Use `ClPropertyPaneManifestModelName` enum and corresponding `*PropertyPaneManifestParamsI` interfaces from `PropertyPaneManifest.interface.d.ts`:

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

## i18n Translations (Input only)

Add `i18n` conforming to `ComponentManifestTranslationI` — `{ [languageCode: string]: { [propPath: string]: string } }`. Minimum `"en"` locale:

```javascript
"i18n": {
  "en": {
    "control,validation,validationErrors,REQUIRED": "Required field",
    "content,label": "Title"
  }
}
```

For full 30-locale translations, copy from `SysChoiceButtons.manifest.js` or `SysInput.manifest.js` in `https://github.com/Claspo/claspo-components-public`.

## i18nPropertyPaneModel (Multiple-choice Input)

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
