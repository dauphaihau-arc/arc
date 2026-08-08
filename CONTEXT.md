# Arc Seller Catalog

This context describes seller-owned product catalog management for Arc, including product drafts and bulk XLSX product imports.

## Language

**Product Import**:
An asynchronous seller operation that parses an uploaded XLSX file and creates product drafts with a row-level report.
_Avoid_: Catalog upload, spreadsheet sync

**Product Import Job**:
The persisted async record created after a valid XLSX upload to track status, progress, source file, and report availability.
_Avoid_: Import request, upload task

**Import Row**:
One row in the `Products` sheet that creates one non-variant product draft in v1.
_Avoid_: Product line, spreadsheet item

**Import Template**:
The backend-generated versioned XLSX structure accepted by product import.
_Avoid_: Sample file, example spreadsheet

**Template Version**:
The declared structure version used by the parser to interpret product import sheets and columns.
_Avoid_: File version

**Products Sheet**:
The worksheet named `Products` containing v1 import headers and product rows.
_Avoid_: First sheet, data sheet

**Metadata Sheet**:
The worksheet containing template metadata such as `template_version`.
_Avoid_: Config sheet

**Import Preview**:
A browser-generated advisory view of the selected XLSX used to help the seller confirm file shape before backend submission.
_Avoid_: Validation result, dry run

**Import Upload**:
The multipart upload of the original XLSX selected by the seller.
_Avoid_: Parsed JSON upload

**Import Source File**:
The original uploaded XLSX stored privately for async processing and short-term audit or debugging.
_Avoid_: Public file, transformed file

**Import Report**:
A downloadable CSV artifact with row-level created and failed outcomes plus error details.
_Avoid_: Log file, final response

**Template Validation**:
Whole-file validation that checks file type, sheet presence, template version, required headers, duplicate headers, formula headers, and row limits.
_Avoid_: Business validation

**Row Validation**:
Per-row business validation that decides whether a single import row can create a product draft.
_Avoid_: Template validation

**SKU Conflict**:
A row-level validation error raised when an imported SKU duplicates another row in the file or an existing SKU in the same shop.
_Avoid_: Duplicate product

**Category Reference**:
A human-readable category path from the import file that must resolve to exactly one existing category.
_Avoid_: Category id in template, new category

**Price Cell**:
A numeric spreadsheet value expressed in shop-currency major units and converted by the backend into minor units.
_Avoid_: Formatted money string

**Boolean Cell**:
A spreadsheet cell normalized from strict `yes`, `no`, `true`, or `false` text, with blank optional cells using documented defaults.
_Avoid_: Localized boolean

**Formula Cell**:
An unsupported XLSX cell containing a spreadsheet formula.
_Avoid_: Computed import value

**Import Draft Creation**:
Product draft creation performed through existing product application use cases rather than direct database writes.
_Avoid_: Bulk insert

**Row Import Attempt**:
The processing of one valid parsed import row into one product draft, recorded independently for retry-safe reporting.
_Avoid_: Batch transaction

**Completed Import**:
An import job that finished processing all parsed rows, regardless of whether some rows failed validation.
_Avoid_: Fully successful import

**Job Failure**:
An import-level failure that stops processing before all rows are attempted while retaining already-created drafts.
_Avoid_: Rollback

**Import Authorization**:
The requirement that product import is allowed only for authenticated actors who can manage the target shop.
_Avoid_: Upload permission

**Import Request Idempotency**:
Duplicate upload protection that maps one seller import submission to one Product Import Job.
_Avoid_: Worker retry safety

**XLSX Parser**:
SheetJS usage on both client and backend, where only backend parsing is authoritative.
_Avoid_: Excel parser

**Image Import**:
An out-of-scope v1 capability for creating product images from spreadsheet data.
_Avoid_: Image URL column

**Import History**:
An out-of-scope v1 UI for browsing previous product import jobs.
_Avoid_: Active import result

**Import Completion Notification**:
An out-of-scope v1 notification that would tell sellers an import finished after they leave the import page.
_Avoid_: Progress state

## Relationships

- A **Product Import** creates exactly one **Product Import Job**.
- A **Product Import Job** belongs to exactly one shop and one requesting seller.
- A **Product Import Job** has one **Import Source File**.
- A **Product Import Job** produces zero or one **Import Report**.
- A **Product Import Job** processes many **Import Rows**.
- An **Import Row** creates at most one product draft in v1.
- A **Completed Import** may contain many failed **Row Import Attempts**.
- A **Template Validation** failure stops the whole **Product Import** before row creation.
- A **Row Validation** failure affects only one **Import Row**.
- A **Category Reference** must resolve to exactly one existing category.
- A **SKU Conflict** fails the affected **Import Row** and does not update existing products.
- An **Import Preview** reads the selected XLSX but does not become the **Import Upload** payload.
- The backend **XLSX Parser** reparses the **Import Source File** and is authoritative.
- **Import Draft Creation** uses existing product use cases to preserve product invariants.
- **Import Request Idempotency** prevents duplicate jobs, while **Row Import Attempt** tracking prevents duplicate drafts on worker retry.

## Example Dialogue

> **Dev:** "Can the seller upload a spreadsheet that updates products with matching SKUs?"
> **Domain expert:** "No. In v1 a **SKU Conflict** fails the **Import Row**. **Product Import** only creates new draft products."
>
> **Dev:** "If one row has an invalid category, should the whole import fail?"
> **Domain expert:** "No. Invalid **Category Reference** is **Row Validation**, so that row fails and valid rows can still create drafts. Missing headers are **Template Validation**, so those stop the import."
>
> **Dev:** "Can the frontend send the parsed SheetJS JSON to the API?"
> **Domain expert:** "No. The frontend only provides an **Import Preview**. The **Import Upload** sends the original XLSX, and the backend **XLSX Parser** is authoritative."

## Flagged Ambiguities

- "Import products" could mean create, update, or publish products. Resolved: **Product Import** v1 creates draft products only.
- "Row" could mean one product or one variant/inventory row. Resolved: an **Import Row** creates one non-variant product draft in v1.
- "Validation" could mean browser preview or backend enforcement. Resolved: **Import Preview** is advisory; **Template Validation** and **Row Validation** on the backend are authoritative.
- "Completed" could mean every row succeeded. Resolved: **Completed Import** means all parsed rows were processed; failed rows may still exist in the **Import Report**.
- "Idempotency" could mean request deduplication or worker retry safety. Resolved: **Import Request Idempotency** deduplicates uploads, while **Row Import Attempt** tracking protects worker retries.
