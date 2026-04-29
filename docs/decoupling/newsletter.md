# Newsletter Module Decoupling Report

## Current State

Newsletter is a **96-file module** that only 3 external modules depend on. The coupling is concentrated almost entirely in the **Customer module**.

### Newsletter's Own Dependencies

- composer.json requires: Backend, Cms, Customer, Eav, Email, RequireJs, Store, Widget, Ui
- module.xml sequence: Store, Customer, Eav, Widget
- **Newsletter → Customer is correct** (newsletter needs customers)

### External Modules Depending on Newsletter

| Module | composer.json | PHP imports | XML | Severity |
|--------|--------------|-------------|-----|----------|
| **Customer** | hard require | 11 files | layout, templates, UI component | **Heavy** |
| **CustomerGraphQl** | hard require + sequence | 3 files | — | Moderate |
| **Review** | hard require | 1 file (inherited constructor param) | — | Phantom |

---

## The Problem: Customer → Newsletter Coupling

The coupling is **bidirectional**:
- **Newsletter → Customer** (correct): Newsletter needs Customer to manage subscriptions
- **Customer → Newsletter** (wrong): Customer admin/frontend blocks directly import Newsletter models

### Customer Module's 11 Newsletter Import Points

**Admin blocks (4 files):**
- `Block/Adminhtml/Edit/Tab/Newsletter.php` — entire admin tab for newsletter subscription
- `Block/Adminhtml/Edit/Tab/Newsletter/Grid.php` — queue grid in admin customer edit
- `Block/Adminhtml/Edit/Tab/Newsletter/Grid/Filter/Status.php` — status filter

**Frontend blocks (3 files):**
- `Block/Form/Register.php` — uses `Newsletter\Model\Config` to check if newsletter enabled
- `Block/Account/Dashboard.php` — uses `Subscriber`, `SubscriberFactory`
- `Block/Account/Dashboard/Info.php` — uses `Subscriber`, `SubscriberFactory`

**Admin controllers (4 files):**
- `Controller/Adminhtml/Index/Save.php` — `SubscriberFactory`, `SubscriptionManagerInterface`
- `Controller/Adminhtml/Index/MassSubscribe.php` — `SubscriptionManagerInterface`
- `Controller/Adminhtml/Index/MassUnsubscribe.php` — `SubscriptionManagerInterface`
- `Controller/Adminhtml/Index/Cart.php` — `SubscriberFactory`

**Frontend controller (1 file):**
- `Controller/Account/CreatePost.php` — `SubscriberFactory`

**Templates (2 files):**
- `view/frontend/templates/form/newsletter.phtml` — subscribe checkbox on registration
- `view/adminhtml/templates/tab/newsletter.phtml` — admin newsletter tab

**Layout (1 file):**
- `view/adminhtml/layout/customer_index_newsletter.xml` — admin newsletter handle

### Newsletter's Plugin INTO Customer (the reverse hook)

`Newsletter/Model/Plugin/CustomerPlugin.php` hooks into `CustomerRepositoryInterface`:
- `afterSave()` — syncs subscription when customer is saved
- `aroundDeleteById()` — deletes subscriptions when customer is deleted
- `afterDelete()` — same, different delete path
- `afterGetById()` — adds `is_subscribed` extension attribute
- `afterGetList()` — adds `is_subscribed` to customer list results

`Newsletter/etc/extension_attributes.xml` adds `is_subscribed` boolean to `CustomerInterface`.

---

## Decoupling Plan

### Phase 1: Break Customer → Newsletter (Move newsletter UI to Newsletter module)

The Customer module should not import anything from Newsletter. All newsletter-related UI that currently lives in Customer should move to Newsletter (where it belongs).

**Move FROM `Customer/` TO `Newsletter/`:**

| Current Location (Customer) | New Location (Newsletter) |
|-----------------------------|---------------------------|
| `Block/Adminhtml/Edit/Tab/Newsletter.php` | `Block/Adminhtml/Customer/Tab/Newsletter.php` |
| `Block/Adminhtml/Edit/Tab/Newsletter/Grid.php` | `Block/Adminhtml/Customer/Tab/Newsletter/Grid.php` |
| `Block/Adminhtml/Edit/Tab/Newsletter/Grid/Filter/Status.php` | `Block/Adminhtml/Customer/Tab/Newsletter/Grid/Filter/Status.php` |
| `view/adminhtml/templates/tab/newsletter.phtml` | Keep in Newsletter (already has admin templates) |
| `view/adminhtml/layout/customer_index_newsletter.xml` | `Newsletter/view/adminhtml/layout/customer_index_newsletter.xml` |

**Refactor in Customer module (make Newsletter optional):**

| File | Change |
|------|--------|
| `Block/Form/Register.php` | Remove `Newsletter\Model\Config` import. Use a config value check instead (`config/newsletter/active`) or make the newsletter form block a separate child block contributed by Newsletter module via layout XML |
| `Block/Account/Dashboard.php` | Remove `SubscriberFactory` from constructor. Newsletter module contributes subscription status via its existing plugin on `CustomerRepositoryInterface` (the `is_subscribed` extension attribute) |
| `Block/Account/Dashboard/Info.php` | Same — read `is_subscribed` from customer extension attributes instead of direct Subscriber model |
| `Controller/Account/CreatePost.php` | Remove `SubscriberFactory`. Dispatch an event (`customer_register_success` already exists) — Newsletter observes it and handles subscription |
| `Controller/Adminhtml/Index/Save.php` | Remove `SubscriberFactory` and `SubscriptionManagerInterface`. Newsletter's existing `CustomerPlugin::afterSave()` already handles this via the `is_subscribed` extension attribute |
| `Controller/Adminhtml/Index/MassSubscribe.php` | Move to Newsletter module as `Newsletter/Controller/Adminhtml/Customer/MassSubscribe.php` |
| `Controller/Adminhtml/Index/MassUnsubscribe.php` | Move to Newsletter module as `Newsletter/Controller/Adminhtml/Customer/MassUnsubscribe.php` |
| `Controller/Adminhtml/Index/Cart.php` | Remove `SubscriberFactory` — it's only passed to parent constructor, likely unused here |

**Frontend templates:**
- `form/newsletter.phtml` — move to Newsletter module, contribute via layout XML handle `customer_account_create`
- `account/dashboard/info.phtml` — the newsletter status line can read from customer's `is_subscribed` extension attribute (set by Newsletter's plugin) instead of directly loading Subscriber model

### Phase 2: Clean Up composer.json Dependencies

**After Phase 1:**

| Module | Change |
|--------|--------|
| `Customer/composer.json` | Remove `magento/module-newsletter` from `require` |
| `CustomerGraphQl/composer.json` | Move `magento/module-newsletter` from `require` to `suggest`; make subscription logic conditional |
| `Review/composer.json` | Remove `magento/module-newsletter` from `require` (phantom — only inherited constructor param) |

### Phase 3: Fix Review's Inherited Dependency

`Review/Block/Customer/ListCustomer.php` extends `Customer/Block/Account/Dashboard` which currently takes `SubscriberFactory` in its constructor. After Phase 1 removes `SubscriberFactory` from Dashboard's constructor, Review's dependency disappears automatically.

---

## How Newsletter Already Does It Right

Newsletter's `CustomerPlugin` is actually the **correct pattern** — it hooks into CustomerRepository via plugins and extension attributes. The problem is that the Customer module ALSO reaches back into Newsletter directly, creating the circular dependency.

After decoupling:
- **Newsletter → Customer**: via plugin on `CustomerRepositoryInterface` (keeps working)
- **Newsletter → Customer UI**: newsletter admin tabs, registration checkbox, dashboard status — all contributed by Newsletter module via layout XML
- **Customer → Newsletter**: **zero dependency** — Customer only sees the `is_subscribed` extension attribute, which is framework-level (`extension_attributes.xml`)

### Architecture After Decoupling

```
Customer module (no Newsletter imports)
    │
    ├── extension_attributes: is_subscribed (boolean)  ← set by Newsletter plugin
    ├── events: customer_register_success              ← observed by Newsletter
    └── layout handles: customer_account_create etc.   ← Newsletter adds blocks via XML
         
Newsletter module (depends on Customer)
    │
    ├── Plugin/CustomerPlugin.php   → hooks CustomerRepositoryInterface
    ├── Observer/CustomerRegister   → handles subscription on registration
    ├── Block/Adminhtml/Customer/*  → admin UI (moved from Customer)
    ├── layout XML contributions    → adds newsletter blocks to Customer pages
    └── extension_attributes.xml    → declares is_subscribed on CustomerInterface
```

If Newsletter is disabled/removed:
- `is_subscribed` extension attribute simply doesn't exist
- No newsletter blocks appear on registration or dashboard
- No newsletter admin tab
- Customer module works perfectly without it

---

## Difficulty Assessment

| Phase | Effort | Risk |
|-------|--------|------|
| Phase 1: Move blocks/controllers | Medium | Low — layout XML handles stay the same, just different module provides them |
| Phase 2: Remove composer deps | Low | Low — after Phase 1, no code references remain |
| Phase 3: Review cleanup | Trivial | Zero — automatic after Dashboard constructor changes |

**Total: Medium effort, low risk, high value** — makes Newsletter fully optional.
