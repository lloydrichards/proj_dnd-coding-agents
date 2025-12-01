# Lit Property Patterns Reference

This document provides comprehensive patterns for reactive property management in Lit components.

**Official Documentation:** https://lit.dev/docs/components/properties/

## Property Decorator Reference

### @property() - Public API Properties

Public properties that are part of the component's API. Can synchronize with attributes.

```typescript
@property({ type: String })
name: string = '';

@property({ type: Number })
count: number = 0;

@property({ type: Boolean })
disabled: boolean = false;

@property({ type: Array })
items: string[] = [];

@property({ type: Object })
config: Config = {};
```

**Full Options:**

```typescript
@property({
  type: String,              // Type hint for attribute conversion
  attribute: 'custom-name',  // Custom attribute name (or false to disable)
  reflect: true,             // Sync property changes back to attribute
  converter: customConverter, // Custom attribute converter
  hasChanged: (newVal, oldVal) => true, // Custom change detection
  noAccessor: false,         // Skip creating property accessor
  state: false,              // Mark as internal state (prefer @state())
  useDefault: true           // Lit 3+: Prevent initial reflection, allow attribute removal reset
})
propertyName: string;
```

**Standard Decorators (Lit 3+ with TypeScript 5.2+):**

```typescript
@property()
accessor myProp: string = 'value';
```

**Experimental Decorators (Default):**

```typescript
@property()
myProp: string = 'value';
```

### @state() - Internal State Properties

Private reactive properties not part of the public API. Never sync with attributes.

```typescript
@state()
private _selectedIndex: number = 0;

@state()
private _loading: boolean = false;

@state()
private _computedData: ProcessedItem[] = [];
```

**Characteristics:**

- Internal only (not exposed in component API)
- No attribute synchronization
- Triggers updates like `@property()`
- Convention: prefix with underscore (`_propertyName`)

### @query() - DOM Query Caching

Query and cache a single element from the component's render root.

```typescript
@query('#submit-button')
submitButton!: HTMLButtonElement;

@query('.error-message')
errorElement?: HTMLDivElement;

@query('slot[name="header"]')
headerSlot!: HTMLSlotElement;
```

**Characteristics:**

- Cached after first access
- Queries within Shadow DOM (render root)
- Use `!` (definite assignment) if element always exists
- Use `?` (optional) if element may not exist

### @queryAll() - Multiple Element Query

Query and cache multiple elements as a NodeList.

```typescript
@queryAll('.list-item')
listItems!: NodeListOf<HTMLElement>;

@queryAll('input[type="checkbox"]')
checkboxes!: NodeListOf<HTMLInputElement>;
```

### @queryAsync() - Async Element Query

For elements that may not be rendered immediately.

```typescript
@queryAsync('#dynamic-content')
dynamicContent!: Promise<HTMLElement>;

async firstUpdated() {
  const element = await this.dynamicContent;
  console.log(element);
}
```

### @queryAssignedElements() - Slot Content Query

Query elements assigned to a specific slot.

```typescript
@queryAssignedElements({ slot: 'header' })
headerElements!: Array<HTMLElement>;

@queryAssignedElements({ slot: '', flatten: true })
defaultSlotElements!: Array<HTMLElement>;
```

## Property Type Configuration

### Simple Types (Attribute Sync)

```typescript
// String
@property({ type: String })
name: string = '';

// Number (auto-converts "123" → 123)
@property({ type: Number })
age: number = 0;

// Boolean (presence = true, absence = false)
@property({ type: Boolean })
active: boolean = false;
```

### Complex Types (No Attribute Sync)

```typescript
// Array (no automatic attribute conversion)
@property({ type: Array, attribute: false })
items: Item[] = [];

// Object (no automatic attribute conversion)
@property({ type: Object, attribute: false })
config: ConfigObject = {};

// Date (requires custom converter)
@property({
  converter: {
    fromAttribute: (value) => value ? new Date(value) : null,
    toAttribute: (value) => value?.toISOString()
  }
})
createdAt: Date | null = null;
```

## Custom Property Converters

### Basic Converter Pattern

```typescript
const numberListConverter = {
  fromAttribute: (value: string | null): number[] => {
    return value ? value.split(',').map(Number) : [];
  },
  toAttribute: (value: number[]): string => {
    return value.join(',');
  }
};

@property({ converter: numberListConverter, reflect: true })
values: number[] = [];
```

### Complex Object Converter

```typescript
interface ComplexConfig {
  enabled: boolean;
  options: Record<string, any>;
}

const configConverter = {
  fromAttribute: (value: string | null): ComplexConfig => {
    if (!value) return { enabled: false, options: {} };
    try {
      return JSON.parse(value);
    } catch {
      return { enabled: false, options: {} };
    }
  },
  toAttribute: (value: ComplexConfig): string => {
    return JSON.stringify(value);
  }
};

@property({ converter: configConverter, reflect: true })
config: ComplexConfig = { enabled: false, options: {} };
```

## Custom Change Detection

### Deep Equality for Objects

```typescript
function deepEqual(a: any, b: any): boolean {
  if (a === b) return true;
  if (a == null || b == null) return false;
  if (typeof a !== 'object' || typeof b !== 'object') return false;

  const keysA = Object.keys(a);
  const keysB = Object.keys(b);

  if (keysA.length !== keysB.length) return false;

  return keysA.every(key => deepEqual(a[key], b[key]));
}

@property({
  type: Object,
  attribute: false,
  hasChanged: (newVal, oldVal) => !deepEqual(newVal, oldVal)
})
data: ComplexObject = {};
```

### Array Length Change Detection

```typescript
@property({
  type: Array,
  attribute: false,
  hasChanged: (newVal, oldVal) => {
    // Only trigger update if array length changes
    return newVal?.length !== oldVal?.length;
  }
})
items: Item[] = [];
```

### Threshold-Based Change Detection

```typescript
@property({
  type: Number,
  hasChanged: (newVal, oldVal) => {
    // Only update if change is significant (> 10%)
    const threshold = Math.abs(oldVal * 0.1);
    return Math.abs(newVal - oldVal) > threshold;
  }
})
percentage: number = 0;
```

## Property Reflection Patterns

### When to Use Reflect

**Use `reflect: true` when:**

- Need CSS attribute selectors (`:host([disabled])`)
- Want attribute to stay in sync with property
- Building form controls (for form association)
- Need attribute for accessibility (ARIA states)

```typescript
@property({ type: Boolean, reflect: true })
disabled: boolean = false;

@property({ type: String, reflect: true })
variant: 'primary' | 'secondary' = 'primary';
```

**Avoid `reflect: true` when:**

- Property updates frequently (performance cost - triggers attribute serialization)
- Property is complex object/array (can't serialize)
- Property is internal implementation detail
- No CSS or external tool needs attribute value

**Modern Alternatives to Reflection:**

- Consider `:state` pseudo selector (WICG proposal) for CSS state styling
- Consider Accessibility Object Model (AOM) for accessibility properties
- Use CSS custom properties for theming instead of reflected attributes

### useDefault Option (Lit 3+)

```typescript
@property({ reflect: true, useDefault: true })
theme: 'light' | 'dark' = 'light';
```

**Behavior:**
- Prevents initial attribute reflection when value equals default
- Resets property to default when attribute is removed
- Useful for avoiding attribute bloat in HTML

## Property Update Lifecycle

### Accessing Old Values in willUpdate

```typescript
willUpdate(changedProperties: PropertyValues) {
  if (changedProperties.has('items')) {
    const oldItems = changedProperties.get('items') as Item[];
    console.log(`Items changed from ${oldItems?.length} to ${this.items.length}`);

    // Compute derived state
    this._filteredItems = this.items.filter(item => item.active);
  }

  if (changedProperties.has('sortBy') || changedProperties.has('items')) {
    // Recompute when either property changes
    this._sortedItems = this._sortItems(this.items, this.sortBy);
  }
}
```

### Side Effects in updated

```typescript
updated(changedProperties: PropertyValues) {
  if (changedProperties.has('value')) {
    // Emit custom event when value changes
    this.dispatchEvent(new CustomEvent('value-changed', {
      detail: { value: this.value },
      bubbles: true,
      composed: true
    }));
  }

  if (changedProperties.has('focused') && this.focused) {
    // Focus input when focused property becomes true
    this.inputElement?.focus();
  }
}
```

## Common Property Patterns

### Validated Properties

```typescript
private _email: string = '';

@property({ type: String })
get email(): string {
  return this._email;
}
set email(value: string) {
  const oldValue = this._email;
  // Validate email format
  if (this._isValidEmail(value)) {
    this._email = value;
    this.requestUpdate('email', oldValue);
  } else {
    console.warn('Invalid email format:', value);
  }
}

private _isValidEmail(email: string): boolean {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}
```

### Coerced Properties

```typescript
private _count: number = 0;

@property({ type: Number })
get count(): number {
  return this._count;
}
set count(value: number | string) {
  const oldValue = this._count;
  // Coerce to number and ensure non-negative
  this._count = Math.max(0, Number(value) || 0);
  this.requestUpdate('count', oldValue);
}
```

### Computed Properties

```typescript
@property({ type: Array, attribute: false })
items: Item[] = [];

@property({ type: String })
filter: string = '';

@state()
private _filteredItems: Item[] = [];

willUpdate(changedProperties: PropertyValues) {
  if (changedProperties.has('items') || changedProperties.has('filter')) {
    this._filteredItems = this.items.filter(item =>
      item.name.toLowerCase().includes(this.filter.toLowerCase())
    );
  }
}

render() {
  return html`
    ${this._filteredItems.map(item => html`<div>${item.name}</div>`)}
  `;
}
```

### Debounced Properties

```typescript
private _searchTerm: string = '';
private _searchDebounceTimer?: number;

@property({ type: String })
get searchTerm(): string {
  return this._searchTerm;
}
set searchTerm(value: string) {
  const oldValue = this._searchTerm;
  this._searchTerm = value;
  this.requestUpdate('searchTerm', oldValue);

  // Debounce search
  clearTimeout(this._searchDebounceTimer);
  this._searchDebounceTimer = window.setTimeout(() => {
    this._performSearch(value);
  }, 300);
}

disconnectedCallback() {
  super.disconnectedCallback();
  clearTimeout(this._searchDebounceTimer);
}
```

## TypeScript Type Safety Patterns

### Union Type Properties

```typescript
type Theme = 'light' | 'dark' | 'auto';

@property({ type: String, reflect: true })
theme: Theme = 'auto';

// With validation
set theme(value: Theme) {
  const validThemes: Theme[] = ['light', 'dark', 'auto'];
  this._theme = validThemes.includes(value) ? value : 'auto';
}
```

### Generic Property Types

```typescript
interface DataTable<T> extends LitElement {
  @property({ type: Array, attribute: false })
  data: T[];

  @property({ type: Function, attribute: false })
  renderRow: (item: T) => TemplateResult;
}
```

### Branded Types for Type Safety

```typescript
declare const brand: unique symbol;
type UserId = string & { [brand]: 'UserId' };
type OrderId = string & { [brand]: 'OrderId' };

@property({ type: String })
userId!: UserId;  // Type-safe, distinct from regular strings

@property({ type: String })
orderId!: OrderId;  // Different type from userId
```

## Property Best Practices

### DO:

- Use `@property()` for public API
- Use `@state()` for internal reactive state
- Set `attribute: false` for objects/arrays
- Use `reflect: true` sparingly and only when needed for CSS/external tools
- Provide default values for all properties
- Use TypeScript types for all properties
- Validate property values in setters
- Use `hasChanged` for deep equality on complex objects
- Consider `useDefault: true` (Lit 3+) to reduce attribute bloat
- Use standard decorators with `accessor` keyword if using TypeScript 5.2+

### DON'T:

- Don't use `@property()` for non-reactive properties
- Don't reflect complex objects to attributes (can't serialize)
- Don't update properties in `render()` (causes infinite loop)
- Don't access DOM in property setters (may not be rendered)
- Don't use public properties for internal state
- Don't mutate array/object properties (create new reference)
- Don't overuse `reflect: true` (performance cost)

## Performance Considerations

### Efficient Property Updates

```typescript
// BAD: Mutating array doesn't trigger update
addItem(item: Item) {
  this.items.push(item);  // No update triggered!
}

// GOOD: Create new array reference
addItem(item: Item) {
  this.items = [...this.items, item];  // Update triggered
}

// BETTER: Use hasChanged for deep comparison
@property({
  type: Array,
  hasChanged: (newVal, oldVal) => {
    return JSON.stringify(newVal) !== JSON.stringify(oldVal);
  }
})
items: Item[] = [];

addItem(item: Item) {
  this.items.push(item);
  this.requestUpdate('items');  // Manually trigger update
}
```

### Batch Property Updates

```typescript
// BAD: Multiple updates triggered
updateMultiple() {
  this.prop1 = 'value1';  // Update 1
  this.prop2 = 'value2';  // Update 2
  this.prop3 = 'value3';  // Update 3
}

// GOOD: Single batched update
updateMultiple() {
  this.prop1 = 'value1';
  this.prop2 = 'value2';
  this.prop3 = 'value3';
  // Update automatically batched in microtask
}
```

## Property Testing Patterns

### Testing Property Reactivity

```typescript
it("should update when property changes", async () => {
  const el = await fixture<MyElement>(html`<my-element></my-element>`);

  el.name = "test";
  await el.updateComplete; // Wait for update to complete

  expect(el.shadowRoot?.textContent).to.include("test");
});
```

### Testing Attribute Synchronization

```typescript
it("should sync property to attribute when reflect=true", async () => {
  const el = await fixture<MyElement>(html`<my-element></my-element>`);

  el.disabled = true;
  await el.updateComplete;

  expect(el.hasAttribute("disabled")).to.be.true;
});
```
