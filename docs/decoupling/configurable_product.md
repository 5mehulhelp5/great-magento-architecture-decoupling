# Configurable Product Decoupling Report

## Modules Scanned

- `Magento_ConfigurableProduct` (`app/code/Magento/ConfigurableProduct`)
- `Magento_ConfigurableImportExport` (`app/code/Magento/ConfigurableImportExport`)
- `Magento_ConfigurableProductGraphQl` (`app/code/Magento/ConfigurableProductGraphQl`)
- `Magento_ConfigurableProductSales` (`app/code/Magento/ConfigurableProductSales`)

---

## Constants Inventory

### ConfigurableProduct

| Constant | Class | Value | Visibility |
|----------|-------|-------|------------|
| `TYPE_CODE` | `Model\Product\Type\Configurable` | `'configurable'` | public |
| `CONFIG_THUMBNAIL_SOURCE` | `Block\Cart\Item\Renderer\Configurable` | `'checkout/cart/configurable_product_image'` | public |
| `CONFIG_THUMBNAIL_SOURCE` | `Model\Product\Configuration\Item\ItemProductResolver` | `'checkout/cart/configurable_product_image'` | public |
| `PRICE_CODE` | `Pricing\Price\ConfigurableRegularPrice` | `'regular_price'` | public |
| `CODE_GROUP_PRICE` | `Ui\..\ConfigurablePrice` | `'container_price'` | public |
| `OPTION_ID` | `Api\Data\ConfigurableItemOptionValueInterface` | `'option_id'` | public |
| `OPTION_VALUE` | `Api\Data\ConfigurableItemOptionValueInterface` | `'option_value'` | public |
| `KEY_ATTRIBUTE_ID` | `Model\Product\Type\Configurable\Attribute` | `'attribute_id'` | public |
| `KEY_LABEL` | `Model\Product\Type\Configurable\Attribute` | `'label'` | public |
| `KEY_POSITION` | `Model\Product\Type\Configurable\Attribute` | `'position'` | public |
| `KEY_IS_USE_DEFAULT` | `Model\Product\Type\Configurable\Attribute` | `'is_use_default'` | public |
| `KEY_VALUES` | `Model\Product\Type\Configurable\Attribute` | `'values'` | public |
| `KEY_PRODUCT_ID` | `Model\Product\Type\Configurable\Attribute` | `'product_id'` | public |
| `KEY_VALUE_INDEX` | `Model\Product\Type\Configurable\OptionValue` | `'value_index'` | public |
| `GROUP_CONFIGURABLE` | `Ui\..\ConfigurablePanel` | `'configurable'` | public |
| `ASSOCIATED_PRODUCT_MODAL` | `Ui\..\ConfigurablePanel` | `'configurable_associated_product_modal'` | public |
| `ASSOCIATED_PRODUCT_LISTING` | `Ui\..\ConfigurablePanel` | `'configurable_associated_product_listing'` | public |
| `CONFIGURABLE_MATRIX` | `Ui\..\ConfigurablePanel` | `'configurable-matrix'` | public |
| `CODE_QUANTITY` | `Ui\..\ConfigurableQty` | `'qty'` | public |
| `CODE_QTY_CONTAINER` | `Ui\..\ConfigurableQty` | `'quantity_and_stock_status_qty'` | public |
| `WARNING_PRICE_TYPE` | `Ui\..\CustomOptions` | `'price_type_warning'` | public |
| `ATTRIBUTE_SET_HANDLER_MODAL` | `Ui\..\ConfigurableAttributeSetHandler` | `'configurable_attribute_set_handler_modal'` | public |
| `ADMIN_RESOURCE` | Various controllers | `'Magento_Catalog::products'` | public |
| `NAME` | Listing Columns (Name, Price, Attributes) | various | public |

### ConfigurableImportExport

| Constant | Class | Value | Visibility |
|----------|-------|-------|------------|
| `ERROR_ATTRIBUTE_CODE_DOES_NOT_EXIST` | `Import\Product\Type\Configurable` | `'attrCodeDoesNotExist'` | public |
| `ERROR_ATTRIBUTE_CODE_NOT_GLOBAL_SCOPE` | `Import\Product\Type\Configurable` | `'attrCodeNotGlobalScope'` | public |
| `ERROR_ATTRIBUTE_CODE_NOT_TYPE_SELECT` | `Import\Product\Type\Configurable` | `'attrCodeNotTypeSelect'` | public |
| `ERROR_ATTRIBUTE_CODE_IS_NOT_SUPER` | `Import\Product\Type\Configurable` | `'attrCodeIsNotSuper'` | public |
| `ERROR_INVALID_OPTION_VALUE` | `Import\Product\Type\Configurable` | `'invalidOptionValue'` | public |
| `ERROR_INVALID_WEBSITE` | `Import\Product\Type\Configurable` | `'invalidSuperAttrWebsite'` | public |
| `ERROR_DUPLICATED_VARIATIONS` | `Import\Product\Type\Configurable` | `'duplicatedVariations'` | public |
| `ERROR_UNIDENTIFIABLE_VARIATION` | `Import\Product\Type\Configurable` | `'unidentifiableVariation'` | public |
| `CONFIGURABLE_VARIATIONS_COLUMN` | `Export\RowCustomizer` | `'configurable_variations'` | public |
| `CONFIGURABLE_VARIATIONS_LABELS_COLUMN` | `Export\RowCustomizer` | `'configurable_variation_labels'` | public |

### ConfigurableProductGraphQl

| Constant | Class | Value | Visibility |
|----------|-------|-------|------------|
| `TYPE_RESOLVER` | `ConfigurableProductTypeResolver` | `'ConfigurableProduct'` | public |
| `OPTION_TYPE` | Various resolvers | `'configurable'` | private |
| `UID_PREFIX` | `SelectionUidFormatter` | `'configurable'` | private |
| `UID_SEPARATOR` | `SelectionUidFormatter` | `'/'` | private |

### ConfigurableProductSales

No constants found.

---

## Cross-Module Constant Usage (Production Code)

The only constant referenced outside the Configurable modules is `Configurable::TYPE_CODE`:

| External Module | File | Usage |
|-----------------|------|-------|
| CatalogRuleConfigurable | `Plugin/CatalogRule/Model/ConfigurableProductsProvider.php` | SQL where clause |
| Swatches | `Model/SwatchAttributesProvider.php` | Type check |
| Swatches | `Model/Plugin/ProductImage.php` | Type check |
| Weee | `Model/Tax.php` | Type check |
| MsrpConfigurableProduct | `Pricing/MsrpPriceCalculator.php` | Type check |

All other constants are used only within their own module.

### Hardcoded String `'configurable'` Outside Configurable Modules

| Module | File | Usage |
|--------|------|-------|
| SalesRule | `Model/Validator.php:618` | `$item->getParentItem()->getProductType() === 'configurable'` |
| QuoteConfigurableOptions | `Model/Cart/BuyRequest/SuperAttributeDataProvider.php` | `private const OPTION_TYPE = 'configurable'` |
| Wishlist | `Model/Wishlist/BuyRequest/SuperAttributeDataProvider.php` | `private const PROVIDER_OPTION_TYPE = 'configurable'` |

---

## PHP `use` Imports From Configurable Modules (Production Code)

| External Module | # Imports | Key Classes |
|-----------------|-----------|-------------|
| Swatches | 11 | `Configurable` type, `Helper\Data`, `ConfigurableAttributeData`, `Attribute`, `Collection`, `AttributesListInterface` |
| CatalogRuleConfigurable | 4 | `Configurable` type, `ConfigurableProductsResourceModel`, `Collection` |
| Weee | 4 | `Configurable` type, `ConfigurableBlock`, `FinalPriceResolver` |
| NewRelicReporting | 2 | `ConfigurableProductManagementInterface` |
| ConfigurableImportExport | 2 | `ConfigurableProductType`, `ChangeParentStockStatus` |
| MsrpConfigurableProduct | 1 | `Configurable` type |

## XML References (di.xml, layout) to Configurable Classes

| External Module | Count | Details |
|-----------------|-------|---------|
| Swatches | 3 | di.xml plugins and observer references |
| Weee | 2 | di.xml plugin on `FinalPriceResolver`, frontend/di.xml plugin on `Configurable` block |
| Wishlist | 2 | Layout XML: configurable block in wishlist configure page |
| CatalogRuleConfigurable | 1 | di.xml reference |

---

## Modules Selected for Decoupling

### 1. Magento_NewRelicReporting — EASY

**Current state:**
- `composer.json`: hard require on `magento/module-configurable-product`
- `module.xml`: `<sequence>` includes `Magento_ConfigurableProduct`

**Coupling points (2 files):**

| File | Import | Usage |
|------|--------|-------|
| `Model/Counter.php` | `ConfigurableProductManagementInterface` | Constructor-injected, calls `->getCount()` in `getConfigurableCount()` |
| `Model/Cron/ReportCounts.php` | `ConfigurableProductManagementInterface` | Constructor-injected, calls `->getCount()` in `reportConfigurableProductsSize()` |

**Decoupling approach:** Make `ConfigurableProductManagementInterface` optional (nullable constructor arg). Gracefully skip configurable count when module is absent. Move `magento/module-configurable-product` from `require` to `suggest`, remove from `<sequence>`.

---

### 2. Magento_Wishlist — MODERATE

**Current state:**
- `composer.json`: `magento/module-configurable-product` already in `suggest` (not require)
- `module.xml`: no sequence dependency on ConfigurableProduct

**Coupling points:**

| File / Location | Type | Details |
|-----------------|------|---------|
| `Model/Wishlist/BuyRequest/SuperAttributeDataProvider.php` | Hardcoded string | `private const PROVIDER_OPTION_TYPE = 'configurable'` — string comparison only, no class import |
| `Pricing/ConfiguredPrice/ConfigurableProduct.php` | Class name | Named after configurable but imports nothing from ConfigurableProduct module — self-contained |
| `etc/di.xml:50` | XML virtualType | `<virtualType name="Magento\ConfigurableProduct\Pricing\Price\Pool">` — injects wishlist price into configurable's price pool |
| `view/base/layout/catalog_product_prices.xml:19` | XML argument | `<argument name="configurable">` — price render template for configurable type |
| `view/frontend/layout/wishlist_index_configure_type_configurable.xml` | Layout XML | Renders `Magento\ConfigurableProduct\Block\...\Configurable` block and template |
| `view/adminhtml/layout/customer_index_wishlist.xml:95` | XML string | `<item name="process" xsi:type="string">configurable</item>` — UI action name, not a class ref |

**Decoupling approach:** PHP code is already clean — no class imports from ConfigurableProduct. Layout XML `wishlist_index_configure_type_configurable.xml` is a type-specific handle only loaded when configurable products are involved. The `di.xml` virtualType could be moved to ConfigurableProduct's own di.xml. Composer `suggest` is already correct.

---

### 3. Magento_Weee — HARD

**Current state:**
- `composer.json`: hard require on `magento/module-configurable-product`
- `module.xml`: no sequence dependency on ConfigurableProduct (inconsistent with hard require)

**Coupling points:**

| File | Type | Details |
|------|------|---------|
| `Model/Tax.php:9,188` | PHP import + constant | `use Configurable; ... $product->getTypeId() !== Configurable::TYPE_CODE` — skips WEEE for configurable products |
| `Plugin/ConfigurableProduct/Block/Product/View/Type/Configurable.php` | Plugin class | Entire plugin targets `ConfigurableBlock` — adds WEEE/FPT data to configurable JSON config |
| `Plugin/ConfigurableProduct/Pricing/FinalPriceResolver.php` | Plugin class | Entire plugin targets `ConfigurableProductFinalPriceResolver` — adjusts final price with WEEE |
| `Plugin/Model/ConfigurableVariationAttributePriority.php` | PHP import + injection | Injects `Configurable` type model, calls `$this->configurable->getParentIdsByChild()` |
| `etc/di.xml:91` | Plugin registration | Plugin on `Magento\Weee\Model\Tax` for configurable variation |
| `etc/di.xml:93` | Plugin registration | Plugin on `Magento\ConfigurableProduct\Pricing\Price\FinalPriceResolver` |
| `etc/frontend/di.xml:23` | Plugin registration | Plugin on `Magento\ConfigurableProduct\Block\...\Configurable` |
| `etc/frontend/di.xml:26` | Plugin registration | Same plugin also targets `Magento\Swatches\Block\...\Configurable` |
| `view/frontend/web/js/price-box-mixin.js:200,207` | JS | `$form.data('configurable')` — jQuery data key, not a PHP dependency |

**Decoupling approach:**
- `Tax.php`: Replace `Configurable::TYPE_CODE` with string `'configurable'` or make excluded types configurable via DI
- 3 Plugin classes: Move to a bridge module (e.g. `Magento_WeeeConfigurableProduct`) or make plugin registrations conditional
- JS mixin: Frontend concern only, no PHP module dependency created
- Move `magento/module-configurable-product` from `require` to `suggest`
