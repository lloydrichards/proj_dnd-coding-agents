# Lit Directives Reference

Concise reference for Lit template directives with practical usage patterns.

**Official Documentation:** https://lit.dev/docs/templates/directives/

## Conditionals

### when()

Renders one of two templates based on condition.

```typescript
import {when} from 'lit/directives/when.js';

${when(
  this.isLoading,
  () => html`<loading-spinner></loading-spinner>`,
  () => html`<div>${this.content}</div>`
)}
```

**Use when:** Simple if-else rendering with lazy evaluation.

### choose()

Renders template based on matching case value.

```typescript
import {choose} from 'lit/directives/choose.js';

${choose(this.view, [
  ['home', () => html`<home-page></home-page>`],
  ['profile', () => html`<profile-page></profile-page>`],
  ['settings', () => html`<settings-page></settings-page>`]
], () => html`<not-found></not-found>`)}
```

**Use when:** Multiple conditional branches (switch-case pattern).

## Loops & Lists

### map()

Simple list rendering.

```typescript
import {map} from 'lit/directives/map.js';

${map(this.items, (item, index) => html`
  <li>${index}: ${item.name}</li>
`)}
```

**Use when:** Simple lists without DOM stability requirements.  
**Bundle size:** Smaller than `repeat()`.  
**Performance:** Faster for simple cases.

### repeat()

Keyed list rendering with DOM stability.

```typescript
import {repeat} from 'lit/directives/repeat.js';

${repeat(
  this.items,
  (item) => item.id,
  (item, index) => html`<li>${item.name}</li>`
)}
```

**Use when:** List items can reorder/shuffle and you need to preserve DOM state.  
**Example:** Drag-and-drop lists, sortable tables, animations.

### join()

Renders items with separator between them.

```typescript
import {join} from 'lit/directives/join.js';

${join(
  map(this.tags, (tag) => html`<span class="tag">${tag}</span>`),
  html`<span class="separator">•</span>`
)}
```

**Use when:** Need separators between list items (breadcrumbs, tags, inline lists).

### range()

Generates numeric range for iteration.

```typescript
import {range} from 'lit/directives/range.js';
import {map} from 'lit/directives/map.js';

${map(range(1, 10), (i) => html`<div>Item ${i}</div>`)}

${map(range(10), (i) => html`<div>Page ${i + 1}</div>`)}
```

**Use when:** Need to render N items without an existing array.

## Caching & Performance

### cache()

Caches DOM for conditional branches.

```typescript
import {cache} from 'lit/directives/cache.js';

${cache(
  this.view === 'grid'
    ? html`<grid-view .items=${this.items}></grid-view>`
    : html`<list-view .items=${this.items}></list-view>`
)}
```

**Use when:** Switching between expensive-to-render templates.  
**Benefit:** Preserves DOM instead of recreating it.

### guard()

Prevents re-rendering unless dependencies change.

```typescript
import {guard} from 'lit/directives/guard.js';

${guard([this.items, this.filter], () => 
  this.items
    .filter(item => item.name.includes(this.filter))
    .map(item => html`<div>${item.name}</div>`)
)}
```

**Use when:** Expensive computations/transformations in templates.  
**Benefit:** Only re-evaluates when dependencies change.

### keyed()

Associates DOM with a value.

```typescript
import {keyed} from 'lit/directives/keyed.js';

${keyed(this.userId, html`
  <user-profile .userId=${this.userId}></user-profile>
`)}
```

**Use when:** Component needs to fully re-render when a key changes.  
**Example:** User profile switching, preventing state bleed between items.

### live()

Syncs input value with live property.

```typescript
import {live} from 'lit/directives/live.js';

<input .value=${live(this.data.value)}>
```

**Use when:** Input value can be changed externally (e.g., undo/redo, data binding).  
**Without live():** Input won't update if user typed then property changed.

## DOM References

### ref()

Gets reference to rendered element.

```typescript
import {ref, createRef} from 'lit/directives/ref.js';

inputRef = createRef<HTMLInputElement>();

render() {
  return html`<input ${ref(this.inputRef)}>`;
}

firstUpdated() {
  this.inputRef.value?.focus();
}
```

**Use when:** Need programmatic access to rendered elements.  
**Alternative:** `@query()` decorator for static selectors.

## Styling

### classMap()

Conditionally applies classes.

```typescript
import {classMap} from 'lit/directives/class-map.js';

<div class=${classMap({
  active: this.isActive,
  disabled: this.disabled,
  'has-error': this.error
})}></div>
```

**Use when:** Dynamic class names based on component state.

### styleMap()

Conditionally applies inline styles.

```typescript
import {styleMap} from 'lit/directives/style-map.js';

<div style=${styleMap({
  color: this.textColor,
  backgroundColor: this.bgColor,
  display: this.visible ? 'block' : 'none'
})}></div>
```

**Use when:** Dynamic inline styles. Prefer CSS custom properties for theming.

### ifDefined()

Only sets attribute if value is defined.

```typescript
import {ifDefined} from 'lit/directives/if-defined.js';

<img src=${ifDefined(this.imageSrc)} alt="...">
```

**Use when:** Avoid setting attributes to `undefined` or `null` strings.

## Async Rendering

### until()

Renders placeholder until Promise resolves.

```typescript
import {until} from 'lit/directives/until.js';

${until(
  this.fetchUserData().then(user => html`<div>${user.name}</div>`),
  html`<loading-spinner></loading-spinner>`
)}
```

**Use when:** Rendering async data with loading state.

### asyncAppend()

Appends values from AsyncIterable.

```typescript
import {asyncAppend} from 'lit/directives/async-append.js';

${asyncAppend(this.streamData(), (value) => html`<div>${value}</div>`)}
```

**Use when:** Streaming data that appends (logs, messages).

### asyncReplace()

Replaces content with values from AsyncIterable.

```typescript
import {asyncReplace} from 'lit/directives/async-replace.js';

${asyncReplace(this.pollData(), (value) => html`<div>${value}</div>`)}
```

**Use when:** Streaming data that replaces (countdown, live updates).

## Unsafe Content

### unsafeHTML()

Renders raw HTML (XSS risk).

```typescript
import {unsafeHTML} from 'lit/directives/unsafe-html.js';

${unsafeHTML(this.sanitizedHtmlString)}
```

**Warning:** Only use with trusted, sanitized content.

### unsafeSVG()

Renders raw SVG (XSS risk).

```typescript
import {unsafeSVG} from 'lit/directives/unsafe-svg.js';

${unsafeSVG(this.svgString)}
```

**Warning:** Only use with trusted, sanitized content.

### templateContent()

Stamps template element content.

```typescript
import {templateContent} from 'lit/directives/template-content.js';

${templateContent(this.templateElement)}
```

**Use when:** Rendering existing `<template>` elements.

## Directive Selection Guide

**Conditionals:**
- Simple if/else → `when()`
- Multiple cases → `choose()`
- Inline ternary → `condition ? a : b`

**Lists:**
- Simple, static order → `map()`
- Reorderable, need DOM stability → `repeat()`
- Need separators → `join()`

**Performance:**
- Expensive branches → `cache()`
- Expensive computations → `guard()`
- Component identity → `keyed()`
- Input synchronization → `live()`

**DOM Access:**
- Element reference → `ref()` or `@query()`

**Styling:**
- Dynamic classes → `classMap()`
- Dynamic styles → `styleMap()`
- Optional attributes → `ifDefined()`

**Async:**
- Promise rendering → `until()`
- Streaming append → `asyncAppend()`
- Streaming replace → `asyncReplace()`
