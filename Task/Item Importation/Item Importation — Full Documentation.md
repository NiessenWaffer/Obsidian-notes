Terminal call: resources/js/views/ProcessAutomation/Finance/Components/SalesMobility/ItemImportation/

## Overview

The Item Importation module manages inventory items with CRUD operations, bulk import via CSV/Excel, approval workflows, M3 ERP integration (item lookup and voucher lines), and reporting (print/CSV export).

---

## 1. Folder Structure

```
resources/js/views/ProcessAutomation/Finance/Components/SalesMobility/ItemImportation/
│
├── index.vue                                   # Main view — mounts the entire feature
├── method/
│   └── itemImportationMethods.js               # All component logic (data, methods, watchers)
│
├── Components/
│   ├── ItemImportationFormModal.vue            # Create / Edit form with M3 auto-fetch
│   ├── ItemDetailModal.vue                     # Read-only detail view
│   ├── VoucherLookupModal.vue                  # Fetch voucher lines from M3
│   ├── PendingItemsPage.vue                    # Pending items approval sub-page
│   ├── ReportTable.vue                         # Report modal with search, sort, print, CSV
│   │
│   └── ReportComponents/
│       ├── ReportPrint.vue                      # Print preview modal
│       └── ReportCSV.vue                       # CSV export with download
│
├── docs.md                                      # This file
│
└── app/Http/Controllers/
    └── ItemImportationController.php            # Backend controller (all endpoints)
```

---

## 2. Database

**Table:** `item_importation`  
**Model:** `app\Models\ItemImportationModel.php`

| Column       | Type     | Notes                            |
|--------------|----------|----------------------------------|
| id           | int (PK) | Auto-increment                   |
| item_code    | string   | Unique — auto-generated `TEST####` or from file/M3 |
| item_name    | string   | Required                         |
| description  | text     | Nullable                         |
| item_qty     | integer  |                                  |
| status       | string   | M3 numeric code (e.g. `20`, `90`, `99`) |
| remarks      | text     | Nullable                         |
| created_at   | datetime |                                  |
| updated_at   | datetime |                                  |

---

## 3. Frontend Architecture

### 3.1 Main View (`index.vue` + `itemImportationMethods.js`)

The template is in `index.vue`, but all `data`, `methods`, `computed`, `watch`, and `components` are defined in `itemImportationMethods.js` and linked via:

```html
<script src="./method/itemImportationMethods.js"></script>
```

### 3.2 Component Tree

```
index.vue
 ├── PendingItemsPage              (when showPendingPage = true)
 │   └── Internal table + approve controls
 │
 └── [when not on pending page]
     ├── Toolbar
     │   ├── Actions dropdown
     │   │   ├── Create Item
     │   │   ├── Sort By (Recent / Oldest / Name A-Z / Name Z-A)
     │   │   ├── Import File (CSV/XLSX)
     │   │   ├── View Report
     │   │   ├── Fetch Voucher
     │   │   └── Pending Items
     │   ├── Search input
     │   └── Bulk Delete button (visible when rows selected)
     │
     ├── b-table (paginated, sortable, checkable)
     │   ├── Columns: Item Code, Item Name, Description, Quantity, Status
     │   └── Details button per row
     │
     ├── ItemForm                   (v-model="showForm")
     ├── ItemDetailModal            (v-model="isDetailModalOpen")
     ├── VoucherLookupModal         (v-model="showVoucherLookup")
     └── ReportTable                (v-model="showReportTable")
```

### 3.3 Data Flow Patterns

| Concern           | Mechanism                                   |
|-------------------|---------------------------------------------|
| Modal open/close  | `v-model` binding to a boolean              |
| Edit item         | Set `editingItem` → open form → `watch.item` populates fields |
| Save create       | Form emits `saved` → parent calls `POST add_data` |
| Save edit         | Form emits `saved` → parent calls `POST edit_data` |
| Delete            | `$buefy.dialog.confirm()` → `POST delete_data` |
| HTTP              | `axios` with `async/await` + try-catch      |
| Feedback          | `this.$buefy.toast.open()`                  |

### 3.4 Watchers

| Watcher             | Action                                    |
|---------------------|-------------------------------------------|
| `searchQuery`       | Reset page to 1, call `getData()`         |
| `showForm` (closing)| Clear `editingItem` and `serverFormErrors`|
| `showForm` (opening)| Clear `serverFormErrors`                  |
| `showPendingPage`   | Refresh main table when returning         |

---

## 4. Backend Architecture

### 4.1 Controller

**File:** `app\Http\Controllers\ItemImportationController.php`  
All endpoints are methods on this single controller.

### 4.2 Routes

**File:** `routes/api.php` (lines 2515–2527)  
Group: `middleware('auth:api')->prefix('/processautomation/finance')`

### 4.3 M3 API Helper

**File:** `app\helpers.php` — function `callM3ApiFunc()`

```php
callM3ApiFunc(
    $env,       // 'tst' | 'prd'
    $multiple,  // true = multiple result rows
    $api,       // M3 program name, e.g. 'MMS200MI'
    $command,   // M3 transaction,   e.g. 'GetItmBasic'
    $params,    // associative array: [ 'CONO' => 110, 'ITNO' => ... ]
    $columns    // array of output fields to return
);
```

Returns: `{ status, message, data }`

---

## 5. Complete API Reference

### 5.1 GET `item-importation/get_data`
**Controller:** `@index`  
**Purpose:** List all items with server-side pagination, sorting, and search.

| Param   | Type   | Default      | Description            |
|---------|--------|--------------|------------------------|
| page    | int    | 1            | Page number            |
| sort    | string | `created_at` | Sort column            |
| order   | string | `desc`       | Sort direction         |
| search  | string | null         | Search `item_name` (LIKE) |

**Response:**
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

---

### 5.2 POST `item-importation/add_data`
**Controller:** `@store`  
**Purpose:** Create a single item. Auto-generates `item_code` (`TEST0001`...).

**Body:** `{ item_name, description?, item_qty, status? }`  
**Default status:** `20`

---

### 5.3 POST `item-importation/edit_data`
**Controller:** `@update`  
**Purpose:** Update an existing item. Validates `id` exists.

**Body:** `{ id, item_code, item_name, description?, item_qty, status }`

---

### 5.4 POST `item-importation/delete_data`
**Controller:** `@destroy`  
**Purpose:** Delete one or more items.

**Body:** `{ ids: [1, 2, 3] }`

---

### 5.5 POST `item-importation/import`
**Controller:** `@import`  
**Purpose:** Upload CSV/XLSX/XLS to bulk-import items.

**Body:** `multipart/form-data` — `file` field  
**Required columns:** `item_code`, `item_name`, `description`, `item_qty`, `status`

**Behavior:**
- Validates column headers exist
- Validates required fields per row
- Skips rows where `item_code` already exists (returns skipped list)
- Default status: `20`

**Response on error (422):**
```json
{
    "success": false,
    "message": "...",
    "errors": ["Row 2: item_code is empty", ...]
}
```

---

### 5.6 GET `item-importation/pending_items`
**Controller:** `@pendingItems`  
**Purpose:** List items awaiting approval (status != `20`), paginated.

**Response:** Same paginated shape as `get_data`.

---

### 5.7 POST `item-importation/approve_items`
**Controller:** `@approveItems`  
**Purpose:** Approve items by setting status to `20`.

**Body:** `{ ids: [1, 2, 3] }`

---

### 5.8 POST `item-importation/update_remarks`
**Controller:** `@updateRemarks`  
**Purpose:** Save remarks on an item.

**Body:** `{ id, remarks? }`

---

### 5.9 POST `item-importation/voucher/lines`
**Controller:** `@lines`  
**Purpose:** Fetch voucher lines from M3 (GLS200MI / LstVoucherLines).

**Body:** `{ voucher_year, voucher_no }`

**M3 Input:** `{ CONO: 110, DIVI: "WBD", YEA4, VONO }`  
**M3 Output:** `[ CONO, DIVI, VONO, JRNO, JSNO, ACDT, CUAM, AIT1..AIT6 ]`

**Response:**
```json
{
    "success": true,
    "data": [ { "VONO": "...", "JRNO": "...", ... } ]
}
```

---

### 5.10 POST `item-importation/m3/item-detail`
**Controller:** `@getM3ItemDetail`  
**Purpose:** Fetch item master data + warehouse stock from M3.

**Body:**

| Field      | Type   | Default | Description                 |
|------------|--------|---------|-----------------------------|
| item_code  | string | —       | M3 item number (required)   |
| warehouse  | string | `"WBD"` | Warehouse for stock balance |

**M3 Calls (sequential):**

| # | Program    | Transaction    | Input Params                  | Output Columns          |
|---|------------|----------------|-------------------------------|-------------------------|
| 1 | `MMS200MI` | `GetItmBasic`  | `CONO`, `ITNO`                | `ITNO, FUDS, ITDS, UNMS, STAT` |
| 2 | `MMS200MI` | `GetItmWhsBal` | `CONO`, `WHLO`, `ITNO`        | `STQT, MBSTQT`         |

**Field Mapping:**

| M3 Field | Form Field   | Description            |
|----------|--------------|------------------------|
| `FUDS`   | `item_name`  | Full description       |
| `ITDS`   | `description`| Item description       |
| `STAT`   | `status`     | M3 numeric status code |
| `MBSTQT` | `item_qty`   | On-hand approved qty   |
| `STQT`   | fallback qty | Total stock qty        |

**Response:**
```json
{
    "success": true,
    "data": {
        "ITNO": "123456",
        "FUDS": "Item Name",
        "ITDS": "Description text",
        "UNMS": "PCS",
        "STAT": "20",
        "STQT": 100,
        "MBSTQT": 90
    }
}
```

**Fallback behavior:** If `GetItmWhsBal` fails, still returns `GetItmBasic` data with `STQT: 0`, `MBSTQT: 0`.

---

### 5.11 GET `item-importation/print`
**Controller:** `@printReport`  
**Purpose:** Fetch items by date range for the print preview.

**Params:** `start_date?`, `end_date?`

---

### 5.12 GET `item-importation/export_csv`
**Controller:** `@exportCSV`  
**Purpose:** Fetch items by date range for CSV export.  
**Note:** The CSV file is assembled client-side by `ReportCSV.vue`.

---

## 6. M3 Integration — Three Functions

| # | Feature          | Program    | Transaction      | Frontend Component    |
|---|------------------|------------|------------------|-----------------------|
| 1 | Voucher Lines    | `GLS200MI` | `LstVoucherLines` | `VoucherLookupModal`  |
| 2 | Item Master Data | `MMS200MI` | `GetItmBasic`     | `ItemImportationFormModal` |
| 3 | Warehouse Stock  | `MMS200MI` | `GetItmWhsBal`    | `ItemImportationFormModal` |

All use the same `callM3ApiFunc()` helper with `env = "tst"` and `CONO = 110`.

---

## 7. User Flows

### 7.1 Create Item with M3 Auto-Fetch
1. Click **Actions → Create Item**
2. Type M3 item code → **Fetch** → auto-fills Name, Description, Status, Quantity
3. Adjust any field → **Save Item**
4. Toast: *"Item created."* → table refreshes

### 7.2 Edit Item
1. Click **eye** → Detail modal → **Edit**
2. Modify fields → **Update Item**
3. Toast: *"Item updated."* → row updates in-place

### 7.3 Bulk Delete
1. Check rows → **Delete (N)** button appears
2. Confirm dialog → items removed

### 7.4 Import File
1. Actions → **Import File** → select CSV/XLSX
2. Backend validates columns, skips duplicates
3. Dialog shows skipped items, if any

### 7.5 Approve Pending Items
1. Actions → **Pending Items** → sub-page loads items with status != 20
2. Check rows → **Approve (N)**
3. Backend sets status → `20`, items removed from pending list

### 7.6 Fetch Voucher
1. Actions → **Fetch Voucher** → enter year + number
2. Results table displays voucher lines from M3

### 7.7 View Report
1. Actions → **View Report** → report modal with search, sort, date-range print, CSV export

---

## 8. Status Values

Status is a **numeric M3 code**, not a string label:

| Code | M3 Meaning     |
|------|----------------|
| `20` | Active/Defined |
| `90` | Inactive       |
| `99` | Obsolete       |
| `11` | (varies by setup) |
| `80` | (varies by setup) |

- Form input accepts any value (free text)
- Main table shows all items regardless of status
- Pending page shows items where status != `20`
- Approve sets status to `20`

---

## 9. Error Handling

All controller methods follow this pattern:

```
try → return { success: true, data }
catch → return { success: false, message }, HTTP 500
```

Frontend catches in `try-catch` and shows `$buefy.toast.open()`:
- Green toast (`is-success`) for success
- Red toast (`is-danger`) for errors
- Yellow toast (`is-warning`) for validation

Validation errors (422) on create/edit are mapped to individual form fields via `serverFormErrors`.

---

## 10. Known Caveats

- **Search fires on every keystroke** — no debounce, resets to page 1 each time.
- **M3 environment** is hardcoded to `"tst"` — change `self::$infor_env` to `"prd"` for production.
- **Modals keep state** (`destroy-on-hide="false"`) — data persists when closed/reopened, except `editingItem` which is cleared on close.
- **M3 item fetch makes 2 sequential calls** — `GetItmWhsBal` is not critical; if it fails, default is `0`.
- **Warehouse is hardcoded to `"WBD"`** — can be overridden via the `warehouse` request field.
