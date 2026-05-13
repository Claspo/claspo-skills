# Accessibility Guide

Mandatory accessibility conventions for all Claspo components (regular and gamified). These rules apply at code-generation time and are validated during edits.

## Why this matters

Claspo widgets render on third-party websites and reach users with screen readers, keyboard-only navigation, and reduced-motion preferences. Source components in `claspo-components` and `claspo-widgets-plugin` have an established a11y baseline (recent commits May 7–12, 2026). Newly scaffolded components must match it or they regress the codebase.

Public mirror of canonical patterns: <https://github.com/Claspo/claspo-components-public>.

---

## Mandatory rules

Every generated component MUST satisfy these:

- **Role** — every interactive element gets an accessible role. Use the closest match:
  - `role="button"` — custom clickable container (gamified items, picture cards)
  - `role="radio"` + parent `role="radiogroup"` — single-choice option group
  - `role="combobox"` + `aria-haspopup="listbox"` — dropdown trigger
  - `role="listbox"` + child `role="option"` — overlay menu
  - `role="group"` — non-form grouped controls (e.g., choice buttons)
- **Native first** — prefer `<button>`, `<input type="radio">`, `<input type="checkbox">` over `<div role="...">`+`tabindex` retrofit. Retrofit only when the visual design forbids a native control (e.g., a 3D gift box).
- **Keyboard** — custom interactive `<div>` MUST have `tabindex="0"` and a keydown handler that triggers click on `Enter` and `Space`:
  ```js
  optionKeydownEventListener(event) {
    if (event.key === 'Enter' || event.key === ' ' || event.key === 'Spacebar') {
      event.preventDefault();
      event.currentTarget.click();
    }
  }
  ```
- **Labels** — every interactive element MUST have an `aria-label` (or `aria-labelledby`). Labels MUST be **localized via an i18n translation map** keyed by language code — never hardcode English. See `selectFeedbackTranslations.js`, `selectScoreTranslations.js` for the pattern.
- **State** — these attributes MUST update reactively as state changes: `aria-checked`, `aria-selected`, `aria-expanded`, `aria-activedescendant`.
- **Decoration** — purely decorative SVG/icons/flags get `aria-hidden="true"`.
- **Focus ring** — `:focus-visible { outline: 2px solid currentColor; outline-offset: 2px; }`. Never bare `outline: none`.
- **Focus restoration** — when an overlay/menu/picker closes, call `triggerElement.focus()` to return focus to the element that opened it.
- **Escape** — overlays/menus MUST close on `Escape` and restore focus.
- **Reduced motion** — animations (spin, scratch, jelly, idle, reveal) MUST respect `@media (prefers-reduced-motion: reduce)`. Provide an instant or short-fade fallback. JS path: `window.matchMedia('(prefers-reduced-motion: reduce)').matches`.

---

## Anti-patterns

Do NOT do any of these (each comes from a real bug fix in the source repos):

- `aria-required` or `aria-multiselectable` on a container `<div>` — those belong only on form controls. (Commit `45f5d7d` removed these from a `ChoiceButtons` container.)
- Moving DOM focus into a listbox to indicate the active option — use `aria-activedescendant` on the trigger/input instead.
- Bare `outline: none` with no `:focus-visible` replacement — keyboard users lose all focus indication.
- Hardcoded English `aria-label="Select option"` — breaks for non-English visitors. Use the i18n translation-map pattern.
- Adding `tabindex` to a native `<button>` or `<input>` — they are already focusable. Adding `tabindex="0"` is redundant; adding `tabindex` > 0 breaks tab order.
- Reading `aria-checked`/`aria-selected` once at render and never updating — these attributes are stateful and must reflect current selection.

---

## Canonical patterns

### Combobox + listbox

Model: `src/SysDropdownInputComponent/`.

```html
<!-- Trigger -->
<div role="combobox"
     aria-haspopup="listbox"
     aria-expanded="false"
     aria-activedescendant=""
     aria-labelledby="cl-myInput-label"
     tabindex="0">
  ...
</div>

<!-- Overlay -->
<div role="listbox">
  <div role="option"
       id="cl-myInput-option-1"
       aria-selected="false"
       tabindex="-1">Option 1</div>
</div>
```

When the user navigates with arrow keys, update `aria-activedescendant` on the combobox to point at the focused option's `id` — do NOT call `.focus()` on the option.

### Radio group

Model: `src/SysRadioGroupComponent/`, plugin `components/SysFeedbackComponent/`, plugin `components/SysNetPromoterScoreComponent/`.

```html
<div role="radiogroup" aria-labelledby="cl-rating-label">
  <div role="radio"
       aria-checked="false"
       aria-label="{i18n-label-for-option-1}"
       tabindex="0">★</div>
  <div role="radio"
       aria-checked="true"
       aria-label="{i18n-label-for-option-2}"
       tabindex="0">★★</div>
</div>
```

On selection, update `aria-checked` on all options in the group.

### Custom button (gamified items)

Model: plugin `components/SysGiftBoxComponent/`, `components/SysSlotMachineComponent/`, `components/SysWheelOfFortuneComponent/`.

```html
<div class="cl-game-item"
     role="button"
     tabindex="0"
     aria-label="{i18n-label}">
  ...
</div>
```

```js
this.gameItemElement.addEventListener('keydown', this.gameItemKeydownEventListener);

gameItemKeydownEventListener(event) {
  if (event.key === 'Enter' || event.key === ' ' || event.key === 'Spacebar') {
    event.preventDefault();
    event.currentTarget.click();
  }
}
```

### i18n aria-label

Model: `selectFeedbackTranslations.js`, `selectScoreTranslations.js`, `selectCountryCodeTranslations.js`, `selectOptionTranslations.js`.

```js
// translations.js
export const selectItemTranslations = {
  en: 'Select item',
  ru: 'Выбрать элемент',
  uk: 'Вибрати елемент',
  // ... 30+ more
};

// component
import { selectItemTranslations } from './selectItemTranslations.js';

const label = selectItemTranslations[props.lang] || selectItemTranslations.en;
element.setAttribute('aria-label', label);
```

### Reduced motion fallback

Model: plugin `components/SysEasterEggHuntComponent/`.

```css
.cl-game-item { transition: transform 600ms ease; }

@media (prefers-reduced-motion: reduce) {
  .cl-game-item { transition: none; }
}
```

```js
const reduced = window.matchMedia('(prefers-reduced-motion: reduce)').matches;
const duration = reduced ? 0 : 600;
```

### Focus restoration

```js
openOverlay() {
  this.triggerElement.setAttribute('aria-expanded', 'true');
  this.overlayElement.style.display = '';
}

closeOverlay() {
  this.overlayElement.style.display = 'none';
  this.triggerElement.setAttribute('aria-expanded', 'false');
  this.triggerElement.focus();
}

handleEscape(event) {
  if (event.key === 'Escape') this.closeOverlay();
}
```

---

## Where to look in the source repos

Live, copy-from references for each pattern (paths relative to repo root):

- **Combobox/listbox** — `claspo-components/src/SysDropdownInputComponent/`, `claspo-components/src/SysPhoneInputComponent/PhoneInputMenu.js`
- **Radio group** — `claspo-components/src/SysRadioGroupComponent/`, `claspo-widgets-plugin/components/SysFeedbackComponent/`, `claspo-widgets-plugin/components/SysNetPromoterScoreComponent/`
- **Choice buttons (group)** — `claspo-components/src/SysChoiceButtonsComponent/`
- **Custom-button gamified items** — `claspo-widgets-plugin/components/SysGiftBoxComponent/`, `components/SysSlotMachineComponent/`, `components/SysWheelOfFortuneComponent/`
- **Reduced motion** — `claspo-widgets-plugin/components/SysEasterEggHuntComponent/`
- **i18n label maps** — `selectFeedbackTranslations.js`, `selectScoreTranslations.js`, `selectCountryCodeTranslations.js`, `selectOptionTranslations.js`
- **Date picker keyboard + focus restoration** — `claspo-components/src/SysDateComponent/`
