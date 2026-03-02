# Code Templates

## Table of Contents

- [File Structure](#file-structure)
- [Default Component Icon](#default-component-icon)
- [Presentation Component JS](#presentation-component-js)
- [Input Component JS](#input-component-js)
- [Input componentTemplate.js](#input-componenttemplatejs)
- [Input componentStyle.js](#input-componentstylejs)

---

## File Structure

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

## Default Component Icon

Create `assets/img/component-icon.svg` (20x20, `#5F5F5F`):

```svg
<svg width="20" height="20" viewBox="0 0 20 20" fill="none" xmlns="http://www.w3.org/2000/svg">
<rect x="3" y="3" width="14" height="14" rx="2" stroke="#5F5F5F" stroke-width="2"/>
<path d="M10 7V13M7 10H13" stroke="#5F5F5F" stroke-width="1.5" stroke-linecap="round"/>
</svg>
```

## Presentation Component JS

Refer to `node_modules/@claspo/renderer/sdk/WcElement.d.ts` for all `WcElement` methods and `ServicesI` for `this.services.*`.

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

## Input Component JS

Refer to `node_modules/@claspo/renderer/sdk/WcControlledElement.d.ts` for `WcControlledElement` methods: `createControlWithValidation(validators, options)`, `getRegisteredControl()`. See `CreateControlConfigI`, `ControlValidationOptionsI`, `ValidatorFnT`.

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

## Input componentTemplate.js

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

## Input componentStyle.js

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
