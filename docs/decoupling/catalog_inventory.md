# CatalogInventory Decoupling Report

## Module Scanned

- `Magento_CatalogInventory` (`app/code/Magento/CatalogInventory`)

---

## Scale of the Problem

- **24 external modules** with CatalogInventory coupling
- **83 PHP `use` imports** (production code, excl. tests)
- **25 XML references** (di.xml, layout, module.xml)
- **23 modules** with hard `require` in composer.json
- **6 modules** with `<sequence>` in module.xml

---

## Key Constants (Most Likely Referenced Externally)

### StockStatusInterface (`Api\Data\StockStatusInterface`)

| Constant | Value |
|----------|-------|
| `STATUS_OUT_OF_STOCK` | `0` |
| `STATUS_IN_STOCK` | `1` |
| `PRODUCT_ID` | `'product_id'` |
| `STOCK_ID` | `'stock_id'` |
| `QTY` | `'qty'` |
| `STOCK_STATUS` | `'stock_status'` |
| `STOCK_ITEM` | `'stock_item'` |

### StockItemInterface (`Api\Data\StockItemInterface`)

| Constant | Value |
|----------|-------|
| `BACKORDERS_NO` | `0` |
| `ITEM_ID` | `'item_id'` |
| `PRODUCT_ID` | `'product_id'` |
| `STOCK_ID` | `'stock_id'` |
| `QTY` | `'qty'` |
| `IS_QTY_DECIMAL` | `'is_qty_decimal'` |
| `IS_IN_STOCK` | `'is_in_stock'` |
| `MANAGE_STOCK` | `'manage_stock'` |
| `BACKORDERS` | `'backorders'` |
| `MIN_QTY` | `'min_qty'` |
| `MIN_SALE_QTY` | `'min_sale_qty'` |
| `MAX_SALE_QTY` | `'max_sale_qty'` |
| `QTY_INCREMENTS` | `'qty_increments'` |
| *(+ 10 more USE_CONFIG_* and other fields)* | |

### Stock Model (`Model\Stock`)

| Constant | Value |
|----------|-------|
| `BACKORDERS_NO` | `0` |
| `BACKORDERS_YES_NONOTIFY` | `1` |
| `BACKORDERS_YES_NOTIFY` | `2` |
| `STOCK_OUT_OF_STOCK` | `0` |
| `STOCK_IN_STOCK` | `1` |
| `DEFAULT_STOCK_ID` | `1` |

### Configuration (`Model\Configuration`)

| Constant | Value |
|----------|-------|
| `XML_PATH_SHOW_OUT_OF_STOCK` | `'cataloginventory/options/show_out_of_stock'` |
| `XML_PATH_MANAGE_STOCK` | `'cataloginventory/item_options/manage_stock'` |
| `XML_PATH_BACKORDERS` | `'cataloginventory/item_options/backorders'` |
| `XML_PATH_MIN_QTY` | `'cataloginventory/item_options/min_qty'` |
| `XML_PATH_MIN_SALE_QTY` | `'cataloginventory/item_options/min_sale_qty'` |
| `XML_PATH_MAX_SALE_QTY` | `'cataloginventory/item_options/max_sale_qty'` |
| *(+ 8 more XML_PATH_* constants)* | |

### Stock Indexer (`Model\Indexer\Stock\Processor`)

| Constant | Value |
|----------|-------|
| `INDEXER_ID` | `'cataloginventory_stock'` |

---

## Coupling by Module (Production Code Only)

### Tier 1: Heaviest Coupling (10+ PHP imports)

#### ConfigurableProduct — 20 PHP uses, 5 XML refs

| File | CatalogInventory Classes Used |
|------|------------------------------|
| `Model/Inventory/ChangeParentStockStatus.php` | `StockItemInterface`, `StockConfigurationInterface`, `StockItemCriteriaInterfaceFactory`, `StockItemRepositoryInterface` |
| `Model/Inventory/ParentItemProcessor.php` | `StockItemCriteriaInterfaceFactory`, `StockItemRepositoryInterface`, `StockConfigurationInterface`, `ParentItemProcessorInterface` |
| `Model/Plugin/UpdateStockChangedAuto.php` | `Stock\Item` (ResourceModel) |
| `Model/Product/Type/Collection/SalableProcessor.php` | `Stock\StatusFactory` |
| `Model/ResourceModel/Indexer/Stock/Configurable.php` | `Indexer\Stock\Action\Full` |
| `Model/ResourceModel/Product/Indexer/Price/BaseStockStatusSelectProcessor.php` | `Stock`, `StockConfigurationInterface` |
| `Model/ResourceModel/Product/StockStatusBaseSelectProcessor.php` | `StockConfigurationInterface`, `Stock\Status`, `Stock\Status` (Resource) |
| `Plugin/Model/ResourceModel/Attribute/InStockOptionSelectBuilder.php` | `StockConfigurationInterface`, `Stock\Status` (Resource) |
| `Pricing/Price/ConfigurableOptionsStockStatusFilter.php` | `StockConfigurationInterface` |
| `Ui/DataProvider/Product/Form/Modifier/Data/AssociatedProducts.php` | `StockRegistryInterface` |
| `etc/di.xml` | 3 type configs (QuantityValidator, SaveInventoryDataObserver, Stock\Item) |
| `etc/module.xml` | sequence dependency |
| `view/frontend/layout/catalog_product_view_type_configurable.xml` | `Stockqty\Type\Configurable` block |

#### Bundle — 11 PHP uses, 5 XML refs

| File | CatalogInventory Classes Used |
|------|------------------------------|
| `Model/Inventory/ChangeParentStockStatus.php` | `StockItemInterface`, `StockConfigurationInterface`, `StockItemCriteriaInterfaceFactory`, `StockItemRepositoryInterface` |
| `Model/Inventory/ParentItemProcessor.php` | `ParentItemProcessorInterface` |
| `Model/ResourceModel/Indexer/DefaultInventoryStockStatusQueryProcessor.php` | `StockConfigurationInterface`, `Stock` |
| `Model/ResourceModel/Indexer/SelectionPriceModifier.php` | `StockConfigurationInterface` |
| `Model/ResourceModel/Indexer/Stock.php` | `Indexer\Stock\Action\Full`, `Indexer\Stock\DefaultStock` |
| `Ui/DataProvider/Product/Form/Modifier/AddSelectionQtyTypeToProductsData.php` | `StockRegistryPreloader` |
| `etc/di.xml` | `StockRegistryInterface\Proxy`, `StockStateInterface\Proxy`, `Indexer\Stock\Action\Full`, `SaveInventoryDataObserver` |
| `view/frontend/layout/catalog_product_view_type_bundle.xml` | `Qtyincrements` block |

---

### Tier 2: Moderate Coupling (4-9 PHP imports)

#### CatalogImportExport — 7 PHP uses, 1 XML ref

| File | CatalogInventory Classes Used |
|------|------------------------------|
| `Model/Export/Product.php` | `StockConfigurationInterface` |
| `Model/Export/Product/Stock.php` | `Configuration`, `Stock\Item` (Resource) |
| `Model/Import/Product.php` | `StockItemInterface` |
| `Model/Import/Product/Validator/Backorders.php` | `Source\Backorders` |
| `Model/StockItemImporter.php` | `Stock\Item` (Resource), `Stock\ItemFactory` |
| `etc/di.xml` | `Indexer\Stock\Processor::INDEXER_ID` |

#### GroupedProduct — 7 PHP uses, 4 XML refs

| File | CatalogInventory Classes Used |
|------|------------------------------|
| `Model/Inventory/ChangeParentStockStatus.php` | `StockItemInterface`, `StockConfigurationInterface`, `StockItemCriteriaInterfaceFactory`, `StockItemRepositoryInterface` |
| `Model/Inventory/ParentItemProcessor.php` | `ParentItemProcessorInterface` |
| `Model/ResourceModel/Indexer/Stock/Grouped.php` | `Indexer\Stock\Action\Full` |
| `ViewModel/ValidateQuantity.php` | `Product\QuantityValidator` |
| `etc/di.xml` | `SaveInventoryDataObserver`, `MassUpdateProductAttribute` |
| `etc/module.xml` | sequence dependency |
| `view/frontend/layout/catalog_product_view_type_grouped.xml` | `Stockqty\Type\Grouped` block |

#### QuoteGraphQl — 6 PHP uses, 1 XML ref

| File | CatalogInventory Classes Used |
|------|------------------------------|
| `Model/Cart/MergeCarts/CartQuantityValidator.php` | `StockRegistryInterface`, `Stock` |
| `Model/CartItem/ProductStock.php` | `StockStatusInterface`, `StockRegistryInterface`, `Configuration`, `StockState` |
| `etc/graphql/di.xml` | `CatalogInventoryGraphQl\Helper\Error\MessageFormatters` |

#### ConfigurableProductGraphQl — 5 PHP uses, 1 XML ref

| File | CatalogInventory Classes Used |
|------|------------------------------|
| `Model/Cart/BuyRequest/SuperAttributeDataProvider.php` | `StockStateInterface`, `StockStateException` |
| `Model/Formatter/OptionValue.php` | `StockRegistry` |
| `Model/Options/DataProvider/Variant.php` | `Stock\StatusFactory` |
| `Plugin/AddStockStatusToCollection.php` | `Stock\Status` (Resource) |
| `etc/module.xml` | sequence dependency |

#### Catalog — 5 PHP uses, 2 XML refs

| File | CatalogInventory Classes Used |
|------|------------------------------|
| `Controller/Adminhtml/Product/Initialization/StockDataFilter.php` | `StockConfigurationInterface` |
| `Setup/CategorySetup.php` | `Form\Field\Stock`, `Source\Stock`, `Stock` model |
| `ViewModel/Product/Checker/AddToCompareAvailability.php` | `StockConfigurationInterface` |
| `etc/di.xml` | `Config\Backend\ShowOutOfStock`, `StockItemInterface[]` |

#### Wishlist — 5 PHP uses, 2 XML refs

| File | CatalogInventory Classes Used |
|------|------------------------------|
| `Model/ResourceModel/Item/Collection.php` | `StockStatusFilterInterface` |
| `Model/Wishlist.php` | `StockConfigurationInterface`, `StockRegistryInterface`, `StockStateException` |
| `ViewModel/AllowedQuantity.php` | `StockRegistry` |
| `view/frontend/layout/wishlist_index_configure_type_bundle.xml` | `Qtyincrements` block |
| `view/frontend/layout/wishlist_index_configure_type_downloadable.xml` | `Stockqty\DefaultStockqty` block |

#### CatalogSearch — 4 PHP uses, 1 XML ref

| File | CatalogInventory Classes Used |
|------|------------------------------|
| `Model/Indexer/Plugin/StockedProductsFilterPlugin.php` | `StockConfigurationInterface`, `StockStatusRepositoryInterface`, `StockStatusCriteriaInterfaceFactory`, `StockStatusInterface` |
| `etc/module.xml` | sequence dependency |

---

### Tier 3: Light Coupling (1-3 PHP imports)

#### Sales — 3 PHP uses

| File | CatalogInventory Classes Used |
|------|------------------------------|
| `Block/Adminhtml/Order/Create/Items/Grid.php` | `StockRegistryInterface`, `StockStateInterface` |
| `CustomerData/LastOrderedItems.php` | `StockRegistryInterface` |

#### SalesInventory — 2 PHP uses, 1 XML ref

| File | CatalogInventory Classes Used |
|------|------------------------------|
| `Observer/RefundOrderInventoryObserver.php` | `StockConfigurationInterface`, `StockManagementInterface` |
| `etc/module.xml` | sequence dependency |

#### GroupedCatalogInventory — 2 PHP uses

| File | CatalogInventory Classes Used |
|------|------------------------------|
| `Plugin/OutOfStockFilter.php` | `StockStatusInterface`, `StockRegistryInterface` |

#### CatalogGraphQl — 2 PHP uses

| File | CatalogInventory Classes Used |
|------|------------------------------|
| `Model/Resolver/Products/DataProvider/Product/CollectionProcessor/StockProcessor.php` | `StockConfigurationInterface`, `Stock\Status` (Resource) |

#### Dhl — 1 PHP use

| File | CatalogInventory Classes Used |
|------|------------------------------|
| `Model/Carrier.php` | `StockRegistryInterface` |

#### Ups — 1 PHP use

| File | CatalogInventory Classes Used |
|------|------------------------------|
| `Model/Carrier.php` | `StockRegistryInterface` |

#### Downloadable — 1 PHP use, 1 XML ref

| File | CatalogInventory Classes Used |
|------|------------------------------|
| `Controller/Download/Sample.php` | `StockConfigurationInterface` |
| `view/frontend/layout/catalog_product_view_type_downloadable.xml` | `Stockqty\DefaultStockqty` block |

#### Checkout — 1 PHP use, 1 XML ref

| File | CatalogInventory Classes Used |
|------|------------------------------|
| `Block/Cart/Crosssell.php` | `Stock` helper |
| `etc/module.xml` | sequence dependency |

#### Elasticsearch — 1 PHP use

| File | CatalogInventory Classes Used |
|------|------------------------------|
| `Model/Indexer/Plugin/DependencyUpdaterPlugin.php` | `Indexer\Stock\Processor` |

#### Quote — 1 PHP use

| File | CatalogInventory Classes Used |
|------|------------------------------|
| `Model/Cart/AddProductsToCart.php` | `StockRegistryInterface` |

---

### Tier 4: Composer-Only Dependency (no PHP imports found)

| Module | Notes |
|--------|-------|
| AdvancedPricingImportExport | ~~Hard require~~ **DONE — removed from composer.json** |
| Fedex | ~~Hard require~~ **DONE — removed from composer.json** |
| Reports | ~~Hard require~~ **DONE — removed from composer.json** |
| Shipping | ~~Hard require~~ **DONE — removed from composer.json** |
| Usps | ~~Hard require~~ **DONE — removed from composer.json** |

---

## Most Referenced CatalogInventory Interfaces

| Interface / Class | # External References | Role |
|-------------------|-----------------------|------|
| `StockConfigurationInterface` | ~15 | Check stock settings (manage stock, show OOS, backorders) |
| `StockRegistryInterface` | ~10 | Get stock item/status by product |
| `StockItemInterface` | ~6 | Stock item data object |
| `StockStatusInterface` | ~5 | Stock status data + IN_STOCK/OUT_OF_STOCK constants |
| `Stock` (model) | ~5 | BACKORDERS_*, STOCK_IN/OUT constants, DEFAULT_STOCK_ID |
| `Stock\Status` (resource model) | ~5 | Join stock status to product collections |
| `StockItemRepositoryInterface` | ~4 | CRUD for stock items |
| `StockItemCriteriaInterfaceFactory` | ~4 | Build stock item search criteria |
| `ParentItemProcessorInterface` | ~3 | Parent stock recalculation (Bundle, Configurable, Grouped) |
| `Indexer\Stock\Action\Full` | ~3 | Stock indexer extension point |

---

## Coupling Pattern Analysis

### Pattern 1: Parent Stock Status Recalculation
**Modules:** Bundle, ConfigurableProduct, GroupedProduct
**Classes:** `ChangeParentStockStatus`, `ParentItemProcessor`
**What it does:** When a child product's stock changes, recalculates parent's stock status.
**Coupling:** 4 CatalogInventory interfaces per module (StockItemInterface, StockConfigurationInterface, StockItemCriteriaInterfaceFactory, StockItemRepositoryInterface)

#### Deep Dive: Code Duplication Analysis

Each product type has 2 classes: `ParentItemProcessor` (thin wrapper implementing the interface) and `ChangeParentStockStatus` (actual logic).

**`ParentItemProcessorInterface`** — defined in `CatalogInventory\Observer`, depends only on `Catalog\Api\Data\ProductInterface`. All 3 implementations are trivial one-liner delegates to `ChangeParentStockStatus::execute()`. The Configurable version still carries deprecated constructor params (`Configurable $configurableType`, `StockItemCriteriaInterfaceFactory`, `StockItemRepositoryInterface`, `StockConfigurationInterface`) that it no longer uses.

**`ChangeParentStockStatus`** — 3 separate implementations sharing identical code:

**Identical across all 3:**

```php
// isNeedToUpdateParent() — copy-pasted verbatim in all 3
private function isNeedToUpdateParent(StockItemInterface $parentStockItem, bool $childrenIsInStock): bool
{
    return $parentStockItem->getIsInStock() !== $childrenIsInStock &&
        ($childrenIsInStock === false || $parentStockItem->getStockStatusChangedAuto());
}
```

```php
// Stock item loading — same logic in all 3
$criteria = $this->criteriaInterfaceFactory->create();
$criteria->setScopeFilter($this->stockConfiguration->getDefaultScopeId());
$criteria->setProductsFilter($productIds);
$stockItemCollection = $this->stockItemRepository->getList($criteria);
```

```php
// Parent update — same in all 3
$parentStockItem->setIsInStock($childrenIsInStock);
$parentStockItem->setStockStatusChangedAuto(1);
$this->stockItemRepository->save($parentStockItem);
```

**Same 4 CatalogInventory dependencies in all 3:**
- `StockItemInterface`
- `StockConfigurationInterface`
- `StockItemCriteriaInterfaceFactory`
- `StockItemRepositoryInterface`

**Only difference — "is children in stock" logic:**

| Module | Children Check | execute() Signature |
|--------|---------------|---------------------|
| **Bundle** | Per-option: at least 1 in-stock child per required option | `execute(array $childrenIds)` |
| **Configurable** | Flat: any single child in-stock = parent in-stock | `execute(array $childrenIds)` |
| **Grouped** | Flat: any single child in-stock = parent in-stock (same as Configurable) | `execute(int $productId)` |

Configurable and Grouped `processStockForParent()` methods are nearly identical. Grouped has an extra `getParentEntityIdsByChild()` that does a raw SQL query instead of using the type model.

#### Recommended Refactoring

Extract an abstract base class in `CatalogInventory` (e.g. `AbstractChangeParentStockStatus`) that handles:
- Stock item loading via criteria factory
- `isNeedToUpdateParent()` check
- Parent stock item save

Each product type module would only provide:
- A method to resolve parent IDs from child IDs (delegates to its own type model)
- A method to determine if children are in-stock (flat check vs per-option check)

This reduces the CatalogInventory coupling surface from 4 interfaces per module to 1 abstract class, and eliminates ~60% of duplicated code across the 3 modules.

Alternatively, `ParentItemProcessorInterface` could be moved out of `CatalogInventory\Observer` to `Catalog\Api` (it only depends on `ProductInterface`), and the stock-specific abstract base would implement it.

### Pattern 2: Stock Status Collection Filtering
**Modules:** ConfigurableProduct, CatalogGraphQl, CatalogSearch, Wishlist
**What it does:** Joins stock_status table to product collections to filter out-of-stock products.
**Coupling:** `StockConfigurationInterface` (to check show_out_of_stock setting) + `Stock\Status` resource model or `StockStatusFilterInterface`.

### Pattern 3: Stock Indexer Extension
**Modules:** Bundle, ConfigurableProduct, GroupedProduct, Elasticsearch
**What it does:** Extends the stock indexer with product-type-specific logic.
**Coupling:** `Indexer\Stock\Action\Full`, `Indexer\Stock\DefaultStock`

### Pattern 4: Stock Registry Lookups
**Modules:** Sales, Quote, Dhl, Ups, Wishlist, QuoteGraphQl, GroupedCatalogInventory
**What it does:** Looks up stock info for a specific product (qty, is_in_stock, min/max sale qty).
**Coupling:** `StockRegistryInterface`

### Pattern 5: Stock Validation in Cart/Checkout
**Modules:** Quote, ConfigurableProductGraphQl, QuoteGraphQl
**What it does:** Validates quantities against stock rules during add-to-cart.
**Coupling:** `StockStateInterface`, `StockStateException`, `StockRegistryInterface`

---

## Decoupling Difficulty Assessment

| Category | Modules | Difficulty | Approach |
|----------|---------|-----------|----------|
| **Composer-only** (no code) | AdvancedPricingImportExport, Fedex, Reports, Shipping, Usps | Trivial | Remove from `require` |
| **Layout XML only** | Downloadable (partial), Wishlist (partial) | Easy | Type-handle layouts, safe to leave or move |
| **Light API usage** (1-3 interfaces) | Dhl, Ups, Quote, Sales, Checkout, Elasticsearch, SalesInventory | Moderate | Make interfaces optional/nullable |
| **Collection filtering** | CatalogSearch, CatalogGraphQl, Wishlist, ConfigurableProduct (partial) | Moderate | Abstract behind a stock filter interface in Catalog |
| **Indexer extension** | Bundle, ConfigurableProduct, GroupedProduct, Elasticsearch | Hard | Needs indexer plugin architecture rework |
| **Deep integration** (10+ refs) | ConfigurableProduct, Bundle, GroupedProduct | Very Hard | Parent stock recalc + indexer + collection filtering — needs a stock abstraction layer |

---

## Recommended Priority

1. ~~**Quick wins (Tier 4):** Remove phantom composer.json dependencies from AdvancedPricingImportExport, Fedex, Reports, Shipping, Usps~~ **DONE**
2. **Shipping carriers:** Dhl, Ups — single `StockRegistryInterface` usage each, easy to make optional
3. **Sales/Quote/Checkout:** Light usage, can be made optional with null checks
4. **Catalog core:** StockDataFilter and CategorySetup — needs careful handling as these are setup/init paths
5. **Product types (Bundle/Configurable/Grouped):** Heaviest coupling — the `ChangeParentStockStatus` pattern is duplicated across all 3 and deeply tied to CatalogInventory internals. Would benefit from a shared stock abstraction layer before decoupling.
6. **GraphQL modules:** Should follow their parent module's decoupling (e.g., decouple ConfigurableProductGraphQl after ConfigurableProduct)
