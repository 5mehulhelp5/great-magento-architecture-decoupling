# Payment ↔ Sales Circular Dependency — Decoupling Plan

## Problem

Payment and Sales have a circular dependency:
- **Sales → Payment:** 14+ PHP imports, composer.json require, module.xml sequence — this is correct, Sales needs Payment
- **Payment → Sales:** 5 PHP imports, 2 event observers, 1 XML source model, composer.json require — this should be broken

The desired direction is **one-way: Sales → Payment**. Payment should not depend on Sales.

---

## Current Payment → Sales Coupling Points

| # | File | Sales Class Used | Purpose |
|---|------|-----------------|---------|
| 1 | `Api/PaymentVerificationInterface.php` | `Sales\Api\Data\OrderPaymentInterface` | Method param type in API interface |
| 2 | `Gateway/Data/PaymentDataObjectFactory.php` | `Sales\Model\Order\Payment` | `instanceof` check: order payment vs quote payment |
| 3 | `Gateway/Data/Order/OrderAdapter.php` | `Sales\Model\Order` | Wraps Order behind `OrderAdapterInterface` |
| 4 | `Gateway/Data/Order/AddressAdapter.php` | `Sales\Api\Data\OrderAddressInterface` | Wraps address behind `AddressAdapterInterface` |
| 5 | `Model/Method/AbstractMethod.php` | `Sales\Model\Order\Payment` | **Deprecated** class — `@see Payment\Model\Method\Adapter` |
| 6 | `Observer/SalesOrderBeforeSaveObserver.php` | `Sales\Model\Order` (via event) | Sets `forcedCanCreditmemo` flag for free payment method |
| 7 | `Observer/UpdateOrderStatusForPaymentMethodsObserver.php` | `Sales\Model\Order\Config`, `Sales\Model\Order::STATE_NEW` | Resets payment method order_status config when status is unassigned |
| 8 | `etc/adminhtml/system.xml` | `Sales\Model\Config\Source\Order\Status\Newprocessing` | Admin config source model for order status dropdown |

### Key Insight

Payment already defines its own abstraction layer that does NOT depend on Sales:
- `Payment\Gateway\Data\OrderAdapterInterface` — generic order data
- `Payment\Gateway\Data\AddressAdapterInterface` — generic address data
- `Payment\Model\InfoInterface` — generic payment info

The problem is that the *implementations* of these interfaces (`OrderAdapter`, `AddressAdapter`) wrap concrete Sales models. The interfaces are clean — only the adapter classes and the factory create the coupling.

---

## Options Evaluated

### Option A: Move Gateway Adapters to Sales (Recommended — First Step)

Move the concrete adapter implementations from Payment to Sales:

**Files to move FROM `Payment/Gateway/Data/Order/` TO `Sales/Model/Payment/Gateway/`:**
- `OrderAdapter.php`
- `OrderAdapterFactory.php`
- `AddressAdapter.php`
- `AddressAdapterFactory.php`

**Interfaces stay in Payment** (no change):
- `Payment\Gateway\Data\OrderAdapterInterface`
- `Payment\Gateway\Data\AddressAdapterInterface`

**Refactor `PaymentDataObjectFactory.php`:**
- Replace `instanceof Sales\Model\Order\Payment` with a check that doesn't import Sales
- Options: check for method `getOrder()` via duck typing, or introduce a marker interface in Payment (e.g. `OrderPaymentInfoInterface extends InfoInterface`)

**Wire via DI:** Add preferences in Sales `di.xml`:
```xml
<preference for="Magento\Payment\Gateway\Data\OrderAdapterInterface"
            type="Magento\Sales\Model\Payment\Gateway\OrderAdapter" />
```

**Impact:** 4 files moved, 1 file refactored. No API break — interfaces unchanged. DI preferences added in Sales.

---

### Option B: Extract a Bridge Module (Cleanest, More Overhead)

Create a new `Magento_PaymentSales` bridge module containing:
- Order/Address adapters (bridge Payment interfaces to Sales models)
- Both event observers (`SalesOrderBeforeSaveObserver`, `UpdateOrderStatusForPaymentMethodsObserver`)
- `PaymentVerificationInterface` (takes `OrderPaymentInterface` param — a Sales type)
- The `system.xml` source model reference

Payment module drops to **zero Sales dependency**. Bridge module requires both.

**Impact:** New module, ~6 files moved. Cleanest separation but adds another module.

---

### Option C: Make Sales Optional (Pragmatic, Minimal Change)

Keep files in Payment but:
1. `AbstractMethod.php` — deprecated, leave as-is
2. Adapters — DI only loads them when Sales is present; move `magento/module-sales` from `require` to `suggest`
3. `PaymentVerificationInterface` — `OrderPaymentInterface` in the signature is essentially a payment-domain type that lives in Sales; accept as necessary
4. Observers — move to Sales `events.xml` (they only fire on Sales events anyway)

**Impact:** Minimal code change, composer.json update, observer relocation.

---

## Recommended Approach: Option A, Then Observers

### Phase 1: Move Adapters to Sales

1. Move `OrderAdapter`, `AddressAdapter` + factories to `Sales\Model\Payment\Gateway\`
2. Refactor `PaymentDataObjectFactory` to remove `instanceof Sales\Model\Order\Payment`
3. Add DI preferences in `Sales/etc/di.xml`
4. Remove `magento/module-sales` from `Payment/composer.json` require

### Phase 2: Move Observers to Sales

1. Move `SalesOrderBeforeSaveObserver` to `Sales\Observer\Payment\` — it operates on `Sales\Model\Order` and checks payment method code
2. Move `UpdateOrderStatusForPaymentMethodsObserver` to `Sales\Observer\Payment\` — it uses `Sales\Model\Order\Config` and `Order::STATE_NEW`
3. Move event registrations from `Payment/etc/events.xml` to `Sales/etc/events.xml`
4. Both observers already depend on Sales models — they belong in Sales

### Phase 3: Handle Remaining Items

1. `AbstractMethod.php` — deprecated, leave the `use Sales\Model\Order\Payment` as-is until the class is removed
2. `PaymentVerificationInterface` — evaluate moving to a shared API package, or accept the single interface dependency
3. `system.xml` source model — move `Newprocessing` source model to Payment module or use a generic status provider

### Result After Phase 1+2

Payment's composer.json drops `magento/module-sales` from `require`. The circular dependency is broken. Sales → Payment remains as the correct one-way dependency.

| What | Before | After |
|------|--------|-------|
| Payment composer.json | requires `magento/module-sales` | no Sales dependency |
| Gateway interfaces | in Payment | in Payment (unchanged) |
| Gateway adapters | in Payment (wrong) | in Sales (correct) |
| Event observers | in Payment listening to Sales events | in Sales (correct) |
| `AbstractMethod` | deprecated, references Sales | deprecated, left as-is |
| `PaymentVerificationInterface` | references `OrderPaymentInterface` | evaluate separately |
