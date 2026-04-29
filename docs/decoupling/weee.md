# Weee (FPT) Module Decoupling Report

## Modules Scanned

- `Magento_Weee` — 50 PHP files
- `Magento_WeeeGraphQl` — 6 PHP files

## Can It Be Removed Entirely? ALMOST

**Zero external modules import from `Magento\Weee`** — no composer dependency, no PHP `use` statements, no XML references from any other module. This is remarkable for a tax-related module.

However, Weee data fields are **baked into core API interfaces** as constants and getter/setter methods. These are database columns that exist regardless of whether Weee is installed.

---

## Inbound Coupling (Who depends on Weee): NONE

| Type | Count |
|------|-------|
| composer.json `require` on `magento/module-weee` | **0** |
| PHP `use Magento\Weee` outside Weee modules | **0** |
| XML references to `Magento\Weee` outside Weee modules | **0** |
| module.xml sequence on `Magento_Weee` | **0** |

**Weee is already fully optional at the module level.** No module depends on it.

---

## Outbound Coupling (What Weee depends on)

Weee requires 14 modules:

```
Framework, Backend, Catalog, Checkout, ConfigurableProduct, Customer,
Directory, Eav, PageCache, Quote, Sales, Store, Tax, Ui
```

Suggests: Bundle, Swatches

This is heavy for a FPT module. The `ConfigurableProduct` dependency was already documented in `configurable_product.md` — Weee has 3 plugin classes targeting ConfigurableProduct.

---

## The Embedded Data Problem

While no module **imports** from Weee, several core API interfaces have Weee fields hardcoded:

### Sales OrderItemInterface — 10 WEEE constants + 20 getter/setter methods

```php
// In Sales\Api\Data\OrderItemInterface (NOT in Weee module)
const BASE_WEEE_TAX_APPLIED_AMOUNT = 'base_weee_tax_applied_amount';
const BASE_WEEE_TAX_APPLIED_ROW_AMNT = 'base_weee_tax_applied_row_amnt';
const WEEE_TAX_APPLIED_AMOUNT = 'weee_tax_applied_amount';
const WEEE_TAX_APPLIED_ROW_AMOUNT = 'weee_tax_applied_row_amount';
const WEEE_TAX_APPLIED = 'weee_tax_applied';
const WEEE_TAX_DISPOSITION = 'weee_tax_disposition';
const WEEE_TAX_ROW_DISPOSITION = 'weee_tax_row_disposition';
const BASE_WEEE_TAX_DISPOSITION = 'base_weee_tax_disposition';
const BASE_WEEE_TAX_ROW_DISPOSITION = 'base_weee_tax_row_disposition';
// + getBaseWeeeTaxAppliedAmount(), setBaseWeeeTaxAppliedAmount(), etc.
```

### Quote TotalsInterface — 1 WEEE constant + getter/setter

```php
// In Quote\Api\Data\TotalsInterface
const KEY_WEEE_TAX_APPLIED_AMOUNT = 'weee_tax_applied_amount';
```

### Quote/Sales Item Models

```php
// In Quote\Model\Quote\Item, Sales\Model\Order\Item, etc.
// Weee data stored as item attributes (weee_tax_applied, weee_tax_disposition, etc.)
```

### Tax Interfaces

```php
// In Tax\Api\Data\TaxDetailsItemInterface
// References "weee" as a tax item type in comments
```

### Tax Calculation

```php
// In Tax\Model\Calculation\RowBaseCalculator and UnitBaseCalculator
// Comments: "Use delta rounding of the product's instead of the weee's"
```

### Catalog Price Rendering

```php
// In Catalog\Api\Data\ProductRender\PriceInfoInterface
// Weee-related formatted price fields
```

---

## Decoupling Assessment

### What's Already Clean (no work needed)

- **Module-level coupling: zero.** Weee can be disabled today via `bin/magento module:disable Magento_Weee` and nothing breaks at the PHP level.
- **WeeeGraphQl** only depends on Weee — auto-disables when Weee is disabled.
- **Weee hooks into other modules via plugins and layout XML** — the correct pattern. When Weee is disabled, plugins don't load, layout blocks don't render.

### What's Messy (API contamination)

The WEEE constants and getter/setter methods in `OrderItemInterface`, `TotalsInterface`, `CreditmemoItemInterface`, etc. are **permanent API pollution**. They exist in core interfaces even when Weee is not installed. The database columns exist regardless.

This is a **design debt**, not a coupling problem. The fields were added directly to core interfaces instead of using extension attributes (the way Newsletter correctly adds `is_subscribed` to CustomerInterface).

### What Could Be Improved

#### Phase 1: Remove ConfigurableProduct Dependency (already documented)

As documented in `configurable_product.md`, Weee has 3 plugin classes + `Tax.php` model that import from ConfigurableProduct. These should be moved to a bridge module or made conditional. This reduces Weee's own dependency count from 14 to 13.

#### Phase 2: Move WEEE Fields to Extension Attributes (breaking change)

The proper pattern (matching how Newsletter handles `is_subscribed`):

```xml
<!-- In Weee/etc/extension_attributes.xml (NOT in Sales) -->
<extension_attributes for="Magento\Sales\Api\Data\OrderItemInterface">
    <attribute code="weee_tax_applied" type="string"/>
    <attribute code="weee_tax_applied_amount" type="float"/>
    <attribute code="weee_tax_disposition" type="float"/>
    <!-- etc. -->
</extension_attributes>
```

Remove the 10 constants and 20 getter/setter methods from `OrderItemInterface`. Same for `TotalsInterface`, `CreditmemoItemInterface`.

**This is a breaking API change** — any code calling `$orderItem->getWeeeTaxApplied()` would need to use `$orderItem->getExtensionAttributes()->getWeeeTaxApplied()` instead.

#### Phase 3: Clean Tax Calculation Comments

The "weee" references in Tax calculation classes are just code comments, not actual coupling. Low priority but clean to fix.

---

## Summary

| Aspect | Status |
|--------|--------|
| Module-level coupling (inbound) | **Zero — fully optional today** |
| Module-level coupling (outbound) | Heavy (14 deps), ConfigurableProduct is the worst |
| API contamination | 10 constants + 20 methods in Sales OrderItemInterface |
| API contamination | 1 constant + getter/setter in Quote TotalsInterface |
| Database columns | WEEE columns exist in sales tables regardless |
| Can be disabled? | **Yes — `module:disable` works cleanly** |
| Can be fully removed? | **Not without API breaking change** (embedded fields in core interfaces) |

### Difficulty

| Action | Effort | Breaking? |
|--------|--------|-----------|
| Disable Weee module | None | No — already works |
| Remove ConfigurableProduct dependency | Medium | No |
| Move WEEE fields to extension_attributes | High | **Yes — API break** |
| Remove WEEE database columns | Very High | **Yes — schema break** |

### Verdict

**Weee is already the best-decoupled tax module in Magento.** Zero inbound coupling. It can be disabled today with no side effects. The only debt is the embedded WEEE fields in core Sales/Quote/Tax API interfaces — that's a long-term cleanup requiring a major version bump.
