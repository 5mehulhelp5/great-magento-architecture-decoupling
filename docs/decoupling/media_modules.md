# Media Modules Decoupling Report

## Overview

25 media-related modules, 323 production PHP files. They break down into 3 distinct systems:

```
MediaStorage (36 files)          ← CORE — file upload, image resize, DB storage
    ↑ depended on by 20 core modules (Catalog, Cms, Customer, etc.)
    │
MediaGallery* (16 modules, 215 files)  ← ENHANCED GALLERY — admin media browser
    │
MediaContent* (8 modules, 73 files)    ← CONTENT RELATIONS — tracks which content uses which media
```

---

## What Each System Does

### MediaStorage — The Foundation (36 files)

**The only essential module.** Handles:
- File/image upload to `pub/media/`
- Image resize and cache
- Database media storage alternative
- Synchronization of media between filesystem and DB

**20 core modules depend on it:** Catalog, Cms, Customer, Eav, Email, Sales, Bundle, ConfigurableProduct, GroupedProduct, Downloadable, Swatches, CatalogImportExport, ImportExport, Config, Store, Theme, ProductVideo, Sitemap, AdminNotification, RemoteStorage

**This cannot be removed.** It's as fundamental as Store or Customer.

### MediaGallery — Enhanced Admin Media Browser (16 modules, 215 files)

The redesigned admin media gallery (introduced in 2.4). Replaces the old WYSIWYG image browser with a full asset management UI.

| Module | Files | Purpose |
|--------|-------|---------|
| `MediaGalleryApi` | 22 | API interfaces — asset, keyword, directory CRUD |
| `MediaGallery` | 27 | Core implementation — asset management, directory operations |
| `MediaGalleryUi` | 55 | Admin UI — React-based media browser, grid, upload |
| `MediaGalleryUiApi` | 2 | UI configuration interface |
| `MediaGalleryCatalog` | 2 | Catalog product image integration |
| `MediaGalleryCatalogIntegration` | 2 | Deeper catalog integration (category images) |
| `MediaGalleryCatalogUi` | 9 | Catalog-specific UI components in gallery |
| `MediaGalleryCmsUi` | 5 | CMS-specific UI components in gallery |
| `MediaGalleryIntegration` | 3 | Integration with legacy media browser |
| **MediaGalleryMetadata** | **32** | **EXIF/IPTC/XMP metadata extraction** — reads JPEG, PNG, GIF image metadata (camera info, GPS, copyright, keywords) |
| `MediaGalleryMetadataApi` | 12 | Metadata interfaces |
| **MediaGalleryRenditions** | **11** | **Image renditions** — generates resized/optimized copies of gallery images for admin preview. Uses queue for async processing |
| `MediaGalleryRenditionsApi` | 3 | Renditions interfaces |
| **MediaGallerySynchronization** | **18** | **Filesystem ↔ DB sync** — scans `pub/media/` filesystem and syncs assets into the `media_gallery_asset` DB table. Runs via queue consumer |
| `MediaGallerySynchronizationApi` | 8 | Sync interfaces |
| `MediaGallerySynchronizationMetadata` | 3 | Syncs metadata during asset sync |

### MediaContent — Content-Asset Relationship Tracking (8 modules, 73 files)

Tracks which CMS pages/blocks and catalog products/categories reference which media files. Enables "where is this image used?" in the gallery UI.

| Module | Files | Purpose |
|--------|-------|---------|
| `MediaContentApi` | 17 | API interfaces — content-asset relation CRUD |
| `MediaContent` | 15 | Core implementation — relation management |
| **MediaContentSynchronization** | **9** | **Scans content (CMS, catalog) and builds a map of which entities use which media files.** Runs via queue consumer. |
| `MediaContentSynchronizationApi` | 7 | Sync interfaces |
| `MediaContentCatalog` | 9 | Parses catalog product/category content for media references |
| `MediaContentCms` | 8 | Parses CMS page/block content for media references |
| `MediaContentSynchronizationCatalog` | 4 | Catalog-specific sync trigger |
| `MediaContentSynchronizationCms` | 4 | CMS-specific sync trigger |

---

## Explaining the Key Concepts

### Renditions

**What:** Pre-generated resized copies of original images for the admin media gallery grid. Instead of loading full 5MB product photos in the admin grid, renditions create smaller preview versions.

**How:** `MediaGalleryRenditions` listens for new/changed assets and generates renditions via `queue_consumer.xml`. The `UpdateRenditions` consumer processes batches of images, creating optimized copies at configured dimensions.

**Why it exists:** Admin performance. Without renditions, opening the media gallery would load full-resolution images in the grid thumbnails.

### ContentSynchronization

**What:** Background process that scans CMS pages/blocks and catalog descriptions for media file references (e.g., `{{media url="wysiwyg/image.jpg"}}` directives), then stores those relationships in the `media_content_asset` table.

**How:** `MediaContentSynchronization` publishes scan jobs to a message queue. The `Consume` class processes them, using `MediaContentCatalog` and `MediaContentCms` to parse entity content and extract media references.

**Why it exists:** Powers the "Used In" column in the media gallery — tells you which products, categories, and CMS pages use a specific image. Without it, you'd have no idea if deleting an image breaks something.

### GallerySynchronization

**What:** Scans the `pub/media/` filesystem and syncs file metadata into the `media_gallery_asset` database table. Creates DB records for files that exist on disk but aren't tracked yet.

**How:** `MediaGallerySynchronization` has a CLI command (`media-gallery:sync`) and a queue consumer. It iterates the filesystem, computes content hashes, and creates/updates asset records.

**Why it exists:** The media gallery DB needs to know about all files on disk, including ones uploaded via FTP, imported, or created by third-party tools. The sync bridges the gap between filesystem reality and database state.

---

## External Coupling

### MediaStorage — 20 modules depend on it (CANNOT be removed)

All 20 are legitimate — every module that handles images/files needs MediaStorage.

### MediaGallery family — 2 external deps

| Module | Depends On | Why |
|--------|-----------|-----|
| RemoteStorage | `media-gallery-metadata`, `media-gallery-synchronization` | Needs to handle metadata and sync for remote (S3) storage |

### MediaContent family — 0 external deps

Zero external modules depend on any MediaContent module.

### Internal Dependency Chain

```
MediaStorage (standalone — no Media* deps)
    
MediaGalleryApi (standalone)
    ↑
MediaGallery (→ Api, Cms)
    ↑
MediaGalleryUi (→ Api, Gallery, Cms, Backend, Store, Ui)
    
MediaGalleryRenditions (→ RenditionsApi, GalleryApi, Cms, ContentApi)
MediaGallerySynchronization (→ GalleryApi, SyncApi)
MediaGalleryMetadata (→ MetadataApi) ← standalone, reads EXIF/IPTC/XMP

MediaContentApi (→ GalleryApi)
    ↑
MediaContent (→ ContentApi, GalleryApi)
    ↑
MediaContentSynchronization (→ ContentSyncApi, ContentApi, GallerySynchronization, AsynchronousOperations)
```

---

## Decoupling Assessment

### Can the MediaGallery/MediaContent systems be removed?

**YES — completely.** Only `MediaStorage` is essential. The entire enhanced gallery (16 modules) and content tracking (8 modules) can be disabled:

```bash
bin/magento module:disable \
  Magento_MediaGallery Magento_MediaGalleryApi \
  Magento_MediaGalleryUi Magento_MediaGalleryUiApi \
  Magento_MediaGalleryCatalog Magento_MediaGalleryCatalogIntegration \
  Magento_MediaGalleryCatalogUi Magento_MediaGalleryCmsUi \
  Magento_MediaGalleryIntegration Magento_MediaGalleryMetadata \
  Magento_MediaGalleryMetadataApi Magento_MediaGalleryRenditions \
  Magento_MediaGalleryRenditionsApi Magento_MediaGallerySynchronization \
  Magento_MediaGallerySynchronizationApi Magento_MediaGallerySynchronizationMetadata \
  Magento_MediaContent Magento_MediaContentApi \
  Magento_MediaContentCatalog Magento_MediaContentCms \
  Magento_MediaContentSynchronization Magento_MediaContentSynchronizationApi \
  Magento_MediaContentSynchronizationCatalog Magento_MediaContentSynchronizationCms
```

**Impact:** Admin falls back to the legacy WYSIWYG media browser. No "Used In" tracking. No EXIF/IPTC metadata display. No renditions (admin loads full images). Product image upload still works via MediaStorage.

### Only blocker: RemoteStorage

RemoteStorage depends on `media-gallery-metadata` and `media-gallery-synchronization`. If you use S3/remote storage, you need those 2 modules. If you use local filesystem (most stores), no issue.

**Fix:** Make those 2 deps `suggest` in RemoteStorage's composer.json and make the functionality conditional.

---

## Summary

| System | Modules | Files | External Deps | Removable? |
|--------|---------|-------|--------------|------------|
| **MediaStorage** | 1 | 36 | 20 modules depend on it | **No — essential** |
| **MediaGallery*** | 16 | 215 | 1 (RemoteStorage, fixable) | **Yes — enhanced gallery is optional** |
| **MediaContent*** | 8 | 73 | 0 | **Yes — content tracking is optional** |
| **Total removable** | **24** | **288** | | **288 files of optional media management** |

### Verdict

MediaStorage is the core — keep it. The other 24 modules (288 files) are an enhanced admin media management system that most headless or API-first stores don't need. They're already well-isolated — zero coupling from core modules (Catalog, Cms, etc.) into the gallery/content systems. The only external touch is RemoteStorage needing 2 gallery modules, easily fixable.

For stores that don't use the admin media browser heavily, disabling all 24 modules eliminates 288 PHP files, several database tables, multiple queue consumers, and a React-based admin UI — significant build and runtime savings.
