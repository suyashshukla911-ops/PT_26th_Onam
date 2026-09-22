# EVENT POS — MongoDB Edition

Private one-day event Point-of-Sale (POS) system for fast counter billing, live inventory, event-day reporting, and exports.

This project is intentionally kept to **two runtime files**:

```text
event_pos_final/
├── app.py
└── index.html
```

There is:

- no local SQLite database
- no `event_pos.db`
- no Flask/Django dependency
- no separate JavaScript file
- no separate CSS file
- no external Excel workbook required at runtime

The browser UI is contained in `index.html`. The Python backend, MongoDB logic, billing rules, inventory logic, APIs, CSV exports, Excel export, reset/backup logic, and HTTP server are contained in `app.py`.

---

# 1. What this application does

The POS is designed for the event counter.

A staff member can:

1. Select products.
2. Increase/decrease quantities.
3. Apply an optional discount.
4. Select Cash, UPI, Card, or Other.
5. Enter amount received.
6. Complete the bill.
7. Immediately see:
   - revenue
   - bill count
   - average bill
   - gross profit
   - current inventory value
   - top-selling products
   - payment totals
   - low-stock items
   - recent bills
8. Edit completed bills.
9. Void completed bills.
10. Manually update inventory.
11. Export sales/inventory/movement data.
12. Export an Excel event report based on the supplied workbook template.
13. Reset the event for testing, with a complete backup automatically downloaded first.

---

# 2. Architecture

```text
                 ┌──────────────────────┐
                 │      index.html      │
                 │ Light-blue POS UI    │
                 │ Inline CSS + JS      │
                 └──────────┬───────────┘
                            │
                     HTTP / JSON API
                            │
                 ┌──────────▼───────────┐
                 │        app.py        │
                 │ Python HTTP server   │
                 │ Billing rules        │
                 │ Inventory rules      │
                 │ Dashboard            │
                 │ Exports              │
                 │ Reset / Backup       │
                 └──────────┬───────────┘
                            │
                       PyMongo driver
                            │
                 ┌──────────▼───────────┐
                 │       MongoDB        │
                 │ products             │
                 │ orders               │
                 │ inventory_movements  │
                 │ counters             │
                 │ settings             │
                 └──────────────────────┘
```

The data is stored in MongoDB, not in the browser and not in a local database file.

---

# 3. File-by-file documentation

## `app.py`

`app.py` is the complete backend.

It contains:

- MongoDB connection
- database/collection setup
- product catalog
- prices
- cost prices
- opening inventory
- low-stock thresholds
- Choco Pops combo rules
- bill creation
- bill editing
- bill voiding
- inventory updates
- dashboard calculations
- CSV export
- Excel export
- full backup creation
- event reset
- HTTP API routes
- application startup/shutdown

### Important design rule

The browser does **not** control the final prices.

The browser sends product SKU/type/flavour/quantity information. `app.py` validates the requested item and recalculates the price on the server.

This means changing browser values manually cannot legitimately change the server's billing price.

---

## `index.html`

`index.html` is the complete frontend.

It contains:

- light-blue visual theme
- header
- database connection indicator
- export buttons
- reset button
- dashboard
- KPI cards
- category tabs
- product cards
- Choco Pops combo controls
- current bill/cart
- discount field
- payment selector
- amount received
- change calculation
- complete bill button
- inventory table
- inventory edit dialog
- recent bills table
- edit/void controls
- refresh controls
- frontend JavaScript

The frontend communicates with the backend through `/api/...` endpoints.

---

# 4. Requirements

## Software

Required:

- Python 3.10 or newer
- MongoDB Atlas or another MongoDB deployment that supports transactions
- Internet access if using MongoDB Atlas

Python package:

```powershell
py -m pip install pymongo
```

No Flask installation is required.

No Node.js/npm is required.

---

# 5. MongoDB setup

MongoDB is the permanent storage layer for this application.

## Recommended setup

Use MongoDB Atlas.

Create:

1. A MongoDB Atlas account/project.
2. A cluster.
3. A database user.
4. A network-access rule allowing the machine/server running this application to connect.
5. A connection string.

The connection string will look similar to:

```text
mongodb+srv://USERNAME:PASSWORD@CLUSTER.mongodb.net/?retryWrites=true&w=majority
```

Do not put your real password into source code.

---

# 6. MongoDB environment variables

`app.py` reads the following environment variables:

| Variable | Required | Default | Purpose |
|---|---|---|---|
| `MONGO_URI` | YES | none | MongoDB connection string |
| `MONGO_DB` | NO | `event_pos` | MongoDB database name |
| `EVENT_NAME` | NO | `Society One-Day Event` | Event name shown in UI/data |
| `EVENT_DATE` | NO | `2026-09-26` | Event date in `YYYY-MM-DD` |
| `HOST` | NO | `127.0.0.1` | HTTP bind address |
| `PORT` | NO | `5000` | HTTP server port |

---

# 7. Windows PowerShell setup

Open PowerShell in the folder containing `app.py` and `index.html`.

Install PyMongo:

```powershell
py -m pip install pymongo
```

Set MongoDB connection:

```powershell
$env:MONGO_URI="mongodb+srv://USERNAME:PASSWORD@CLUSTER.mongodb.net/?retryWrites=true&w=majority"
```

Optional event name:

```powershell
$env:EVENT_NAME="Society Onam Event"
```

Optional date:

```powershell
$env:EVENT_DATE="2026-09-26"
```

Optional database name:

```powershell
$env:MONGO_DB="event_pos"
```

Run:

```powershell
py app.py
```

Open:

```text
http://127.0.0.1:5000
```

---

# 8. Windows CMD setup

If using Command Prompt instead of PowerShell:

```cmd
set MONGO_URI=mongodb+srv://USERNAME:PASSWORD@CLUSTER.mongodb.net/?retryWrites=true&w=majority
set EVENT_NAME=Society Onam Event
set EVENT_DATE=2026-09-26
set MONGO_DB=event_pos
py app.py
```

Then open:

```text
http://127.0.0.1:5000
```

---

# 9. What happens at startup

When `app.py` starts:

1. It checks whether `MONGO_URI` exists.
2. It connects to MongoDB.
3. It sends a MongoDB `ping`.
4. It creates the application's required indexes.
5. It creates/initializes the event collections.
6. It inserts the product catalog if a product does not already exist.
7. It does **not** overwrite an existing `current_stock` value when the product already exists.
8. It creates the event settings document if it does not exist.
9. It starts the Python HTTP server.

This means restarting the program does not automatically reset inventory.

---

# 10. MongoDB database structure

The default database name is:

```text
event_pos
```

The application uses these collections.

## `products`

Stores the live product catalog and inventory.

Representative document fields:

```json
{
  "sku": "mewa-laddus",
  "name": "Mewa Laddus",
  "category": "Mewa Laddus",
  "price": 40,
  "cost_price": 24,
  "unit": "pc",
  "opening_stock": 180,
  "current_stock": 180,
  "low_stock_threshold": 20,
  "active": true
}
```

Important:

- `price` = selling price
- `cost_price` = internal cost used for gross-profit/inventory-value calculation
- `opening_stock` = stock to which the RESET EVENT operation returns
- `current_stock` = current live quantity
- `low_stock_threshold` = warning threshold

---

## `orders`

Stores every bill.

Representative fields:

```json
{
  "order_no": "260926-001",
  "created_at": "...",
  "event_date": "2026-09-26",
  "status": "completed",
  "payment_mode": "Cash",
  "subtotal": 140,
  "discount": 0,
  "total": 140,
  "amount_paid": 200,
  "change_due": 60,
  "items": [],
  "consumption": []
}
```

Possible order statuses:

```text
completed
voided
```

---

## `inventory_movements`

This is the inventory audit trail.

It records events such as:

```text
sale
edit restore
edited sale
void restore
manual stock adjustment
```

Representative fields:

```json
{
  "created_at": "...",
  "sku": "mewa-laddus",
  "item": "Mewa Laddus",
  "reason": "sale",
  "delta_qty": -2,
  "before_qty": 180,
  "after_qty": 178,
  "order_no": "260926-001",
  "event_date": "2026-09-26"
}
```

This is useful when checking exactly why inventory changed.

---

## `counters`

Stores the bill-number counter.

The application uses the counter to create sequential bill numbers.

---

## `settings`

Stores event-level settings such as:

```text
event_name
event_date
currency
```

---

# 11. Product catalog currently embedded in `app.py`

Current sale prices:

| Product | Category | Sale price | Unit |
|---|---|---:|---|
| Mewa Laddus | Mewa Laddus | ₹40 | pc |
| Filter Coffee | Coffee | ₹60 | cup |
| Instant Coffee | Coffee | ₹50 | cup |
| Makhana Bhel | Makhana Bhel | ₹99 | plate |
| Bhel Puri | Bhel Puri | ₹70 | plate |
| Badame Khaas | Nawabi Delights | ₹80 | tray |
| Cashew Delight | Nawabi Delights | ₹80 | tray |
| Choco Pops | Choco Pops | ₹40 | pc |
| Water | Water | ₹6 | pc |

The current code also contains cost prices and opening-stock values. These are internal application seed values and should be checked against the final physical inventory before the live event.

---

# 12. Coffee rules

Coffee is one category containing:

```text
Filter Coffee — ₹60
Instant Coffee — ₹50
```

They are separate SKUs and therefore separate inventory quantities.

---

# 13. Nawabi Delights rules

Nawabi Delights contains:

```text
Badame Khaas — ₹80
Cashew Delight — ₹80
```

They are separate products with separate inventory.

---

# 14. Choco Pops rules

Choco Pops has two flavours:

```text
Chana
Peanuts
```

### Single piece

```text
1 Chana = ₹40
1 Peanuts = ₹40
```

### Same-flavour combo

```text
2 Chana = ₹70
2 Peanuts = ₹70
```

### Different-flavour combo

```text
1 Chana + 1 Peanuts = ₹70
```

## Important inventory model

The application keeps **one physical Choco Pops stock pool**:

```text
Choco Pops current_stock
```

The chosen flavour is recorded on the bill for sales information.

Consumption rules:

```text
Single Choco Pops:
1 sale = 1 physical piece

Any 2-for-₹70 combo:
1 combo = 2 physical pieces
```

Therefore:

```text
2 same-flavour combos = 4 pieces consumed
1 same-flavour combo + 1 different-flavour combo = 4 pieces consumed
```

The code does not create separate Chana and Peanuts physical inventory pools.

---

# 15. Billing workflow

## Step 1 — select items

Click a product card.

The frontend adds the product to the current bill.

## Step 2 — change quantity

Use the quantity controls in the cart.

## Step 3 — discount

Enter a discount if required.

The server rejects:

- negative discounts
- discounts greater than subtotal

## Step 4 — payment

Choose:

```text
Cash
UPI
Card
Other
```

## Step 5 — amount received

Enter the amount received.

The server rejects the bill if:

```text
amount received < final total
```

## Step 6 — complete bill

The frontend sends the bill to:

```text
POST /api/orders
```

The server then:

1. validates the items
2. recalculates prices
3. calculates subtotal
4. validates discount
5. calculates total
6. validates amount received
7. calculates change
8. creates a bill number
9. inserts the order
10. deducts inventory
11. records inventory movements

These operations are wrapped in a MongoDB transaction.

---

# 16. Why transactions matter

A completed sale should not produce a situation such as:

```text
Bill saved
Inventory not reduced
```

or:

```text
Inventory reduced
Bill not saved
```

The application performs bill creation and stock consumption inside a MongoDB transaction.

The same approach is used for:

- bill editing
- bill voiding
- manual inventory updates
- event reset

Use a MongoDB deployment that supports transactions.

---

# 17. Bill editing

The Recent Bills section allows an existing completed bill to be opened again.

When a bill is edited:

```text
Old consumption restored
        ↓
New consumption calculated
        ↓
New stock deducted
        ↓
Bill updated
        ↓
Dashboard refreshed
```

For example:

Original:

```text
2 Mewa Laddus
1 Water
Total ₹86
```

Change to:

```text
3 Mewa Laddus
1 Water
Total ₹126
```

The inventory is restored for the old bill and then recalculated for the new bill.

---

# 18. Bill voiding

Void is different from delete.

The application marks the bill as:

```text
voided
```

and restores the inventory consumed by that bill.

Voided bills are not included in completed revenue totals.

This preserves the bill's history while removing it from operational sales totals.

---

# 19. Manual inventory editing

Each product can have:

```text
Current Stock
Low Stock Threshold
```

changed from the Inventory section.

When manual stock is changed, the application records the difference in:

```text
inventory_movements
```

Example:

```text
Before: 100
After: 85
Delta: -15
Reason: manual stock adjustment
```

This makes manual changes traceable.

---

# 20. Dashboard calculations

The dashboard is calculated from MongoDB data.

## Revenue

Sum of the `total` field for:

```text
status = completed
```

for the selected date.

## Bills

Number of completed bills for the selected date.

## Average Bill

```text
Revenue / completed bill count
```

## Gross Profit

For every sold line:

```text
Line Revenue - Line Cost
```

summed across completed bills.

## Inventory Value

For every active product:

```text
current_stock × cost_price
```

summed across products.

## Top Products

The dashboard groups sold items and sorts them by revenue.

For Choco Pops combos, physical quantity is counted as two pieces per combo.

## Payment Mix

Completed bill totals grouped by:

```text
Cash
UPI
Card
Other
```

## Low Stock

A product is shown as low stock when:

```text
current_stock <= low_stock_threshold
```

---

# 21. Date handling

The application uses Indian Standard Time (IST):

```text
UTC + 05:30
```

Dashboard/order queries convert the selected Indian calendar date into UTC database boundaries.

Therefore the event date is treated as an IST event date rather than a UTC date.

---

# 22. API reference

## Health

```http
GET /api/health
```

Checks MongoDB connectivity.

---

## Event metadata

```http
GET /api/meta
```

Returns:

- event name
- event date
- currency
- category list

---

## Products

```http
GET /api/products
```

Returns active products and current stock.

---

## Dashboard

```http
GET /api/dashboard?date=YYYY-MM-DD
```

Returns:

- revenue
- bills
- average bill
- gross profit
- inventory value
- payments
- top products
- low stock

---

## Orders for date

```http
GET /api/orders?date=YYYY-MM-DD
```

Returns recent bills for the selected date.

---

## Single order

```http
GET /api/orders/<ORDER_NO>
```

Returns one bill.

---

## Create order

```http
POST /api/orders
```

Creates a new completed bill and updates inventory.

Example body:

```json
{
  "payment_mode": "Cash",
  "amount_paid": 200,
  "discount": 0,
  "notes": "",
  "items": [
    {
      "kind": "single",
      "sku": "mewa-laddus",
      "qty": 2
    },
    {
      "kind": "single",
      "sku": "filter-coffee",
      "qty": 1
    }
  ]
}
```

Do not rely on client-side prices. The backend recalculates them.

---

## Edit order

```http
PUT /api/orders/<ORDER_NO>
```

Replaces the completed bill after restoring old inventory and applying the new consumption.

---

## Void order

```http
DELETE /api/orders/<ORDER_NO>
```

Voids the bill and restores its inventory.

---

## Update inventory

```http
POST /api/inventory/<SKU>/stock
```

Example:

```json
{
  "current_stock": 150,
  "low_stock_threshold": 20
}
```

---

# 23. Export endpoints

## Sales CSV

```text
GET /api/export/sales.csv
```

Downloads:

```text
event_sales.csv
```

---

## Inventory CSV

```text
GET /api/export/inventory.csv
```

Downloads:

```text
event_inventory.csv
```

---

## Inventory movements CSV

```text
GET /api/export/inventory-movements.csv
```

Downloads:

```text
inventory_movements.csv
```

---

## Summary CSV

```text
GET /api/export/summary.csv
```

Downloads:

```text
event_summary.csv
```

---

## Excel report

```text
GET /api/export/excel.xlsx
```

Downloads:

```text
event_report.xlsx
```

---

# 24. Excel export design

The Excel template is embedded directly inside `app.py`.

This means the application does not need a separate `.xlsx` file to run.

The export process:

1. Loads the embedded copy of the supplied workbook.
2. Preserves the workbook's existing structure.
3. Writes live POS data into the dedicated POS sheets.
4. Returns the finished `.xlsx`.

The application currently writes live POS data into these dedicated workbook sheet XML files:

```text
sheet8.xml
sheet9.xml
sheet10.xml
```

These correspond to the POS export sheets created for:

```text
POS Sales Log
POS Inventory
POS Inventory Movements
```

## Important

Do not casually replace the embedded workbook template without checking the sheet structure.

If the workbook is changed so the expected sheets are moved/reordered, the Excel export logic may need to be updated.

---

# 25. Full reset / testing

The header contains:

```text
RESET EVENT
```

This is intended for testing before the real event.

## What RESET EVENT does

The frontend asks for confirmation.

Then the server:

1. Generates a complete backup ZIP.
2. Returns that ZIP to the browser for download.
3. Deletes all orders.
4. Deletes all inventory movements.
5. Restores every product to its opening stock.
6. Restores low-stock thresholds to configured values.
7. Resets the bill counter.
8. Returns the application to its initial event state.

The reset operation is performed transactionally.

---

# 26. Reset backup contents

The reset backup ZIP contains:

```text
event_reset_backup_YYYY-MM-DD.zip
├── sales.csv
├── inventory.csv
├── inventory_movements.csv
├── summary.csv
├── event_report.xlsx
└── backup.json
```

`backup.json` contains structured copies of:

- event information
- orders
- inventory
- inventory movements

Therefore the reset is destructive to live MongoDB event data, but the previous event state is exported first.

---

# 27. Recommended test procedure

Before the event, use this exact test sequence.

## Test 1 — MongoDB connection

Run:

```powershell
py app.py
```

Open:

```text
http://127.0.0.1:5000
```

Confirm the header says MongoDB is connected.

---

## Test 2 — Single product sale

Create:

```text
2 × Mewa Laddus
```

Expected:

```text
Subtotal = ₹80
Total = ₹80
Bills = 1
Revenue = ₹80
```

Inventory should decrease by 2.

Recent Bills should show the bill.

---

## Test 3 — Multiple products

Create:

```text
2 × Mewa Laddus
1 × Filter Coffee
1 × Water
```

Expected subtotal:

```text
₹80 + ₹60 + ₹6 = ₹146
```

Revenue should become:

```text
₹146
```

Average bill should become:

```text
₹146
```

---

## Test 4 — Change calculation

Suppose total is:

```text
₹146
```

Amount received:

```text
₹200
```

Expected change:

```text
₹54
```

---

## Test 5 — Discount

Subtotal:

```text
₹146
```

Discount:

```text
₹10
```

Expected total:

```text
₹136
```

---

## Test 6 — Choco Pops single

Sell:

```text
1 Chana
```

Expected:

```text
₹40
```

Physical Choco Pops stock decreases by:

```text
1
```

---

## Test 7 — Same-flavour combo

Sell:

```text
2 Chana
```

through the combo button.

Expected:

```text
₹70
```

Physical Choco Pops stock decreases by:

```text
2
```

---

## Test 8 — Different-flavour combo

Sell:

```text
1 Chana + 1 Peanuts
```

Expected:

```text
₹70
```

Physical Choco Pops stock decreases by:

```text
2
```

---

## Test 9 — Edit bill

1. Create a bill.
2. Open it from Recent Bills.
3. Add/remove an item.
4. Save changes.

Check:

- old stock restored
- new stock applied
- new total calculated
- dashboard updated
- Recent Bills updated

---

## Test 10 — Void bill

1. Create a bill.
2. Void it.

Check:

- bill status becomes voided
- stock is restored
- completed revenue excludes that bill

---

## Test 11 — Manual inventory edit

Change a product's stock.

Check:

- inventory table changes
- low-stock status changes if applicable
- movement history/export records the adjustment

---

## Test 12 — CSV exports

Download:

```text
Sales CSV
Inventory CSV
Movements CSV
```

Open them in Excel.

Confirm the rows match MongoDB data.

---

## Test 13 — Excel export

Download:

```text
Excel
```

Open the resulting workbook.

Verify:

```text
POS Sales Log
POS Inventory
POS Inventory Movements
```

---

## Test 14 — Reset

After all testing:

1. Click `RESET EVENT`.
2. Confirm the warning.
3. Verify the ZIP is downloaded.
4. Verify bills are gone from active totals.
5. Verify inventory equals opening inventory.
6. Verify bill numbering starts over.

---

# 28. How to inspect the data in MongoDB Atlas

Open MongoDB Atlas.

Go to:

```text
Database
→ Browse Collections
```

Select your database:

```text
event_pos
```

Then inspect:

```text
products
orders
inventory_movements
counters
settings
```

## Check inventory

Open:

```text
products
```

Look at:

```text
current_stock
```

## Check bills

Open:

```text
orders
```

Look at:

```text
order_no
status
total
payment_mode
items
consumption
```

## Check why stock changed

Open:

```text
inventory_movements
```

Look at:

```text
reason
delta_qty
before_qty
after_qty
order_no
```

This is the primary audit trail.

---

# 29. Data integrity rules

The following rules are enforced by the backend:

### Product validation

Unknown SKUs are rejected.

### Quantity validation

Quantity must be greater than zero.

### Discount validation

Discount cannot be:

```text
negative
```

or greater than:

```text
subtotal
```

### Payment validation

Only:

```text
Cash
UPI
Card
Other
```

are accepted.

### Amount received

Amount received must be at least:

```text
total
```

### Inventory

Inventory cannot be manually set below zero.

### Edit restrictions

Only completed bills can be edited.

### Void restrictions

A bill cannot be voided twice.

---

# 30. Security notes

This application is designed as an internal event POS.

At the current stage it does **not** implement user login/authentication.

Therefore:

```text
Do not expose the application directly to the public internet
```

unless authentication and access control are added.

Especially protect:

```text
POST /api/reset
```

because it intentionally deletes the event's active orders and movement records after creating the backup.

Also never commit:

```text
MONGO_URI
```

or the MongoDB password to GitHub.

Use environment variables/secrets instead.

---

# 31. Deployment notes

The Python server binds to:

```text
127.0.0.1
```

by default.

For a server environment, configure:

```text
HOST=0.0.0.0
PORT=<platform-provided-port>
```

and provide:

```text
MONGO_URI=<secret MongoDB URI>
MONGO_DB=event_pos
EVENT_NAME=<event name>
EVENT_DATE=<YYYY-MM-DD>
```

The application expects MongoDB to be reachable from the deployed server.

For a public deployment, place authentication and HTTPS/access controls in front of the application.

---

# 32. Updating the event date/name

Use environment variables rather than editing the Python source every time.

Example:

```powershell
$env:EVENT_NAME="Ganesh Chaturthi Society Event"
$env:EVENT_DATE="2026-09-26"
```

Then run:

```powershell
py app.py
```

---

# 33. Changing product prices

Prices are defined in the `PRODUCTS` list inside `app.py`.

Example:

```python
{
    "sku": "mewa-laddus",
    "name": "Mewa Laddus",
    "category": "Mewa Laddus",
    "price": 40.0,
    ...
}
```

When changing a sale price:

1. Change the server-side `price`.
2. Restart the application.
3. Verify the product card.
4. Create a test bill.
5. Verify the dashboard.
6. Verify the export.

Because existing MongoDB product documents are seeded with `$setOnInsert`, restarting alone does not overwrite already-existing catalog fields.

For a live event, change product/catalog settings deliberately and re-test.

---

# 34. Changing opening inventory

Opening inventory is also defined in the `PRODUCTS` list.

For example:

```python
"opening_stock": 180
```

Important distinction:

```text
opening_stock
```

is the baseline used by RESET EVENT.

```text
current_stock
```

is the live quantity in MongoDB.

Changing the Python value does not automatically overwrite a product's existing MongoDB `current_stock`.

If the event has already started, do not change opening values casually.

---

# 35. What not to edit casually

Do not change these without understanding their relationship:

```text
SKU values
order counter
consumption rules
Choco Pops combo logic
MongoDB collection names
API route names
HTML element IDs used by JavaScript
Excel sheet mapping
```

Changing an HTML ID can break the corresponding frontend action.

Changing a SKU in the backend while existing MongoDB records use the old SKU can create data inconsistencies.

---

# 36. If the UI button does not work

Check these in order.

## 1. Is `app.py` running?

The browser must be opened through:

```text
http://127.0.0.1:5000
```

not by opening `index.html` directly from File Explorer.

Correct:

```text
http://127.0.0.1:5000
```

Incorrect:

```text
file:///C:/.../index.html
```

---

## 2. Check MongoDB status

The header contains a MongoDB connection status indicator.

If MongoDB is unavailable, data actions will fail.

---

## 3. Open browser developer tools

Press:

```text
F12
```

Go to:

```text
Console
```

and:

```text
Network
```

Look for failed `/api/...` requests.

---

## 4. Check the Python terminal

The backend prints requests and errors.

Read the terminal where:

```text
py app.py
```

is running.

---

# 37. Common MongoDB connection problems

## `MONGO_URI is missing`

Set:

```powershell
$env:MONGO_URI="..."
```

before running the app.

---

## Authentication error

Check:

- database username
- database password
- cluster hostname
- database user permissions

---

## Network timeout

Check MongoDB Atlas Network Access.

The server/device running `app.py` must be allowed to connect.

---

## Password contains special characters

MongoDB URI credentials must be URI-safe.

Characters such as:

```text
@
:
/
?
#
%
```

may need URL encoding when placed in a connection string.

---

# 38. Important note about MongoDB transactions

The backend uses transactions for multi-step operations.

Therefore the MongoDB environment must support transactions.

MongoDB Atlas is the intended deployment model.

If using a self-hosted MongoDB server, configure it in a deployment mode that supports transactions.

---

# 39. Backup philosophy

There are three useful backup/export paths.

## Sales CSV

Best for simple sales analysis.

## Excel report

Best for event reporting while preserving the supplied workbook structure.

## RESET backup ZIP

Best for preserving the entire current event state before a test reset.

The reset ZIP includes both business-readable files and a machine-readable `backup.json`.

---

# 40. Recommended event-day operating procedure

## Before opening the stall

1. Confirm MongoDB is reachable.
2. Start `app.py`.
3. Open the POS in the browser.
4. Check inventory quantities.
5. Check the event name/date.
6. Test one dummy bill.
7. Export a starting Excel report if desired.
8. Reset the event if the dummy bill should not remain.
9. Verify opening stock one final time.

## During sales

Use only the billing section for normal sales.

Avoid manually changing inventory unless there is a real physical stock correction.

## After the event

1. Stop accepting new bills.
2. Export Sales CSV.
3. Export Inventory CSV.
4. Export Movements CSV.
5. Export Excel.
6. Keep the MongoDB data.
7. Keep the exported files as event records.

---

# 41. Quick command reference

### Install

```powershell
py -m pip install pymongo
```

### Run

```powershell
py app.py
```

### Open

```text
http://127.0.0.1:5000
```

### Stop

Press:

```text
Ctrl + C
```

---

# 42. Minimal working setup

The minimum required files are:

```text
event_pos_final/
├── app.py
└── index.html
```

Then:

```powershell
py -m pip install pymongo
$env:MONGO_URI="YOUR_MONGODB_URI"
py app.py
```

Open:

```text
http://127.0.0.1:5000
```

That is the complete runtime setup.

---

# 43. Current implementation summary

### Storage

```text
MongoDB
```

### Backend

```text
Python
```

### Frontend

```text
HTML
CSS
JavaScript
```

### Database driver

```text
PyMongo
```

### HTTP server

```text
Python ThreadingHTTPServer
```

### Local database

```text
None
```

### Excel library dependency

```text
None
```

The Excel workbook is manipulated using the embedded workbook/package XML logic already contained in `app.py`.

---

# 44. Final file responsibilities

```text
app.py
│
├── MongoDB connection
├── Product catalog
├── Prices
├── Costs
├── Inventory
├── Billing
├── Combos
├── Transactions
├── Orders
├── Dashboard
├── CSV export
├── Excel export
├── Backup
├── Reset
├── HTTP API
└── Python server

index.html
│
├── UI
├── Styling
├── Dashboard display
├── Product selection
├── Cart
├── Payment form
├── Inventory controls
├── Recent bills
├── Edit controls
├── Export buttons
├── Reset button
└── Browser-side API calls
```

---

# 45. Final operational rule

Treat MongoDB as the source of truth.

```text
MongoDB
   ↓
Backend calculations
   ↓
Frontend display
```

Do not use the browser display itself as the authoritative inventory or sales record.

The authoritative records are:

```text
orders
products
inventory_movements
```

in MongoDB.

---

# 46. Important current-code note

The current implementation creates application indexes automatically.

You should **not manually create the same indexes in MongoDB Atlas**.

If MongoDB reports an index-definition error during startup, check the Python index definitions in `ensure_collections()` before changing database indexes manually. In particular, the `_id` index is already special and does not need an application-created uniqueness index.

---

# 47. Version / project state

Documentation prepared for the current two-file MongoDB POS build:

```text
Backend:  app.py
Frontend: index.html
Storage:  MongoDB
Currency: INR / ₹
Timezone: IST
```

Before the live event, complete the full test procedure above and verify the opening inventory and cost values against the final physical stock and accounting sheet.
