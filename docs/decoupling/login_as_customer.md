# Login As Customer (LAC) Decoupling Report

## Modules Scanned (10 modules, 95 PHP files total)

| Module | Files | Role |
|--------|-------|------|
| `LoginAsCustomerApi` | 17 | API interfaces + service contracts (only depends on Framework) |
| `LoginAsCustomer` | 20 | Core logic — authentication, admin login action |
| `LoginAsCustomerAdminUi` | 8 | Admin UI — buttons, columns in customer/order grid |
| `LoginAsCustomerAssistance` | 18 | Shopping assistance consent + admin notices |
| `LoginAsCustomerFrontendUi` | 6 | Frontend UI — confirmation popup, notifications |
| `LoginAsCustomerGraphQl` | 5 | GraphQL resolvers for LAC |
| `LoginAsCustomerLog` | 14 | Login logging + admin log grid |
| `LoginAsCustomerPageCache` | 2 | Page cache context handling for LAC sessions |
| `LoginAsCustomerQuote` | 2 | Quote cleanup on LAC session start |
| `LoginAsCustomerSales` | 3 | Order notes — marks orders placed during LAC |

## Can It Be Removed Entirely? YES — trivially

**Only 1 external module has any dependency on LAC.** Zero PHP imports exist outside the LAC family.

---

## External Coupling

### Persistent Module — 1 plugin, 1 PHP import, 1 composer dep

| Type | Detail |
|------|--------|
| `Persistent/composer.json` | `"magento/module-login-as-customer-api": "*"` (hard require) |
| `Persistent/etc/frontend/di.xml:61` | Plugin on `LoginAsCustomerApi\Api\AuthenticateCustomerBySecretInterface` |
| `Persistent/Model/Plugin/LoginAsCustomerCleanUp.php` | `use Magento\LoginAsCustomerApi\Api\AuthenticateCustomerBySecretInterface` |

**What it does:** When an admin logs in as a customer, Persistent clears the persistent cookie to prevent session confusion.

**Fix:** Move this plugin from Persistent into one of the LAC modules (e.g., `LoginAsCustomer` or a new `LoginAsCustomerPersistent` bridge). The plugin is about LAC behavior, not Persistent behavior — it belongs in the LAC module family. Persistent then drops its LAC dependency.

**That's it.** No other external module references LAC in any way.

---

## Internal Architecture (Already Well-Designed)

The 10 modules follow a clean layered pattern:

```
LoginAsCustomerApi (pure interfaces — depends only on Framework)
    ↑
    │ All other LAC modules depend on Api
    │
LoginAsCustomer (core logic)
    ↑
    ├── LoginAsCustomerAdminUi (admin buttons/grid columns)
    ├── LoginAsCustomerAssistance (consent management)
    ├── LoginAsCustomerFrontendUi (frontend popup/notifications)
    ├── LoginAsCustomerGraphQl (GraphQL layer)
    ├── LoginAsCustomerLog (audit logging)
    ├── LoginAsCustomerPageCache (cache context)
    ├── LoginAsCustomerQuote (quote cleanup)
    └── LoginAsCustomerSales (order annotation)
```

**Key design decisions done right:**
- `LoginAsCustomerApi` has **zero module dependencies** (only Framework) — the perfect API module
- Each sub-module is single-responsibility (PageCache handles caching, Quote handles quotes, Sales handles orders)
- Sub-modules only depend on `LoginAsCustomerApi` (interface layer), not on each other (except AdminUi→FrontendUi and Assistance→AdminUi)
- Every sub-module can be disabled independently

### Dependencies on Core Modules

| LAC Module | Core Module Dependencies |
|------------|-------------------------|
| Api | Framework only |
| Core | Backend, Customer, User, Authorization |
| AdminUi | Backend, Customer, Sales, Store |
| Assistance | Authorization, Backend, Customer, Store |
| FrontendUi | Customer, Store, Checkout |
| GraphQl | Integration, Store, Customer |
| Log | Backend, Customer, Ui, User |
| PageCache | Store, PageCache |
| Quote | Checkout, Customer, Quote |
| Sales | Backend, User, Sales |

All core dependencies are appropriate — each LAC module depends on exactly the modules it interacts with.

---

## Decoupling Plan

### Phase 1: Move Persistent Plugin to LAC (only change needed)

Move FROM `Persistent/` TO `LoginAsCustomer/` (or `LoginAsCustomerApi/`):

| Current | New |
|---------|-----|
| `Persistent/Model/Plugin/LoginAsCustomerCleanUp.php` | `LoginAsCustomer/Plugin/Persistent/CleanUp.php` |
| `Persistent/etc/frontend/di.xml` (plugin registration) | `LoginAsCustomer/etc/frontend/di.xml` |

Add `magento/module-persistent` to `LoginAsCustomer/composer.json` as `suggest` (plugin only activates when Persistent is installed).

Remove `magento/module-login-as-customer-api` from `Persistent/composer.json`.

### Phase 2: Verify Clean Removal

After Phase 1, disabling all 10 LAC modules means:
- No "Login as Customer" button in admin customer/order grids
- No shopping assistance consent
- No LAC session handling or logging
- Persistent, Customer, Sales, Quote, Checkout — zero impact
- **All 10 modules can be removed via composer with zero side effects**

---

## Difficulty Assessment

| Phase | Effort | Risk |
|-------|--------|------|
| Move 1 plugin + 1 DI registration | Trivial | Zero |
| **Overall** | **Trivial** | **LAC is already near-perfect** |

## Verdict

**LoginAsCustomer is the second-best-designed module family in Magento** (after ApplicationPerformanceMonitor). Clean API separation, single-responsibility sub-modules, minimal external coupling (1 plugin in 1 module). The only fix is a single plugin relocation.

This should be the reference architecture for how to build optional Magento feature modules.
