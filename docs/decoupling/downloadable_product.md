# Downloadable Product Decoupling Report

## Modules Scanned

- `Magento_Downloadable` (`app/code/Magento/Downloadable`)
- `Magento_DownloadableGraphQl` (`app/code/Magento/DownloadableGraphQl`)
- `Magento_DownloadableImportExport` (`app/code/Magento/DownloadableImportExport`)

---

## Constants Inventory

### Downloadable (Main Module)

| Constant | Class | Value | Visibility |
|----------|-------|-------|------------|
| `TYPE_DOWNLOADABLE` | `Model\Product\Type` | `'downloadable'` | public |
| `PRICE_CODE` | `Pricing\Price\LinkPrice` | `'link_price'` | public |
| `LINK_TYPE_URL` | `Helper\Download` | `'url'` | public |
| `LINK_TYPE_FILE` | `Helper\Download` | `'file'` | public |
| `XML_PATH_CONTENT_DISPOSITION` | `Helper\Download` | `'catalog/downloadable/content_disposition'` | public |
| `CODE_IS_DOWNLOADABLE` | `Api\Data\ProductAttributeInterface` | `'is_downloadable'` | public |
| `DOWNLOADABLE_LINKS` | `Api\Data\DownloadableOptionInterface` | `'downloadable_links'` | public |
| `XML_PATH_LINKS_TITLE` | `Model\Link` | `'catalog/downloadable/links_title'` | public |
| `XML_PATH_DEFAULT_DOWNLOADS_NUMBER` | `Model\Link` | `'catalog/downloadable/downloads_number'` | public |
| `XML_PATH_TARGET_NEW_WINDOW` | `Model\Link` | `'catalog/downloadable/links_target_new_window'` | public |
| `XML_PATH_CONFIG_IS_SHAREABLE` | `Model\Link` | `'catalog/downloadable/shareable'` | public |
| `LINK_SHAREABLE_YES` | `Model\Link` | `1` | public |
| `LINK_SHAREABLE_NO` | `Model\Link` | `0` | public |
| `LINK_SHAREABLE_CONFIG` | `Model\Link` | `2` | public |
| `KEY_TITLE` | `Model\Link` | `'title'` | public |
| `KEY_SORT_ORDER` | `Model\Link` | `'sort_order'` | public |
| `KEY_IS_SHAREABLE` | `Model\Link` | `'is_shareable'` | public |
| `KEY_PRICE` | `Model\Link` | `'price'` | public |
| `KEY_NUMBER_OF_DOWNLOADS` | `Model\Link` | `'number_of_downloads'` | public |
| `KEY_LINK_TYPE` | `Model\Link` | `'link_type'` | public |
| `KEY_LINK_FILE` | `Model\Link` | `'link_file'` | public |
| `KEY_LINK_FILE_CONTENT` | `Model\Link` | `'link_file_content'` | public |
| `KEY_LINK_URL` | `Model\Link` | `'link_url'` | public |
| `KEY_SAMPLE_TYPE` | `Model\Link` | `'sample_type'` | public |
| `KEY_SAMPLE_FILE` | `Model\Link` | `'sample_file'` | public |
| `KEY_SAMPLE_FILE_CONTENT` | `Model\Link` | `'sample_file_content'` | public |
| `KEY_SAMPLE_URL` | `Model\Link` | `'sample_url'` | public |
| `XML_PATH_SAMPLES_TITLE` | `Model\Sample` | `'catalog/downloadable/samples_title'` | public |
| `KEY_TITLE` | `Model\Sample` | `'title'` | public |
| `KEY_SORT_ORDER` | `Model\Sample` | `'sort_order'` | public |
| `KEY_SAMPLE_TYPE` | `Model\Sample` | `'sample_type'` | public |
| `KEY_SAMPLE_FILE` | `Model\Sample` | `'sample_file'` | public |
| `KEY_SAMPLE_FILE_CONTENT` | `Model\Sample` | `'sample_file_content'` | public |
| `KEY_SAMPLE_URL` | `Model\Sample` | `'sample_url'` | public |
| `XML_PATH_ORDER_ITEM_STATUS` | `Model\Link\Purchased\Item` | `'catalog/downloadable/order_item_status'` | public |
| `LINK_STATUS_PENDING` | `Model\Link\Purchased\Item` | `'pending'` | public |
| `LINK_STATUS_AVAILABLE` | `Model\Link\Purchased\Item` | `'available'` | public |
| `LINK_STATUS_EXPIRED` | `Model\Link\Purchased\Item` | `'expired'` | public |
| `LINK_STATUS_PENDING_PAYMENT` | `Model\Link\Purchased\Item` | `'pending_payment'` | public |
| `LINK_STATUS_PAYMENT_REVIEW` | `Model\Link\Purchased\Item` | `'payment_review'` | public |
| `DEFAULT_MIME_TYPE` | `Model\File\ContentUploader` | `'application/octet-stream'` | public |
| `PARAM_DOWNLOADABLE_DOMAINS` | `Model\Url\DomainValidator` | `'downloadable_domains'` | public |
| `DATA` | `Model\File\Content` | `'file_data'` | public |
| `NAME` | `Model\File\Content` | `'name'` | public |
| `DATA_KEY` | `Model\Product\TypeHandler\Sample` | `'sample'` | public |
| `IDENTIFIER_KEY` | `Model\Product\TypeHandler\Sample` | `'sample_id'` | public |
| `FIELD_IS_DELETE` | `Model\Product\TypeHandler\AbstractTypeHandler` | `'is_delete'` | public |
| `FIELD_FILE` | `Model\Product\TypeHandler\AbstractTypeHandler` | `'file'` | public |
| `CHILDREN_PATH` | `Ui\..\Composite` | `'downloadable/children'` | public |
| `CONTAINER_LINKS` | `Ui\..\Composite` | `'container_links'` | public |
| `CONTAINER_SAMPLES` | `Ui\..\Composite` | `'container_samples'` | public |
| `INPUT_KEY_DOMAINS` | `Console\Command\DomainsAddCommand` | `'domains'` | public |
| `INPUT_KEY_DOMAINS` | `Console\Command\DomainsRemoveCommand` | `'domains'` | public |
| `ADMIN_RESOURCE` | `Controller\Adminhtml\Downloadable\File` | `'Magento_Catalog::products'` | public |

### DownloadableGraphQl

| Constant | Class | Value | Visibility |
|----------|-------|-------|------------|
| `DOWNLOADABLE_PRODUCT` | `Model\DownloadableProductTypeResolver` | `'DownloadableProduct'` | public |
| `OPTION_TYPE` | `Resolver\Product\DownloadableLinksValueUid` | `'downloadable'` | private |

### DownloadableImportExport

| Constant | Class | Value | Visibility |
|----------|-------|-------|------------|
| `PAIR_VALUE_SEPARATOR` | `Import\Product\Type\Downloadable` | `'='` | public |
| `DEFAULT_SORT_ORDER` | `Import\Product\Type\Downloadable` | `0` | public |
| `DEFAULT_NUMBER_OF_DOWNLOADS` | `Import\Product\Type\Downloadable` | `0` | public |
| `DEFAULT_IS_SHAREABLE` | `Import\Product\Type\Downloadable` | `2` | public |
| `DEFAULT_WEBSITE_ID` | `Import\Product\Type\Downloadable` | `0` | public |
| `DOWNLOADABLE_PATCH_SAMPLES` | `Import\Product\Type\Downloadable` | `'downloadable/files/samples'` | public |
| `DOWNLOADABLE_PATCH_LINKS` | `Import\Product\Type\Downloadable` | `'downloadable/files/links'` | public |
| `DOWNLOADABLE_PATCH_LINK_SAMPLES` | `Import\Product\Type\Downloadable` | `'downloadable/files/link_samples'` | public |
| `URL_OPTION_VALUE` | `Import\Product\Type\Downloadable` | `'url'` | public |
| `FILE_OPTION_VALUE` | `Import\Product\Type\Downloadable` | `'file'` | public |
| `COL_DOWNLOADABLE_SAMPLES` | `Import\Product\Type\Downloadable` | `'downloadable_samples'` | public |
| `COL_DOWNLOADABLE_LINKS` | `Import\Product\Type\Downloadable` | `'downloadable_links'` | public |
| `DEFAULT_GROUP_TITLE` | `Import\Product\Type\Downloadable` | `''` | public |
| `DEFAULT_PURCHASED_SEPARATELY` | `Import\Product\Type\Downloadable` | `1` | public |
| `ERROR_OPTIONS_NOT_FOUND` | `Import\Product\Type\Downloadable` | `'optionsNotFound'` | public |
| `ERROR_GROUP_TITLE_NOT_FOUND` | `Import\Product\Type\Downloadable` | `'groupTitleNotFound'` | public |
| `ERROR_OPTION_NO_TITLE` | `Import\Product\Type\Downloadable` | `'optionNoTitle'` | public |
| `ERROR_MOVE_FILE` | `Import\Product\Type\Downloadable` | `'moveFile'` | public |
| `ERROR_COLS_IS_EMPTY` | `Import\Product\Type\Downloadable` | `'emptyOptions'` | public |

---

## Cross-Module Constant Usage (Production Code)

**No Downloadable constants are referenced by external modules.** All constant usage is internal within the Downloadable module family.

---

## External Module Coupling to Downloadable

### PHP `use` Imports (Production Code)

| External Module | File | Import |
|-----------------|------|--------|
| CustomerDownloadableGraphQl | `Model/Resolver/CustomerDownloadableProducts.php` | `Magento\DownloadableGraphQl\Model\ResourceModel\GetPurchasedDownloadableProducts` |

### Hardcoded String `'downloadable'` Outside Downloadable Modules

| Module | File | Usage |
|--------|------|-------|
| Sales | `Block/Adminhtml/Order/Create/Search/Grid/Renderer/Price.php:21` | `if ($row->getTypeId() == 'downloadable')` |
| Reports | `Block/Adminhtml/Product/Downloads/Grid.php:73` | `->addAttributeToFilter('type_id', ['downloadable'])` |
| Wishlist | `Model/Wishlist/BuyRequest/DownloadableLinkDataProvider.php:17` | `private const PROVIDER_OPTION_TYPE = 'downloadable'` |
| QuoteDownloadableLinks | `Model/Cart/BuyRequest/DownloadableLinkDataProvider.php:19` | `private const OPTION_TYPE = 'downloadable'` |

### XML References to Downloadable Classes

| External Module | File | Reference |
|-----------------|------|-----------|
| RemoteStorage | `etc/di.xml:152` | `<type name="Magento\DownloadableImportExport\Helper\Uploader">` |
| Wishlist | `etc/di.xml:57` | `<virtualType name="Magento\Downloadable\Pricing\Price\Pool">` — injects wishlist price |
| Wishlist | `view/frontend/layout/wishlist_index_configure_type_downloadable.xml` | Renders `Magento\Downloadable\Block\Catalog\Product\Samples`, `View\Type`, `Product\Links` blocks |
| Msrp | `view/frontend/layout/catalog_product_view_type_downloadable.xml:10` | `<referenceBlock name="product.info.downloadable.options">` |
| QuoteGraphQl | `etc/di.xml:18` | `<item name="downloadable" xsi:type="string">DownloadableCartItem</item>` |
| WishlistGraphQl | `etc/graphql/di.xml:22` | `<item name="downloadable" xsi:type="object">...DownloadableLinkDataProvider</item>` |
| QuoteDownloadableLinks | `etc/graphql/di.xml:12` | `<item name="downloadable" xsi:type="object">...DownloadableLinkDataProvider</item>` |

### Composer `require` Dependencies on `magento/module-downloadable`

| Module | Type |
|--------|------|
| Msrp | hard require |
| RemoteStorage | hard require |
| Reports | hard require |
| QuoteGraphQl | hard require |
| GiftMessageGraphQl | hard require (no actual code reference found — phantom dependency) |
| Wishlist | **suggest** (already optional) |

---

## Modules Selected for Decoupling

### 1. Magento_Sales — EASY

**Current state:**
- `composer.json`: no dependency on `magento/module-downloadable`
- `module.xml`: no sequence dependency

**Coupling points (1 file):**

| File | Type | Details |
|------|------|---------|
| `Block/Adminhtml/Order/Create/Search/Grid/Renderer/Price.php:21` | Hardcoded string | `if ($row->getTypeId() == 'downloadable')` — adds link price to product price in admin order create grid |

**Decoupling approach:** Replace type-specific check with a generic approach — either make downloadable price handling pluggable, or use product type's price model directly.

---

### 2. Magento_Reports — EASY

**Current state:**
- `composer.json`: hard require on `magento/module-downloadable`
- `module.xml`: no sequence dependency

**Coupling points (1 file):**

| File | Type | Details |
|------|------|---------|
| `Block/Adminhtml/Product/Downloads/Grid.php:73` | Hardcoded string | `->addAttributeToFilter('type_id', ['downloadable'])` — filters grid to only show downloadable products |

**Decoupling approach:** The downloads grid is specifically for downloadable products. The filter uses a string, not a class constant. Move `magento/module-downloadable` from `require` to `suggest`. The grid will simply return no results if downloadable products don't exist.

---

### 3. Magento_Wishlist — MODERATE (already partially decoupled)

**Current state:**
- `composer.json`: `magento/module-downloadable` already in `suggest`
- `module.xml`: no sequence dependency

**Coupling points:**

| File / Location | Type | Details |
|-----------------|------|---------|
| `Model/Wishlist/BuyRequest/DownloadableLinkDataProvider.php:17` | Hardcoded string | `private const PROVIDER_OPTION_TYPE = 'downloadable'` — string comparison only, no class import |
| `Pricing/ConfiguredPrice/Downloadable.php` | Class name | Named after downloadable, likely self-contained (similar pattern to ConfigurableProduct) |
| `etc/di.xml:57` | XML virtualType | `<virtualType name="Magento\Downloadable\Pricing\Price\Pool">` — injects wishlist pricing into downloadable's price pool |
| `view/frontend/layout/wishlist_index_configure_type_downloadable.xml` | Layout XML | Renders Downloadable blocks (Samples, Type, Links) |

**Decoupling approach:** Same pattern as Configurable — PHP is already clean, coupling is XML-only. Layout is type-handle-based (loaded only for downloadable products). The `di.xml` virtualType could be moved to the Downloadable module's own DI.

---

### 4. Magento_Msrp — EASY

**Current state:**
- `composer.json`: hard require on `magento/module-downloadable`
- `module.xml`: no sequence dependency

**Coupling points (1 file):**

| File | Type | Details |
|------|------|---------|
| `view/frontend/layout/catalog_product_view_type_downloadable.xml:10` | Layout XML | `<referenceBlock name="product.info.downloadable.options">` — adds MSRP notice to downloadable options |

**Decoupling approach:** This is a type-specific layout handle (`catalog_product_view_type_downloadable`). It only loads when viewing a downloadable product page. Move `magento/module-downloadable` to `suggest`. The layout handle simply won't be triggered when Downloadable is absent.

---

### 5. Magento_RemoteStorage — MODERATE

**Current state:**
- `composer.json`: hard require on both `magento/module-downloadable` and `magento/module-downloadable-import-export`

**Coupling points (1 file):**

| File | Type | Details |
|------|------|---------|
| `etc/di.xml:152` | XML type override | `<type name="Magento\DownloadableImportExport\Helper\Uploader">` — configures filesystem driver for downloadable import uploads |

**Decoupling approach:** The DI `<type>` configuration is only applied when the target class exists. Move dependencies to `suggest`. Magento's DI will safely ignore the type configuration when DownloadableImportExport is not installed.

---

### 6. Magento_GiftMessageGraphQl — EASY (phantom dependency)

**Current state:**
- `composer.json`: hard require on `magento/module-downloadable`
- No actual code, XML, or string references to Downloadable found

**Coupling points:** None found. This appears to be a phantom/unnecessary dependency.

**Decoupling approach:** Simply remove `magento/module-downloadable` from `require`.

---

## Summary

| Module | Difficulty | Key Change |
|--------|-----------|------------|
| **GiftMessageGraphQl** | Trivial | Remove phantom `require` dependency |
| **Sales** | Easy | Hardcoded `'downloadable'` string in 1 admin block |
| **Reports** | Easy | Hardcoded `'downloadable'` string filter, move to `suggest` |
| **Msrp** | Easy | Layout handle only, move to `suggest` |
| **RemoteStorage** | Easy | DI type config only, move to `suggest` |
| **Wishlist** | Already done | Already `suggest`, XML coupling is type-handle-based |
