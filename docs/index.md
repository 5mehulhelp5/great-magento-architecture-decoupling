# GMAD: Great Magento Architecture Decoupling

<div style="margin: 1.5rem 0; border-radius: 12px; overflow: hidden; box-shadow: 0 4px 20px rgba(0,0,0,0.15);">
  <img src="gmad_jakub.png" alt="GMAD - Great Magento Architecture Decoupling" style="width: 100%; display: block;">
</div>

<div style="text-align: center; margin: 1.5rem 0;">
  <p style="font-size: 1.3em; font-style: italic; font-weight: 500;">"I'm Jakub, from qoliber. Let's fix Magento — nobody else is going to do it."</p>
  <p><strong><a href="https://www.linkedin.com/in/jakubwinkler/">Jakub Winkler</a></strong> — The MAD man behind the GMAD project, Founder of <a href="https://qoliber.com">qoliber</a> | <a href="https://www.linkedin.com/in/jakubwinkler/">Connect on LinkedIn :fontawesome-brands-linkedin:</a></p>
</div>

**The open-source project to decouple, modernize, and slim down Magento 2.**

[Contribute on GitHub :fontawesome-brands-github:](https://github.com/qoliber/great-magento-architecture-decoupling){ .md-button .md-button--primary }

!!! tip "Want to contribute?"
    This is a community project. All discussions, ideas, and contributions happen on GitHub. **[Open an Issue](https://github.com/qoliber/great-magento-architecture-decoupling/issues)** to discuss, or **[Submit a Pull Request](https://github.com/qoliber/great-magento-architecture-decoupling/pulls)** with your changes. Pick any workstream from the [Implementation Plan](IMPLEMENTATION_PLAN.md) and start.

---

## The Problem

Magento 2 ships with **221 modules** — and that's without MSI! Most stores use less than half of them. But disabling unused modules isn't straightforward — decades of coupling mean that removing one module can break another that shouldn't care about it at all.

Newsletter code in the Customer module. Wishlist controllers in Sales. Payment depending on Sales which depends on Payment. Product type checks hardcoded into Catalog. CatalogInventory wired into 24 modules. 

**This project fixes that.**

## The Numbers

| Metric | Count |
|--------|-------|
| Total modules scanned | **221 / 221 (100%)** |
| Modules that can be removed | **~121 (55%)** |
| Removable PHP files | **~1,470** |
| Circular dependencies found | **2** (Sales-Payment, QuoteGraphQl-SalesGraphQl) |
| Phantom composer deps found & fixed | **8** |
| Parallel workstreams for developers | **12** |

## What We've Done

### Scanned Every Module

Every one of Magento's 221 modules has been analyzed for:

- Inbound coupling (who depends on this module?)
- Outbound coupling (what does this module import?)
- Phantom dependencies (composer.json `require` with zero code usage)
- Constants leaking across module boundaries
- Hardcoded type checks that should be plugins
- DI configurations in the wrong module

### Identified 4 Decoupling Patterns

From the completed [PR #1](https://github.com/qoliber/great-magento-architecture-decoupling), we established 4 reusable patterns that apply across the entire codebase:

**Pattern A: Hardcoded Type Check to Plugin** — Remove `switch/case` product type checks from core modules; each product type registers its own plugin.

**Pattern B: Hardcoded DI Array to Self-Registration** — Core modules define empty arrays; product types add themselves via DI merging.

**Pattern C: Hard Dependency to Nullable Constructor** — Make dependencies optional with `?Type $dep = null` and null-checks.

**Pattern D: Move DI Config to Correct Module** — When Module A configures Module B's classes, move the config to where it belongs.

### Created the Implementation Plan

[**12 independent workstreams, 46 tasks**](IMPLEMENTATION_PLAN.md) — all with step-by-step instructions, file paths, before/after code, and the exact pattern to follow. 10 of 12 can start Day 1 with no dependencies between them.

### Built the Module Disable Guide

[**3 tiers of modules to disable**](disable_groups.md) with copy-paste `bin/magento module:disable` commands:

- **Tier 1:** 48 modules — disable today, zero risk
- **Tier 2:** 35 modules — disable today, minor admin side effects
- **Tier 3:** 12 modules — disable after decoupling work

### Mapped Framework Modernization (PHP 8.4)

- [**206 Proxy classes**](magento_framework_modernization.md) replaceable with native lazy objects
- **99 constant-bag classes** convertible to PHP 8.1 enums
- **401 `@deprecated` annotations** upgradable to `#[\Deprecated]` attribute
- [**6 View/Layout proposals**](magento_view_modernization.md) including lazy block instantiation and compiled layout cache

## How to Contribute

1. **Pick a workstream** from the [Implementation Plan](IMPLEMENTATION_PLAN.md)
2. **Follow the patterns** established in [PR #1](https://github.com/qoliber/great-magento-architecture-decoupling)
3. **Submit a PR** with your changes
4. **Run the verification checklist** at the bottom of the Implementation Plan

Every workstream is independent. Multiple developers can work simultaneously without conflicts.

## Documentation

### Decoupling Reports

Each report covers: what the module does, who depends on it, who it depends on, the exact coupling points, and a step-by-step decoupling plan.

| Report | Modules | Key Finding |
|--------|---------|-------------|
| [Configurable Product](decoupling/configurable_product.md) | 4 modules | `TYPE_CODE` constant used by 5 external modules |
| [Downloadable Product](decoupling/downloadable_product.md) | 3 modules | GiftMessageGraphQl coupling fixed |
| [CatalogInventory](decoupling/catalog_inventory.md) | 24 modules, 83 PHP imports | 5 phantom deps removed. Identical code in 3 product types |
| [Sales / Shipping / Payment](decoupling/sales_shipping_payment.md) | 16 modules | Circular dependency: Sales-Payment |
| [Payment-Sales Circular](decoupling/payment_sales_circular.md) | 2 modules | 3-phase fix: move adapters, observers, cleanup |
| [Newsletter](decoupling/newsletter.md) | 2 modules | Customer has 11 Newsletter imports — all movable |
| [Review](decoupling/review.md) | 3 modules | Catalog already has null-object pattern. Move 16 report files |
| [Wishlist](decoupling/wishlist.md) | 3 modules | Zero PHP imports from outside — all XML/template coupling |
| [Weee (FPT)](decoupling/weee.md) | 2 modules | Zero inbound coupling. Already optional |
| [Cron & Message Queue](decoupling/cron_and_message_queue.md) | 5 modules | MQ transports fully decoupled. Proposed `Async\PublisherInterface` |
| [Login As Customer](decoupling/login_as_customer.md) | 10 modules | Near-perfect. 1 plugin to relocate |
| [MSRP / Multishipping / MysqlMq](decoupling/msrp_multishipping_mysqlmq.md) | 7 modules | MSRP: 3 modules coupled. Multishipping: 2 plugins to move |
| [Media Modules](decoupling/media_modules.md) | 25 modules | 24 modules (288 files) fully optional. MediaStorage essential |
| [RequireJs & Ui Removal](decoupling/requirejs_ui_removal.md) | 2 modules | Ui: 44 modules, 553 imports. Solvable via composer `replace` |
| [APM](decoupling/application_performance_monitor.md) | 2 modules | Perfectly decoupled. Zero coupling. Reference architecture |
| [Swatches / SendFriend](decoupling/swatches_sendfriend.md) | 5 modules | SendFriend: zero coupling. SwatchesLayeredNavigation: empty module |
| [Misc](decoupling/related_product_graphql_release_notification.md) | 2 modules | Both clean, trivial fixes |

### Framework Modernization

| Report | Key Proposals |
|--------|--------------|
| [PHP 8.4 Opportunities](magento_framework_modernization.md) | Lazy objects, enums, property hooks, deprecated attribute, component replacement |
| [View/Layout System](magento_view_modernization.md) | Lazy blocks, compiled layout cache, template maps, asset pipeline extraction |

## Project Info

**Created by:** [Jakub Winkler](https://www.linkedin.com/in/jakubwinkler/) at [qoliber](https://qoliber.com)

**License:** Open source, same as Magento 2 (OSL-3.0 / AFL-3.0)

**Reference PR:** [github.com/qoliber/great-magento-architecture-decoupling](https://github.com/qoliber/great-magento-architecture-decoupling) — Decouples Catalog from Bundle, Configurable, Downloadable + makes CatalogInventory optional (77 PHP files, 6 XML files, 18 modules)
