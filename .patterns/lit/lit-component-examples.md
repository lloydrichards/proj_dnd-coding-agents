# Lit Component Examples

Comprehensive component architecture examples for Lit framework.

## Data Table Component

A sortable, filterable data table component with pagination support.

### Public API Design

#### Properties

| Property | Type | Default | Reflects | Description |
|----------|------|---------|----------|-------------|
| `data` | `Array<Record<string, any>>` | `[]` | No | Dataset to display |
| `columns` | `ColumnConfig[]` | `[]` | No | Column definitions |
| `sortColumn` | `string` | `''` | Yes | Currently sorted column ID |
| `sortDirection` | `'asc' \| 'desc'` | `'asc'` | Yes | Sort direction |
| `pageSize` | `number` | `10` | Yes | Rows per page |
| `currentPage` | `number` | `0` | Yes | Current page index |

#### Events

| Event | Detail Type | Description |
|-------|-------------|-------------|
| `sort-changed` | `{column: string, direction: 'asc' \| 'desc'}` | User changed sort |
| `page-changed` | `{page: number}` | User navigated to page |
| `row-selected` | `{rowIndex: number, rowData: any}` | User selected row |

#### CSS Custom Properties

| Property | Default | Description |
|----------|---------|-------------|
| `--table-header-bg` | `#f5f5f5` | Header background |
| `--table-row-hover` | `#e8e8e8` | Row hover background |
| `--table-border` | `#ddd` | Border color |

### Implementation

```typescript
@customElement("data-table")
export class DataTable extends LitElement {
  @property({ type: Array, attribute: false, hasChanged: deepEqual })
  data: Array<Record<string, any>> = [];

  @property({ type: Array, attribute: false })
  columns: ColumnConfig[] = [];

  @property({ type: String, reflect: true, attribute: "sort-column" })
  sortColumn: string = "";

  @property({ type: String, reflect: true, attribute: "sort-direction" })
  sortDirection: "asc" | "desc" = "asc";

  @state()
  private _sortedData: Array<Record<string, any>> = [];

  @state()
  private _paginatedData: Array<Record<string, any>> = [];
}
```

### Template Design

```typescript
import {repeat} from 'lit/directives/repeat.js';
import {map} from 'lit/directives/map.js';
import {when} from 'lit/directives/when.js';

render() {
  return html`
    <table>
      <thead>
        <tr>
          ${map(this.columns, col => html`
            <th @click=${() => this._handleSort(col.id)}>
              ${col.label}
              ${this._renderSortIcon(col.id)}
            </th>
          `)}
        </tr>
      </thead>
      <tbody>
        ${when(
          this._paginatedData.length > 0,
          () => repeat(this._paginatedData, (row, idx) => idx, (row, idx) => html`
            <tr @click=${() => this._handleRowClick(idx, row)}>
              ${map(this.columns, col => html`
                <td>${col.format ? col.format(row[col.id]) : row[col.id]}</td>
              `)}
            </tr>
          `),
          () => html`<tr><td colspan="${this.columns.length}"><slot name="empty">No data</slot></td></tr>`
        )}
      </tbody>
    </table>
  `;
}
```

### Styling Strategy

```css
:host {
  display: block;
}

:host([sort-direction="desc"]) th[data-sorted]::after {
  content: "▼";
}

table {
  border-collapse: collapse;
  background: var(--table-bg, white);
}

th {
  background: var(--table-header-bg, #f5f5f5);
  cursor: pointer;
}

tr:hover {
  background: var(--table-row-hover, #e8e8e8);
}
```

### Lifecycle Implementation

```typescript
willUpdate(changedProperties: PropertyValues) {
  // Recompute sorted data when data or sort changes
  if (changedProperties.has('data') ||
      changedProperties.has('sortColumn') ||
      changedProperties.has('sortDirection')) {
    this._sortedData = this._computeSortedData();
  }

  // Recompute paginated data when sorted data or pagination changes
  if (changedProperties.has('_sortedData') ||
      changedProperties.has('currentPage') ||
      changedProperties.has('pageSize')) {
    this._paginatedData = this._computePaginatedData();
  }
}
```

### Event Handling

```typescript
private _handleSort(columnId: string) {
  const newDirection = this.sortColumn === columnId && this.sortDirection === 'asc'
    ? 'desc'
    : 'asc';

  this.sortColumn = columnId;
  this.sortDirection = newDirection;

  this.dispatchEvent(new CustomEvent('sort-changed', {
    detail: { column: columnId, direction: newDirection },
    bubbles: true,
    composed: true
  }));
}

private _handleRowClick(rowIndex: number, rowData: any) {
  this.dispatchEvent(new CustomEvent('row-selected', {
    detail: { rowIndex, rowData },
    bubbles: true,
    composed: true
  }));
}
```

### Performance Considerations

- **Deep equality on data**: Prevents re-render when reference changes but content identical
- **`map()` for columns**: Simpler, smaller bundle than `repeat()` for stable column order
- **`repeat()` for rows**: Preserves DOM stability for row interactions
- **`when()` for conditionals**: Lazy evaluation of empty state template
- **Derived state in `willUpdate()`**: Compute sorted/paginated data once, not on every render
- **Static styles**: Parsed once, shared across all instances
- **Virtual scrolling**: Consider for datasets > 1000 rows (use `lit-virtualizer`)

### TypeScript Interfaces

```typescript
interface ColumnConfig {
  id: string;
  label: string;
  format?: (value: any) => string | TemplateResult;
  sortable?: boolean;
}

declare global {
  interface HTMLElementEventMap {
    "sort-changed": CustomEvent<{ column: string; direction: "asc" | "desc" }>;
    "page-changed": CustomEvent<{ page: number }>;
    "row-selected": CustomEvent<{ rowIndex: number; rowData: any }>;
  }
}
```

### Usage Examples

```html
<!-- Basic usage -->
<data-table
  .data=${userData}
  .columns=${[
    { id: 'name', label: 'Name' },
    { id: 'email', label: 'Email' },
    { id: 'age', label: 'Age', format: (val) => `${val} years` }
  ]}
></data-table>

<!-- With empty state -->
<data-table .data=${[]} .columns=${columns}>
  <div slot="empty">No users found</div>
</data-table>

<!-- Event handling -->
<data-table
  @sort-changed=${(e) => console.log('Sort:', e.detail)}
  @row-selected=${(e) => showDetails(e.detail.rowData)}
></data-table>
```

### Testing Considerations

- Unit test sorting logic with various data types
- Test pagination boundary conditions (first page, last page, single page)
- Integration test event emission on user interactions
- Accessibility: Verify ARIA labels, keyboard navigation
- Performance: Benchmark with 1000+ rows

### Browser Compatibility

- Minimum: Chrome 90+, Firefox 88+, Safari 14+ (native Shadow DOM)
- No polyfills required for modern browsers
