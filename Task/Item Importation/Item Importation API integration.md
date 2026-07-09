# Architecture & API Reference

## Overview

The Item Importation module provides a UI for managing imported inventory items. Users can create, edit, delete, search, sort, bulk‑approve, import via CSV/Excel, fetch voucher lines from the Infor M3 ERP system, fetch M3 item details, and generate reports.

**Related:** [[Technical Assessment]]

---

## Folder Structure

```
ItemImportation/
├── index.vue                         # Main view — toolbar, table, modals
├── method/
│   └── itemImportationMethods.js     # Vue component logic (data, methods, watchers)
├── Components/
│   ├── ItemImportationFormModal.vue  # Create / Edit form modal (M3 fetch added here)
│   ├── ItemDetailModal.vue           # Read‑only detail modal
│   ├── VoucherLookupModal.vue        # Fetch voucher lines from M3
│   ├── PendingItemsPage.vue          # Pending items approval page
│   ├── ReportTable.vue               # Report table modal
│   └── ReportComponents/
│       ├── ReportPrint.vue           # Print report
│       └── ReportCSV.vue             # CSV export
└── docs.md                           # This file
```

---

## Calling m3

### Feature: Fetch M3 Item Detail (MMS200MI / GetItmBasic + GetItmWhsBal)

**Why:** Users needed a way to pull item details (name, description, status, quantity) directly from the Infor M3 ERP by entering an item code, instead of manually typing them. This reduces data-entry errors and speeds up item creation.

**What was added:**
| File | Change |
|------|--------|
| `routes/api.php:2526` | New route: `POST item-importation/m3/item-detail` |
| `ItemImportationController.php:368-436` | New method `getM3ItemDetail()` |
| `ItemImportationFormModal.vue` | M3 Item Code input + Fetch button (visible when creating) |

---

## How the Fetch Flow Works

```
User types item code → clicks "Fetch"
        │
        ▼
ItemImportationFormModal.vue — fetchM3Item()
        │
        ├── Validates input is not empty
        │
        ├── POST /api/processautomation/finance/item-importation/m3/item-detail
        │      Body: { item_code: "123456", warehouse: "WBD" }
        │             (warehouse defaults to "WBD" if omitted)
        │
        ▼
routes/api.php → ItemImportationController@getM3ItemDetail
        │
        ▼
Controller::getM3ItemDetail($request)
        │
        ├── 1. Get master data ──────────────────────────────────
        │    callM3ApiFunc(
        │      env: "tst", multi: false,
        │      program: "MMS200MI",
        │      trans:   "GetItmBasic",
        │      params:  [ CONO => 110, ITNO => $itemCode ],
        │      columns: [ ITNO, FUDS, ITDS, UNMS, STAT ]
        │    )
        │    └── returns: { ITNO, FUDS (item name), ITDS (desc), UNMS, STAT }
        │
        ├── 2. Get warehouse balance ────────────────────────────
        │    callM3ApiFunc(
        │      env: "tst", multi: false,
        │      program: "MMS200MI",
        │      trans:   "GetItmWhsBal",
        │      params:  [ CONO => 110, WHLO => $warehouse, ITNO => $itemCode ],
        │      columns: [ STQT, MBSTQT ]
        │    )
        │    └── returns: { STQT (total stock), MBSTQT (on-hand approved) }
        │
        ├── Merge data from both calls
        │
        └── return { success: true, data: { ITNO, FUDS, ITDS, UNMS, STAT, STQT, MBSTQT } }
        │
        ▼
ItemImportationFormModal.vue receives response
        │
        ├── form.item_name    = res.data.data.FUDS
        ├── form.description  = res.data.data.ITDS
        ├── form.status       = res.data.data.STAT
        ├── form.item_qty     = res.data.data.MBSTQT ?? res.data.data.STQT ?? 0
        └── Toast: "Item details fetched from M3."
```

---

## Frontend Architecture

### Component Tree

```
index.vue
 ├── PendingItemsPage         (when showPendingPage = true)
 ├── b-table                  (main data table)
 ├── ItemForm                 (create / edit modal — M3 fetch added here)
 ├── ItemDetailModal          (read‑only detail modal)
 ├── VoucherLookupModal       (M3 voucher lookup)
 └── ReportTable              (report modal)
```

### Key Data Flow

| Concern              | File                          | Details                                 |
|----------------------|-------------------------------|-----------------------------------------|
| Component logic      | `method/itemImportationMethods.js` | All data, methods, watchers live here (script src‑linked from `index.vue`) |
| Props / Events       | Parent → child via `v-model` + `@` events | Standard Vue 2 + Buefy pattern         |
| HTTP calls           | `axios`                       | Always `async/await` with try‑catch     |
| UI feedback          | `this.$buefy.toast.open()`    | Success / error toasts                  |
| Confirmations        | `this.$buefy.dialog.confirm()`| Delete confirmations                    |

---

## Backend Architecture

### Database Table: `item_importation`

**Model:** `app\Models\ItemImportationModel.php` — Eloquent model, `$fillable`:

| Column       | Type     | Notes                        |
|--------------|----------|------------------------------|
| id           | int (PK) | Auto‑increment               |
| item_code    | string   | Unique, auto‑generated `TEST####` or from import |
| item_name    | string   | Required                     |
| description  | text     | Nullable                     |
| item_qty     | integer  |                              |
| status       | string   | Numeric M3 status code (e.g. `20`, `90`, `99`) |
| remarks      | text     | Nullable                     |
| created_at   | datetime |                              |
| updated_at   | datetime |                              |

### Controller: `app\Http\Controllers\ItemImportationController.php`

Single controller class handling all endpoints via named methods.

### M3 API Helper: `callM3ApiFunc()` — `app/helpers.php:36`

```php
function callM3ApiFunc(
    $env       = 'tst',    // 'tst' | 'prd'
    $multiple  = false,    // single or multiple result rows
    $api,                  // M3 program name, e.g. 'MMS200MI'
    $command,              // M3 transaction,   e.g. 'GetItmBasic'
    $params,               // associative array of input fields
    $columns  = []         // array of output field names to return
);
```

**Internally** instantiates `M3Api` (from `app/InforM3Apis.php`), sets properties, calls `$m3->get()`, returns result object with `->status`, `->message`, `->data`.

### Routes: `routes/api.php` (lines 2515–2527)

All routes under `middleware('auth:api')->prefix('/processautomation/finance')`.

| Method | URI | Controller | Purpose |
|--------|-----|------------|---------|
| GET  | item-importation/get_data | @index | List active items, paginated |
| POST | item-importation/add_data | @store | Create single item |
| POST | item-importation/edit_data | @update | Update existing item |
| POST | item-importation/delete_data | @destroy | Delete one or more items |
| POST | item-importation/import | @import | Bulk import CSV/XLSX |
| GET  | item-importation/print | @printReport | Report by date range |
| GET  | item-importation/pending_items | @pendingItems | List pending items |
| POST | item-importation/approve_items | @approveItems | Approve pending items |
| POST | item-importation/update_remarks | @updateRemarks | Save remarks |
| POST | item-importation/voucher/lines | @lines | Fetch M3 voucher lines |
| **POST** | **item-importation/m3/item-detail** | **@getM3ItemDetail** | **Fetch M3 item details** |
| GET  | item-importation/export_csv | @exportCSV | Export to CSV |

---

## M3 API Integration — Three Use Cases

### 1. Voucher Lines (GLS200MI / LstVoucherLines)
- **Input:** `voucher_year`, `voucher_no`
- **M3 Params:** `CONO`, `DIVI`, `YEA4`, `VONO`
- **Output:** VONO, JRNO, JSNO, ACDT, CUAM, AIT1-6
- **Modal:** `VoucherLookupModal.vue`

### 2. Item Detail (MMS200MI / GetItmBasic + GetItmWhsBal)
- **Input:** `item_code`, `warehouse` (optional, defaults to `"WBD"`)
- **M3 Transactions (2 calls):**

  | Order | Transaction    | Purpose              | Output Fields                              |
  |-------|----------------|----------------------|--------------------------------------------|
  | 1st   | `GetItmBasic`  | Master data          | `ITNO`, `FUDS`, `ITDS`, `UNMS`, `STAT`     |
  | 2nd   | `GetItmWhsBal` | Warehouse balance    | `STQT` (total stock), `MBSTQT` (on-hand approved) |

- **M3 Params:** `CONO: 110`, `ITNO`, `WHLO` (2nd call only)
- **Frontend Mapping:**

  | Response Field | Form Field      |
  |----------------|-----------------|
  | `FUDS`         | `item_name`     |
  | `ITDS`         | `description`   |
  | `STAT`         | `status`        |
  | `MBSTQT`       | `item_qty`      |
  | `STQT`         | fallback for qty|

- **Location:** `ItemImportationFormModal.vue`
- **Behavior:** Auto-fills name, description, status, and quantity after fetch

---

## API Endpoints Reference

### GET `item-importation/get_data`
- **Controller:** `ItemImportationController@index`
- **Purpose:** List active items with server‑side pagination, sorting, search.
- **Params:**

| Param   | Type   | Default      | Description                 |
|---------|--------|--------------|-----------------------------|
| page    | int    | 1            | Page number                 |
| sort    | string | `created_at` | Sort column                 |
| order   | string | `desc`       | Sort direction              |
| search  | string | null         | Search by `item_name` (LIKE)|

- **Response:**
```json
{
    "success": true,
    "data": {
        "data": [ ... ],
        "total": 42,
        "per_page": 10,
        "current_page": 1,
        "last_page": 5
    }
}
```

### POST `item-importation/m3/item-detail`
- **Controller:** `ItemImportationController@getM3ItemDetail`
- **Purpose:** Fetch item details + warehouse quantity from Infor M3 by item code.
- **Body:**

| Field      | Type   | Default | Description                |
|------------|--------|---------|----------------------------|
| item_code  | string | —       | M3 item number (required)  |
| warehouse  | string | `"WBD"` | Warehouse for stock lookup |

- **M3 Calls (sequential):**

  | # | Program    | Transaction    | Params                          | Columns                |
  |---|------------|----------------|---------------------------------|------------------------|
  | 1 | `MMS200MI` | `GetItmBasic`  | `CONO: 110`, `ITNO`             | `ITNO, FUDS, ITDS, UNMS, STAT` |
  | 2 | `MMS200MI` | `GetItmWhsBal` | `CONO: 110`, `WHLO`, `ITNO`     | `STQT, MBSTQT`        |

- **Response:**

| Status | Condition         | Response |
|--------|-------------------|----------|
| 200    | Both calls succeed| `{ success: true, data: { ITNO, FUDS, ITDS, UNMS, STAT, STQT, MBSTQT } }` |
| 200    | GetItmBasic only  | `{ success: true, data: { ITNO, FUDS, ITDS, UNMS, STAT, STQT: 0, MBSTQT: 0 } }` |
| 200    | GetItmBasic fails | `{ success: false, message: "..." }` |
| 422    | Empty item_code   | `{ success: false, message: "Item code is required." }` |
| 500    | Exception         | `{ success: false, message }` |

---

## Error Handling Convention

All controller methods follow the same try‑catch pattern:

| Layer      | On error                          |
|------------|-----------------------------------|
| Controller | Returns `{ success: false, message }` with HTTP 500 |
| Vue        | Shows `$buefy.toast.open({ type: 'is-danger' })`    |

Validation errors (422) are handled specifically in `handleSave()` for the create/edit form — `serverFormErrors` is populated from `error.response.data.errors`.

---

## Notes

- `destroy-on-hide="false"` on modals means state persists across opens.
- The search watcher in `itemImportationMethods.js` fires on every keystroke (no debounce).
- The `index()` method uses `orderBy('id', $sortOrder)` as a secondary sort for stable pagination.
- `self::$infor_env = "tst"` — change to `"prd"` for production.
- The M3 Item Code input + Fetch button is only visible when **creating** a new item, not when editing.
- The `getM3ItemDetail` endpoint makes **two sequential** M3 calls — if `GetItmWhsBal` fails, it still returns `GetItmBasic` data with quantity defaulted to `0`.
- Status is a **numeric** M3 code (e.g. `20` = active, `90` = inactive), not a string label.
- `MBSTQT` (on-hand approved) is preferred over `STQT` (total stock) for the quantity field.
- Default warehouse is `"WBD"` — override via the `warehouse` request field.
