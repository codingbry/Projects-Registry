# 🏪 Office Mini Honesty Store

A lightweight, full-stack web application built using **Google Apps Script**, **Google Sheets**, and **HTML5/JavaScript** designed to manage an office mini honesty store. It supports user authentication, multi-product checkout, credit tracking ("Vale"), real-time inventory management with inline restocking, and daily transaction reporting.

---

## ✨ Key Features

* **Secure User Authentication**: ID and PIN-based login with browser session persistence (`localStorage`) so users stay logged in across page refreshes.
* **Multi-Product Cart & Checkout**: Add multiple items to a single checkout session, view live totals, and pay via **Cash** or charge to **Vale (Credit)**.
* **Smart Searchable Dropdowns**: Type product names or IDs to filter items instantly. **Out-of-stock items are automatically hidden** from the purchase dropdown.
* **Interactive Inventory Management**: Toggle into "Edit / Restock Mode" to update prices, names, and stock levels inline, add brand-new products, or filter/sort items by price, stock status, and name.
* **Daily Transaction Logs**: Filter transaction histories by date with clear time stamps, user IDs, quantities, totals, and payment type badges.
* **Zero Cost / Serverless Hosting**: Runs entirely inside your Google Workspace ecosystem via Google Sheets and Google Apps Script Web Apps.

---

## 📁 Project Architecture & Files

The project consists of two core files inside the Google Apps Script editor:

1. **`Code.gs`**: The backend server-side logic handling automated sheet creation (`Users`, `Products`, `Transactions`), user login validation, transaction logging, stock deductions, and inventory updates.
2. **`Index.html`**: The single-page frontend user interface built with responsive HTML/CSS and vanilla JavaScript, featuring tabbed navigation and asynchronous communication (`google.script.run`) with the backend.

---

## 🚀 Setup & Installation Guide

### Step 1: Create a Google Sheet
1. Go to [Google Sheets](https://sheets.google.com) and create a new blank spreadsheet.
2. Name it something recognizable, e.g., `Office Mini Honesty Store`.

### Step 2: Open the Apps Script Editor
1. In your spreadsheet toolbar, click on **Extensions > Apps Script**.
2. Delete any default code in `Code.gs`.

### Step 3: Add the Project Files
1. Create/paste the backend code into **`Code.gs`**.
2. Create an HTML file named **`Index`** (Apps Script automatically appends `.html`) and paste the frontend code.

### Step 4: Deploy as a Web App
1. In the top-right corner of the Apps Script editor, click **Deploy > New deployment**.
2. Click the gear icon next to *Select type* and choose **Web app**.
3. Configure the deployment settings:
   * **Description**: `Version 1.0`
   * **Execute as**: `Me` ( your Google account )
   * **Who has access**: `Anyone within your organization` (or `Anyone` if external users need access)
4. Click **Deploy**, review/grant permissions, and copy the **Web app URL**.

---

## 🔑 Default Seed Credentials

Upon first launch, the backend automatically seeds initial demo data into your Google Sheets:

* **Admin User**: 
  * **User ID**: `U001`
  * **PIN**: `1234`
* **Employee User**: 
  * **User ID**: `U002`
  * **PIN**: `0000`

---

## 📊 Database Schema (`Google Sheets`)

The app automatically generates and manages three sheets:

1. **`Users`**: Stores user credentials and access roles.
   * `User ID` | `Name` | `PIN` | `Role`
2. **`Products`**: Stores active catalog items, pricing, and live inventory counts.
   * `Product ID` | `Name` | `Price` | `Stock Remaining`
3. **`Transactions`**: Logs every checkout event.
   * `Transaction ID` | `Timestamp` | `User ID` | `Product ID` | `Quantity` | `Total Price` | `Payment Type` (Cash / Vale)

---

## 🛠️ Usage Guide

1. **Login**: Open the Web App URL and enter your User ID and PIN.
2. **Buying Items**: Go to the **Buy / Vale** tab, search for products using the search input, specify quantities, and choose either **Pay Cash** or **Charge to Vale**.
3. **Checking Inventory**: Switch to the **Inventory** tab to view stock levels. Click **Edit / Restock Mode** to modify prices, update stock, or add new rows directly.
4. **Viewing Daily Logs**: Switch to the **Daily Log** tab to filter and review transactions by specific dates.