# Merge Notes: Fork Fix vs. Upstream Dev

This document describes upstream changes in the `dev` branch that may cause
conflicts when merging a future `paperless-ngx/main` release into the fork.

Applies to: the 6 files modified by our fix.

---

## 1. `abstract-paperless-service.ts`

**Upstream change:**
`publishReplay(1), refCount()` replaced with `shareReplay({ bufferSize: 1, refCount: true })`.

**Conflict risk:** Low
Our version already uses `shareReplay` — identical to the upstream change.

---

## 2. `document-detail.component.ts`

**Upstream changes (extensive):**

- `ZoomSetting` enum **removed** from this file, moved to `settings.component.ts`
- New feature: **Document Versioning**
  - Import `DocumentVersionDropdownComponent`
  - Import `DocumentVersionInfo` from `document.ts`
  - New methods: `onVersionSelected()`, `onVersionsUpdated()`
  - New properties: `selectedVersionId`, `document?.versions`
- New imports: `TagService`, `TagEditDialogComponent`, `PasswordRemovalConfirmDialogComponent`,
  `DocumentDetailFieldID`, `RouterModule`
- PDF viewer refactored: `PdfViewerModule` (ng2-pdf-viewer) removed, replaced with `PngxPdfViewerComponent`
- New types: `PdfRenderMode`, `PdfSource`, `PdfZoomLevel`, `PdfZoomScale`, `PngxPdfDocumentProxy`
- `DocumentHistoryComponent` path changed: `../document-history/` to `./document-history/`
- New enum value: `Duplicates = 8` in `DocumentDetailNavIDs`

**Our fix (3 lines that must be preserved):**

```typescript
fetchCorrespondent = (id: number) => this.correspondentService.get(id)
fetchDocumentType = (id: number) => this.documentTypeService.get(id)
fetchStoragePath = (id: number) => this.storagePathService.get(id)
```

**Conflict risk:** High
The file was heavily refactored. After merging, locate the 3 lines above in the new
context and re-add them manually if needed.

---

## 3. `document-detail.component.html`

**Upstream changes (extensive):**

- `<pngx-page-header>` gets `[id]="documentId"`
- New element `<pngx-document-version-dropdown>`:
  ```html
  <pngx-document-version-dropdown
    [documentId]="documentId"
    [versions]="document?.versions ?? []"
    [selectedVersionId]="selectedVersionId"
    [userIsOwner]="userIsOwner"
    [userCanEdit]="userCanEdit"
    (versionSelected)="onVersionSelected($event)"
    (versionsUpdated)="onVersionsUpdated($event)"
  />
  ```
- Download button gets `[selectedVersionId]` parameter
- PDF viewer: `<pdf-viewer>` replaced with `<pngx-pdf-viewer>`

**Our fix (3 attributes that must be preserved):**

```html
[fetchItem]="fetchCorrespondent" <!-- on pngx-input-select for correspondent -->
[fetchItem]="fetchDocumentType" <!-- on pngx-input-select for document_type -->
[fetchItem]="fetchStoragePath" <!-- on pngx-input-select for storage_path -->
```

**Conflict risk:** High
After merging, verify the 3 `[fetchItem]` bindings are present on their respective
`pngx-input-select` elements.

---

## 4. `select.component.ts` / `tags.component.ts` / `object-name.pipe.ts`

**Upstream changes:** None relevant between v2.20.8 and dev.

**Conflict risk:** Low

---

## How to Merge on Next Upstream Release

```bash
git fetch origin
git checkout main
git merge origin/main

# On conflicts in document-detail.component.ts:
# accept upstream version, then re-add the 3 fetchXxx lines

# On conflicts in document-detail.component.html:
# accept upstream version, then re-add the 3 [fetchItem] bindings

# Run tests
cd src-ui && npx jest --no-coverage

git push fork main
```

---

## Notable Upstream Dev Features (not yet in main/latest)

For reference — coming in the next major release:

- **Document Versioning**
- **Password Removal Workflow Action**
- **New PDF Viewer** (`pngx-pdf-viewer` replaces ng2-pdf-viewer)
- **Sharelink Bundles**
- **Paperless AI**
- **Remote OCR (Azure AI)**
- **Duplicate Detection**
- **Breaking:** API v1 removed
- **Breaking:** Document and thumbnail encryption removed
- **Breaking:** pybzar barcode reader removed
- **DB migrations:** New tables/fields for versioning and other features —
  rolling back to an older version after upgrading requires manual intervention
