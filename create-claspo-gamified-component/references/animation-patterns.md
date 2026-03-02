# Animation Patterns

All gamified component game items MUST include hover, idle jelly, and ready-to-play animations.

## Hover Animation (always include)

Every game item must have hover feedback. Items with `hover-ready` class (applied after ready-to-play) get a subtle idle animation that pauses on hover.

**CSS pattern:**

```css
/* Hover: wobble on each game item */
.{itemSelector}:hover .{wrapperElement} {
  animation: itemHover 0.5s ease 1;
}

/* Idle jelly after ready-to-play (4s cycle = ~3.4s pause between wiggles) */
.{itemSelector}.hover-ready .{wrapperElement} {
  animation: itemJelly 4s ease infinite;
}

/* Stop ALL idle animations when user hovers the container */
.{containerRow}:hover .{itemSelector}.hover-ready .{wrapperElement} {
  animation: none;
}

/* Only the hovered item gets hover animation */
.{containerRow}:hover .{itemSelector}.hover-ready:hover .{wrapperElement} {
  animation: itemHover 0.5s ease 1;
}
```

## Ready-to-Play Animation (always include)

When form validation passes (`connectToForm` callback), ALL game items play a sequential attention-grabbing pop animation, then get `hover-ready` class for idle jelly.

**Constructor:**
```javascript
this.readyToPlayAnimationCb = debounce(this.runReadyToPlayAnimation, 1_500);
```

**connectToForm callback:**
```javascript
this.connectToForm(() => {
  this.readyToPlay = true;
  this.readyToPlayAnimationCb();
});
```

**Animation method:**
```javascript
runReadyToPlayAnimation() {
  const items = this.getRootElement().querySelectorAll(`.${this.itemClassName}`);
  items?.forEach((item, index) => {
    setTimeout(() => {
      const wrapper = item.querySelector('.{wrapperElement}');
      if (!wrapper) return;
      wrapper.style.animation = 'itemPop 0.7s 1';
      setTimeout(() => {
        wrapper.style.animation = '';
        item.classList.add('hover-ready');
      }, 700);
    }, (index + 1) * 200);
  });
}
```

**CSS:**
```css
@keyframes itemPop {
  0% { transform: scale(1); }
  50% { transform: scale(1.2); }
  100% { transform: scale(1); }
}
```

## Demo Mode Pattern

```javascript
const demoMode = this.services.config.getConfig(PreviewMode.DEMO);
```

In demo mode:
- Skip form validation, skip `savePrizeOptionToStorage()` / `setGameDataToContext()` / `completeSubmitAction()`
- Auto-play the game with timeouts stored in `this.demoTimeouts`
- Loop the demo after completion

## ResizeObserver Pattern

For components that re-render on container size changes:

```javascript
addResizeObserver() {
  let skipFirst = true;
  this.resizeObserver = new ResizeObserver(() => {
    if (skipFirst) { skipFirst = false; return; }
    this.debouncedRender(this.getProps());
  });
  this.resizeObserver.observe(
    this.getRootElement().querySelector(`.${this.containerClassName}`)
  );
}
```

Always disconnect in `disconnectedCallback()`.
