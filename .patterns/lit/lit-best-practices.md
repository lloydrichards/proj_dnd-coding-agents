# Lit Framework Best Practices Reference

This document contains reference material for web component development using Lit. It covers framework comparisons, version features, and best practices.

**Official Documentation:** https://lit.dev/docs/

## Web Components vs React Components

**Key Conceptual Differences:**

| Aspect | React Component | Web Component (Lit) |
| --- | --- | --- |
| **Rendering** | Virtual DOM diffing | lit-html template stamping |
| **State** | useState, setState | Reactive properties (`@property`, `@state`) |
| **Encapsulation** | CSS Modules, styled-components | Shadow DOM (native encapsulation) |
| **Lifecycle** | useEffect, componentDidMount | connectedCallback, updated |
| **Events** | Callback props (`onClick={fn}`) | CustomEvents (`addEventListener`) |
| **Children** | JSX children, render props | `<slot>` (native projection) |
| **Props** | JavaScript objects | Attributes (strings) + properties (objects) |
| **Update Control** | React scheduler, useMemo | Automatic on property change, `requestUpdate()` |
| **Styling** | className, inline styles | `:host`, CSS custom properties, Shadow DOM |
| **Composition** | Component composition, HOCs | Element composition, slots, mixins |

**Design Implications:**

Web components require different thinking:

- **No automatic prop drilling** - use events for upward communication, properties for downward
- **Attribute vs property distinction** - simple values via attributes, complex via properties
- **Shadow DOM isolation** - global styles don't leak in, component styles don't leak out
- **Framework agnostic** - must work in vanilla JS, React, Vue, Angular
- **Standard lifecycle** - follows DOM spec, not framework conventions
- **Imperative API** - components expose methods and properties, not just props

## Lit-Specific Best Practices

### Component Registration

- Use `@customElement('my-element')` decorator for element registration
- Custom element names **must** contain a hyphen (e.g., `my-component`, not `mycomponent`)
- Register globally or use scoped registries for isolation

### Styling Patterns

- Define static styles with `static styles = css\`...\`` for performance (reused across instances)
- Use `:host` selector for component root styling
- Use `:host([attr])` for attribute-based variants
- Use `::slotted()` for styling projected content
- Design with CSS custom properties (`--component-*`) for themability
- Leverage `adoptedStyleSheets` for shared style efficiency

### Template Optimization

**Documentation:** https://lit.dev/docs/templates/directives/

- Leverage directives: `when`, `choose`, `map`, `repeat`, `cache`, `guard`, `keyed`, `live`, `ref`
- Use styling directives: `classMap`, `styleMap`, `ifDefined`
- Use `@query` and `@queryAll` instead of `querySelector` for cached DOM access
- Minimize dynamic parts (only wrap changing values in `${}`)
- Use keyed `repeat()` for list rendering when DOM stability needed
- Use `map()` for simple lists (smaller, faster than `repeat()`)
- Use `cache()` for conditional template branches
- Use `guard()` to prevent re-evaluation of expensive expressions
- Use `keyed()` to associate DOM with changing values
- Use `ref()` for DOM element references

### Property Management

**Documentation:** https://lit.dev/docs/components/properties/

- Use `@property()` for public API, `@state()` for internal state
- Specify `@property({ type: Object })` for complex types to prevent automatic conversion
- Use `hasChanged` option for deep equality checks on object/array properties
- Set `attribute: false` for properties that shouldn't sync to attributes
- Set `reflect: true` sparingly for properties that should update attributes (performance cost)
- Use `useDefault: true` (Lit 3+) to prevent initial attribute reflection and allow attribute removal reset
- Consider `:state` pseudo selector or Accessibility Object Model instead of reflection when possible

### Update Control

**Documentation:** https://lit.dev/docs/components/lifecycle/

- Implement `requestUpdate()` only when automatic change detection is insufficient
- Use `scheduleUpdate()` override for custom scheduling (not `performUpdate()`)
- Plan `renderRoot` customization only for advanced use cases (default is Shadow DOM)
- Use `willUpdate()` for computing derived state before render
- Use `updated()` for DOM interactions after render
- Use `await this.updateComplete` before dispatching events for rendered state

### Performance Considerations

- Static styles are parsed once and shared across all component instances
- Reactive properties trigger targeted updates (only affected parts re-render)
- Template caching with `cache()` and `guard()` prevents unnecessary work
- Use `repeat()` with keys for efficient list updates
- Lazy load large components or features with dynamic imports

## Lit Version Feature Matrix

**Documentation:** https://lit.dev/docs/releases/upgrade/

| Version | Key Features |
| --- | --- |
| 3.0+ | Standard decorators support, ES2021 output, improved SSR with declarative shadow DOM, new directives, official React integration |
| 2.0+ | Experimental decorators, reactive controllers, `@lit/reactive-element` package, improved tree-shaking |
| 1.0+ | Core lit-html + LitElement integration, Shadow DOM by default, stable API |

### Version-Specific Features

**Lit 3.0+ (Current):**

- **Standard Decorators:** Support for TC39 stage 3 decorators with `accessor` keyword (TypeScript 5.2+)
- **ES2021 Target:** Published as ES2021 (was ES2019 in Lit 2)
- **SSR Improvements:** Declarative shadow DOM, better hydration with `@lit-labs/ssr-client`
- **Official React Package:** `@lit/react` for creating React wrappers
- **New Directives:** `when()`, `choose()`, `map()`, `join()`, `range()`, `keyed()`, `live()`, `ref()`
- **Property Options:** New `useDefault` option for better attribute reflection control
- **Type Improvements:** Better TypeScript inference and stricter types
- **Breaking Changes:** Minimal, mostly removed deprecated APIs from Lit 1.x

**Migration Note:** Lit 2.x and 3.x are interoperable. Most projects can use version range `"^2.7.0 || ^3.0.0"`.

**Lit 2.0+:**

- Experimental decorators (TypeScript `experimentalDecorators`)
- Reactive controllers for reusable behaviors
- `@lit/reactive-element` as foundation package
- Improved bundle size and tree-shaking

**Lit 1.0+:**

- Stable decorators (`@property`, `@state`, `@query`, `@customElement`)
- lit-html efficient template rendering
- Shadow DOM encapsulation by default
- Full TypeScript support

### Decorator Syntax Options (Lit 3+)

**Experimental Decorators (TypeScript default):**

```typescript
@property()
myProp: string = 'value';
```

**Standard Decorators (TypeScript 5.2+, optional):**

```typescript
@property()
accessor myProp: string = 'value';
```

**Configuration:**
- Experimental: `"experimentalDecorators": true` in `tsconfig.json`
- Standard: Remove `experimentalDecorators` and use TypeScript 5.2+
- Both syntaxes work in Lit 3.x

## Shadow DOM Best Practices

### When to Use Shadow DOM

- **Use Shadow DOM (default):**

  - Component needs style encapsulation
  - Building reusable component library
  - Component will be used in unknown contexts
  - Need to prevent global style leakage

- **Avoid Shadow DOM (light DOM):**
  - Need global styles to apply to component internals
  - Need direct CSS selector access from outside
  - Integrating with frameworks that expect light DOM
  - Building simple wrapper components

### Shadow DOM Styling Patterns

**Documentation:** https://lit.dev/docs/components/styles/

**`:host` Selector:**

```css
:host {
  display: block;
  position: relative;
}

:host([hidden]) {
  display: none;
}

:host([disabled]) {
  opacity: 0.5;
  pointer-events: none;
}
```

**CSS Custom Properties for Theming:**

```css
:host {
  --_padding: var(--component-padding, 1rem);
  --_bg: var(--component-bg, white);

  background: var(--_bg);
  padding: var(--_padding);
}
```

**Slotted Content Styling:**

```css
::slotted(*) {
  margin: 0;
}

::slotted(h2) {
  font-size: 1.5rem;
}
```

**Unicode Characters in Styles:**

```css
div::before {
  content: '\u2022';
}
```

## Slot Composition Patterns

### Basic Slot Usage

```html
<!-- Component template -->
<div class="header">
  <slot name="header"></slot>
</div>
<div class="content">
  <slot></slot>
  <!-- Default slot -->
</div>
<div class="footer">
  <slot name="footer">Default footer</slot>
  <!-- With fallback -->
</div>
```

### Slot Change Detection

```typescript
@query('slot[name="header"]')
headerSlot!: HTMLSlotElement;

firstUpdated() {
  this.headerSlot.addEventListener('slotchange', () => {
    const hasHeader = this.headerSlot.assignedElements().length > 0;
    // Update component state based on slot content
  });
}
```

## Lifecycle Method Best Practices

**Documentation:** https://lit.dev/docs/components/lifecycle/

### Constructor

- Initialize non-reactive properties
- Set up event listeners on `this` only (DOM not available)
- **Do NOT** access DOM or attributes (not parsed yet)
- Keep synchronous and lightweight

### connectedCallback

- Set up external listeners (window, document, stores)
- Initialize third-party libraries requiring DOM
- Start timers or subscriptions
- Call `super.connectedCallback()` first
- **SSR Note:** Only runs on client (not during server rendering)

### disconnectedCallback

- Clean up external listeners and subscriptions
- Cancel pending async operations
- Dispose third-party library instances
- Call `super.disconnectedCallback()` first
- **Critical:** Prevent memory leaks

### willUpdate

- Compute derived state from changed properties
- Validate property combinations
- Prepare data transformations before render
- Access old property values via `changedProperties` map
- **SSR Note:** Runs on both server and client

### updated

- Interact with rendered DOM (measurements, focus)
- Trigger side effects based on property changes
- Integrate with non-Lit libraries requiring DOM
- Check `changedProperties` to avoid unnecessary work
- Dispatch events after `await this.updateComplete` for consistency
- **SSR Note:** Only runs on client (not during server rendering)

### firstUpdated

- One-time setup after first render
- Initialize libraries requiring rendered DOM
- Set initial focus or scroll position
- Only called once per component lifetime
- **SSR Note:** Only runs on client (not during server rendering)

### SSR Lifecycle Subset

Only these lifecycle methods run during server-side rendering:
- `constructor`
- `willUpdate`
- `render`

Client-only methods (run during hydration):
- `connectedCallback`
- `updated`
- `firstUpdated`

## Performance Optimization Patterns

### Static Styles

```typescript
static styles = css`
  :host { display: block; }
  /* Parsed once, shared across all instances */
`;
```

### Template Caching

```typescript
render() {
  return html`
    ${cache(this.viewMode === 'list'
      ? this.renderListView()
      : this.renderGridView()
    )}
  `;
}
```

### Guarded Expensive Operations

```typescript
render() {
  return html`
    ${guard([this.items], () => this.expensiveTransform(this.items))}
  `;
}
```

### List Rendering

```typescript
import {map} from 'lit/directives/map.js';
import {repeat} from 'lit/directives/repeat.js';

render() {
  return html`
    <ul>
      ${map(this.items, (item) => html`<li>${item.name}</li>`)}
    </ul>
  `;
}

render() {
  return html`
    <ul>
      ${repeat(this.items, (item) => item.id, (item) => html`
        <li>${item.name}</li>
      `)}
    </ul>
  `;
}
```

**When to use:**
- Use `map()` for simple lists (smaller bundle, faster)
- Use `repeat()` with keys when DOM stability is important

### Lazy Loading Components

```typescript
async loadEditor() {
  const { CodeEditor } = await import('./code-editor.js');
  this.editorLoaded = true;
  this.requestUpdate();
}

render() {
  return html`
    ${this.editorLoaded
      ? html`<code-editor></code-editor>`
      : html`<button @click=${this.loadEditor}>Load Editor</button>`
    }
  `;
}
```

## Validation Checklist

Before finalizing component architecture, verify:

- [ ] Component name contains hyphen (custom element requirement)
- [ ] Public API uses `@property()`, internal state uses `@state()`
- [ ] Complex properties have `attribute: false` option
- [ ] Events follow CustomEvent pattern with typed detail
- [ ] Shadow DOM styles use CSS custom properties for theming
- [ ] Lifecycle methods properly clean up resources
- [ ] TypeScript interfaces defined for all complex types
- [ ] Framework interop requirements addressed (if needed)
- [ ] Performance optimizations specified (static styles, directives)
- [ ] Accessibility considerations documented (ARIA, keyboard nav)
- [ ] Component works without JavaScript (progressive enhancement where applicable)
- [ ] Error states and loading states handled gracefully
