# WCAG Audit Patterns

## Quick Assessment Checklist

- [ ] Keyboard navigable (Tab, Arrow keys, Enter, Escape)
- [ ] Screen reader announcements clear and meaningful
- [ ] Color contrast meets WCAG AA (4.5:1 normal, 3:1 large text)
- [ ] Focus indicators visible and clear
- [ ] Interactive elements have adequate touch targets (44x44px)
- [ ] Meaningful alt text for images
- [ ] Form labels properly associated
- [ ] Error messages clear and associated with inputs

## Structured Report Template

```markdown
# Accessibility Analysis Report

## Executive Summary
**Overall Compliance:** [WCAG Level A/AA/AAA]
**Critical Issues:** [Count]
**Risk Level:** [High/Medium/Low]

## Issues by Impact

### 🔴 Critical (Blockers)
**Issue:** [Description]
- **Location:** [file:line]
- **WCAG Criterion:** [e.g., 2.1.1 Keyboard]
- **User Impact:** [Who is affected and how]
- **Fix:** [Solution with code example]
- **Testing:** [Verification method]

### 🟡 Major (Should Fix)
[Similar structure]

### 🟢 Minor (Consider Fixing)
[Similar structure]
```

## Component-Specific Patterns

### Forms
- Label associations (explicit via `for`/`id`, implicit via nesting)
- Error announcements (`role="alert"`, `aria-live="polite"`)
- Required indicators (`aria-required="true"`, visual indicator)
- Group labeling (`<fieldset>` and `<legend>`)
- Autocomplete (`autocomplete` attribute values)

### Modals/Dialogs
- Focus trap implementation
- `role="dialog"` and `aria-modal="true"`
- Escape key handling
- Focus restoration on close
- Background `aria-hidden="true"`

### Navigation
- Landmark roles (`<nav>`, `role="navigation"`)
- Skip links for keyboard users
- `aria-current` for active states
- Mobile menu patterns
- Breadcrumb `aria-label="Breadcrumb"`

### Data Tables
- Header associations (`<th>`, `scope` attribute)
- Caption and summary
- Sort state announcements
- Complex tables (`headers` attribute)
- Responsive patterns

### Interactive Widgets
- Custom controls ARIA patterns
- State changes (`aria-pressed`, `aria-expanded`)
- Live regions (`aria-live`, `role="status"`)
- Loading states (`aria-busy`)
- Progress (`role="progressbar"`)

## Testing Methods

### Manual Testing
1. Navigate with keyboard only (no mouse)
2. Check tab order follows visual flow
3. Verify all interactive elements reachable
4. Test with screen reader (NVDA/JAWS/VoiceOver)
5. Validate color contrast ratios
6. Check touch target sizes (mobile)

### Automated Tools
- **axe-core/axe DevTools**: Comprehensive WCAG testing
- **WAVE**: Visual feedback on accessibility issues
- **Lighthouse**: Performance and accessibility audit
- **Pa11y**: CI/CD integration for automated testing

### Screen Reader Testing Matrix
| Screen Reader | Platform | Key Testing Points |
|--------------|----------|-------------------|
| NVDA | Windows | Free, widely used, good standards support |
| JAWS | Windows | Most popular commercial, extensive features |
| VoiceOver | macOS/iOS | Built-in Apple screen reader |
| TalkBack | Android | Built-in Android screen reader |