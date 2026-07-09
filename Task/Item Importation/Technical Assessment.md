# Junior Developer Technical Assessment: Item Importation Module

**Related:** [[Item Importation API integration]]

## Objective
Develop a simple **Item Importation** module using **Laravel** and **Vue.js**. The module should support basic CRUD operations, file uploads, approval workflow preparation, and report viewing.

---

## 🎨 Frontend Requirements (Vue.js)

### 1. Navigation & Access
- [x] Create a new menu item named **Item Importation**.
- [x] Include a placeholder **Approval Module** page linked to future approval workflows. → [[Item Importation API integration#Approval Workflow|Implementation]]
- [x] Ensure menu visibility follows existing application standards.

### 2. Item Importation Page
Implement the following components:
#### 📤 Upload Functionality
- [x] Add an **Upload** button for importing item data. → [[Item Importation API integration#File Import|Implementation]]
- [x] Accept CSV or Excel files.
- [x] Display appropriate success and error messages.
- [x] **Validation Criteria for Templates**:
  - Template is empty
  - Template with wrong column name
  - Required fields: `item_code` and `item_qty`

#### 📊 Data Table
- [x] Implement a server-side paginated table supporting:
  - Pagination
  - Sorting
  - Search/Filtering
- [x] Display imported item records. → [[Item Importation API integration#1. Main Table Data Fetch getData|Implementation]]

#### 📝 CRUD Operations (Modal Forms)
- [x] **Create Item** → [[Item Importation API integration#ItemImportationFormModal vue|Implementation]]
- [x] **View Item Details**
- [x] **Edit Item**
- [x] **Delete Item** (with confirmation dialog) → [[Item Importation API integration#Bulk Delete|Implementation]]

#### ⚠️ Validation Handling
- [x] Display validation errors returned from the backend. → [[Item Importation API integration#State data|serverFormErrors]]
- [x] Prevent submission of invalid data.

### 3. Report Viewing
- [x] Create a page or modal for viewing a custom report. → [[Item Importation API integration#Toolbar Dropdown Menu|Implementation]]
- [x] Display report data in a readable tabular format.
- [x] Allow filtering by available criteria.
- [x] *Optional Bonus:* Include an option to export or print. → [[Item Importation API integration#Structure|ReportPrint / ReportCSV]]

---

## ⚙️ Backend Requirements (Laravel)

### 1. Controller Development
Create a Resource Controller following Laravel conventions containing:
- `index()`
- `store()`
- `show()`
- `update()`
- `destroy()`

### 2. Request Validation
Create Form Request classes and implement validation rules for:
- Item creation
- Item update
- File upload
- **Requirements**:
  - Return meaningful error messages.
  - Prevent invalid records from being saved.

### 3. Business Logic & Error Handling
- [x] Implement service/controller methods with proper exception handling using `try-catch` blocks.
- [x] Implement database transaction handling where applicable.
- [x] Return a consistent API response structure.
- [x] Ensure error logging for unexpected exceptions.

#### 📬 Example API Response Formats
**Success:**
```json
{
  "success": true,
  "message": "Record saved successfully",
  "data": {}
}
```

**Error:**
```json
{
  "success": false,
  "message": "Unable to process request"
}
```

### 4. API Development (`routes/api.php`)
Define the following RESTful API endpoints:
- `GET /api/items` - Fetch/filter item records (paginated) → [[Item Importation API integration#3. All API Endpoints|Implementation: get_data]]
- `POST /api/items` - Create a single item → [[Item Importation API integration#3. All API Endpoints|Implementation: add_data]]
- `GET /api/items/{id}` - Fetch single item details
- `PUT /api/items/{id}` - Update an item → [[Item Importation API integration#3. All API Endpoints|Implementation: edit_data]]
- `DELETE /api/items/{id}` - Delete an item → [[Item Importation API integration#3. All API Endpoints|Implementation: delete_data]]
- `POST /api/items/upload` - Upload and parse CSV/Excel file → [[Item Importation API integration#3. All API Endpoints|Implementation: import]]

---

## 🗄️ Database Schema
Create migration(s) for the Item Importation module with the following suggested fields:
- `id` (Primary Key)
- `item_code` (string, required/unique)
- `item_name` (string)
- `description` (text, nullable)
- `quantity` (integer)
- `status` (string, e.g., pending, approved, rejected)
- `created_at`
- `updated_at`

---

## 🌟 Bonus Merits
- Clean code structure
- Repository or Service pattern implementation
- Reusable Vue components → [[Item Importation API integration#Structure|Separated component files]]
- Unit or Feature tests 
- Proper Git commit history
- Responsive UI design
- Loading indicators and user-friendly notifications → [[Item Importation API integration#State data|isLoading state]]
