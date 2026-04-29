# Sales / Shipping / Payment Coupling Report

## Modules Scanned

**Sales family:**
- Sales, SalesAnalytics, SalesInventory, SalesRule, SalesSequence

**Shipping family:**
- Shipping, Dhl, Fedex, Ups, Usps, OfflineShipping

**Payment family:**
- Payment, Paypal, Vault, OfflinePayments

---

## Direction 1: Sales → Shipping/Payment

### Sales Module

**Payment coupling (14+ PHP imports):**

| File | Classes Imported |
|------|-----------------|
| `Block/Adminhtml/Order/Payment.php` | `Payment\Model\Info` |
| `Block/Order/Info.php` | `Payment\Helper\Data` |
| `Ui/Component/Listing/Column/PaymentMethod.php` | `Payment\Helper\Data` |
| `Model/AdminOrder/Create.php` | `Payment\Model\Checks\SpecificationFactory`, `Payment\Model\MethodInterface` |
| `Model/Order/Payment.php` | `Payment\Model\SaleOperationInterface` |
| `Model/OrderRepository.php` | `Payment\Api\Data\PaymentAdditionalInfoInterface`, `PaymentAdditionalInfoInterfaceFactory` |
| `Model/Order/Invoice/Validation/CanRefund.php` | `Payment\Model\InfoInterface`, `Payment\Model\MethodInterface`, hardcoded `Payment\Model\Method\Free` |
| `Model/Order/Payment/Info.php` | `Payment\Model\Method\Substitution`, `Payment\Model\InfoInterface` |
| `Model/Order/Email/Sender/InvoiceSender.php` | `Payment\Helper\Data` |
| `Model/Order/Email/Sender/CreditmemoSender.php` | `Payment\Helper\Data` |
| `Model/Order/Email/Sender/OrderSender.php` | `Payment\Helper\Data` |
| `Model/Order/Email/Sender/ShipmentSender.php` | `Payment\Helper\Data` |
| `Model/Order/Shipment/Sender/EmailSender.php` | `Payment\Helper\Data` |
| `Model/Service/OrderService.php` | `Payment\Gateway\Command\CommandException` |
| `Controller/Adminhtml/Order/Create/Save.php` | `Payment\Model\Method\AbstractMethod` constants |

**DI config:** 6 `Payment\Model\Checks\*` classes in `etc/di.xml`, `Payment\Model\MethodInterface` constants in `etc/adminhtml/di.xml`

**Shipping coupling (3 PHP imports):**

| File | Classes Imported |
|------|-----------------|
| `Block/DataProviders/Email/Shipment/TrackingUrl.php` | `Shipping\Helper\Data` |
| `Block/Adminhtml/Order/AbstractOrder.php` | `Shipping\Helper\Data` |
| `Model/Service/ShipmentService.php` | `Shipping\Model\ShipmentNotifier` |

**Dependencies:**
- composer.json: hard require on both `magento/module-payment` and `magento/module-shipping`
- module.xml: `<sequence>` includes `Magento_Payment`

### SalesRule Module

- composer.json: hard require on both `magento/module-payment` and `magento/module-shipping`
- No direct PHP use statements from Payment/Shipping in production code
- Uses shipping/payment through configuration and indirect references

### SalesAnalytics, SalesInventory, SalesSequence

- No dependencies on Shipping or Payment modules

---

## Direction 2: Shipping → Sales

### Shipping Module — HEAVY (26 PHP imports, 11 XML refs)

| File | # Imports | Sales Classes Used |
|------|-----------|-------------------|
| `Controller/Adminhtml/Order/ShipmentLoader.php` | 7 | `ShipmentTrackCreationInterface`, `ShipmentTrackCreationInterfaceFactory`, `ShipmentItemCreationInterfaceFactory`, `ShipmentRepositoryInterface`, `OrderRepositoryInterface`, `ShipmentDocumentFactory`, `ShipmentItemCreationInterface` |
| `Controller/Adminhtml/Order/Shipment/Save.php` | 2 | `Sales\Helper\Data`, `Shipment\Validation\QuantityValidator` |
| `Controller/Adminhtml/Order/Shipment/MassPrintShippingLabel.php` | 2 | `Shipment\CollectionFactory`, `Order\CollectionFactory` |
| `Controller/Adminhtml/Order/Shipment/AddTrack.php` | 2 | `ShipmentTrackInterfaceFactory`, `ShipmentRepositoryInterface` |
| `Controller/Adminhtml/Order/Shipment/AddComment.php` | 3 | `ShipmentCommentSender`, `Shipment\Comment`, `Shipment\Comment` (Resource) |
| `Controller/Adminhtml/Shipment/MassPrintShippingLabel.php` | 1 | `Shipment\CollectionFactory` |
| `Model/ShipmentNotifier.php` | 2 | `ShipmentSender`, `Status\History\CollectionFactory` |
| `Model/Shipping.php` | 1 | `Order\Shipment` |
| `Model/Info.php` | 1 | `Order\Shipment` |
| `Model/Shipping/Labels.php` | 2 | `Order\Shipment`, `Order\Address` |
| `Model/Shipping/LabelGenerator.php` | 1 | `Order\Shipment` |

**Layout XML:** 11 references to `Sales\Block\Adminhtml\*` blocks in shipment admin layouts

**Assessment:** This coupling is **architectural** — shipment admin controllers and views are in the Shipping module but render Sales blocks and work with Sales data objects. This is by design (Shipping handles the "how to ship" while Sales owns the "what was shipped" data model).

### Dhl — 3 PHP imports

| File | Sales Classes Used |
|------|-------------------|
| `Model/Carrier.php` | `Sales\Model\Order\Shipment`, `Sales\Exception\DocumentValidationException` |
| `Model/Validator/XmlValidator.php` | `Sales\Exception\DocumentValidationException` |

### Fedex — 1 PHP import

| File | Sales Classes Used |
|------|-------------------|
| `Model/Carrier.php` | `Sales\Model\Order` |

### Ups — 1 PHP import

| File | Sales Classes Used |
|------|-------------------|
| `Model/Carrier.php` | `Sales\Model\Order\Shipment` |

### Usps — 0 PHP imports

composer.json requires `magento/module-sales` but no actual code usage found. **Phantom dependency.**

### OfflineShipping — 1 PHP import (SalesRule, not Sales)

| File | Sales Classes Used |
|------|-------------------|
| `Model/SalesRule/Calculator.php` | `SalesRule\Model\Validator` |

composer.json requires both `magento/module-sales` and `magento/module-sales-rule`. module.xml sequences both.

---

## Direction 3: Payment → Sales

### Payment Module — 5 PHP imports

| File | Sales Classes Used |
|------|-------------------|
| `Api/PaymentVerificationInterface.php` | `Sales\Api\Data\OrderPaymentInterface` |
| `Gateway/Data/PaymentDataObjectFactory.php` | `Sales\Model\Order\Payment` |
| `Gateway/Data/Order/OrderAdapter.php` | `Sales\Model\Order` |
| `Gateway/Data/Order/AddressAdapter.php` | `Sales\Api\Data\OrderAddressInterface` |
| `Model/Method/AbstractMethod.php` | `Sales\Model\Order\Payment` |

**XML:**
- `system.xml`: `Sales\Model\Config\Source\Order\Status\Newprocessing` source model
- `events.xml`: listeners for `sales_order_save_before` and `sales_order_status_unassign`

composer.json requires `magento/module-sales`. No module.xml sequence.

### Paypal — 31 PHP imports (HEAVIEST)

| Area | # Imports | Key Sales Classes |
|------|-----------|-------------------|
| Model layer (Express, Payflow, IPN, etc.) | 20 | `Order`, `Order\Payment`, `Payment\Transaction`, `OrderPaymentInterface`, `OrderSender`, `CreditmemoSender`, `OrderMutexInterface`, `TransactionRepositoryInterface` |
| Controllers | 6 | `OrderFactory`, `PaymentFailuresInterface`, `OrderManagementInterface`, `OrderRepositoryInterface` |
| Plugins | 5 | `Order`, `Order\Payment`, `InvoiceRepositoryInterface`, `CanInvoice`, `OrderInterface` |
| Blocks | 4 | `Adminhtml\Order\View`, `Reorder`, `Config`, `Order\Address` |
| Setup | 1 | `SalesSetupFactory` |

**XML:** plugins on `Sales\Model\Order`, `Sales\Model\Order\Payment`, `Sales\Model\Order\Validation\CanInvoice`; event listeners for `sales_order_payment_*` events; ACL refs to `Magento_Sales` resources

composer.json requires `magento/module-sales`. module.xml sequences `Magento_Sales`.

### Vault — 6 PHP imports

| File | Sales Classes Used |
|------|-------------------|
| `Observer/AfterPaymentSaveObserver.php` | `OrderPaymentExtensionInterface`, `OrderPaymentInterface` |
| `Plugin/PaymentVaultAttributesLoad.php` | `OrderPaymentExtensionInterface`, `OrderPaymentInterface`, `OrderPaymentExtensionFactory` |
| `Model/Ui/Adminhtml/TokensConfigProvider.php` | `OrderRepositoryInterface` |
| `Model/PaymentTokenManagement.php` | `OrderPaymentInterface`, `Order\Payment` |
| `Api/PaymentTokenManagementInterface.php` | `OrderPaymentInterface` |
| `Model/Method/Vault.php` | `OrderPaymentExtensionInterfaceFactory`, `OrderPaymentInterface`, `Order\Payment` |

**XML:** extension_attributes for `OrderPaymentInterface`, plugin on `OrderPaymentInterface`, event listener for `sales_order_payment_save_after`

composer.json requires `magento/module-sales`. module.xml sequences `Magento_Sales`.

### OfflinePayments — 0 PHP imports

- `system.xml`: references `Sales\Model\Config\Source\Order\Status\NewStatus` (4 times)
- `events.xml`: listener for `sales_order_payment_save_before`
- No composer.json dependency on Sales. **No module.xml sequence.**

---

## Summary Matrix

### Coupling Density (PHP imports, production code)

| From → To | Sales | Shipping | Payment |
|-----------|-------|----------|---------|
| **Sales** | — | 3 | 14+ |
| **SalesRule** | — | 0 (composer only) | 0 (composer only) |
| **Shipping** | 26 | — | 0 |
| **Dhl** | 3 | *(parent)* | 0 |
| **Fedex** | 1 | *(parent)* | 0 |
| **Ups** | 1 | *(parent)* | 0 |
| **Usps** | 0 (phantom) | *(parent)* | 0 |
| **OfflineShipping** | 0 | *(parent)* | 0 |
| **Payment** | 5 | 0 | — |
| **Paypal** | 31 | 0 | *(parent)* |
| **Vault** | 6 | 0 | *(parent)* |
| **OfflinePayments** | 0 (XML only) | 0 | *(parent)* |

### Phantom Dependencies (composer.json require, zero code usage)

| Module | Phantom Dependency |
|--------|--------------------|
| Usps | `magento/module-sales` |
| SalesRule | `magento/module-payment` (no direct PHP imports) |
| SalesRule | `magento/module-shipping` (no direct PHP imports) |

---

## Key Architectural Observations

### 1. Sales ↔ Payment is a Circular Dependency
- Sales requires Payment (14+ PHP imports, composer require, module.xml sequence)
- Payment requires Sales (5 PHP imports, composer require, events)
- This is the tightest bidirectional coupling in this group

### 2. Sales ↔ Shipping is Architecturally Split
- Sales has light coupling TO Shipping (3 imports: helper + notifier)
- Shipping has heavy coupling TO Sales (26 imports) because Shipping admin controllers manage Sales shipment entities
- The admin UI for shipments lives in Shipping but renders Sales blocks — this is by design but creates strong coupling

### 3. Paypal is Deeply Embedded in Sales
- 31 PHP imports from Sales — Paypal directly manipulates Order, Payment, Invoice, and Transaction objects
- Uses Sales email senders, order mutexes, payment state commands
- This is NOT a thin integration — Paypal is wired into Sales internals

### 4. Carrier Modules (Dhl, Fedex, Ups) Have Light Sales Coupling
- Each carrier only needs `Order\Shipment` for label generation
- Dhl also uses `DocumentValidationException`
- Usps has zero code usage — phantom dependency

### 5. OfflinePayments is Nearly Decoupled
- Zero PHP imports from Sales
- Only XML config references (system.xml source models + event observer)
- Could be fully decoupled with minor XML adjustments

---

## Decoupling Difficulty Assessment

| Category | Modules | Difficulty | Notes |
|----------|---------|-----------|-------|
| **Phantom deps** | Usps→Sales, SalesRule→Payment, SalesRule→Shipping | Trivial | Remove from composer.json |
| **XML-only** | OfflinePayments→Sales | Easy | Move source model refs, event could stay |
| **Light carrier** | Fedex→Sales, Ups→Sales | Easy | 1 import each, could abstract shipment interface |
| **Moderate carrier** | Dhl→Sales | Easy | 3 imports, same approach |
| **Payment base** | Payment→Sales | Hard | API interfaces reference Sales order payment — foundational |
| **Vault** | Vault→Sales | Hard | Extension attributes on OrderPaymentInterface |
| **Sales→Payment** | Sales→Payment | Very Hard | 14+ imports, DI checks, email senders — deeply architectural |
| **Sales→Shipping** | Sales→Shipping | Moderate | Only 3 imports but includes ShipmentNotifier |
| **Shipping→Sales** | Shipping→Sales | Very Hard | 26 imports + 11 XML layout refs — admin UI is built on Sales blocks |
| **Paypal→Sales** | Paypal→Sales | Extreme | 31 imports across models, controllers, plugins — complete rewrite needed |

## Recommended Priority

1. **Quick wins:** Remove phantom deps from Usps, SalesRule (3 composer.json changes)
2. **OfflinePayments:** Nearly decoupled already, small XML work
3. **Carriers (Fedex, Ups, Dhl):** Abstract `Order\Shipment` behind a carrier-facing interface
4. **Sales↔Payment circular:** This is architectural debt — both modules are designed as a unit. Decoupling would require extracting shared interfaces into a new base module (e.g. `SalesPaymentApi`)
5. **Shipping→Sales admin UI:** The 26 imports are mostly admin controllers that build Sales shipment entities. Would need a complete admin architecture rethink to decouple.
6. **Paypal→Sales:** Do not attempt without first resolving the Sales↔Payment circular dependency
