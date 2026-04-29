# MSRP, Multishipping & MysqlMq Decoupling Report

---

## 1. MSRP (Minimum Suggested Retail Price)

### Module Family (32 files total)

| Module | Files | Role |
|--------|-------|------|
| `Msrp` | 27 | Core MSRP logic — price rendering, helper, admin UI |
| `MsrpConfigurableProduct` | 2 | MSRP price calculator for configurable products |
| `MsrpGroupedProduct` | 3 | MSRP price calculator for grouped products |

### Who Depends on MSRP

| Module | composer.json | PHP imports | Type |
|--------|--------------|-------------|------|
| **Catalog** | hard require | 1 file: `Pricing/Render/FinalPriceBox.php` imports `MsrpPrice` | Direct code coupling |
| **Checkout** | hard require | 1 file: `CustomerData/DefaultItem.php` imports `Msrp\Helper\Data` | Direct code coupling |
| **GroupedProduct** | hard require | 1 file: `Model/Product/Type/Grouped.php` imports `Msrp\Helper\Data` | Direct code coupling |
| **ConfigurableProduct** | **suggest** (already correct) | 0 | Clean |
| MsrpConfigurableProduct | hard require (expected) | 1 file | Internal to MSRP family |
| MsrpGroupedProduct | hard require (expected) | 1 file | Internal to MSRP family |

### Coupling Details

**Catalog — `Pricing/Render/FinalPriceBox.php`:**
```php
use Magento\Msrp\Pricing\Price\MsrpPrice;
// Checks if MSRP applies, hides actual price if "show before order confirm" is set
if ($this->isMsrpPriceApplicable()) {
    $msrpPriceType = $this->getSaleableItem()->getPriceInfo()->getPrice(MsrpPrice::PRICE_CODE);
}
```

**Checkout — `CustomerData/DefaultItem.php`:**
```php
protected $msrpHelper; // Magento\Msrp\Helper\Data
// Adds 'canApplyMsrp' flag to cart item data
'canApplyMsrp' => $this->msrpHelper->isShowBeforeOrderConfirm($product)
    && $this->msrpHelper->isMinimalPriceLessMsrp($product),
```

**GroupedProduct — `Model/Product/Type/Grouped.php`:**
```php
protected $msrpData; // Magento\Msrp\Helper\Data
// getChildrenMsrp() — returns MSRP for grouped children
```

### Decoupling Plan

**Catalog `FinalPriceBox`:** The MSRP check should be a plugin from the Msrp module, not hardcoded in Catalog's price renderer. Msrp module hooks into `FinalPriceBox::toHtml()` via `afterToHtml()` or `aroundToHtml()` to apply MSRP logic. Catalog renders the price normally; Msrp wraps/overrides the output when applicable.

**Checkout `DefaultItem`:** The `canApplyMsrp` flag should be added via plugin on `DefaultItem::getItemData()` from the Msrp module, not via constructor injection in Checkout.

**GroupedProduct `Grouped.php`:** The `msrpData` helper and `getChildrenMsrp()` method should move to `MsrpGroupedProduct` module as a plugin or be provided via extension attributes.

**Composer changes:**

| Module | Change |
|--------|--------|
| `Catalog/composer.json` | Remove `magento/module-msrp` |
| `Checkout/composer.json` | Remove `magento/module-msrp` |
| `GroupedProduct/composer.json` | Move `magento/module-msrp` to `suggest` |

**Difficulty: Medium.** The price rendering coupling in Catalog is the trickiest — MSRP fundamentally changes how prices are displayed. But the plugin approach is well-established in Magento.

---

## 2. Multishipping

### Module Size: 65 PHP files

A complete alternative checkout flow for splitting orders across multiple shipping addresses.

### Who Depends on Multishipping

| Module | composer.json | PHP/XML coupling | Type |
|--------|--------------|------------------|------|
| **GiftMessage** | hard require | 2 plugins in `frontend/di.xml` targeting Multishipping blocks/models | Code + XML |
| **SalesRule** | hard require | 1 plugin file `CouponUsagesIncrementMultishipping.php` + DI registration | Code + XML |

**That's it.** Only 2 external modules.

### Coupling Details

**GiftMessage — 2 plugins:**
```xml
<!-- GiftMessage/etc/frontend/di.xml -->
<type name="Magento\Multishipping\Block\Checkout\Shipping">
    <plugin name="getItemsBoxTextAfter" type="Magento\GiftMessage\Block\Message\Multishipping\Plugin\ItemsBox"/>
</type>
<type name="Magento\Multishipping\Model\Checkout\Type\Multishipping">
    <plugin name="save_gift_messages" type="Magento\GiftMessage\Model\Type\Plugin\Multishipping"/>
</type>
```
GiftMessage hooks into Multishipping to add gift message functionality to multishipping checkout.

**SalesRule — 1 plugin:**
```php
// SalesRule/Plugin/CouponUsagesIncrementMultishipping.php
use Magento\Multishipping\Model\Checkout\Type\Multishipping\PlaceOrderDefault;
// Tracks coupon usage when orders are placed via multishipping
```

### Decoupling Plan

**GiftMessage plugins:** Move plugin registrations and the 2 plugin classes from GiftMessage into either:
- A new `GiftMessageMultishipping` bridge module, OR
- The Multishipping module itself (Multishipping contributes gift message support)

The second option is cleaner — Multishipping already depends on many modules, adding GiftMessage as a `suggest` is natural.

**SalesRule plugin:** Move `CouponUsagesIncrementMultishipping.php` + DI registration from SalesRule into the Multishipping module. Multishipping already depends on Quote/Sales, adding SalesRule as a `suggest` for coupon usage tracking is appropriate.

**Composer changes:**

| Module | Change |
|--------|--------|
| `GiftMessage/composer.json` | Remove `magento/module-multishipping` |
| `SalesRule/composer.json` | Remove `magento/module-multishipping` |

After this, Multishipping can be entirely disabled/removed:
- Standard checkout works perfectly without it
- GiftMessage works without multishipping (plugins just don't load)
- SalesRule coupon tracking works for normal orders (multishipping-specific tracking just doesn't happen)

**Difficulty: Low-Medium.** Move 3 plugin files + 3 DI registrations. Straightforward.

### Multishipping's Own Dependencies

Multishipping requires 10 core modules (Checkout, Customer, Directory, Payment, Quote, Sales, Store, Tax, Theme, Captcha). This is expected — it's a full checkout flow. These dependencies are all legitimate; Multishipping is a consumer, not a dependency.

---

## 3. MysqlMq

### Module Size: 18 PHP files

MySQL-based message queue transport — the fallback when RabbitMQ is not installed.

### Who Depends on MysqlMq

| Type | Count |
|------|-------|
| composer.json require | **0** |
| PHP imports | **0** |
| XML references | **0** |

**Zero external coupling.** Already documented in `cron_and_message_queue.md`.

### MysqlMq's Own Dependencies

- `magento/framework` + `magento/framework-message-queue` + `magento/module-store`
- Has a `crontab.xml` (cleans old queue messages)

### Verdict

**MysqlMq is already fully decoupled.** It's a pure SPI (Service Provider Interface) implementation — the MessageQueue framework discovers and uses it at runtime via DI configuration. Can be disabled/removed via `module:disable` or composer with zero impact. When removed, RabbitMQ (Amqp module) must be configured instead, or MessageQueue module itself must be disabled.

No work needed.

---

## Summary

| Module | Inbound Coupling | Decoupling Effort | Can Be Removed? |
|--------|-----------------|-------------------|-----------------|
| **Msrp** | 3 modules (Catalog, Checkout, GroupedProduct) | Medium — move price logic to plugins | Yes, after decoupling |
| **MsrpConfigurableProduct** | 0 external | None | Yes — already optional |
| **MsrpGroupedProduct** | 0 external (only MsrpGroupedProduct itself) | None | Yes — already optional |
| **Multishipping** | 2 modules (GiftMessage, SalesRule) | Low-Medium — move 3 plugins | Yes, after decoupling |
| **MysqlMq** | 0 | None needed | **Yes — already fully decoupled** |

### Quick Wins

| Action | Effort |
|--------|--------|
| Remove MysqlMq (if RabbitMQ is used) | Already possible |
| Remove MsrpConfigurableProduct / MsrpGroupedProduct | Already possible |
| Move GiftMessage + SalesRule multishipping plugins to Multishipping | Low |

### Medium Effort

| Action | Effort |
|--------|--------|
| Decouple MSRP from Catalog FinalPriceBox | Medium — refactor to plugin-based MSRP rendering |
| Decouple MSRP from Checkout DefaultItem | Medium — move to plugin |
| Decouple MSRP from GroupedProduct | Low — move to MsrpGroupedProduct |
