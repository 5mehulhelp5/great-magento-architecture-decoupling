# Swatches & SendFriend Decoupling Report

---

## 1. Swatches (Color/Image Swatches)

### Module Family (47 files total)

| Module | Files | Role |
|--------|-------|------|
| `Swatches` | 40 | Core — swatch attribute rendering, admin UI, product images |
| `SwatchesGraphQl` | 6 | GraphQL resolvers for swatch data |
| `SwatchesLayeredNavigation` | 1 | **Empty module** — only `registration.php`, zero logic |

### Inbound Coupling

| Module | Type | Detail |
|--------|------|--------|
| **Weee** | composer require + XML | Plugin on `Swatches\Block\Product\Renderer\Listing\Configurable` in `Weee/etc/frontend/di.xml` |

**That's it.** 1 external module. Zero PHP imports from outside.

The Weee plugin adds WEEE/FPT data to swatch listing JSON config — the same plugin that also targets `ConfigurableProduct\Block\...\Configurable` (already documented in `configurable_product.md`).

### Swatches' Own Dependencies

Requires: Backend, Catalog, Config, **ConfigurableProduct**, Customer, Eav, PageCache, MediaStorage, Store, Theme
Suggests: LayeredNavigation

**Note:** Swatches has a hard dependency on ConfigurableProduct (already flagged in `configurable_product.md` as the tightest product-type coupling — 11 PHP imports + 3 XML refs). This is architectural: swatches are fundamentally about configurable product options. The coupling is legitimate.

### SwatchesLayeredNavigation

**This module is empty.** It contains only `registration.php` and `composer.json`. No PHP classes, no XML config, no templates. It requires only Framework. It's effectively a placeholder — can be safely removed with zero impact.

### Decoupling Fix

**Weee → Swatches:** The plugin in `Weee/etc/frontend/di.xml` targeting `Swatches\Block\...\Configurable` is already documented. It should move to Weee's `suggest` list (it's the same WEEE FPT plugin pattern — functional when Swatches is present, safely ignored when absent since DI skips plugins on non-existent classes).

Remove `magento/module-swatches` from Weee's composer.json `require` (it's already in `suggest`).

**SwatchesLayeredNavigation:** Remove entirely — it's an empty shell.

### Verdict

**Swatches is already near-fully decoupled.** 1 external plugin from Weee (already in suggest). The empty SwatchesLayeredNavigation module can be deleted.

---

## 2. SendFriend (Email to a Friend)

### Module Family (17 files total)

| Module | Files | Role |
|--------|-------|------|
| `SendFriend` | 12 | Core — "Email to a Friend" form on product page, rate limiting, admin config |
| `SendFriendGraphQl` | 5 | GraphQL mutation for sending product to friend |

### Inbound Coupling

| Direction | Count |
|-----------|-------|
| Modules depending on it (composer) | **0** |
| PHP imports from outside | **0** |
| XML refs from outside | **0** |

**Zero external coupling.** Nobody depends on SendFriend.

### SendFriend's Own Dependencies

Requires: Framework, Catalog, Customer, Store, Captcha, Authorization, Theme

Lightweight and appropriate — it needs Catalog (products), Customer (sender identity), Captcha (spam prevention).

### Verdict

**SendFriend is already fully decoupled.** Zero inbound coupling. Both modules can be disabled/removed via `module:disable` or composer remove with zero impact.

Most stores disable this feature (it's a legacy email-sharing feature that's been largely replaced by social sharing). Safe to remove entirely.

---

## Summary

| Module | Files | Inbound Coupling | Effort to Decouple | Can Remove? |
|--------|-------|-----------------|---------------------|-------------|
| **Swatches** | 40 | 1 (Weee plugin — already suggest) | Trivial — remove Weee hard require | Yes |
| **SwatchesGraphQl** | 6 | 0 | None | Yes |
| **SwatchesLayeredNavigation** | 1 | 0 | None — **empty module, delete it** | Yes — it's empty |
| **SendFriend** | 12 | 0 | None | **Yes — already clean** |
| **SendFriendGraphQl** | 5 | 0 | None | **Yes — already clean** |
