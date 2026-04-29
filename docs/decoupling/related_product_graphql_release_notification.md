# RelatedProductGraphQl & ReleaseNotification Decoupling Report

---

## 1. RelatedProductGraphQl — 10 files

GraphQL resolvers for related, upsell, and crosssell products.

### Coupling

| Direction | Count |
|-----------|-------|
| Modules depending on it | **0** |
| PHP imports from outside | **0** |
| XML refs from outside | **0** |

### Dependencies

Requires: Catalog, CatalogGraphQl, Framework. Suggests: GraphQl.

### Verdict

**Already fully decoupled.** Zero inbound coupling. Disable/remove freely. No work needed.

---

## 2. ReleaseNotification — 11 files

Admin panel "what's new" notification popup shown after Magento upgrades. Fetches content from Adobe's content CDN and shows it once per admin user per version.

### Coupling

| Direction | Detail |
|-----------|--------|
| Modules depending on it | **1** — `AdminAnalytics` |
| PHP imports from outside | `AdminAnalytics/ViewModel/Notification.php` imports `ReleaseNotification\Model\Condition\CanViewNotification` |
| XML refs from outside | None |

### What AdminAnalytics Uses

```php
use Magento\ReleaseNotification\Model\Condition\CanViewNotification as ReleaseNotification;
// Constructor-injected, used to check if admin user has seen the notification
```

AdminAnalytics piggybacks on ReleaseNotification's "can view notification" check to decide whether to show its own analytics opt-in popup alongside the release notification modal.

### Dependencies

ReleaseNotification requires: User, Backend, Ui, Framework. Suggests: Config. Lightweight.

### Decoupling Fix

**Move the coupling from AdminAnalytics to ReleaseNotification:**

The `CanViewNotification` condition check is generic — it just checks if a user has been marked as notified for the current version. AdminAnalytics should either:
1. Duplicate this simple check internally (it's just a DB flag lookup), OR
2. ReleaseNotification could extract the interface to a shared location

**Simplest fix:** Make `magento/module-release-notification` a `suggest` in AdminAnalytics' composer.json and make the constructor param nullable. If ReleaseNotification is absent, AdminAnalytics always shows its popup (or never — depending on desired behavior).

### Verdict

**Nearly fully decoupled.** 1 import in 1 module. Trivial to fix. Both modules can be safely disabled on production stores — they only show admin notifications about new Magento versions.

---

## Summary

| Module | Files | Inbound Coupling | Effort | Can Remove? |
|--------|-------|-----------------|--------|-------------|
| RelatedProductGraphQl | 10 | Zero | None | **Yes — already clean** |
| ReleaseNotification | 11 | 1 module (AdminAnalytics) | Trivial | **Yes — after making AdminAnalytics dep optional** |
