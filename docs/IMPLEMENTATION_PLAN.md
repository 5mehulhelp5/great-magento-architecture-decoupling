# Magento 2 Decouplifier — Implementation Plan

**221 modules scanned. ~121 removable. 12 parallel workstreams for maximum developer throughput.**

**Reference PR:** https://github.com/jakwinkler/magento2/pull/1

---

## How to Read This Plan

- **Workstreams** are independent — multiple developers can work on different workstreams simultaneously
- **Tasks within a workstream** are sequential — complete them in order
- **Difficulty:** T = Trivial, E = Easy, M = Medium, H = Hard
- **Each task** lists: what to change, which files, the pattern to follow
- **Patterns** are proven — PR #1 already demonstrates each pattern with working code

---

## Completed Work (PR #1)

PR #1 decouples the Catalog module from Bundle, Configurable, Downloadable product types and makes CatalogInventory optional across multiple modules. **77 PHP files changed, 6 XML files changed, 1 new plugin created across 18 modules.**

### 4 Decoupling Patterns Used (Reference for All Future Work)

#### Pattern A: Hardcoded Type Check → Plugin

**Problem:** Catalog's `CartConfiguration` had a hardcoded `case TYPE_BUNDLE` check.

**Before** (`Catalog/Model/Product/CartConfiguration.php`):
```php
switch ($product->getTypeId()) {
    case \Magento\Catalog\Model\Product\Type::TYPE_SIMPLE:
    case \Magento\Catalog\Model\Product\Type::TYPE_VIRTUAL:
        return isset($config['options']);
    case \Magento\Catalog\Model\Product\Type::TYPE_BUNDLE:    // ← hardcoded
        return isset($config['bundle_option']);                // ← hardcoded
}
```

**After:** Remove the `bundle` case from Catalog. Bundle registers its own plugin:

```php
// Bundle/Model/Product/CartConfiguration/Plugin/Bundle.php (NEW FILE)
class Bundle
{
    public function aroundIsProductConfigured(
        CartConfiguration $subject,
        \Closure $proceed,
        Product $product,
        $config
    ) {
        if ($product->getTypeId() === Type::TYPE_CODE) {
            return isset($config['bundle_option']);
        }
        return $proceed($product, $config);
    }
}
```

```xml
<!-- Bundle/etc/di.xml (ADDED) -->
<type name="Magento\Catalog\Model\Product\CartConfiguration">
    <plugin name="bundle_product" type="Magento\Bundle\Model\Product\CartConfiguration\Plugin\Bundle" sortOrder="50" />
</type>
```

**Use this pattern when:** A core module has `switch/case` or `if/elseif` blocks checking product type IDs, order states, payment methods, etc. Each case moves to the responsible module as a plugin.

#### Pattern B: Hardcoded DI Array → Module Self-Registration

**Problem:** Catalog's `di.xml` listed specific product types in `allowedProductTypes` arrays.

**Before** (`Catalog/etc/di.xml`):
```xml
<type name="Magento\Catalog\Model\Product\Price\BasePriceStorage">
    <arguments>
        <argument name="allowedProductTypes" xsi:type="array">
            <item name="0" xsi:type="string">simple</item>
            <item name="1" xsi:type="string">virtual</item>
            <item name="2" xsi:type="string">bundle</item>    <!-- hardcoded -->
        </argument>
    </arguments>
</type>
```

**After:** Catalog defines only its own types. Bundle self-registers in its own `di.xml`:

```xml
<!-- Catalog/etc/di.xml — only core types -->
<argument name="allowedProductTypes" xsi:type="array">
    <item name="simple" xsi:type="string">simple</item>
    <item name="virtual" xsi:type="string">virtual</item>
</argument>

<!-- Bundle/etc/di.xml — adds itself -->
<type name="Magento\Catalog\Model\Product\Price\BasePriceStorage">
    <arguments>
        <argument name="allowedProductTypes" xsi:type="array">
            <item name="bundle" xsi:type="string">bundle</item>
        </argument>
    </arguments>
</type>
```

**Use this pattern when:** A core module's `di.xml` has arrays listing product types, entity types, or feature flags that belong to other modules. Each module adds its own entry via DI merging.

#### Pattern C: Hard Dependency → Nullable Constructor (Optional Dep)

**Problem:** `Quote/Model/Cart/AddProductsToCart.php` required `StockRegistryInterface` in constructor.

**Before:**
```php
public function __construct(
    // ...
    private readonly StockRegistryInterface $stockRegistry
) {}
```

**After:**
```php
public function __construct(
    // ...
    private readonly ?StockRegistryInterface $stockRegistry = null
) {}

// Usage: null-check before calling
if ($product && $this->stockRegistry !== null) {
    $stockItem = $this->stockRegistry->getStockItem(...);
}
```

**Use this pattern when:** A module uses another module's service for a secondary feature (e.g., stock check during add-to-cart). The primary functionality works without it; the feature degrades gracefully.

#### Pattern D: Move DI Config to Correct Module

**Problem:** `Downloadable/etc/di.xml` configured a class from `GiftMessageGraphQl`.

**Before** (`Downloadable/etc/di.xml`):
```xml
<type name="Magento\GiftMessageGraphQl\Model\Resolver\Product\GiftMessage">
    <arguments>
        <argument name="nonGiftMessageProductTypes" xsi:type="array">
            <item name="downloadable" xsi:type="string">downloadable</item>
        </argument>
    </arguments>
</type>
```

**After:** Removed from Downloadable, added to `GiftMessageGraphQl/etc/graphql/di.xml`.

**Use this pattern when:** Module A configures a class from Module B, but Module B depends on Module A (or should). The config belongs in the module that owns the class, or the module whose feature it is.

### Modules Changed in PR #1

| Module | What Changed | Pattern Used |
|--------|-------------|-------------|
| **Catalog** | Removed Bundle/Configurable/Downloadable type checks from `CartConfiguration`, `Option\SaveHandler`, `InvalidSkuProcessor`, `TierPriceValidator`, `CategorySetup`, `di.xml` arrays. Added empty injectable arrays for extension. | A, B |
| **Bundle** | Added `CartConfiguration\Plugin\Bundle`, self-registered in 6 DI argument arrays for pricing, reporting, options, EAV | A, B |
| **ConfigurableProduct** | Self-registered in DI arrays (`di.xml`), decoupled stock status processor | B, C |
| **GroupedProduct** | Self-registered in DI arrays, made `QuantityValidator` optional | B, C |
| **Downloadable** | Moved GiftMessageGraphQl DI config out, added own `Plugin/Downloadable` for product init | D, A |
| **CatalogInventory consumers** (12 modules) | Made `StockRegistryInterface`, `StockConfigurationInterface`, `StockHelper` nullable in constructors | C |
| **Sales** | Made `Order` state/status handling configurable via DI, removed hardcoded checks | B, C |
| **Dhl, Ups, Shipping** | Made stock registry optional in carrier models | C |
| **5 composer.json** | Removed phantom `magento/module-catalog-inventory` deps | Phantom removal |

---

## Workstream 1: Phantom Dependency Removal

**Developer count:** 1
**Difficulty:** T
**Prerequisite:** None

Remove `require` entries from `composer.json` where zero code references exist.

### Task 1.1: Already Completed in PR #1

`AdvancedPricingImportExport`, `Fedex`, `Reports`, `Shipping`, `Usps` — removed `magento/module-catalog-inventory`.

### Task 1.2: Remaining Phantom Dependencies

| Module | Dependency to Remove | Verify with |
|--------|---------------------|------------|
| `Usps/composer.json` | `magento/module-sales` | `grep -r "Magento\\\\Sales\|Magento.Sales" app/code/Magento/Usps/ --include="*.php"` returns nothing |
| `SalesRule/composer.json` | `magento/module-payment` | Zero PHP imports from Payment in production code |
| `SalesRule/composer.json` | `magento/module-shipping` | Zero PHP imports from Shipping in production code |
| `Reports/composer.json` | `magento/module-downloadable` | Only hardcoded string `'downloadable'` — not a class dep |
| `Msrp/composer.json` | `magento/module-downloadable` | Already removed in PR #1 |

---

## Workstream 2: Plugin Relocation

**Developer count:** 1-2
**Difficulty:** E
**Prerequisite:** None

**Pattern used:** D (Move DI Config to Correct Module)

### Task 2.1: LoginAsCustomer — Move Persistent Plugin

| Step | Action |
|------|--------|
| 1 | Copy `Persistent/Model/Plugin/LoginAsCustomerCleanUp.php` → `LoginAsCustomer/Plugin/Persistent/CleanUp.php` |
| 2 | Update namespace in new file |
| 3 | Move plugin registration from `Persistent/etc/frontend/di.xml` to `LoginAsCustomer/etc/frontend/di.xml` |
| 4 | Remove old file and DI registration from Persistent |
| 5 | Remove `magento/module-login-as-customer-api` from `Persistent/composer.json` |
| 6 | Add `magento/module-persistent` to `LoginAsCustomer/composer.json` `suggest` |
| 7 | Run `bin/magento setup:di:compile` |

### Task 2.2: Multishipping — Move GiftMessage Plugins (2 files)

| Step | Action |
|------|--------|
| 1 | Copy `GiftMessage/Block/Message/Multishipping/Plugin/ItemsBox.php` → `Multishipping/Plugin/GiftMessage/ItemsBox.php` |
| 2 | Copy `GiftMessage/Model/Type/Plugin/Multishipping.php` → `Multishipping/Plugin/GiftMessage/TypePlugin.php` |
| 3 | Update namespaces in new files |
| 4 | Move 2 plugin registrations from `GiftMessage/etc/frontend/di.xml` to `Multishipping/etc/frontend/di.xml` |
| 5 | Remove old files and DI registrations from GiftMessage |
| 6 | Remove `magento/module-multishipping` from `GiftMessage/composer.json` |
| 7 | Add `magento/module-gift-message` to `Multishipping/composer.json` `suggest` |

### Task 2.3: Multishipping — Move SalesRule Plugin (1 file)

| Step | Action |
|------|--------|
| 1 | Copy `SalesRule/Plugin/CouponUsagesIncrementMultishipping.php` → `Multishipping/Plugin/SalesRule/CouponUsagesIncrement.php` |
| 2 | Update namespace |
| 3 | Move plugin registration from `SalesRule/etc/di.xml` to `Multishipping/etc/di.xml` |
| 4 | Remove old file and DI registration from SalesRule |
| 5 | Remove `magento/module-multishipping` from `SalesRule/composer.json` |

### Task 2.4: Customer — Move Wishlist Plugin Registration

| Step | Action |
|------|--------|
| 1 | Move plugin registration from `Customer/etc/di.xml` to `Wishlist/etc/di.xml` |
| 2 | (Plugin class already lives in Wishlist — only the registration is misplaced) |

### Task 2.5: Persistent — Fix Cron Type-Hint

In `Persistent/Observer/ClearExpiredCronJobObserver.php`:

```php
// Before:
use Magento\Cron\Model\Schedule;
public function execute(Schedule $schedule)

// After:
public function execute($schedule)
```

Remove `magento/module-cron` from `Persistent/composer.json`.

---

## Workstream 3: Newsletter Decoupling

**Developer count:** 2 (one admin, one frontend)
**Difficulty:** M
**Prerequisite:** None
**Pattern used:** A (hardcoded checks → plugins), D (move UI to correct module)

### Task 3.1: Move Admin Newsletter Blocks (Developer A — Admin)

| FROM (Customer) | TO (Newsletter) |
|-----------------|----------------|
| `Block/Adminhtml/Edit/Tab/Newsletter.php` | `Block/Adminhtml/Customer/Tab/Newsletter.php` |
| `Block/Adminhtml/Edit/Tab/Newsletter/Grid.php` | `Block/Adminhtml/Customer/Tab/Newsletter/Grid.php` |
| `Block/Adminhtml/Edit/Tab/Newsletter/Grid/Filter/Status.php` | `Block/Adminhtml/Customer/Tab/Newsletter/Grid/Filter/Status.php` |
| `view/adminhtml/templates/tab/newsletter.phtml` | `Newsletter/view/adminhtml/templates/customer/tab/newsletter.phtml` |
| `view/adminhtml/layout/customer_index_newsletter.xml` | `Newsletter/view/adminhtml/layout/customer_index_newsletter.xml` |

Update namespaces, template paths. Verify admin customer edit → newsletter tab works.

### Task 3.2: Move Admin Mass Actions (Developer A — Admin)

| FROM (Customer) | TO (Newsletter) |
|-----------------|----------------|
| `Controller/Adminhtml/Index/MassSubscribe.php` | `Newsletter/Controller/Adminhtml/Customer/MassSubscribe.php` |
| `Controller/Adminhtml/Index/MassUnsubscribe.php` | `Newsletter/Controller/Adminhtml/Customer/MassUnsubscribe.php` |

Update route definitions. Verify admin customer grid mass actions work.

### Task 3.3: Refactor Customer Registration (Developer B — Frontend)

**`Customer/Block/Form/Register.php`:** Remove `Newsletter\Model\Config`. Newsletter contributes the checkbox via layout XML child block on handle `customer_account_create`.

**`Customer/Controller/Account/CreatePost.php`:** Remove `SubscriberFactory`. Newsletter observes `customer_register_success` event.

Create `Newsletter/view/frontend/layout/customer_account_create.xml`:
```xml
<referenceContainer name="form.additional.info">
    <block class="Magento\Newsletter\Block\Subscribe" 
           name="customer.form.register.newsletter" 
           template="Magento_Newsletter::form/register/newsletter.phtml"/>
</referenceContainer>
```

### Task 3.4: Refactor Customer Dashboard (Developer B — Frontend)

**`Customer/Block/Account/Dashboard.php`** and **`Dashboard/Info.php`:** Remove `SubscriberFactory`. Read `is_subscribed` from customer extension attributes (already set by Newsletter's `CustomerPlugin`).

### Task 3.5: Refactor Admin Save Controller

**`Customer/Controller/Adminhtml/Index/Save.php`:** Remove `SubscriberFactory` and `SubscriptionManagerInterface`. Newsletter's `CustomerPlugin::afterSave()` already handles this.

### Task 3.6: Remove Composer Dependencies

Remove `magento/module-newsletter` from: `Customer/composer.json`, `Review/composer.json`.
Move to `suggest` in `CustomerGraphQl/composer.json`.

---

## Workstream 4: Review Decoupling

**Developer count:** 1
**Difficulty:** M
**Prerequisite:** None
**Pattern used:** D (move code to correct module)

### Task 4.1: Move Review Report Collections (2 files)

| FROM (Reports) | TO (Review) |
|----------------|------------|
| `Model/ResourceModel/Review/Collection.php` | `Review/Model/ResourceModel/Report/Collection.php` |
| `Model/ResourceModel/Review/Customer/Collection.php` | `Review/Model/ResourceModel/Report/Customer/Collection.php` |

### Task 4.2: Move Review Report Controllers (10 files)

Move entire `Reports/Controller/Adminhtml/Report/Review/` directory → `Review/Controller/Adminhtml/Report/`

Update route definitions to maintain same admin URLs.

### Task 4.3: Move Review Report Blocks (4 files)

Move `Reports/Block/Adminhtml/Review/` → `Review/Block/Adminhtml/Report/`

### Task 4.4: Move ACL Resources

Move review-related ACL entries from `Reports/etc/acl.xml` to `Review/etc/acl.xml`.

### Task 4.5: Remove Composer Dependency

Remove `magento/module-review` from `Reports/composer.json`.

---

## Workstream 5: Wishlist Decoupling

**Developer count:** 2 (one admin, one frontend)
**Difficulty:** M
**Prerequisite:** None
**Pattern used:** A (hardcoded → plugin), D (move UI)

### Task 5.1: Move Admin Controllers from Customer (Developer A)

Move 5 files from `Customer/Controller/Adminhtml/*Wishlist*` to `Wishlist/Controller/Adminhtml/Customer/`.

### Task 5.2: Decouple Sales AdminOrder\Create (Developer A)

Extract `getCustomerWishlist()` from `Sales/Model/AdminOrder/Create.php` into a Wishlist plugin. The method uses ObjectManager to create `Wishlist\Model\Wishlist` — move this to `Wishlist/Plugin/Sales/AdminOrder/CreatePlugin.php`.

### Task 5.3: Fix Catalog/CatalogWidget Templates (Developer B)

Replace `$this->helper(Magento\Wishlist\Helper\Data::class)->isAllow()` in templates with a named child block:

```php
// Before (in Catalog templates):
<?php if ($this->helper(Magento\Wishlist\Helper\Data::class)->isAllow() && $showWishlist): ?>

// After:
<?= $block->getChildHtml('product.addto.wishlist') ?>
```

Wishlist contributes the child block via layout XML.

### Task 5.4: Update Composer Dependencies

Remove `magento/module-wishlist` from: `Customer`, `Sales`, `Catalog`, `CatalogWidget`.
Move to `suggest` in: `GroupedProduct`, `Reports`.

---

## Workstream 6: MSRP Decoupling

**Developer count:** 1
**Difficulty:** M
**Prerequisite:** None
**Pattern used:** A (hardcoded → plugin)

### Task 6.1: Catalog FinalPriceBox → Msrp Plugin

Move MSRP logic from `Catalog/Pricing/Render/FinalPriceBox.php` to `Msrp/Plugin/Catalog/Pricing/Render/FinalPriceBoxPlugin.php`.

Remove `use Magento\Msrp\Pricing\Price\MsrpPrice` and `isMsrpPriceApplicable()` from FinalPriceBox.

### Task 6.2: Checkout DefaultItem → Msrp Plugin

Move `canApplyMsrp` flag from `Checkout/CustomerData/DefaultItem.php` to `Msrp/Plugin/Checkout/CustomerData/DefaultItemPlugin.php`.

Remove `$msrpHelper` from DefaultItem constructor.

### Task 6.3: GroupedProduct → MsrpGroupedProduct

Move `$msrpData` and `getChildrenMsrp()` from `GroupedProduct/Model/Product/Type/Grouped.php` to `MsrpGroupedProduct` as a plugin.

### Task 6.4: Remove Composer Dependencies

Remove `magento/module-msrp` from `Catalog/composer.json` and `Checkout/composer.json`.

---

## Workstream 7: Payment ↔ Sales Circular Dependency

**Developer count:** 1-2
**Difficulty:** H
**Prerequisite:** None
**Pattern used:** D (move adapters to correct module)

### Task 7.1: Move Gateway Adapters from Payment to Sales

| FROM (Payment) | TO (Sales) |
|----------------|-----------|
| `Gateway/Data/Order/OrderAdapter.php` | `Sales/Model/Payment/Gateway/OrderAdapter.php` |
| `Gateway/Data/Order/OrderAdapterFactory.php` | `Sales/Model/Payment/Gateway/OrderAdapterFactory.php` |
| `Gateway/Data/Order/AddressAdapter.php` | `Sales/Model/Payment/Gateway/AddressAdapter.php` |
| `Gateway/Data/Order/AddressAdapterFactory.php` | `Sales/Model/Payment/Gateway/AddressAdapterFactory.php` |

Add DI preferences in `Sales/etc/di.xml`.

### Task 7.2: Refactor PaymentDataObjectFactory

Replace `instanceof Sales\Model\Order\Payment` with a check that doesn't import Sales.

### Task 7.3: Move Observers from Payment to Sales

| FROM (Payment) | TO (Sales) |
|----------------|-----------|
| `Observer/SalesOrderBeforeSaveObserver.php` | `Sales/Observer/Payment/OrderBeforeSaveObserver.php` |
| `Observer/UpdateOrderStatusForPaymentMethodsObserver.php` | `Sales/Observer/Payment/UpdateOrderStatusObserver.php` |

Move event registrations from `Payment/etc/events.xml` to `Sales/etc/events.xml`.

### Task 7.4: Remove Sales Dependency from Payment

Remove `magento/module-sales` from `Payment/composer.json`.

---

## Workstream 8: CatalogInventory — Product Type Stock Abstraction

**Developer count:** 1-2
**Difficulty:** H
**Prerequisite:** PR #1 merged (Pattern C already applied to light-usage modules)
**Pattern used:** New — abstract base class for shared stock logic

### Task 8.1: Create Abstract ChangeParentStockStatus

Create `CatalogInventory/Model/Inventory/AbstractChangeParentStockStatus.php` containing the shared code (identical in Bundle, Configurable, Grouped):

```php
abstract class AbstractChangeParentStockStatus
{
    // Shared: getStockItems(), isNeedToUpdateParent(), saveParentStockStatus()
    // Abstract: getParentIds(array $childrenIds), isChildrenInStock(int $parentId)
}
```

### Task 8.2: Refactor Bundle ChangeParentStockStatus

`Bundle/Model/Inventory/ChangeParentStockStatus.php` → extends abstract. Only implements `isChildrenInStock()` (per-option check).

### Task 8.3: Refactor Configurable ChangeParentStockStatus

Same pattern. Only implements the flat "any child in stock" check.

### Task 8.4: Refactor Grouped ChangeParentStockStatus

Same pattern. Nearly identical to Configurable.

---

## Workstream 9: Framework Async Publisher

**Developer count:** 1
**Difficulty:** M
**Prerequisite:** None

### Task 9.1: Create Interface

```php
// lib/internal/Magento/Framework/Async/PublisherInterface.php
namespace Magento\Framework\Async;

interface PublisherInterface
{
    public function publish(string $topicName, mixed $data): void;
}
```

### Task 9.2: Create MQ-backed Implementation

`MessageQueue/Model/Async/Publisher.php` — delegates to `Framework\MessageQueue\PublisherInterface`.

### Task 9.3: Create Synchronous Fallback

`Framework/Async/SynchronousPublisher.php` — runs handler inline when MQ is absent.

### Task 9.4: Migrate Light-Usage Modules (9 modules)

Replace `use Magento\Framework\MessageQueue\PublisherInterface` with `use Magento\Framework\Async\PublisherInterface` in: ProductAlert, MediaStorage, MediaGallerySynchronization, MediaGalleryRenditions, MediaContentSynchronization, ImportExport, CatalogUrlRewrite, Eav, Config.

---

## Workstream 10: Framework PHP 8.4 Quick Wins

**Developer count:** 1-2
**Difficulty:** T-E
**Prerequisite:** PHP 8.4 runtime

### Task 10.1: `@deprecated` → `#[\Deprecated]` (401 annotations, 196 files)

Mechanical — can be scripted:
```bash
# Find and replace pattern
grep -rn "@deprecated" lib/internal/Magento/Framework/ --include="*.php" -l
```

### Task 10.2: `json_validate()` Replacement (3-5 files)

Replace `json_decode()` + `json_last_error()` with `json_validate()` in `Framework/Serialize/JsonValidator.php` and similar.

### Task 10.3: Pure Constant Classes → Enums (99 files)

Start with the simplest: classes with ONLY constants, zero methods. Keep old interfaces with `@deprecated` for backward compat.

---

## Workstream 11: Framework PHP 8.4 Proxy Elimination

**Developer count:** 1-2
**Difficulty:** H
**Prerequisite:** PHP 8.4, Workstream 10 complete

### Task 11.1: Modify ObjectManager Proxy Resolution

Change `Framework/ObjectManager/Factory/AbstractFactory.php` to use `ReflectionClass::newLazyProxy()` instead of generated Proxy classes.

### Task 11.2: Remove Proxy Generator

Remove `Framework/ObjectManager/Code/Generator/Proxy.php` from `generatedEntities` config.

### Task 11.3: Verify All 206 Proxy References

Test that every `\Proxy` reference in di.xml across the codebase resolves correctly with native lazy objects.

---

## Workstream 12: View Layout — Lazy Blocks

**Developer count:** 1
**Difficulty:** M
**Prerequisite:** PHP 8.4

### Task 12.1: Wrap Block Creation in LazyGhost

Modify `Framework/View/Layout/Generator/Block.php`:

```php
// Before:
$block = $this->blockFactory->createBlock($class, $arguments);

// After:
$reflector = new ReflectionClass($class);
$block = $reflector->newLazyGhost(function ($instance) use ($class, $arguments) {
    $this->blockFactory->initializeBlock($instance, $arguments);
});
```

### Task 12.2: Verify Block Rendering

Test that all block types render correctly — the lazy ghost initializes transparently on `toHtml()`.

---

## Workstream Dependencies

```
WS 1  (Phantom deps)          ─── independent
WS 2  (Plugin relocation)     ─── independent
WS 3  (Newsletter)            ─── independent
WS 4  (Review)                ─── independent
WS 5  (Wishlist)              ─── independent
WS 6  (MSRP)                  ─── independent
WS 7  (Payment↔Sales)         ─── independent
WS 8  (Stock abstraction)     ─── depends on PR #1 merged
WS 9  (Async Publisher)       ─── independent
WS 10 (PHP 8.4 quick wins)    ─── independent
WS 11 (Proxy elimination)     ─── depends on WS 10
WS 12 (Lazy blocks)           ─── depends on PHP 8.4
```

**10 of 12 workstreams can start Day 1.** WS 8 waits on PR #1. WS 11 waits on WS 10.

---

## Verification Checklist (After Each Task)

```bash
# 1. Compilation
bin/magento setup:di:compile

# 2. Database
bin/magento setup:upgrade --dry-run

# 3. Unit tests for affected modules
vendor/bin/phpunit -c dev/tests/unit/phpunit.xml.dist app/code/Magento/MODULE/Test/Unit/

# 4. Module disable test (for newly decoupled modules)
bin/magento module:disable Magento_MODULE
bin/magento setup:di:compile  # Must pass with module disabled
bin/magento module:enable Magento_MODULE
```

---

## Summary

| WS | Workstream | Tasks | Files | Difficulty | Devs | Status |
|----|-----------|-------|-------|------------|------|--------|
| — | PR #1: Catalog/Bundle/CatalogInventory decoupling | — | 77 PHP + 6 XML | M | — | **DONE** |
| 1 | Phantom deps | 2 | ~5 composer.json | T | 1 | Ready |
| 2 | Plugin relocation | 5 | ~12 files | E | 1 | Ready |
| 3 | Newsletter | 6 | ~15 files | M | 2 | Ready |
| 4 | Review | 5 | ~16 files | M | 1 | Ready |
| 5 | Wishlist | 4 | ~12 files | M | 2 | Ready |
| 6 | MSRP | 4 | ~6 files | M | 1 | Ready |
| 7 | Payment↔Sales | 4 | ~8 files | H | 1-2 | Ready |
| 8 | Stock abstraction | 4 | ~6 files | H | 1-2 | Blocked (PR #1) |
| 9 | Async Publisher | 4 | ~12 files | M | 1 | Ready |
| 10 | PHP 8.4 quick wins | 3 | ~300 files | T-E | 1-2 | Ready |
| 11 | Proxy elimination | 3 | ~5 files | H | 1-2 | Blocked (WS 10) |
| 12 | Lazy blocks | 2 | ~2 files | M | 1 | Ready |
| **Total** | | **46** | **~500 files** | | **10-15 devs** | |
