# Magento View/Layout System — Modernization Plan

## Current Architecture Overview

**Scale:** 356 production PHP files, 846 layout XML files across modules, 509 unique layout handles

### The Three-Phase Pipeline

```
Phase 1: loadLayoutUpdates()     → Merge XML files from modules/themes
Phase 2: generateLayoutXml()     → Parse merged XML string into element tree
Phase 3: generateLayoutBlocks()  → Create block objects + build structure tree
         renderElement()         → Render blocks to HTML (lazy, on output)
```

Orchestrated by `Layout/Builder.php` calling `Layout.php` (16 constructor deps, suppresses 6 PHPMD warnings).

### What Happens For a Product Page

1. **32 `default.xml` files** merged (every page load) + 12 `catalog_product_view.xml` files
2. ~80KB of raw XML parsed via `simplexml_load_string()`
3. **102+ block class instantiations** from default.xml alone — all created eagerly
4. 4-level cache hierarchy (file XML → merged XML → scheduled structure → block output)
5. Events fired: `layout_load_before`, `layout_generate_blocks_before`, `layout_generate_blocks_after`, plus per-block render events

### Current Caching (4 Levels)

| Level | What | Key | Hit = Skip |
|-------|------|-----|-----------|
| 1 | File layout XML per theme/store | `LAYOUT_{area}_STORE{id}_{themeId}` | XML file scanning |
| 2 | Merged XML per handle combination | `LAYOUT_{...}{md5(handles)}` | XML merging |
| 3 | ScheduledStructure (interpreted XML) | `structure_{cacheId}` | XML interpretation (ReaderPool) |
| 4 | Block HTML output | `BLOCK_{custom_key}` | Block rendering |

**When cache is warm**, phases 1-2 are skipped, phase 3 deserializes from cache, and only block generation + rendering runs. This is the happy path in production.

---

## Key Bottlenecks

### 1. Eager Block Instantiation (Biggest Problem)

**ALL blocks are instantiated in phase 3**, even if they're in a sidebar that won't render, or in a container that's removed by another layout handle. Block constructors run, dependencies are injected, data is loaded.

For a product page, this means ~100-150 block objects created upfront. Many are never rendered.

**Current flow:**
```
GeneratorPool::process()
  → Generator/Block::process() 
    → creates ALL scheduled blocks via BlockFactory
    → injects constructor arguments via ArgumentInterpreter
    → registers in Layout::$_blocks[]
```

**No lazy creation.** If a block has expensive constructor logic or loads data, it runs even if the block is hidden.

### 2. Serialization of ScheduledStructure

Level 3 cache serializes/deserializes the entire `ScheduledStructure` + `pageConfigStructure` graph via `serialize()`/`unserialize()`. For complex pages this can be 200KB+ of serialized PHP data.

### 3. XML Merging on Cache Miss

On cache miss, 32+ XML files get loaded individually via `simplexml_load_string()`, string-concatenated, then validated against XSD schema. This is O(n) in file count and expensive for the first request after cache flush.

### 4. Template File Resolution

Each `.phtml` template is resolved through a fallback chain:
```
theme/<module>/templates/ → parent_theme/<module>/templates/ 
  → module/view/<area>/templates/ → module/view/base/templates/
```
Multiple filesystem `stat()` calls per template. Cached per-request but not across requests unless full-page cache is on.

### 5. Event Dispatch Overhead

5+ events dispatched during layout build. Each event goes through the observer pattern with XML config lookup. On uncached pages, this adds measurable overhead.

---

## Modernization Proposals

### Proposal 1: Lazy Block Instantiation via PHP 8.4

**Problem:** All blocks instantiated eagerly, most never rendered.
**Solution:** Use `ReflectionClass::newLazyGhost()` for block creation.

```php
// Current: BlockFactory creates real instance immediately
$block = $this->blockFactory->createBlock($class, $arguments);

// Proposed: block is a lazy ghost, constructor runs only on first property access/method call
$reflector = new ReflectionClass($class);
$block = $reflector->newLazyGhost(function ($block) use ($arguments) {
    // Constructor logic runs here, only when block is actually used
    $block->__construct(...$arguments);
});
```

**Impact:**
- Blocks that are never rendered = zero construction cost
- Blocks in removed containers = zero cost
- Sidebar blocks on pages that don't render sidebar = zero cost
- Estimated 40-60% reduction in block instantiation work on typical pages

**Compatibility:** Blocks are accessed via `$layout->getBlock($name)->toHtml()`. The lazy ghost is transparent — initialization triggers on `toHtml()` call.

---

### Proposal 2: Component-Based Rendering (Replace Block Tree)

**Problem:** The block tree is a flat registry (`Layout::$_blocks[]`) with parent/child relationships managed externally in `Data\Structure`. Rendering walks the tree recursively.

**Solution:** Move toward a component model where each component declares its own children, data requirements, and rendering.

```php
// Current: blocks declared in XML, structure externally managed
// <block class="Magento\Catalog\Block\Product\View" name="product.info" template="...">
//     <block class="Magento\Catalog\Block\Product\View\Price" name="product.info.price"/>
// </block>

// Proposed: self-contained component
#[LayoutComponent('product.info')]
class ProductView implements ComponentInterface
{
    public function __construct(
        private readonly ProductRepositoryInterface $productRepository,
        private readonly PricingService $pricingService,
    ) {}

    public function getData(RequestInterface $request): array
    {
        $product = $this->productRepository->getById($request->getParam('id'));
        return ['product' => $product, 'price' => $this->pricingService->getPrice($product)];
    }

    public function getTemplate(): string
    {
        return 'Magento_Catalog::product/view.phtml';
    }

    /** @return ComponentInterface[] */
    public function getChildren(): array
    {
        return ['price' => ProductPriceComponent::class];
    }
}
```

**Impact:**
- Components are self-describing — no external XML needed for structure
- Data fetching separate from rendering (enables async/parallel data loading)
- Children declared in code — IDE navigable, type-safe
- XML still available for overrides/customization, but not the primary definition

**This is a long-term architectural shift**, not a quick win.

---

### Proposal 3: Replace ScheduledStructure Serialization

**Problem:** `serialize()`/`unserialize()` of large object graphs for cache level 3.
**Solution:** Use `igbinary_serialize` (if available) or `msgpack`.

Short-term option:
```php
// Current
$this->cache->save(serialize($scheduledStructure), $cacheId);
$data = unserialize($this->cache->load($cacheId));

// Proposed: use json or igbinary (30-50% smaller, 2-3x faster)
$this->cache->save(igbinary_serialize($scheduledStructure), $cacheId);
$data = igbinary_unserialize($this->cache->load($cacheId));
```

---

### Proposal 4: Template Pre-compilation

**Problem:** `.phtml` templates are PHP files included at runtime via `include`. Template file resolution requires filesystem lookups through the theme fallback chain.

**Solution:** At deploy time, resolve all template paths and create a lookup map:

```php
// generated/template_map.php (opcached)
return [
    'Magento_Catalog::product/view/form.phtml' => '/app/design/frontend/MyTheme/default/Magento_Catalog/templates/product/view/form.phtml',
    'Magento_Checkout::cart.phtml' => '/vendor/magento/module-checkout/view/frontend/templates/cart.phtml',
    // ...
];
```

**Impact:**
- Zero filesystem fallback lookups at runtime
- Single array lookup per template
- Generated during `setup:static-content:deploy`
- Development mode keeps dynamic resolution

---

### Proposal 5: Decouple Asset Pipeline from Layout

**Problem:** Asset collection (CSS/JS) is tangled into the layout XML system. Adding a CSS file requires a layout XML update. The asset preprocessing chain (LESS compilation, merging, minification) runs in PHP.

**Solution:** 
- Extract asset manifest from layout at build time
- Use native bundlers (esbuild, Vite) for JS/CSS processing
- Layout only handles structure/blocks, not assets

```php
// Current: assets declared in layout XML
// <head><css src="Magento_Catalog::css/gallery.css"/></head>

// Proposed: asset manifest per page type (build-time generated)
// generated/assets/catalog_product_view.json
{
    "css": ["Magento_Catalog::css/gallery.css", "Magento_Theme::css/base.css"],
    "js": ["Magento_Catalog::js/gallery.js"],
    "preload": ["Magento_Catalog::images/placeholder.svg"]
}
```

The PHP LESS processor (`Css/` — 12 files) becomes unnecessary. Modern CSS (custom properties, nesting in CSS4) or Sass via Node.js replaces it.

---

## Summary: Priority & Impact Matrix

| Proposal | Effort | Performance Gain | Breaking Change? |
|----------|--------|-----------------|-----------------|
| **1. Lazy blocks (PHP 8.4)** | Medium | 40-60% block instantiation | No — transparent proxy |
| **2. Component model** | Very High | Architectural — better DX + performance | Yes — new API, backward compat layer needed |
| **3. Better serialization** | Low | 30-50% faster cache load/save | No — internal change |
| **4. Template map** | Low | Eliminates filesystem lookups | No — deploy-time generation |
| **5. Asset pipeline extraction** | High | Decouples build from runtime | Partial — layout XML asset syntax changes |

### Recommended Order

1. **Template pre-compilation map** (Proposal 4) — trivial to implement, immediate win
2. **Lazy block instantiation** (Proposal 1) — PHP 8.4 makes this nearly free to implement
3. **Better serialization** (Proposal 3) — swap `serialize()` for `igbinary_serialize()`, 1-line change
4. **Asset pipeline extraction** (Proposal 5) — decouple CSS/JS from PHP entirely
5. **Component model** (Proposal 2) — long-term vision, new projects can adopt incrementally

---

## Current File Distribution

| Directory | Files | Role |
|-----------|-------|------|
| Element/ | 94 | Block classes, UI components, templates |
| Asset/ | 59 | CSS/JS collection, merging, preprocessing |
| Design/ | 58 | Theme fallback, customization |
| Layout/ | 50 | Build pipeline, readers, generators, caching |
| TemplateEngine/ | 21 | PHP/PHTML + XHTML engine |
| Page/ | 16 | Page config (title, meta, body classes) |
| File/ | 13 | View file resolution, collectors |
| Result/ | 4 | Response types (Page, Layout, Json, Raw) |
| Other | 41 | Render, URL, Model, Helper, Xsd |
| **Total** | **356** | |

## Scale of What Gets Merged

| Metric | Count |
|--------|-------|
| Total layout XML files in modules | 846 |
| Unique layout handles | 509 |
| `default.xml` files (merged every page) | 32 |
| XML merged for product page | ~80KB |
| Block class refs in default.xml alone | 102 |
| Modules contributing to `checkout_index_index` | 19 |
