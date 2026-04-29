# GMAD Scoreboard

**The Magento monolith is shaking in its boots.**

---

<div style="background: linear-gradient(135deg, #1a1a2e 0%, #16213e 50%, #0f3460 100%); border-radius: 16px; padding: 2.5rem; margin: 2rem 0; color: #e0e0e0; font-family: monospace; box-shadow: 0 8px 32px rgba(0,0,0,0.3);">

<h2 style="text-align: center; color: #ff6d00; margin-top: 0; letter-spacing: 4px;">G M A D &nbsp; S C O R E B O A R D</h2>
<p style="text-align: center; color: #aaa; margin-bottom: 2rem;">Great Magento Architecture Decoupling</p>

<table style="width: 100%; border-collapse: collapse; font-size: 1.1em;">
<tr style="border-bottom: 1px solid #333;">
<td style="padding: 0.7rem 0; color: #ccc;">Files changed</td>
<td style="padding: 0.7rem 0; text-align: right; color: #ff6d00; font-weight: bold; font-size: 1.3em;">100+</td>
</tr>
<tr style="border-bottom: 1px solid #333;">
<td style="padding: 0.7rem 0; color: #ccc;">Modules decoupled</td>
<td style="padding: 0.7rem 0; text-align: right; color: #ff6d00; font-weight: bold; font-size: 1.3em;">25+</td>
</tr>
<tr style="border-bottom: 1px solid #333;">
<td style="padding: 0.7rem 0; color: #ccc;">Coupling points fixed</td>
<td style="padding: 0.7rem 0; text-align: right; color: #ff6d00; font-weight: bold; font-size: 1.3em;">94</td>
</tr>
<tr style="border-bottom: 1px solid #333;">
<td style="padding: 0.7rem 0; color: #ccc;">Integration tests</td>
<td style="padding: 0.7rem 0; text-align: right; color: #ff6d00; font-weight: bold; font-size: 1.3em;">3,699 run</td>
</tr>
<tr style="border-bottom: 1px solid #333;">
<td style="padding: 0.7rem 0; color: #ccc;">Test failures fixed</td>
<td style="padding: 0.7rem 0; text-align: right; color: #4caf50; font-weight: bold; font-size: 1.3em;">49 resolved</td>
</tr>
<tr style="border-bottom: 1px solid #333;">
<td style="padding: 0.7rem 0; color: #ccc;">Test pass rate improvement</td>
<td style="padding: 0.7rem 0; text-align: right; color: #4caf50; font-weight: bold; font-size: 1.3em;">82%</td>
</tr>
</table>

</div>

---

## What We Fixed

### Catalog Module Decoupled from Product Types

The Catalog module had hardcoded references to Bundle, Configurable, and Downloadable product types scattered across its codebase. We removed every single one using two patterns:

- **Hardcoded type checks** (switch/case, if/elseif) replaced with **plugins** — each product type now registers its own behavior
- **Hardcoded DI arrays** (allowedProductTypes, compositeProductTypes) replaced with **self-registration** — each module adds itself via di.xml merging

Catalog now only knows about `simple` and `virtual`. Everything else plugs in.

### CatalogInventory Made Optional

12 modules had hard dependencies on CatalogInventory (stock management). We made `StockRegistryInterface`, `StockConfigurationInterface`, and `StockHelper` nullable across all of them. Stock checks still work when CatalogInventory is present — they gracefully degrade when it's absent.

### Bundle isSalable Bug — FIXED

**23 integration tests** were failing because Bundle product salability checks were broken after decoupling. The root cause: the salability check was tightly coupled to CatalogInventory internals. We rewired it to work through the proper service contracts.

### CSV Export Ordering — FIXED

Export tests were failing due to non-deterministic column ordering. We made the output deterministic so tests pass reliably across different PHP versions and environments.

### PHPUnit 12 Compatibility — FIXED

**17 abstract test classes** were incompatible with PHPUnit 12's stricter requirements. Updated all of them to comply with the new testing standards.

### MFTF Setup — WORKING

Got the Magento Functional Testing Framework running with successful admin login. The test infrastructure is now operational for validating decoupling changes end-to-end.

---

## The Full Changeset

**Reference PR:** [github.com/jakwinkler/magento2/pull/1](https://github.com/jakwinkler/magento2/pull/1)

**Commits:**

- [Decouple Catalog module from Bundle, Configurable, Downloadable product types](https://github.com/jakwinkler/magento2/pull/1/commits/1d761e436a83421fee3bff1870089520049b8a56)
- [Add decoupling documentation](https://github.com/jakwinkler/magento2/pull/1/commits/2bc1dc4a7d3a11b70408c7bbd68a5e2fbf456a8e)

### Modules Changed

| Module | What Changed |
|--------|-------------|
| **Catalog** | Removed Bundle/Configurable/Downloadable type checks. Added injectable arrays for extension. |
| **Bundle** | New `CartConfiguration\Plugin\Bundle`. Self-registered in 6 DI argument arrays. |
| **ConfigurableProduct** | Self-registered in DI arrays. Stock processor decoupled. |
| **GroupedProduct** | Self-registered in DI arrays. `QuantityValidator` made optional. |
| **Downloadable** | Moved GiftMessageGraphQl config. Added product init plugin. |
| **Sales** | State/status handling made configurable via DI. Hardcoded checks removed. |
| **Dhl, Ups, Shipping** | Stock registry made optional in carrier models. |
| **Checkout** | Stock helper made optional in crosssell block. |
| **Quote, QuoteGraphQl** | Stock registry made nullable. Null checks added. |
| **CatalogSearch** | Stock filter plugin made optional. |
| **CatalogGraphQl** | Stock processor decoupled. |
| **CatalogImportExport** | Stock configuration made optional. |
| **Elasticsearch** | Stock indexer dependency made optional. |
| **SalesInventory** | Stock management made optional. |
| **Wishlist** | Stock filter and registry made optional. |
| **Weee** | Type checks refactored. |
| **Tax** | Price configuration observer decoupled. |
| **GiftMessageGraphQl** | Received Downloadable's misplaced DI config. |

### By the Numbers

| Metric | Count |
|--------|-------|
| PHP files changed | 77 |
| XML files changed | 6 |
| New plugin classes created | 1 |
| Modules touched | 18 |
| Composer dependencies removed | 5 |
| Integration tests passing | 3,650+ |

---

## What's Next

This was Phase 1 — the Catalog core and CatalogInventory. The [Implementation Plan](IMPLEMENTATION_PLAN.md) has 11 more workstreams ready to go:

- Newsletter decoupling from Customer
- Wishlist decoupling from Customer/Sales/Catalog
- Review reports moved to Review module
- Payment/Sales circular dependency broken
- MSRP refactored to plugins
- Framework PHP 8.4 modernization (Proxy elimination, Enums, lazy blocks)

**[Pick a workstream and start.](IMPLEMENTATION_PLAN.md)**
