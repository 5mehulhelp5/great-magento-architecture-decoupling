# Wishlist Module Decoupling Report

## Modules Scanned

- `Magento_Wishlist` — 111 PHP files
- `Magento_WishlistGraphQl` — 30 PHP files
- `Magento_WishlistAnalytics` — exists (analytics data provider)

## Can It Be Removed Entirely? YES — with moderate effort

**Zero PHP `use` imports from outside** — no external module imports a Wishlist class in production PHP code. All coupling is via XML config, PHTML templates, ObjectManager string references, and composer.json.

---

## Who Depends on Wishlist

### composer.json hard `require` (7 modules)

| Module | Actual Coupling | Type |
|--------|----------------|------|
| **Customer** | Admin controllers + 1 DI plugin reference | Code + XML |
| **Sales** | `AdminOrder\Create` uses Wishlist via ObjectManager | Code (string ref) |
| **GroupedProduct** | Plugin on `Wishlist\Model\Item` | Code + XML |
| **Catalog** | PHTML templates call `Wishlist\Helper\Data` | Template only |
| **CatalogWidget** | PHTML template calls `Wishlist\Helper\Data` | Template only |
| **Reports** | Event observers for wishlist events | XML + config |
| **WishlistAnalytics** | Analytics data provider | Expected |

### XML-only references (GraphQL modules)

5 GraphQL modules configure `WishlistGraphQl\Model\Resolver\Type\WishlistItemType` in their `graphql/di.xml`:
- BundleGraphQl, ConfigurableProductGraphQl, DownloadableGraphQl, CatalogGraphQl, GroupedProductGraphQl

These are DI type configs — safely ignored when WishlistGraphQl is not installed.

---

## Coupling Details Per Module

### Customer (Heaviest — admin wishlist management)

**Controllers (5 files) — wishlist admin UI lives in Customer:**
- `Controller/Adminhtml/Index/Wishlist.php` — delete wishlist item (uses ObjectManager to create `Wishlist\Model\Item`)
- `Controller/Adminhtml/Index/ViewWishlist.php` — AJAX wishlist view
- `Controller/Adminhtml/Wishlist/Product/Composite/Wishlist.php` — abstract base
- `Controller/Adminhtml/Wishlist/Product/Composite/Wishlist/Update.php`
- `Controller/Adminhtml/Wishlist/Product/Composite/Wishlist/Configure.php`

**DI plugin (XML only):**
- `Customer/etc/di.xml` registers `Magento\Wishlist\Plugin\SaveWishlistDataAndAddReferenceKeyToBackUrl` — this is a Wishlist class plugged into Customer. The plugin registration belongs in Wishlist's di.xml, not Customer's.

**Fix:** Move all 5 admin wishlist controllers from Customer to Wishlist module. Move the plugin registration from Customer's di.xml to Wishlist's di.xml.

### Sales (Admin order create — ObjectManager string refs)

**1 file: `Sales/Model/AdminOrder/Create.php`**
- Property `$_wishlist` typed as `\Magento\Wishlist\Model\Wishlist`
- `getCustomerWishlist()` method uses `$this->_objectManager->create(\Magento\Wishlist\Model\Wishlist::class)`
- `switch` case `'wishlist'` handles adding wishlist items to admin order

**Fix:** Extract wishlist integration from `AdminOrder\Create` into a plugin provided by the Wishlist module. The `getCustomerWishlist()` method and the `'wishlist'` case in the switch can be handled by Wishlist plugging into the admin order create flow.

### GroupedProduct (Plugin on Wishlist\Model\Item)

**1 PHP file + 1 XML:**
- `GroupedProduct/Model/Wishlist/Product/Item.php` — plugin on `Wishlist\Model\Item::beforeRepresentProduct()` to handle grouped product qty in wishlist
- `GroupedProduct/etc/frontend/di.xml` — registers the plugin

**Fix:** This is the correct pattern (GroupedProduct extends Wishlist behavior) but creates a hard dependency. Move to `suggest` in composer.json. The DI plugin registration is safely ignored when Wishlist is absent — Magento skips plugin configs when the target class doesn't exist.

### Catalog & CatalogWidget (PHTML template refs only)

**Catalog templates (4+ files):**
- `product/widget/new/content/new_list.phtml` — `$this->helper(Magento\Wishlist\Helper\Data::class)->isAllow()`
- `product/widget/new/content/new_grid.phtml` — same pattern
- Other widget templates with "Add to Wishlist" buttons

**CatalogWidget templates (1 file):**
- `product/widget/content/grid.phtml` — `use Magento\Wishlist\Helper\Data;` + `$this->helper(Data::class)->isAllow()`

**Fix:** These are template-level references using the `$this->helper()` pattern. If Wishlist is disabled, `helper()` throws an exception. Options:
1. Wrap in try/catch or `class_exists()` check (ugly but backward compatible)
2. Move the "Add to Wishlist" button to a child block contributed by Wishlist via layout XML (clean — Wishlist adds the button, Catalog doesn't know about it)
3. Create a simple `Catalog\ViewModel\WishlistAvailability` that returns false when Wishlist is absent (DI-based)

### Reports (Event observers — XML only)

**events.xml observers:**
- `wishlist_add_product` → `Magento\Reports\Observer\WishlistAddProductObserver`
- `wishlist_share` → `Magento\Reports\Observer\WishlistShareObserver`

**config.xml + system.xml:**
- `product_to_wishlist_enabled` and `wishlist_share_enabled` report config fields

**Setup patch:**
- `InitializeReportEntityTypesAndPages.php` — registers wishlist event types

**Fix:** Move wishlist-related observers and config from Reports to Wishlist (or a bridge). Reports module doesn't need to know about Wishlist — Wishlist dispatches events that Reports observes. The observers themselves (`WishlistAddProductObserver`, `WishlistShareObserver`) are Reports classes that only depend on Reports internals. The event listener registrations could stay in Reports (events are loosely coupled). Move composer dep to `suggest`.

### GraphQL Modules (5 modules — DI config only)

Each GraphQL module adds a wishlist item type mapping:
```xml
<type name="Magento\WishlistGraphQl\Model\Resolver\Type\WishlistItemType">
    <arguments>
        <argument name="supportedTypes">
            <item name="simple" xsi:type="string">SimpleWishlistItem</item>
        </argument>
    </arguments>
</type>
```

**No fix needed.** DI type configs on non-existent classes are safely ignored by Magento's ObjectManager. If WishlistGraphQl is absent, these configs are dead code with no runtime effect.

---

## Decoupling Plan

### Phase 1: Move Admin Wishlist UI from Customer to Wishlist

Move FROM `Customer/` TO `Wishlist/`:

| Current (Customer) | New (Wishlist) |
|-------------------|----------------|
| `Controller/Adminhtml/Index/Wishlist.php` | `Wishlist/Controller/Adminhtml/Customer/Wishlist.php` |
| `Controller/Adminhtml/Index/ViewWishlist.php` | `Wishlist/Controller/Adminhtml/Customer/ViewWishlist.php` |
| `Controller/Adminhtml/Wishlist/Product/Composite/Wishlist.php` | `Wishlist/Controller/Adminhtml/Product/Composite/Wishlist.php` |
| `Controller/Adminhtml/Wishlist/Product/Composite/Wishlist/Update.php` | Same, under Wishlist module |
| `Controller/Adminhtml/Wishlist/Product/Composite/Wishlist/Configure.php` | Same, under Wishlist module |

Move plugin registration from `Customer/etc/di.xml` to `Wishlist/etc/di.xml`:
```xml
<plugin name="saveWishlistDataAndAddReferenceKeyToBackUrl" 
        type="Magento\Wishlist\Plugin\SaveWishlistDataAndAddReferenceKeyToBackUrl"/>
```

### Phase 2: Decouple Sales Admin Order Create

Extract `getCustomerWishlist()` and the `'wishlist'` switch case from `Sales/Model/AdminOrder/Create.php` into a Wishlist plugin. Wishlist module hooks into admin order creation via plugin, Sales module has zero knowledge of Wishlist.

### Phase 3: Fix Catalog/CatalogWidget Templates

Replace direct `$this->helper(Wishlist\Helper\Data::class)` calls with a child block contributed by Wishlist via layout XML. Catalog templates render a named child block (`product.addto.wishlist`) — if Wishlist is installed it provides the block, if not the child doesn't exist and nothing renders.

### Phase 4: Update Composer Dependencies

| Module | Change |
|--------|--------|
| `Customer/composer.json` | Remove `magento/module-wishlist` |
| `Sales/composer.json` | Remove `magento/module-wishlist` |
| `GroupedProduct/composer.json` | Move to `suggest` |
| `Catalog/composer.json` | Remove `magento/module-wishlist` |
| `CatalogWidget/composer.json` | Remove `magento/module-wishlist` |
| `Reports/composer.json` | Move to `suggest` (event observers are loosely coupled) |

---

## Architecture After Decoupling

```
Catalog (zero Wishlist dependency)
    └── Layout: renders child block "product.addto.wishlist" if it exists
         ↑
         │ Wishlist adds this block via layout XML (when installed)
         │
Customer (zero Wishlist dependency)
    └── Admin customer edit: no wishlist tab/controllers
         ↑
         │ Wishlist adds admin tabs/controllers via layout XML + routes
         │
Sales (zero Wishlist dependency)  
    └── AdminOrder\Create: no getCustomerWishlist()
         ↑
         │ Wishlist hooks into admin order create via plugin
         │
Wishlist module (optional)
    ├── Frontend: add-to-wishlist buttons, wishlist pages
    ├── Admin: customer wishlist tab, wishlist management
    ├── Plugins: hooks into Customer, Sales, GroupedProduct
    └── Layout XML: contributes blocks to Catalog product pages
```

If Wishlist disabled: no wishlist buttons, no admin tabs, no wishlist in admin order create. Everything else works.

---

## Difficulty Assessment

| Phase | Effort | Risk |
|-------|--------|------|
| Move admin controllers (5 files) | Medium | Low — route URLs stay the same |
| Decouple Sales AdminOrder\Create | Medium | Moderate — the `getCustomerWishlist()` method is public API |
| Fix Catalog/CatalogWidget templates | Medium | Low — child block pattern is standard |
| Update composer.json (6 modules) | Trivial | Zero |
| **Overall** | **Medium** | **Wishlist becomes fully optional** |
