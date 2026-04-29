# Cron & Message Queue (AMQP/RabbitMQ) Decoupling Report

---

## Part 1: Cron Module

### Current Architecture

```
Framework/Crontab/          → 3 files (CLI crontab install/remove)
app/code/Magento/Cron/      → 30 PHP files (scheduler, admin UI, cleanup)
```

Cron module is lean — only requires Framework, Config, Store. No heavy dependencies.

### Who Depends on Cron

**composer.json `require`:** Only 4 non-Cron modules

| Module | Why |
|--------|-----|
| Backup | Scheduled backups |
| Config | Cron schedule config |
| Persistent | Expired session cleanup |
| Sitemap | Scheduled sitemap generation |

**PHP imports from `Magento\Cron`:** Only 1 file

| Module | File | Import |
|--------|------|--------|
| Persistent | `Observer/ClearExpiredCronJobObserver.php` | `Cron\Model\Schedule` (type-hint on execute method) |

### Who Declares Cron Jobs (crontab.xml)

**26 modules** define `crontab.xml` — but they DON'T depend on the Cron module for this. The `crontab.xml` files are parsed by the Cron module's scheduler. The modules just declare config.

| Modules with crontab.xml |
|--------------------------|
| Analytics, AsynchronousOperations, Backend, Backup, Captcha, Catalog, CatalogRule, Cron, Customer, Directory, ImportExport, Indexer, Integration, MessageQueue, MysqlMq, NewRelicReporting, Newsletter, Paypal, Persistent, ProductAlert, Sales, SalesRule, Security, Shipping, Sitemap, Tax |

### Decoupling Assessment

**Cron is already well-decoupled:**
- Only 4 modules have composer dependency, and only 1 has a PHP import
- The `crontab.xml` config pattern is a proper plugin architecture — modules declare jobs, Cron discovers and runs them. No coupling.
- The single PHP coupling (`Persistent` importing `Cron\Model\Schedule`) is a type-hint on the cron job method. This could be replaced with a generic interface or simply `mixed`.

**Quick fix for Persistent:**

```php
// Current: type-hints Cron\Model\Schedule
public function execute(Schedule $schedule)

// Proposed: remove type-hint (schedule object is unused anyway)
public function execute($schedule)
```

Then remove `magento/module-cron` from Persistent's composer.json.

**Verdict: Cron is nearly fully decoupled already. 1 minor fix needed.**

---

## Part 2: Message Queue (AMQP / RabbitMQ / MysqlMq)

### Current Architecture

The MQ system has 3 layers:

```
Layer 1: Framework (abstractions)
├── Framework/Communication/     →  12 files (config parsing, topic/handler definitions)  
├── Framework/MessageQueue/      → 158 files (consumer, publisher, topology, lock, routing)
├── Framework/Amqp/              →  20 files (AMQP protocol abstraction)
└── Framework/Bulk/              →   6 files (bulk operation interfaces)

Layer 2: Transport modules (implementations)
├── Magento/Amqp/                →   8 files (RabbitMQ connection via Framework/Amqp)
├── Magento/MysqlMq/             →  18 files (MySQL-based queue fallback)
└── Magento/MessageQueue/        →  14 files (consumer management, cron runner)

Layer 3: Feature modules (consumers/publishers)
├── Magento/AsynchronousOperations/  → 73 files (bulk operations UI + API)
├── Magento/WebapiAsync/             → 18 files (async REST/SOAP endpoints)
└── Magento/Stomp/                   →  8 files (STOMP protocol support)
```

### Who Depends on MQ Modules (composer.json)

**Nobody.** The 3 MQ modules (`Amqp`, `MysqlMq`, `MessageQueue`) are only required by themselves. No external module has a composer dependency on them.

### Who Uses Framework MessageQueue (PHP imports)

| Module | Import Count | What They Use |
|--------|-------------|---------------|
| AsynchronousOperations | 19 | Bulk operations, consumers, publishers — the primary MQ consumer |
| MessageQueue | 10 | (Self — consumer management) |
| MysqlMq | 5 | (Self — MySQL transport) |
| Stomp | 4 | STOMP protocol adapter |
| WebapiAsync | 3 | Async webapi dispatching |
| SalesRule | 3 | Async coupon generation |
| Catalog | 3 | Async category/product operations |
| Store | 2 | Store config consumer |
| Amqp | 2 | (Self — RabbitMQ adapter) |
| ProductAlert | 1 | Product alert notifications |
| MediaStorage | 1 | Media sync |
| MediaGallerySynchronization | 1 | Gallery sync |
| MediaGalleryRenditions | 1 | Renditions processing |
| MediaContentSynchronization | 1 | Content sync |
| ImportExport | 1 | Async import/export |
| Eav | 1 | EAV attribute processing |
| Config | 1 | Config change consumers |
| CatalogUrlRewrite | 1 | URL rewrite generation |
| AsyncConfig | 1 | Async config save |

### Who Declares Queue Config (XML)

**11 modules** define MQ configuration:

| Module | consumer | publisher | topology | communication |
|--------|----------|-----------|----------|---------------|
| AsyncConfig | x | x | x | x |
| Catalog | x | x | — | x |
| CatalogUrlRewrite | x | x | — | x |
| ImportExport | x | x | — | x |
| MediaContentSynchronization | x | x | — | x |
| MediaGalleryRenditions | x | x | — | x |
| MediaGallerySynchronization | x | x | — | x |
| MediaStorage | x | x | — | x |
| ProductAlert | x | x | — | x |
| SalesRule | x | x | x | x |
| WebapiAsync | x | — | x | — |

### The Coupling Problem

**Framework/MessageQueue is the issue** — 158 files baked into the framework. Even if you remove the Amqp and MysqlMq modules, the framework still carries the full MQ abstraction layer. Modules like Catalog, SalesRule, and ProductAlert import directly from `Framework\MessageQueue`.

The transport modules (Amqp, MysqlMq) are already clean — no external module imports from them directly. They're pure SPI implementations.

### Decoupling Options

#### Option A: Make MQ Framework Optional (extract from core framework)

**Problem:** `Framework/MessageQueue` (158 files) is bundled with the core framework. It can't be removed via composer.

**Solution:** Extract into a separate framework package:

```
magento/framework                   → core (no MQ)
magento/framework-message-queue     → already exists as a package name!
magento/framework-amqp              → already exists
magento/framework-bulk              → already exists
magento/framework-communication     → extract
```

Modules that use MQ (Catalog, SalesRule, etc.) would `require` `magento/framework-message-queue` explicitly. Stores that don't use async operations could skip the MQ packages entirely.

**Impact:** 158 files removed from core framework. 11 modules add explicit require.

#### Option B: Make Queue Declarations Conditional

**Problem:** Modules like Catalog declare `queue_consumer.xml` — if MQ is absent, these configs cause errors.

**Solution:** Queue XML configs are already only loaded by the MessageQueue module's config reader. If MessageQueue module is disabled, the XML files are simply ignored (same as `crontab.xml` without Cron). **This already works.**

The real issue is the PHP imports in modules like Catalog and SalesRule that use `Framework\MessageQueue` classes directly.

#### Option C: Lightweight Async Publisher in Framework (for simple cases)

**Problem:** Modules like ProductAlert, MediaStorage, and CatalogUrlRewrite publish 1-2 messages. They don't need the full MQ framework — they just want "do this work later."

**Solution:** Introduce a lightweight publisher interface in the existing `Framework\Async` namespace (which already exists for ProxyDeferred, etc.):

```php
namespace Magento\Framework\Async;

/**
 * Lightweight async publisher — mirrors MessageQueue\PublisherInterface
 * but lives in core Framework so modules don't need the full MQ stack.
 */
interface PublisherInterface
{
    /**
     * Publish data to a named topic for asynchronous processing.
     *
     * @param string $topicName
     * @param mixed $data
     * @return void
     */
    public function publish(string $topicName, mixed $data): void;
}
```

The MessageQueue module provides the real implementation (queue-backed via RabbitMQ or MySQL). Without MQ installed, a synchronous fallback runs the handler inline. Same API, swappable backend — modules only import from `Framework\Async`, not `Framework\MessageQueue`.

**Impact:** Light-usage modules (ProductAlert, Media*, CatalogUrlRewrite, ImportExport, Config, Store, Eav) stop importing from MQ entirely. Only heavy MQ users (AsynchronousOperations, WebapiAsync, SalesRule) keep the direct `Framework\MessageQueue` dependency.

#### Option D: Do Nothing (it's already mostly decoupled)

The transport layer (Amqp, MysqlMq) has **zero external coupling** — no module imports from them. They're pure swappable backends. If you don't install RabbitMQ, MysqlMq handles everything. If you don't want queues at all, disable `MessageQueue` module and all queue features gracefully stop.

The only real coupling is at the Framework level (158 files that ship with every install).

---

## Summary

### Cron

| Aspect | Status |
|--------|--------|
| Module-level coupling | Nearly zero (1 type-hint in Persistent) |
| Config coupling (crontab.xml) | Clean — declarative, no imports needed |
| composer deps | 4 modules, all legitimate |
| **Verdict** | **Already well-decoupled. 1 trivial fix.** |

### Message Queue

| Aspect | Status |
|--------|--------|
| Transport modules (Amqp, MysqlMq) | **Fully decoupled** — zero external imports |
| MessageQueue module | **Clean** — no external module depends on it via composer |
| Framework/MessageQueue (158 files) | **Bundled in core** — can't be removed without framework extraction |
| Queue config XML | **Clean** — ignored if MessageQueue disabled |
| PHP imports from Framework MQ | **19 modules** import from it, but most are light (1-3 imports) |
| **Verdict** | **Transport is clean. Framework MQ is the problem — 158 files that could be extracted.** |

### Recommended Actions

| Action | Effort | Impact |
|--------|--------|--------|
| Fix Persistent→Cron type-hint | Trivial | Removes last Cron coupling |
| Extract Framework/MessageQueue into standalone package | High | 158 files removable from core |
| Add `Magento\Framework\Async\PublisherInterface` | Medium | 9 light-usage modules stop needing MQ framework |
| Remove Amqp/MysqlMq transport modules | Already possible | They have zero external coupling — just disable |
