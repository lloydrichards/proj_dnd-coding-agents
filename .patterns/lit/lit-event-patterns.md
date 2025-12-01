# Lit Event Patterns Reference

This document provides comprehensive patterns for event handling and emission in Lit components, including framework interoperability.

**Official Documentation:** https://lit.dev/docs/components/events/

## CustomEvent Pattern Fundamentals

### Basic CustomEvent Emission

```typescript
class MyElement extends LitElement {
  private _handleClick() {
    this.dispatchEvent(
      new CustomEvent("item-selected", {
        detail: { itemId: "123", timestamp: Date.now() },
        bubbles: true,
        composed: true,
        cancelable: false,
      })
    );
  }
}
```

**CustomEvent Options:**

| Option | Description | Default | When to Use |
| --- | --- | --- | --- |
| `bubbles` | Event bubbles up DOM tree | `false` | `true` for most component events (enables delegation) |
| `composed` | Event crosses shadow DOM boundary | `false` | `true` if parent needs to listen (most cases) |
| `cancelable` | Can be canceled with preventDefault() | `false` | `true` for events that can be prevented |
| `detail` | Custom payload data | `undefined` | Always include structured data |

### TypeScript Event Type Definitions

```typescript
// Define event detail interfaces
interface ItemSelectedDetail {
  itemId: string;
  item: Item;
  source: "click" | "keyboard";
}

interface ValueChangedDetail {
  value: string;
  previousValue: string;
}

// Augment HTMLElementEventMap for type safety
declare global {
  interface HTMLElementEventMap {
    "item-selected": CustomEvent<ItemSelectedDetail>;
    "value-changed": CustomEvent<ValueChangedDetail>;
    "data-loaded": CustomEvent<{ count: number }>;
  }
}

// Usage with type safety
class MyElement extends LitElement {
  private _emitSelection(item: Item) {
    this.dispatchEvent(
      new CustomEvent<ItemSelectedDetail>("item-selected", {
        detail: {
          itemId: item.id,
          item: item,
          source: "click",
        },
        bubbles: true,
        composed: true,
      })
    );
  }
}
```

## Event Naming Conventions

### Best Practices

```typescript
// GOOD: Descriptive, kebab-case, namespace-aware
"item-selected";
"file-uploaded";
"form-submitted";
"data-table-sorted";
"user-profile-updated";

// BAD: Vague, inconsistent casing
"select";
"click";
"itemSelected"; // camelCase not web standard
"ITEM_SELECTED"; // SCREAMING_SNAKE_CASE not standard
```

**Naming Rules:**

- Use kebab-case (lowercase with hyphens)
- Be descriptive and specific
- Use past tense for completed actions (`item-selected`, not `item-select`)
- Prefix with component name for namespacing if needed (`data-table-sorted`)
- Avoid generic names that conflict with native events

## Event Bubbling and Composition

### When to Bubble

```typescript
// Bubble when parent containers need to know
this.dispatchEvent(
  new CustomEvent("item-selected", {
    bubbles: true, // Parent can use event delegation
    composed: true, // Crosses shadow boundary
  })
);
```

### When NOT to Bubble

```typescript
// Don't bubble for internal implementation events
this.dispatchEvent(
  new CustomEvent("_internal-state-changed", {
    bubbles: false, // Internal only
    composed: false, // Stays in shadow DOM
  })
);
```

### Event Delegation Pattern

```typescript
// Parent component can listen to events from many children
class ParentElement extends LitElement {
  render() {
    return html`
      <div @item-selected=${this._handleSelection}>
        ${this.items.map(
          (item) => html` <child-element .item=${item}></child-element> `
        )}
      </div>
    `;
  }

  private _handleSelection(e: CustomEvent<ItemSelectedDetail>) {
    console.log("Selected:", e.detail.itemId);
    // Event bubbled up from child
  }
}
```

## Cancelable Events

### Request/Response Pattern

```typescript
class MyElement extends LitElement {
  private _handleAction() {
    // Dispatch cancelable event to ask for permission
    const event = this.dispatchEvent(
      new CustomEvent("before-delete", {
        detail: { itemId: this.itemId },
        bubbles: true,
        composed: true,
        cancelable: true, // Parent can prevent with preventDefault()
      })
    );

    if (event.defaultPrevented) {
      console.log("Delete canceled by parent");
      return;
    }

    // Proceed with delete
    this._deleteItem();
  }
}

// Parent can cancel
class ParentElement extends LitElement {
  render() {
    return html`
      <my-element @before-delete=${this._handleBeforeDelete}></my-element>
    `;
  }

  private _handleBeforeDelete(e: CustomEvent) {
    if (!confirm("Are you sure?")) {
      e.preventDefault(); // Cancel the delete
    }
  }
}
```

## Event Listening Patterns

### Template Event Listeners

```typescript
render() {
  return html`
    <!-- Property binding with @ -->
    <button @click=${this._handleClick}>Click</button>

    <!-- Custom events -->
    <child-element @value-changed=${this._handleValueChange}></child-element>

    <!-- With event modifiers (not built-in, must implement) -->
    <input @input=${this._handleInput}>
  `;
}
```

### Programmatic Event Listeners

```typescript
connectedCallback() {
  super.connectedCallback();

  // Listen to events on this element
  this.addEventListener('custom-event', this._handleCustom);

  // Listen to events on window/document
  window.addEventListener('resize', this._handleResize);
  document.addEventListener('click', this._handleClickOutside);
}

disconnectedCallback() {
  super.disconnectedCallback();

  // CRITICAL: Remove listeners to prevent memory leaks
  this.removeEventListener('custom-event', this._handleCustom);
  window.removeEventListener('resize', this._handleResize);
  document.removeEventListener('click', this._handleClickOutside);
}
```

### Event Listener Options

**Programmatic Options:**

```typescript
connectedCallback() {
  super.connectedCallback();

  this.addEventListener('touchstart', this._handleTouch, { passive: true });
  this.addEventListener('focus', this._handleFocus, { capture: true });
  this.addEventListener('load', this._handleLoad, { once: true });
}
```

**Decorator Options:**

```typescript
import {eventOptions} from 'lit/decorators.js';

@eventOptions({passive: true})
private _handleTouchStart(e: TouchEvent) {
  console.log(e.type);
}
```

**Template Options (Object Listener):**

```typescript
render() {
  return html`
    <button @click=${{
      handleEvent: () => this.onClick(),
      once: true
    }}>Click Once</button>
  `;
}
```

## Framework Interoperability

### React Integration

**Documentation:** https://lit.dev/docs/frameworks/react/

**Problem:** React doesn't automatically bind to CustomEvents or web component properties.

**Solution:** Use the official `@lit/react` package.

#### Official @lit/react Package (Recommended)

```typescript
import React from 'react';
import {createComponent} from '@lit/react';
import {MyElement} from './my-element.js';

export const MyElementComponent = createComponent({
  tagName: 'my-element',
  elementClass: MyElement,
  react: React,
  events: {
    onvaluechange: 'value-changed',
    onitemselected: 'item-selected',
  },
});
```

**With Event Typing:**

```typescript
import {createComponent, type EventName} from '@lit/react';

interface MyElementEvents {
  valueChanged: CustomEvent<{value: string}>;
  itemSelected: CustomEvent<{itemId: string}>;
}

export const MyElementComponent = createComponent({
  tagName: 'my-element',
  elementClass: MyElement,
  react: React,
  events: {
    onvaluechange: 'value-changed' as EventName<MyElementEvents['valueChanged']>,
    onitemselected: 'item-selected' as EventName<MyElementEvents['itemSelected']>,
  },
});
```

**Usage in React:**

```typescript
function App() {
  const [value, setValue] = useState('');
  const [items, setItems] = useState<Item[]>([]);

  return (
    <MyElementComponent
      value={value}
      items={items}
      onvaluechange={(e) => setValue(e.detail.value)}
    />
  );
}
```

#### Using Reactive Controllers in React

```typescript
import {useController} from '@lit/react/use-controller.js';
import {MouseController} from './mouse-controller.js';

const useMouse = () => {
  const controller = useController(React, (host) => new MouseController(host));
  return controller.pos;
};

function MyComponent() {
  const mousePos = useMouse();
  return <div>Mouse: {mousePos.x}, {mousePos.y}</div>;
}
```

#### Manual React Wrapper (Legacy Pattern)

For cases where `@lit/react` cannot be used:

```typescript
import React, {useRef, useEffect} from 'react';

interface MyElementProps {
  value?: string;
  items?: Item[];
  onValueChanged?: (detail: {value: string}) => void;
}

export function MyElementReact({value, items, onValueChanged}: MyElementProps) {
  const ref = useRef<any>(null);

  useEffect(() => {
    if (ref.current && items) {
      ref.current.items = items;
    }
  }, [items]);

  useEffect(() => {
    if (ref.current && value !== undefined) {
      ref.current.value = value;
    }
  }, [value]);

  useEffect(() => {
    const element = ref.current;
    if (!element || !onValueChanged) return;

    const handler = (e: Event) => {
      onValueChanged((e as CustomEvent).detail);
    };

    element.addEventListener('value-changed', handler);
    return () => element.removeEventListener('value-changed', handler);
  }, [onValueChanged]);

  return <my-element ref={ref} />;
}
```

### Vue Integration

Vue has better native support for web components.

```vue
<script setup lang="ts">
import { ref } from "vue";

const value = ref("");
const items = ref<Item[]>([]);

function handleValueChanged(e: CustomEvent<{ value: string }>) {
  value.value = e.detail.value;
}
</script>

<template>
  <!-- Vue automatically handles properties and events -->
  <my-element
    :value="value"
    :items="items"
    @value-changed="handleValueChanged"
  />
</template>
```

**Vue Configuration for Web Components:**

```typescript
// vite.config.ts
import { defineConfig } from "vite";
import vue from "@vitejs/plugin-vue";

export default defineConfig({
  plugins: [
    vue({
      template: {
        compilerOptions: {
          // Treat all tags starting with 'my-' as custom elements
          isCustomElement: (tag) => tag.startsWith("my-"),
        },
      },
    }),
  ],
});
```

### Angular Integration

Angular also has good web component support.

```typescript
// app.module.ts
import { CUSTOM_ELEMENTS_SCHEMA, NgModule } from "@angular/core";

@NgModule({
  schemas: [CUSTOM_ELEMENTS_SCHEMA], // Enable custom elements
  // ...
})
export class AppModule {}

// component.ts
import { Component } from "@angular/core";

@Component({
  selector: "app-root",
  template: `
    <my-element
      [value]="value"
      [items]="items"
      (value-changed)="handleValueChanged($event)"
    ></my-element>
  `,
})
export class AppComponent {
  value = "";
  items: Item[] = [];

  handleValueChanged(event: CustomEvent<{ value: string }>) {
    this.value = event.detail.value;
  }
}
```

## Common Event Patterns

### Form Input Events

```typescript
class InputElement extends LitElement {
  @property({ type: String })
  value: string = "";

  private _handleInput(e: InputEvent) {
    const input = e.target as HTMLInputElement;
    this.value = input.value;

    // Emit change event with debouncing
    this._emitChange();
  }

  private _changeTimeout?: number;

  private _emitChange() {
    clearTimeout(this._changeTimeout);
    this._changeTimeout = window.setTimeout(() => {
      this.dispatchEvent(
        new CustomEvent("value-changed", {
          detail: { value: this.value },
          bubbles: true,
          composed: true,
        })
      );
    }, 300);
  }

  disconnectedCallback() {
    super.disconnectedCallback();
    clearTimeout(this._changeTimeout);
  }
}
```

### Selection Events

```typescript
class ListElement extends LitElement {
  @property({ type: Array, attribute: false })
  items: Item[] = [];

  @property({ type: String })
  selectedId: string = "";

  private _handleItemClick(item: Item) {
    const oldValue = this.selectedId;
    this.selectedId = item.id;

    // Emit with both old and new values
    this.dispatchEvent(
      new CustomEvent("selection-changed", {
        detail: {
          selectedId: item.id,
          selectedItem: item,
          previousId: oldValue,
        },
        bubbles: true,
        composed: true,
      })
    );
  }
}
```

### Loading State Events

```typescript
class DataElement extends LitElement {
  @state()
  private _loading: boolean = false;

  async loadData() {
    this._loading = true;
    this.dispatchEvent(
      new CustomEvent("loading-started", {
        bubbles: true,
        composed: true,
      })
    );

    try {
      const data = await fetch("/api/data").then((r) => r.json());

      this.dispatchEvent(
        new CustomEvent("data-loaded", {
          detail: { data, count: data.length },
          bubbles: true,
          composed: true,
        })
      );
    } catch (error) {
      this.dispatchEvent(
        new CustomEvent("loading-error", {
          detail: {
            error: error instanceof Error ? error.message : "Unknown error",
          },
          bubbles: true,
          composed: true,
        })
      );
    } finally {
      this._loading = false;
      this.dispatchEvent(
        new CustomEvent("loading-complete", {
          bubbles: true,
          composed: true,
        })
      );
    }
  }
}
```

### Drag and Drop Events

```typescript
class DraggableElement extends LitElement {
  @property({ type: String })
  itemId: string = "";

  private _handleDragStart(e: DragEvent) {
    e.dataTransfer!.effectAllowed = "move";
    e.dataTransfer!.setData("text/plain", this.itemId);

    this.dispatchEvent(
      new CustomEvent("drag-started", {
        detail: { itemId: this.itemId },
        bubbles: true,
        composed: true,
      })
    );
  }

  private _handleDrop(e: DragEvent) {
    e.preventDefault();
    const draggedId = e.dataTransfer!.getData("text/plain");

    this.dispatchEvent(
      new CustomEvent("item-dropped", {
        detail: {
          draggedId,
          targetId: this.itemId,
        },
        bubbles: true,
        composed: true,
      })
    );
  }

  render() {
    return html`
      <div
        draggable="true"
        @dragstart=${this._handleDragStart}
        @drop=${this._handleDrop}
        @dragover=${(e: DragEvent) => e.preventDefault()}
      >
        <slot></slot>
      </div>
    `;
  }
}
```

## Event Testing Patterns

### Testing Event Emission

```typescript
import { fixture, expect, oneEvent } from "@open-wc/testing";

it("should emit custom event when clicked", async () => {
  const el = await fixture<MyElement>(html`<my-element></my-element>`);

  setTimeout(() => el.click());
  const { detail } = await oneEvent(el, "item-selected");

  expect(detail.itemId).to.exist;
});
```

### Testing Event Handling

```typescript
it("should handle parent events", async () => {
  let capturedDetail: any;

  const el = await fixture<ParentElement>(html`
    <parent-element
      @item-selected=${(e: CustomEvent) => {
        capturedDetail = e.detail;
      }}
    ></parent-element>
  `);

  // Trigger child event
  const child = el.shadowRoot!.querySelector("child-element");
  child!.dispatchEvent(
    new CustomEvent("item-selected", {
      detail: { itemId: "123" },
      bubbles: true,
      composed: true,
    })
  );

  await el.updateComplete;
  expect(capturedDetail.itemId).to.equal("123");
});
```

## Event Best Practices

### DO:

- Use kebab-case for event names
- Always include `bubbles: true` for component events
- Always include `composed: true` to cross shadow boundaries
- Provide strongly-typed event details
- Augment `HTMLElementEventMap` for TypeScript autocomplete
- Use past tense for event names (`item-selected`, not `item-select`)
- Clean up event listeners in `disconnectedCallback()`
- Document all events in component API documentation
- Dispatch events after `await this.updateComplete` when emitting based on rendered state

### DON'T:

- Don't use camelCase or PascalCase for event names
- Don't emit events in `render()` (causes infinite loops)
- Don't forget to remove event listeners (memory leaks)
- Don't dispatch events before component is connected
- Don't use generic event names that conflict with native events
- Don't emit events synchronously in property setters (use `updated()`)
- Don't forget `bubbles` and `composed` for cross-boundary events

## Event Performance Considerations

### Debouncing Frequent Events

```typescript
private _scrollTimeout?: number;

private _handleScroll() {
  clearTimeout(this._scrollTimeout);
  this._scrollTimeout = window.setTimeout(() => {
    this.dispatchEvent(new CustomEvent('scroll-end', {
      detail: { scrollTop: this.scrollTop },
      bubbles: true
    }));
  }, 150);
}
```

### Throttling High-Frequency Events

```typescript
private _lastEmit: number = 0;
private _throttleMs: number = 100;

private _handleMouseMove(e: MouseEvent) {
  const now = Date.now();
  if (now - this._lastEmit < this._throttleMs) return;

  this._lastEmit = now;
  this.dispatchEvent(new CustomEvent('position-changed', {
    detail: { x: e.clientX, y: e.clientY },
    bubbles: true
  }));
}
```

### Event Pooling for Large Lists

```typescript
// Single listener on container (event delegation)
render() {
  return html`
    <div @click=${this._handleClick}>
      ${this.items.map(item => html`
        <div data-item-id=${item.id}>${item.name}</div>
      `)}
    </div>
  `;
}

private _handleClick(e: MouseEvent) {
  const target = e.target as HTMLElement;
  const itemId = target.dataset.itemId;
  if (itemId) {
    this.dispatchEvent(new CustomEvent('item-clicked', {
      detail: { itemId },
      bubbles: true
    }));
  }
}
```
