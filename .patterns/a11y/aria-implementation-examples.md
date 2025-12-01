# ARIA Implementation Examples

## Form Accessibility

### Inaccessible Error Messages

**Problem:**
```html
<input type="email" id="email" />
<span class="error">Invalid email format</span>
```

**Issues:**
- Error not programmatically associated
- No screen reader announcement
- Color-only indication

**Solution:**
```html
<input 
  type="email" 
  id="email"
  aria-invalid="true"
  aria-describedby="email-error"
/>
<span 
  id="email-error" 
  class="error"
  role="alert"
  aria-live="polite"
>
  Invalid email format. Please enter a valid email address.
</span>
```

## Dialog Pattern

### Modal Dialog Implementation

**WAI-ARIA Reference:** https://www.w3.org/WAI/ARIA/apg/patterns/dialog-modal/

```html
<div
  role="dialog"
  aria-modal="true"
  aria-labelledby="dialog-title"
  aria-describedby="dialog-description"
>
  <h2 id="dialog-title">Confirm Action</h2>
  <p id="dialog-description">
    Are you sure you want to delete this item?
  </p>
  <button>Cancel</button>
  <button>Delete</button>
</div>
```

**Focus Management:**
```javascript
// On open:
const previousFocus = document.activeElement;
dialog.querySelector('[autofocus]')?.focus();
trapFocus(dialog);

// On close:
previousFocus?.focus();
```

**Keyboard Interactions:**
- **Tab**: Move through focusable elements
- **Shift+Tab**: Reverse direction
- **Escape**: Close dialog
- **Enter**: Activate focused button

## Color Contrast

### Fixing Contrast Issues

**Failing Example:**
```css
.text-muted {
  color: #777777; /* 3.5:1 on white - FAILS AA */
}
```

**WCAG AA Compliant:**
```css
.text-muted {
  color: #595959; /* 4.5:1 on white - PASSES AA */
}
```

**WCAG AAA Compliant:**
```css
.text-muted {
  color: #454545; /* 7.1:1 on white - PASSES AAA */
}
```

**Required Ratios:**
- Normal text: 4.5:1 (AA), 7:1 (AAA)
- Large text (18pt+): 3:1 (AA), 4.5:1 (AAA)
- UI components: 3:1 (AA)

## Common ARIA Patterns

### Accordion
```html
<button
  aria-expanded="false"
  aria-controls="panel-1"
>
  Section Title
</button>
<div id="panel-1" hidden>
  Content
</div>
```

### Tab Panel
```html
<div role="tablist">
  <button role="tab" aria-selected="true" aria-controls="panel-1">Tab 1</button>
  <button role="tab" aria-selected="false" aria-controls="panel-2">Tab 2</button>
</div>
<div role="tabpanel" id="panel-1">Content 1</div>
<div role="tabpanel" id="panel-2" hidden>Content 2</div>
```

### Live Region
```html
<!-- Polite announcement (waits for screen reader to finish) -->
<div role="status" aria-live="polite">
  Form saved successfully
</div>

<!-- Assertive announcement (interrupts immediately) -->
<div role="alert" aria-live="assertive">
  Error: Invalid input
</div>
```

## Framework-Specific Patterns

### React
```jsx
// Focus management
const buttonRef = useRef(null);
useEffect(() => {
  buttonRef.current?.focus();
}, []);

// Live regions with react-aria-live
<LiveMessage message={status} aria-live="polite" />
```

### Lit/Web Components
```javascript
// Shadow DOM with semantic HTML
render() {
  return html`
    <button
      part="trigger"
      aria-expanded=${this.open}
      @click=${this.toggle}
    >
      <slot name="trigger">Open</slot>
    </button>
  `;
}
```

### Vue
```vue
<!-- Dynamic announcements -->
<div 
  role="status"
  aria-live="polite"
  :aria-atomic="true"
>
  {{ statusMessage }}
</div>
```