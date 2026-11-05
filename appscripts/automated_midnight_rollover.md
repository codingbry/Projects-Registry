# 🏪 Office Mini Honesty Store — Automated Midnight Rollover & Inventory Architecture Report

**Date**: November 5, 2026

**Subject**: Implementing Automated Midnight Inventory Rollover and Separating Lazy Initialization

**Tech Stack**: Google Apps Script, Google Sheets, JavaScript

## 📌 Executive Summary

To balance automated daily inventory management with strict bulk import isolation, the system architecture has been refined. We have separated real-time, on-demand record creation from scheduled daily processing. 

The system now features a dedicated **Automated Midnight Rollover** routine that runs via Google Apps Script time-driven triggers, ensuring products with active stock automatically carry over their closing balances into the new day. Meanwhile, unlisted items and zero-stock products remain completely excluded unless explicitly imported, purchased, or edited.

---

## 🛠️ Core Architectural Components

### 1. Automated Midnight Rollover (`dailyMidnightRollOver`)
* **Trigger Source**: Google Apps Script Time-Driven Trigger (Daily at 12:00 AM – 1:00 AM).
* **Functionality**: 
  - Scans yesterday's audit logs in `DailyStocks`.
  - Identifies products that finished yesterday with a **Closing Stock > 0**.
  - Automatically creates today's audit row, setting yesterday's closing stock as today's **Opening Stock**.
- **Benefit**: Automates daily transitions for active store items without human intervention, while ignoring dead inventory or zero-stock products.

### 2. On-Demand / Lazy Initialization (`ensureDailyRecordForProduct`)
* **Trigger Source**: Real-time operations (Bulk imports, individual product edits, and checkout purchases).
* **Functionality**: 
  - Ensures a `DailyStocks` record exists strictly for a single specific product ID when an action is actively performed on it.
  - Carries over previous closing stock if available, defaulting to `0` otherwise.
- **Benefit**: Guarantees that active transactions or imports never fail due to missing rows, while maintaining strict isolation for unlisted catalog items.

---

## 📊 Database & Workflow Integration

1. **At Midnight (12:00 AM)**: 
   - `dailyMidnightRollOver()` executes in the background.
   - Active products with positive stock get their opening balances established for the new day.
2. **During Daytime Operations (Imports / Purchases)**: 
   - If an item was skipped at midnight because it had zero stock, but is included in a bulk import or purchased today, `ensureDailyRecordForProduct()` instantly creates its row on demand.
3. **Unlisted Products**: 
   - Items not present in the master catalog or file imports are completely bypassed by both routines, keeping the `DailyStocks` sheet clean and free of unwanted zero-filled rows.

---

## 📂 Updated Backend Implementation (`Database.gs`)

```javascript
/**
 * AUTOMATED MIDNIGHT ROLLOVER:
 * Runs daily at 12:00 AM via a time-driven trigger.
 * Automatically creates today's DailyStocks rows for products that had 
 * positive closing stock yesterday, carrying it over as Opening Stock.
 */
function dailyMidnightRollOver() {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const dailySheet = ss.getSheetByName('DailyStocks');
  if (!dailySheet) return;

  const todayStr = getTodayDateString();
  const dailyData = dailySheet.getDataRange().getValues();

  const existingTodayProducts = new Set();
  for (let i = 1; i < dailyData.length; i++) {
    const rowDate = formatDateValue(dailyData[i][0]);
    if (rowDate === todayStr) {
      existingTodayProducts.add(String(dailyData[i][1]).trim());
    }
  }

  const d = new Date(todayStr + 'T00:00:00');
  d.setDate(d.getDate() - 1);
  const prevDateStr = Utilities.formatDate(d, "Asia/Manila", "yyyy-MM-dd");

  const rowsToAppend = [];
  for (let i = 1; i < dailyData.length; i++) {
    const rowDate = formatDateValue(dailyData[i][0]);
    const pId = String(dailyData[i][1]).trim();
    const closingStock = Number(dailyData[i][5]) || 0;

    if (rowDate === prevDateStr && closingStock > 0) {
      if (!existingTodayProducts.has(pId)) {
        const opening = closingStock;
        const restocked = 0;
        const sales = 0;
        const closing = opening;

        rowsToAppend.push([todayStr, pId, opening, restocked, sales, closing]);
        existingTodayProducts.add(pId);
      }
    }
  }

  if (rowsToAppend.length > 0) {
    dailySheet.getRange(dailySheet.getLastRow() + 1, 1, rowsToAppend.length, 6).setValues(rowsToAppend);
  }
}
```

*Report compiled for the Office Mini Honesty Store Development Team.*