# Magento Framework Modernization Plan (PHP 8.4)

**Scope:** `/lib/internal/Magento/Framework` — 2,777 production PHP files, 80+ sub-components

---

## 1. Proxy Generation → PHP 8.4 Native Lazy Objects

### Current State

Magento auto-generates Proxy classes at build time for lazy-loading dependencies. The system:
- Generator: `ObjectManager/Code/Generator/Proxy.php` (294 lines)
- Also: `Async/Code/Generator/ProxyDeferredGenerator.php` (241 lines)
- Infrastructure: `Code/Generator/EntityAbstract.php`, `Code/Generator.php`, `Code/Generator/Autoloader.php`
- **206 `\Proxy` references** across 86 module di.xml files
- Generated files written to `generated/code/` directory

**How it works:** Proxy extends the source class, stores a `$_subject` that's only instantiated on first method call. All public methods delegate to `$this->_getSubject()->method()`.

### PHP 8.4 Replacement

PHP 8.4 introduces `ReflectionClass::newLazyProxy()` and `newLazyGhost()`:

```php
// Current: requires code generation, disk writes, autoloader hooks
$proxy = new \Magento\Framework\App\Cache\Proxy($objectManager, Cache::class);

// PHP 8.4: runtime lazy object, zero code generation
$reflector = new ReflectionClass(Cache::class);
$proxy = $reflector->newLazyProxy(function () use ($objectManager) {
    return $objectManager->get(Cache::class);
});
```

### Impact

| Metric | Current | After |
|--------|---------|-------|
| Generated Proxy files | ~206 | 0 |
| Build time code generation | Required | Eliminated |
| `generated/code/` Proxy classes | ~206 files | None |
| Proxy.php generator | 294 lines | Removable |
| ProxyDeferredGenerator | 241 lines | Replaceable |

### Migration Plan

1. Modify `ObjectManager` to use `ReflectionClass::newLazyProxy()` when creating `\Proxy` type-hinted dependencies
2. Keep backward compatibility: existing `\Proxy` suffix in di.xml continues to work, but resolved at runtime instead of code-gen
3. Remove Proxy generator from `generatedEntities` config
4. Phase out `generated/code/*_Proxy.php` files

**Priority: HIGH** — eliminates entire code generation subsystem for proxies, speeds up builds, reduces disk I/O.

---

## 2. DataObject / Getter-Setter → PHP 8.4 Property Hooks

### Current State

**Core class hierarchy:**

| Class | Lines | Purpose |
|-------|-------|---------|
| `DataObject.php` | 607 | Base class — `$_data` array + `__call()` magic for get/set/has/uns |
| `Model/AbstractModel.php` | 1,056 | Extends DataObject — adds change tracking, persistence lifecycle |
| `Api/AbstractSimpleObject.php` | 82 | API DTOs — explicit `_get()`/`setData()`, no magic |
| `Api/AbstractExtensibleObject.php` | 198 | Extends SimpleObject — adds custom + extension attributes |
| `Model/AbstractExtensibleModel.php` | 423 | Combines AbstractModel + extensible attributes |

**The pattern:**
```php
// Magic method dispatch in DataObject::__call()
$product->getName();      // → __call() → getData('name')
$product->setName('x');   // → __call() → setData('name', 'x')
```

**33 Framework files** have explicit getter/setter pairs that just delegate to `getData`/`setData` — **78 methods total** (25 getters, 53 setters).

### PHP 8.4 Property Hooks Replacement

```php
// Current: explicit delegation method
public function getOutput() {
    return $this->getData('output');
}

// PHP 8.4: property hook
public string $output {
    get => $this->getData('output');
    set(string $value) => $this->setData('output', $value);
}
```

### Feasibility by Class Type

| Class Type | Feasibility | Notes |
|------------|-------------|-------|
| **API SimpleObject DTOs** (SearchCriteria, Filter, etc.) | Excellent | No magic methods, clear typed contracts |
| **Simple DataObject extensions** (Observer, Lock, etc.) | Good | Can coexist with `__call()` magic |
| **AbstractModel subclasses** | Moderate | Need to keep change tracking in setData |
| **DataObject core `__call()`** | Not feasible | Dynamic nature can't be replicated with hooks |

### Migration Plan

**Phase 1 — API Data Objects (low risk):**
Convert `Api/SearchCriteria.php`, `Api/Filter.php`, `Api/SearchResults.php`, etc. — ~18 files

**Phase 2 — Simple DataObject extensions (medium risk):**
Convert `Event/Observer.php`, `MessageQueue/Lock.php`, `Shell/Response.php`, etc. — ~15 files

**Phase 3 — New code policy:**
All new data classes use property hooks instead of getData/setData delegation

**Keep:** DataObject's `__call()` magic for backward compatibility on unmapped properties

**Priority: MEDIUM** — improves IDE support and type safety, but DataObject core stays as-is.

---

## 3. Constants → PHP 8.1 Enums

### Current State

| Category | Count |
|----------|-------|
| Files with 3+ constants, zero methods (pure constant bags) | 99 |
| Files with 3+ constants, 1-2 methods (mostly constants) | 100 |
| Source models with `toOptionArray()` | 18 |
| Total constant definitions in Framework | 872 |

### Top Enum Candidates

| File | Constants | Description |
|------|-----------|-------------|
| `HTTP/Mime.php` | 14 | MIME types + encodings |
| `DB/Ddl/Table.php` | 50+ | Column types, actions, options |
| `Jwt/Jwk.php` | 42 | Key types, algorithms |
| `App/Filesystem/DirectoryList.php` | 30+ | Directory path constants |
| `Bulk/OperationInterface.php` | 9 | Operation status types |
| `App/AreaInterface.php` | 3 | Area part constants |

### Migration Example

```php
// Current: constant bag interface
interface OperationInterface {
    const STATUS_TYPE_COMPLETE = 1;
    const STATUS_TYPE_OPEN = 4;
    const STATUS_TYPE_NOT_STARTED = 0;
}

// PHP 8.1: backed enum
enum OperationStatus: int {
    case Complete = 1;
    case Open = 4;
    case NotStarted = 0;
}
```

Source models become trivial:
```php
enum StockStatus: int implements HasLabel {
    case InStock = 1;
    case OutOfStock = 0;

    public function label(): string {
        return match($this) {
            self::InStock => 'In Stock',
            self::OutOfStock => 'Out of Stock',
        };
    }
}
```

**Priority: HIGH** — 99 pure constant files are trivial to convert. Improves type safety everywhere constants are used.

---

## 4. `@deprecated` → `#[\Deprecated]` Attribute (PHP 8.4)

### Current State

- **401 `@deprecated` PHPDoc annotations** across 196 Framework files
- **0 files** use the `#[\Deprecated]` attribute

### PHP 8.4 Replacement

```php
// Current: only visible in IDE/docs
/** @deprecated Use NewClass instead */
class OldClass {}

// PHP 8.4: triggers E_USER_DEPRECATED at runtime
#[\Deprecated("Use NewClass instead", since: "2.5.0")]
class OldClass {}
```

**Priority: HIGH** — mechanical replacement across 401 annotations. Enables runtime deprecation detection.

---

## 5. JSON Validation → `json_validate()` (PHP 8.3)

### Current State

50 files use `json_*` functions. Several use the pattern:
```php
json_decode($string);
if (json_last_error() === JSON_ERROR_NONE) { ... }
```

Key file: `Serialize/JsonValidator.php` (30 lines) — entire purpose is JSON validation.

### PHP 8.3 Replacement

```php
// Current: decode just to validate (wasteful)
json_decode($string);
return json_last_error() === JSON_ERROR_NONE;

// PHP 8.3: validation without decoding
return json_validate($string);
```

**Priority: LOW** — small scope, easy win, but minor impact.

---

## 6. Framework Components: Replace vs Keep

### Replace (Modern alternatives exist)

| Component | Files | Replace With | Effort | Priority |
|-----------|-------|-------------|--------|----------|
| **Mail** | 23 | Symfony Mailer (already in composer) | Medium | High |
| **Serialization** | 9 | Simplify to `json_encode`/`json_decode` + remove legacy `serialize()` | Low | Medium |
| **HTTP Client** | 23 | PSR-18 (Guzzle already in composer) | Medium | Medium |
| **Validator** | 41 | Symfony Validator + custom Magento constraints | High | Medium |
| **Lock** | 7 | Symfony Lock | Low-Medium | Low |
| **Archive** | 9 | Flysystem or native PHP | Low | Low |
| **Image** | 9 | Intervention/Image | Low-Medium | Low |
| **CSS/LESS** | 12 | Node.js/PostCSS tooling (deprecate PHP LESS) | High | Low |

### Keep (Already modern or deeply integrated)

| Component | Files | Reason |
|-----------|-------|--------|
| **Console** | 9 | Already wraps Symfony Console |
| **Logger** | 7 | Already wraps Monolog (PSR-3) |
| **Cache** | 47 | Has PSR-6/PSR-16 adapters; tag-based invalidation is Magento-specific and valuable |
| **Session** | 26 | Deeply integrated with PHP sessions + custom handlers (Redis, DB) |
| **Filesystem** | 36 | Works well; Flysystem is optional for new code |

### Simplify (Reduce without replacing)

| Component | Files | Action |
|-----------|-------|--------|
| **Convert** | 6 | `DataSize` is 5 lines; `Excel` could use phpspreadsheet |
| **Measure** | 4 | Niche; fine as-is |
| **Validation** | 2 | Just `ValidationException` + `ValidationResult`; inline-able |

---

## Summary: Prioritized Roadmap

### Phase 1 — Quick Wins (Low effort, high value)

| Item | Files Affected | Effort |
|------|---------------|--------|
| `@deprecated` → `#[\Deprecated]` | 196 files, 401 annotations | Mechanical |
| Pure constant classes → Enums | 99 files | Mechanical |
| `JsonValidator` → `json_validate()` | 1-5 files | Trivial |
| Remove phantom composer deps (already done for some) | Various | Trivial |

### Phase 2 — Proxy Elimination (High value, medium effort)

| Item | Files Affected | Effort |
|------|---------------|--------|
| Replace Proxy code-gen with `ReflectionClass::newLazyProxy()` | ObjectManager + 206 di.xml refs | Medium |
| Remove Proxy generator | 2 generator files | Low |
| Clean up `generated/code/` Proxy classes | ~206 generated files | Automated |

### Phase 3 — Property Hooks & API Modernization (Medium effort)

| Item | Files Affected | Effort |
|------|---------------|--------|
| API DTO property hooks | ~18 files | Medium |
| Simple DataObject extension hooks | ~15 files | Medium |
| Source models → backed enums | 18 files | Low-Medium |

### Phase 4 — Component Replacement (High effort, long-term)

| Item | Current Files | Replace With |
|------|--------------|-------------|
| Mail → Symfony Mailer | 23 | Already in composer |
| HTTP → PSR-18/Guzzle | 23 | Already in composer |
| Validator → Symfony Validator | 41 | Needs adding |
| Serialization cleanup | 9 | Simplify |

---

## Key Numbers

| Metric | Count |
|--------|-------|
| Framework production PHP files | 2,777 |
| Proxy references in codebase di.xml | 206 |
| `@deprecated` annotations | 401 |
| Pure constant bag classes (→ enums) | 99 |
| Explicit getter/setter files (→ property hooks) | 33 |
| Delegating methods (→ property hooks) | 78 |
| Components replaceable by modern packages | 8 |
| Components already modern (keep) | 4 |
