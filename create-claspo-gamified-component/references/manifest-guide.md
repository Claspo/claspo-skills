# Manifest Guide

## componentType

All gamified components use `"PRIZE_BASED_GAMING"`. Confirm via `ClComponentType` enum in `Document.interface.d.ts`.

## Choosing the Right Control Type

- Numeric values → `NUMBER_INPUT` (no `valueTransformers` — plain numbers, not CSS). Use `params.validators: { min, max }`.
- String/text → `TEXT_INPUT`
- Color → `COLOR_SELECT`
- Enum/option → `SELECT`
- Icon/image → `ICON_SELECT`

## Manifest Template

```javascript
import { cloneControlsToAllEnvs } from '@claspo/renderer/sdk/ManifestUtils';
import { cloneStylesToAllEnvs } from '@claspo/renderer/sdk/ModelStyleUtils';
// If component displays prize label:
// import defaultPrizeStyles from './defaultPrizeStyles';

export default {
  "name": "{Name}Component",
  "componentType": "PRIZE_BASED_GAMING",
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

  // --- Floating controls ---
  "floatingControlsModel": cloneControlsToAllEnvs([
    // SIZE — always, height hidden (auto-height)
    {
      "type": "CONTROL",
      "name": "SIZE",
      "elementProp": "styleAttributes",
      "element": "host",
      "params": {
        "height": { "hide": true }
      }
    },
    // MARGIN — always
    {
      "type": "CONTROL",
      "name": "MARGIN",
      "elementProp": "styleAttributes",
      "element": "host"
    }
    // VIEW_SWITCHER — only if has view switcher (Q5)
    // {
    //   "type": "CONTROL",
    //   "name": "VIEW_SWITCHER",
    //   "propPath": ["content", "activeView"],
    //   "params": {
    //     "views": [
    //       { "label": "DOCUMENT_FLOATING_CONTROL_{NAME}_GAME_LAYER", "value": "game" },
    //       { "label": "DOCUMENT_FLOATING_CONTROL_{NAME}_REWARD_LAYER", "value": "reward" }
    //     ]
    //   }
    // }
  ]),

  // --- Property pane ---
  "propertyPaneModel": {
    "content": cloneControlsToAllEnvs([
      // SIZE — always, width only
      {
        "type": "CONTROL",
        "name": "SIZE",
        "element": "host",
        "elementProp": "styleAttributes",
        "params": {
          "width": {
            "options": ["fixed", "fill"]
          }
        }
      },

      // --- Game-specific controls from Q2/Q4 ---

      // SELECT example (grid size, theme):
      // {
      //   "type": "CONTROL",
      //   "name": "SELECT",
      //   "propPath": ["content", "game", "{optionName}"],
      //   "params": {
      //     "label": "Option Label",
      //     "options": [
      //       { "label": "Option 1", "value": "option_1" },
      //       { "label": "Option 2", "value": "option_2" }
      //     ]
      //   }
      // },

      // NUMBER_INPUT example (item count, gap, max attempts):
      // {
      //   "type": "CONTROL",
      //   "name": "NUMBER_INPUT",
      //   "propPath": ["content", "game", "{optionName}"],
      //   "params": {
      //     "label": "Option Label",
      //     "validators": { "min": 1, "max": 10 }
      //   }
      // },

      // ICON_SELECT example:
      // {
      //   "type": "CONTROL",
      //   "name": "ICON_SELECT",
      //   "propPath": ["content", "{gameKey}"],
      //   "params": {
      //     "label": "Image",
      //     "options": getPredefinedIcons().map((icon, index) => ({
      //       icon, inlineSvg: true, value: String(index),
      //     })),
      //     enableCustomIcons: true,
      //     storageType: '{ICON_STORAGE_TYPE}',
      //   }
      // },

      // COLOR_SELECT example:
      // {
      //   "type": "CONTROL",
      //   "name": "COLOR_SELECT",
      //   "propPath": ["content", "game", "{colorProp}"],
      //   "params": { "label": "Element Color" }
      // },

      // TEXT_PARAMS for prize label — only if displays prize text
      // Only in reward view (if has view switcher):
      // {
      //   "type": "CONTROL",
      //   "name": "TEXT_PARAMS",
      //   "propPath": ["content", "prize", "styles"],
      //   "params": [{ "fontSizePlaceholder": "Font Size", "hideStyleSelect": true }],
      //   "displayCondition": "return sdk.component.getProps().content.activeView === 'reward';"
      // },

      // BACKGROUND for prize label — only if displays prize text:
      // {
      //   "type": "CONTROL",
      //   "name": "BACKGROUND",
      //   "propPath": ["content", "prize", "styles"],
      //   "displayCondition": "return sdk.component.getProps().content.activeView === 'reward';"
      // },

      // INDENTATION (MARGIN) — always
      {
        "type": "CONTROL",
        "name": "INDENTATION",
        "elementProp": "styleAttributes",
        "element": "host",
        "params": {
          "indentationType": "MARGIN"
        }
      }
    ]),

    // Or when desktop/mobile need different controls, use GROUP pattern:
    // "content": [
    //   {
    //     "type": "GROUP",
    //     "propPath": ["adaptiveStyles", "desktop"],
    //     "children": [ /* desktop controls */ ]
    //   },
    //   {
    //     "type": "GROUP",
    //     "propPath": ["adaptiveStyles", "mobile"],
    //     "children": [ /* mobile controls */ ]
    //   }
    // ],

    // --- General tab: always include PRIZE_SETTINGS ---
    "general": [
      {
        "type": "CONTROL",
        "name": "PRIZE_SETTINGS",
        "propPath": ["content", "prize"]
        // Optional: "params": { "minOptions": 1 }
        // Optional: "params": { "prizeImageEnabled": true }
      }
    ]
  },

  // --- Props ---
  "props": {
    "adaptiveStyles": cloneStylesToAllEnvs([
      {
        "element": "host",
        "styleAttributes": {
          "position": "relative",
          "width": "300px",
          "minWidth": null,
          "height": "auto",
          "minHeight": null,
          "marginTop": "0px",
          "marginBottom": "0px",
          "marginLeft": "0px",
          "marginRight": "0px",
          "_marginEnabled": false
        },
        "classes": ""
      }
    ]),
    // Or separate desktop/mobile:
    // "adaptiveStyles": {
    //   "desktop": [{ "element": "host", "styleAttributes": { "width": "400px", ... }, "classes": "" }],
    //   "mobile": [{ "element": "host", "styleAttributes": { "width": "300px", ... }, "classes": "" }]
    // },

    "content": {
      "game": {
        // Game-specific config from Q2
      },
      "readyToPlayEventName": "CONTACT_ID_WAS_RECEIVED",
      "prize": {
        "sourceType": "MANUAL",
        "distributionType": "PERCENT_CHANCE",
        "options": [
          {
            "valueType": "DISCOUNT_PERCENTAGE",
            "value": "10",
            "code": "PROMO_10",
            "weight": 10
          }
        ]
        // If displays prize text: "styles": defaultPrizeStyles
      }
      // If has view switcher: "activeView": "game"
    }
  },

  "metaDescription": {
    "icon": "/{Name}Component/assets/img/component-icon.svg",
    "label": {
      "en": "{Auto Label}"
    }
  }
}
```
