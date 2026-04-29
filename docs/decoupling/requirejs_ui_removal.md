# RequireJs & Ui Module Removal Plan (Hyva + Nebula)

## Context

- **Frontend:** Hyva theme (Alpine.js, no RequireJS, no Luma UI components)
- **Backend:** Nebula admin theme (replaces Magento_Ui component rendering)

---

## 1. RequireJs Module (3 PHP files)

### What It Does

- `Block/Html/Head/Config.php` — renders RequireJS config in `<head>`
- `Model/FileManager.php` — generates `requirejs-config.js` bundles

### Coupling

| Layer | Count | Detail |
|-------|-------|--------|
| Module deps (composer) | 6 | Backend, Csp, Deploy, Email, Newsletter, Theme |
| Framework files | 5 | `Framework/RequireJs/Config.php`, `View/Page/Config/Renderer.php`, asset bundle system |
| `requirejs-config.js` files | 58 | Across all modules |

### Can It Be Removed?

**The module (3 files): YES** — disable it and the `requirejs-config.js` block stops rendering in `<head>`. Hyva already does this at the theme level.

**The Framework layer (5 files): needs cleanup.** `Framework/View/Page/Config/Renderer.php` has RequireJS bundle rendering logic hardcoded. With Hyva this code path simply returns empty output, but the classes still exist.

**The 58 `requirejs-config.js` files across modules:** Dead code with Hyva. They do nothing. Can be ignored (they're JS files, not PHP — no loading cost).

### Removal Plan

```bash
bin/magento module:disable Magento_RequireJs
```

That's it for the module. The 6 dependent modules will still work — they only `require` it in composer.json but don't import PHP classes from it (except Newsletter which just passes it to `require` for admin WYSIWYG — Nebula replaces this).

For a full cleanup, also remove RequireJS logic from `Framework/View/Page/Config/Renderer.php` — but this is optional since Hyva bypasses it anyway.

---

## 2. Ui Module (180 PHP files) — The Big One

### What It Provides

The entire admin UI component framework:
- Admin grids (product grid, order grid, customer grid, etc.)
- Admin forms (product edit, category edit, CMS page edit, etc.)
- Mass actions, filters, columns, field types
- Bookmarks (saved grid state)
- Data providers + modifiers
- CSV/XML export
- WYSIWYG editor config

### Coupling Scale

| Metric | Count |
|--------|-------|
| Modules that depend on Ui (composer) | **44** |
| Unique Ui classes imported externally | **129** |
| Total PHP import lines across codebase | **553** |
| `ui_component/*.xml` definitions across modules | **113** |
| Framework UiComponent files | **53** |

### Import Breakdown by Category

| Category | Import Lines | What |
|----------|-------------|------|
| `Component\*` | 333 | Columns, Forms, Fields, Filters, MassAction, etc. |
| `DataProvider\*` | 78 | AbstractDataProvider, Modifiers, EavValidationRules |
| `Config\*` | 65 | Converter, Reader, Definition |
| `Model\*` | 34 | Bookmark, Export, Manager |
| `Api\*` | 26 | BookmarkManagement, BookmarkRepository |
| `Controller\*` | 11 | Export actions, Render controllers |
| `Block\*` | 4 | StepsWizard, Wrapper |

### The 3 Layers to Replace

#### Layer 1: Framework UiComponent (53 files in `Framework/View/Element/UiComponent/`)

Core interfaces and base classes:
- `UiComponentInterface`, `ContextInterface`, `DataSourceInterface`
- `DataProvider/DataProvider.php`, `CollectionFactory.php`, `FilterPool.php`
- `Config/Reader.php`, `DomMerger.php` — ui_component XML parsing
- `Control/ButtonProviderInterface.php` — admin button rendering

**These are the contracts.** Nebula either implements these interfaces or provides its own.

#### Layer 2: Ui Module (180 files in `app/code/Magento/Ui/`)

Concrete implementations:
- 70+ `Component\*` classes (Column, Field, Fieldset, Filter types, etc.)
- 20+ `DataProvider\*` classes (modifiers, mappers, validation)
- 15+ `Config\*` classes (XML converters, readers)
- 15+ `Model\*` classes (Bookmark CRUD, export to CSV/XML)
- 10+ `Controller\*` classes (AJAX render, export, bookmark save)

**These are what Nebula replaces.** If Nebula provides grid/form rendering, these become dead code.

#### Layer 3: Per-Module ui_component XML (113 files)

Each module defines its admin grids and forms:
```
Catalog/view/adminhtml/ui_component/product_listing.xml
Sales/view/adminhtml/ui_component/sales_order_grid.xml
Customer/view/adminhtml/ui_component/customer_listing.xml
...
```

**These are consumed by the Ui module's XML reader.** If Nebula reads them too, they stay. If Nebula has its own definition format, they become dead code.

### Can It Be Removed?

**If Nebula fully replaces the admin UI component rendering:** YES, but it requires:

#### Step 1: Provide Stub/Noop Implementations for Critical Interfaces

44 modules import 129 Ui classes. These modules won't compile if the classes vanish. Options:

**Option A: Keep Ui as empty stubs** — replace the 129 imported classes with no-op implementations that satisfy type-hints but do nothing. The Ui module becomes a thin compatibility shim (~129 one-liner classes).

**Option B: Nebula provides the same namespaces** — if Nebula's module registers classes under `Magento\Ui\*` namespace, everything keeps working. The module is "replaced" not "removed."

**Option C: Gradual migration** — Each module's DataProviders, Modifiers, and Column classes get refactored to use Nebula's interfaces over time. Long-term clean, but massive effort (553 import lines across 44 modules).

#### Step 2: Handle Framework UiComponent Layer (53 files)

`Framework/View/Element/UiComponent/` provides the interfaces that both Ui and Nebula implement. **Keep these** — they're contracts, not implementations. They have zero cost if not used.

`Framework/View/Layout/Generator/UiComponent.php` and `Reader/UiComponent.php` handle `<uiComponent>` tags in layout XML. If Nebula uses a different rendering path, these can be stubbed out.

#### Step 3: Handle 113 ui_component XML Files

If Nebula reads them: keep them.
If Nebula ignores them: they're dead XML. No cost (only loaded on demand by the Ui config reader which wouldn't be running).

### Recommended Approach

**Option B is the pragmatic winner** — Nebula provides a `magento/module-ui` replacement package that:
1. Registers under the same `Magento\Ui\*` namespace (composer `replace`)
2. Provides the 129 classes that other modules import
3. Implements them using Nebula's rendering instead of the vanilla JS-heavy Ui components
4. Keeps `BookmarkManagement` working (grid state saving is useful regardless of renderer)

```json
{
    "name": "nebula/module-ui",
    "replace": {
        "magento/module-ui": "*"
    }
}
```

This way: zero changes in the 44 dependent modules. They still `use Magento\Ui\Component\Listing\Columns\Column` — but the class they get is Nebula's implementation.

---

## Summary

| Module | Files | Can Disable? | Approach |
|--------|-------|-------------|----------|
| **RequireJs** | 3 | **Yes — today** | `module:disable`. Hyva already bypasses it |
| **Ui** | 180 + 53 Framework | **Yes — via composer `replace`** | Nebula provides replacement package under same namespace |

### The Numbers If Both Are Removed

| What | Count |
|------|-------|
| PHP files eliminated (Ui module) | 180 |
| PHP files eliminated (RequireJs module) | 3 |
| Framework files made dead code | 53 (UiComponent) + 5 (RequireJs) |
| Admin JS eliminated | Entire `Magento_Ui/view/base/web/js/` (~200+ JS files) |
| ui_component XML files (dead code) | 113 |
| requirejs-config.js files (dead code) | 58 |
| **Total PHP files** | **~236** |
| **Total JS/XML dead code** | **~371 files** |

Combined with Hyva eliminating Luma's frontend JS, this removes the two heaviest asset-loading systems in Magento.
