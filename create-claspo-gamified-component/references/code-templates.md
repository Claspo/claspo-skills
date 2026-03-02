# Code Templates

## File Structure

```
{componentsDir}/{Name}Component/
  {Name}Component.js              # Main component class
  {Name}.manifest.js               # Manifest configuration
  getStyleElement.js               # CSS styles function
  assets/img/
    component-icon.svg             # Editor icon (20x20, #5F5F5F)
  # Optional based on complexity:
  defaultPrizeStyles.js            # Prize text styling defaults (if displays prize label)
  {Name}Utils.js                   # Utility functions (for complex games)
  {Name}Config.js                  # Configuration constants (for complex games)
  assets/themes/{name}/1.svg ...   # Theme images with distinct colors per item (if themes)
```

## Default Component Icon

Create `assets/img/component-icon.svg`:

```svg
<svg width="20" height="20" viewBox="0 0 20 20" fill="none" xmlns="http://www.w3.org/2000/svg">
<rect x="3" y="3" width="14" height="14" rx="2" stroke="#5F5F5F" stroke-width="2"/>
<path d="M10 7V13M7 10H13" stroke="#5F5F5F" stroke-width="1.5" stroke-linecap="round"/>
</svg>
```

## Component JS Template

Refer to `WcElement.d.ts` for all available methods and `BaseGamifiedComponent.js` for inherited methods.

```javascript
import {Name}Manifest from "./{Name}.manifest";
import getStyleElement from './getStyleElement';
import debounce from '@claspo/common/async/debounce';
import BaseGamifiedComponent from "../BaseGamifiedComponent";
import { PreviewMode } from '@claspo/renderer/sdk/PreviewMode';
import insertHtmlIntoElement from '@claspo/common/dom/insertHtmlIntoElement';
// If component displays prize label:
// import defaultPrizeStyles from './defaultPrizeStyles';
// import { resizeElementTextToFitContainer } from '@claspo/renderer/sdk/HtmlStyleUtils';

export default class {Name}Component extends BaseGamifiedComponent {

  static define = {
    name: '{tag-name}',
    model: {Name}Manifest.name,
    manifest: {Name}Manifest,
  };

  constructor() {
    super();
    this.containerClassName = '{kebab-name}-container';
    this.demoTimeouts = [];
    this.gameCompleted = false;

    this.debouncedRender = debounce((props) => this.render(props), 50);
    this.readyToPlayAnimationCb = debounce(this.runReadyToPlayAnimation, 1_500);

    this.getRootElement().innerHTML = `
      ${getStyleElement(this.containerClassName)}
      <div class="${this.containerClassName}"></div>
    `;
  }

  connectedCallback() {
    super.connectedCallback();

    // --- For event-triggered games (START_GAME pattern) only: ---
    // this.services.eventEmitter.emit('emitActionOnSubmit', {
    //   componentId: this.getModel().id,
    //   actionName: 'START_GAME',
    // });
    // this.services.eventEmitter.on('START_GAME', () => {
    //   this.startGame();
    // });

    this.connectToPrizePool(() => {
      this.observeProps((prev, next) => {
        this.applyAutoAdaptiveStyles(next.adaptiveStyles);
        this.debouncedRender(next);
      });

      this.observeEnvironment(() => {
        this.debouncedRender(this.getProps());
      });

      this.observeShared(() => {
        this.debouncedRender(this.getProps());
      });
    }, () => {
      this.handlePrizeUpdate();
    });

    this.connectToForm(() => {
      this.readyToPlay = true;
      this.readyToPlayAnimationCb();
    });
  }

  disconnectedCallback() {
    super.disconnectedCallback();
    this.demoTimeouts.forEach(clearTimeout);
    this.demoTimeouts = [];
    this.resizeObserver?.disconnect();
    this.gameCompleted = false;
  }

  render(props) {
    const container = this.getRootElement().querySelector(`.${this.containerClassName}`);
    const demoMode = this.services.config.getConfig(PreviewMode.DEMO);

    // Build game UI from props.content
    // ...

    // If view switcher exists and reward view is active:
    // if (this.isStaticRewardShouldBeShown()) {
    //   this.renderStaticReward();
    // }
  }

  handleItemInteraction(event) {
    const demoMode = this.services.config.getConfig(PreviewMode.DEMO);

    if (!demoMode) {
      const isValid = this.services.form.isValid();
      if (!isValid) {
        this.services.form.markAsTouched();
        return;
      }
    }

    // ... game interaction logic ...

    // On game completion (non-demo):
    if (!demoMode) {
      this.savePrizeOptionToStorage();
      this.setGameDataToContext();

      setTimeout(() => {
        this.completeSubmitAction();
        this.handlers?.forEach(h => h.release());
      }, /* animation delay ms */);
    }
  }

  handlePrizeUpdate() {
    // Re-render prize display if visible (for view-switcher components)
  }
}
```

### connectedCallback — Key Rules

Observers (`observeProps`, `observeEnvironment`, `observeShared`) MUST be inside `connectToPrizePool` prizeLoadedCallback.

Order:
1. `super.connectedCallback()` — always first
2. Optional: event listener setup (START_GAME for event-triggered games)
3. `this.connectToPrizePool(prizeLoadedCb, prizeUpdatedCb)` — prize system
4. `this.connectToForm(allControlsValidCb)` — form validation

### Game Completion — Key Rules

Always in this order:
1. `this.savePrizeOptionToStorage()` — persist prize selection
2. `this.setGameDataToContext()` — sync prize to context records
3. `this.completeSubmitAction()` — trigger form submission flow (after animation delay)

Skip all three in demo mode. In demo mode, loop the game instead.

---

## getStyleElement.js Template

```javascript
export default function getStyleElement(containerClassName) {
  return `
<style>
  .${containerClassName} {
    display: flex;
    align-items: center;
    justify-content: center;
    width: 100%;
    position: relative;
  }

  /* Game-specific styles here */

  /* --- Hover + ready-to-play animations (see animation-patterns.md) --- */

  @keyframes fadeIn {
    0% { opacity: 0; }
    100% { opacity: 1; }
  }

  @keyframes itemHover {
    0% { transform: scale(1); }
    25% { transform: scale(0.92, 1.08); }
    50% { transform: scale(1.08, 0.92); }
    75% { transform: scale(0.96, 1.04); }
    100% { transform: scale(1); }
  }

  @keyframes itemPop {
    0% { transform: scale(1); }
    50% { transform: scale(1.2); }
    100% { transform: scale(1); }
  }

  @keyframes itemJelly {
    0%, 85%, 100% { transform: scale(1) rotate(0deg); }
    88% { transform: scale(1.03) rotate(1deg); }
    91% { transform: scale(0.97) rotate(-1deg); }
    94% { transform: scale(1.02) rotate(0.5deg); }
    97% { transform: scale(1) rotate(0deg); }
  }

  @keyframes shake {
    0% { opacity: 0; transform: scale(1); }
    5% { transform: scale(1.05) rotate(5deg); }
    20% { transform: scale(1.1) rotate(-5deg); }
    55% { opacity: 0.6; transform: scale(1.15) rotate(0deg); }
    100% { opacity: 1; transform: scale(1); }
  }
</style>
  `;
}
```

The function receives class name constants from the constructor. Can receive additional parameters for dynamic CSS values.

---

## defaultPrizeStyles.js Template

Only needed for components that display prize label/text (like GiftBox, TreasureHunt, ScratchCard). Not for components where prize display is handled by a separate view (like MemoryCards, SlotMachine).

```javascript
const defaultPrizeStyles = {
  color: 'var(--cl-text-accent)',
  fontWeight: '700',
  background: '#fff',
  textAlign: 'center',
  letterSpacing: '0px',
  textShadow: 'none',
  textTransform: 'none',
};

export default defaultPrizeStyles;
```

Referenced in manifest as `"styles": defaultPrizeStyles` inside `props.content.prize`.
