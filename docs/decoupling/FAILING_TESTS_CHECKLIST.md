# Failing Integration Tests Checklist

> All failures are pre-existing in 2.4-develop branch (not caused by decoupling).
> Total: 3716 tests, 14977 assertions. Failures: 42, Errors: 18.

---

## 1. Bundle — testIsSalable (22 failures)

**Root cause:** Stock/backorder salability logic returns wrong result for bundle selections
when child product is out of stock with various backorder settings.

**Test file:** `dev/tests/integration/testsuite/Magento/Bundle/Model/ProductTest.php:203`

**Assertion:** `$this->assertEquals($isSalable, $bundle->isSalable())` — expects `false`, gets `true`

**Failing data sets:**
- [ ] selectionQty: 5, qty: 0, isInStock: 0, manageStock: 1, backorders: 0
- [ ] selectionQty: 10, qty: 0, isInStock: 0, manageStock: 1, backorders: 0
- [ ] selectionQty: 15, qty: 0, isInStock: 0, manageStock: 1, backorders: 0
- [ ] selectionQty: 5, qty: 0, isInStock: 0, manageStock: 1, backorders: 1
- [ ] selectionQty: 10, qty: 0, isInStock: 0, manageStock: 1, backorders: 1
- [ ] selectionQty: 15, qty: 0, isInStock: 0, manageStock: 1, backorders: 1
- [ ] selectionQty: 5, qty: 0, isInStock: 0, manageStock: 1, backorders: 2
- [ ] selectionQty: 10, qty: 0, isInStock: 0, manageStock: 1, backorders: 2
- [ ] selectionQty: 15, qty: 0, isInStock: 0, manageStock: 1, backorders: 2
- [ ] selectionQty: 5, qty: 0, isInStock: 1, manageStock: 1, backorders: 0
- [ ] selectionQty: 10, qty: 0, isInStock: 1, manageStock: 1, backorders: 0
- [ ] selectionQty: 15, qty: 0, isInStock: 1, manageStock: 1, backorders: 0
- [ ] selectionQty: 5, qty: 10, isInStock: 0, manageStock: 1, backorders: 0
- [ ] selectionQty: 10, qty: 10, isInStock: 0, manageStock: 1, backorders: 0
- [ ] selectionQty: 15, qty: 10, isInStock: 0, manageStock: 1, backorders: 0
- [ ] selectionQty: 5, qty: 10, isInStock: 0, manageStock: 1, backorders: 1
- [ ] selectionQty: 10, qty: 10, isInStock: 0, manageStock: 1, backorders: 1
- [ ] selectionQty: 15, qty: 10, isInStock: 0, manageStock: 1, backorders: 1
- [ ] selectionQty: 5, qty: 10, isInStock: 0, manageStock: 1, backorders: 2
- [ ] selectionQty: 10, qty: 10, isInStock: 0, manageStock: 1, backorders: 2
- [ ] selectionQty: 15, qty: 10, isInStock: 0, manageStock: 1, backorders: 2
- [ ] selectionQty: 15, qty: 10, isInStock: 1, manageStock: 1, backorders: 0

---

## 2. Bundle — IsSaleableTest + PriceTest (3 failures)

- [ ] `dev/tests/integration/testsuite/Magento/Bundle/Model/Product/IsSaleableTest.php` — testIsSaleableOnBundleWithNotEnoughQtyOfSelection
- [ ] `dev/tests/integration/testsuite/Magento/Bundle/Model/ResourceModel/Indexer/PriceTest.php:84` — testBundleDynamicPriceWhenShowOutOfStockIsDisabled
- [ ] `dev/tests/integration/testsuite/Magento/Bundle/Model/ResourceModel/Indexer/PriceTest.php:157` — testBundleDynamicPriceWhenShowOutOfStockIsEnabled

**Root cause:** Bundle price indexer and salability calculation bugs with OOS children.

---

## 3. Downloadable — File Upload/ACL (11 failures)

**Root cause:** Admin session/authentication issues in test sandbox. All tests fail at `AbstractBackendController.php:76` (session setup).

**Test file:** `dev/tests/integration/testsuite/Magento/Downloadable/Controller/Adminhtml/Downloadable/FileTest.php:29`

- [ ] testAclHasAccess
- [ ] testAclNoAccess
- [ ] testUploadAction
- [ ] testUploadProhibitedExtensions — data set #0 (sample.php)
- [ ] testUploadProhibitedExtensions — data set #1 (sample.php3)
- [ ] testUploadProhibitedExtensions — data set #2 (sample.php4)
- [ ] testUploadProhibitedExtensions — data set #3 (sample.php5)
- [ ] testUploadProhibitedExtensions — data set #4 (sample.php7)
- [ ] testUploadWrongUploadType — data set #0
- [ ] testUploadWrongUploadType — data set #1

---

## 4. Downloadable — Observers (2 failures)

- [ ] `dev/tests/integration/testsuite/Magento/Downloadable/Block/Sales/Order/Email/Items/Order/DownloadableTest.php` — testShouldSendDownloadableLinksInTheEmail
- [ ] `dev/tests/integration/testsuite/Magento/Downloadable/Model/Observer/SetLinkStatusObserverTest.php` — testCheckStatusOnOrderCancel

**Root cause:** Fixture rollback issues (`quote_with_configurable_downloadable_product.php` unable to revert).

---

## 5. Elasticsearch (2 failures)

- [ ] `dev/tests/integration/testsuite/Magento/Elasticsearch/SearchAdapter/AdapterTest.php:546` — testAdvancedSearchCompositeProductWithOutOfStockOption
- [ ] `dev/tests/integration/testsuite/Magento/Elasticsearch/SearchAdapter/AdapterTest.php` — testAdvancedSearchCompositeProductWithDisabledChild

**Root cause:** Search indexer doesn't correctly filter composite products with OOS/disabled children.

---

## 6. ConfigurableProduct (2 failures)

- [ ] `dev/tests/integration/testsuite/Magento/ConfigurableProduct/Model/OptionRepositoryTest.php` — testGetListWithExtensionAttributes
- [ ] `dev/tests/integration/testsuite/Magento/ConfigurableProduct/Pricing/Price/ConfigurablePriceTest.php` — testGetProductMinimalPriceIfOneOfChildIsOutOfStock

**Root cause:** Extension attributes list mismatch; price calculation with OOS child.

---

## 7. CatalogImportExport (3 failures)

- [ ] `dev/tests/integration/testsuite/Magento/CatalogImportExport/Model/Export/ProductTest.php` — testExportWithStock
- [ ] `dev/tests/integration/testsuite/Magento/CatalogImportExport/Model/Export/ProductTest.php` — testFilterByQuantityAndStockStatus (data set #1)
- [ ] `dev/tests/integration/testsuite/Magento/CatalogImportExport/Model/Export/ProductTest.php` — testSaveLongNames

**Root cause:** CSV export header column order mismatch — expects specific column order that doesn't match runtime output.

---

## 8. Sales (2 failures)

- [ ] `dev/tests/integration/testsuite/Magento/Sales/Model/ResourceModel/Order/Grid/CollectionTest.php` — testRefreshBySchedule (data set "Invoice Grid")
- [ ] `dev/tests/integration/testsuite/Magento/Sales/Model/Order/Email/Sender/InvoiceSenderTest.php` — testInvoiceEmailSenderExecute

**Root cause:** Grid refresh timing issue; email sender mock/stub mismatch.

---

## 9. Quote (1 failure)

- [ ] `dev/tests/integration/testsuite/Magento/Quote/Model/QuoteManagementTest.php` — testAddProductToOrderFromShoppingCart (data set "as_json")

**Root cause:** Quote-to-order conversion with JSON format cart data.

---

## 10. Checkout (1 failure)

- [ ] `dev/tests/integration/testsuite/Magento/Checkout/Model/Cart/ReindexTest.php` — testReindexRowAfterUpdateStockStatus

**Root cause:** Stock status reindex after stock update doesn't propagate correctly in test.

---

## 11. CatalogSearch / ConfigurableProductGraphQl (1 failure)

- [ ] `dev/tests/integration/testsuite/Magento/ConfigurableProduct/Model/Product/Type/Configurable/StockTest.php` — testOutOfStockProductWithDisabledConfigView

**Root cause:** Configurable product visibility when all children OOS with config setting.

---

## 12. PHPUnit 12 Abstract Class Warnings (17 errors)

**Root cause:** PHPUnit 12 reports abstract test classes as errors. Not actual failures.

- [ ] `testsuite/Magento/Catalog/Block/Adminhtml/Product/Edit/Tab/Alerts/AbstractAlertTest.php`
- [ ] `testsuite/Magento/Catalog/Block/Product/ProductList/AbstractLinksTest.php`
- [ ] `testsuite/Magento/Catalog/Block/Product/View/AbstractCurrencyTest.php`
- [ ] `testsuite/Magento/Catalog/Block/Product/View/Attribute/AbstractAttributeTest.php`
- [ ] `testsuite/Magento/Catalog/Block/Product/View/Options/AbstractRenderCustomOptionsTest.php`
- [ ] `testsuite/Magento/Catalog/Controller/Adminhtml/Product/AbstractAlertTest.php`
- [ ] `testsuite/Magento/Catalog/Controller/Adminhtml/Product/Attribute/Delete/AbstractDeleteAttributeControllerTest.php`
- [ ] `testsuite/Magento/Catalog/Controller/Adminhtml/Product/Attribute/Save/AbstractSaveAttributeTest.php`
- [ ] `testsuite/Magento/Catalog/Controller/Adminhtml/Product/Attribute/Update/AbstractUpdateAttributeTest.php`
- [ ] `testsuite/Magento/Catalog/Model/Product/Attribute/Save/AbstractAttributeTest.php`
- [ ] `testsuite/Magento/Catalog/Ui/DataProvider/Product/Form/Modifier/AbstractEavTest.php`
- [ ] `testsuite/Magento/Catalog/Ui/DataProvider/Product/Related/AbstractRelationsDataProviderTest.php`
- [ ] `testsuite/Magento/Bundle/Block/Catalog/Product/View/Type/AbstractBundleOptionsViewTest.php`
- [ ] `testsuite/Magento/Bundle/Controller/Adminhtml/Product/AbstractBundleProductSaveTest.php`
- [ ] `testsuite/Magento/Sales/Block/Adminhtml/Order/Create/AbstractAddressFormTest.php`
- [ ] `testsuite/Magento/Sales/Controller/Adminhtml/Order/Invoice/AbstractInvoiceControllerTest.php`
- [ ] `testsuite/Magento/Sales/Model/AbstractCollectorPositionsTest.php`

**Fix:** Add `#[DoesNotPerformAssertions]` or exclude abstract classes from test runner config.

---

## Summary

| Priority | Category | Count | Difficulty |
|----------|----------|-------|------------|
| P1 | Bundle isSalable stock logic | 22 | Medium — fix `isSalable()` backorder/qty logic |
| P2 | PHPUnit 12 abstract class errors | 17 | Easy — phpunit.xml exclude or attribute |
| P3 | Downloadable file upload/ACL | 11 | Medium — fix admin session in test sandbox |
| P3 | CatalogImportExport CSV headers | 3 | Easy — update expected CSV header in test |
| P4 | Bundle pricing OOS | 3 | Medium — price indexer with OOS children |
| P4 | Elasticsearch OOS search | 2 | Medium — search indexer composite filter |
| P4 | ConfigurableProduct | 2 | Medium — extension attrs + price OOS |
| P4 | Downloadable observers | 2 | Medium — fixture rollback |
| P5 | Sales/Quote/Checkout | 4 | Mixed — various integration issues |
