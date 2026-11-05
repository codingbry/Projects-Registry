# 🏪 Office Mini Honesty Store — Refactored Architecture & Update Report

**Date**: November 4, 2026  
**Subject**: Architecture Refactoring, On-Demand Inventory Initialization, and Bulk Import Fixes  
**Tech Stack**: Google Apps Script, Google Sheets, JavaScript, SheetJS  

---

## 📌 Executive Summary

Today’s release focuses on resolving critical data leakage issues during bulk Excel/CSV file imports. The system previously suffered from unintended background audit row generation in `DailyStocks`, where unlisted catalog items were automatically initialized with zeroed records (`0` stock) during imports or app loads.

Through a targeted architectural refactor, mass-initialization logic has been completely replaced with **Lazy / On-Demand Stock Initialization**. The system now strictly guarantees that bulk imports and single-item updates interact **only** with products explicitly referenced in the transaction or file import payload.

---

## 🛠️ Key Improvements & Resolved Issues

### 1. Mass-Population Bug Elimination
* **Root Cause**: The legacy `initializeDailyRecordsForDate` routine scanned the entire master catalog and appended blank audit rows into `DailyStocks` for every product in the store whenever `getProducts()`, `purchaseMultipleProducts()`, or `getInventoryReport()` ran.
* **Fix**: Removed blanket initialization calls across all backend controllers. Unlisted products remain untouched in `DailyStocks` until explicitly imported, purchased, or edited.

### 2. On-Demand "Lazy" Inventory Initialization
* **Implementation**: Created `ensureDailyRecordForProduct(todayStr, pId)` in `Database.gs`.
* **Behavior**: When a product is imported or purchased, the system checks if a record exists for today. If absent, it carries over yesterday's closing stock as today's opening balance **strictly for that individual product ID**.

### 3. Streamlined Bulk Import Flow
* **File Precision**: Bulk imports using SheetJS now process exclusively the products present in the uploaded document.
* **Explicit Zero Preservation**: Restocked quantities explicitly set to `0` in the file payload are accurately captured without falling back to blank defaults or being skipped.

### 4. Controller Consolidation & Deduplication
* **Single Source of Truth**: Refactored `bulkUpdateProducts` to delegate catalog updating and stock synchronization directly to `updateSingleProduct` and `_syncDailyStock`.
* **Code Cleanliness**: Eliminated duplicated function declarations and redundant spreadsheet lookup logic across server scripts.

---

## 📊 Modernized Database Architecture (`Google Sheets`)

The underlying Google Sheets database consists of four synchronized sheets:

1. **`Users`**: Authentication & Roles
   * `User ID` | `Name` | `PIN` | `Role`
2. **`Products`**: Master Catalog *(Strictly 3 Columns)*
   * `Product ID` | `Name` | `Price`
3. **`DailyStocks`**: Historical Daily Audit Log
   * `Date` | `Product ID` | `Opening Stock` | `Restocked` | `Sales` | `Closing Stock`
4. **`Transactions`**: Order Audit Trail
   * `Transaction ID` | `Timestamp` | `User ID` | `Product ID` | `Quantity` | `Total Price` | `Payment Type`

---

## 📂 Updated Backend Architecture (`Apps Script`)

| File Name | Functionality & Fixes Applied Today |
| :--- | :--- |
| **`Database.gs`** | Contains core sheet setups and `ensureDailyRecordForProduct(todayStr, pId)` for single-item lazy initialization. |
| **`ProductController.gs`** | Handles `getProducts()`, `updateSingleProduct()`, and `bulkUpdateProducts()`. Generates zero background rows for unlisted catalog items during file imports. |
| **`TransactionController.gs`** | Manages `purchaseMultipleProducts()`, `getTransactions()`, and `cancelTransaction()`. Lazily initializes `DailyStocks` records strictly for products present in the cart payload. |
| **`InventoryController.gs`** | Controls `getInventoryReport()`. Dynamically reads stock status without writing blank audit rows to Google Sheets. |

---

## 🎯 Verification & Testing Results

- **Bulk Import Isolation**: Uploaded an Excel file containing 2 items out of 50 catalog items. `DailyStocks` received exactly 2 updated/appended rows. The remaining 48 unlisted products generated 0 audit rows.
- **Stock Rollover Continuity**: Unlisted products continue to track their latest closing stock balance correctly in memory without polluting the sheet with zeroed historical records.
- **Checkout Integrity**: Purchases of initialized or uninitialized items succeed smoothly, creating targeted `DailyStocks` rows on demand.

---
*Report compiled for the Office Mini Honesty Store Development Team.*