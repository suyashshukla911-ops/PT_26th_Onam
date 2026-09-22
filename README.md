# PT_26th_Onam Event POS — GitHub Pages + Render + MongoDB Atlas

This is the final deployment setup for the private event billing/POS website.

## Final architecture

```text
                     ┌────────────────────────────┐
                     │         GitHub Pages        │
                     │        index.html           │
                     │      Light-blue POS UI      │
                     └──────────────┬─────────────┘
                                    │ HTTPS API
                                    ▼
                     ┌────────────────────────────┐
                     │           Render            │
                     │          app.py             │
                     │ Python HTTP backend + API   │
                     └──────────────┬─────────────┘
                                    │ PyMongo
                                    ▼
                     ┌────────────────────────────┐
                     │       MongoDB Atlas         │
                     │       event_pos database    │
                     └────────────────────────────┘
```

GitHub Pages is used only for the frontend. GitHub documents Pages as a static publishing service and states that it does not support server-side Python.

Render runs the Python backend as a Web Service. Render requires a public web service to bind to `0.0.0.0` and to listen on the provided `PORT` environment variable. The final `app.py` does this automatically.

---

# 1. Final files

The final runtime code is intentionally only:

```text
PT_26th_Onam/
├── app.py
├── index.html
└── README.md
```

There is no local SQLite database.

There is no `.env` file in the final package.

There is no Node.js/npm requirement.

There is no Flask/Django requirement.

`app.py` contains the complete Python backend.

`index.html` contains the complete frontend, including its CSS and browser-side JavaScript.

---

# 2. IMPORTANT security note

The previous ZIP contained a `.env` file with a MongoDB URI containing credentials.

**Do not upload that `.env` file to GitHub.**

The final package deliberately removes it.

Because the old credential appeared in the previous archive and may already have been placed into Git history, the safest action is to rotate/change the MongoDB database-user password before the final deployment.

Never put the MongoDB connection string in `index.html`.

MongoDB credentials belong in Render environment variables/secrets. Render specifically recommends environment variables for secrets such as database connection strings and warns against committing `.env` files to source control.

---

# 3. What changed from the local version

The important deployment problem was:

```text
Local:
Browser → 127.0.0.1:5000/app.py → MongoDB
```

but after publishing only the HTML:

```text
GitHub Pages → index.html
             X
         no Python
             X
         no /api/*
```

The final version changes this to:

```text
GitHub Pages → Render app.py → MongoDB Atlas
```

The frontend now has an API-base setting.

On localhost it automatically uses the same origin:

```text
http://127.0.0.1:5000
```

On GitHub Pages it uses:

```text
RENDER_API_URL
```

which you set once after creating the Render service.

---

# 4. Frontend API configuration

Near the beginning of the JavaScript section in `index.html`:

```javascript
const RENDER_API_URL = "https://YOUR-RENDER-SERVICE.onrender.com";
```

After Render creates your service, replace only this value.

Example:

```javascript
const RENDER_API_URL = "https://pt-26th-onam-pos.onrender.com";
```

Do not add a trailing `/`.

Do not add `/api`.

Correct:

```text
https://pt-26th-onam-pos.onrender.com
```

Incorrect:

```text
https://pt-26th-onam-pos.onrender.com/
```

Incorrect:

```text
https://pt-26th-onam-pos.onrender.com/api
```

---

# 5. MongoDB Atlas

MongoDB is the permanent data store.

The application uses the database name configured by:

```text
MONGO_DB
```

The default is:

```text
event_pos
```

Collections are created automatically:

```text
products
orders
inventory_movements
counters
settings
```

The Python backend connects using:

```text
MONGO_URI
```

and checks the connection with MongoDB `ping`.

---

# 6. Render environment variables

Create a Render Web Service and add:

```text
MONGO_URI = your complete MongoDB connection URI
MONGO_DB = event_pos
EVENT_NAME = Society One-Day Event
EVENT_DATE = 2026-09-26
CORS_ORIGIN = https://suyashshukla911-ops.github.io
```

`HOST` and `PORT` do not need to be entered manually for the normal setup:

- `HOST` defaults to `0.0.0.0`
- `PORT` is read from Render's `PORT` environment variable

Render automatically provides environment variables at runtime.

### CORS_ORIGIN

For the current GitHub Pages site:

```text
https://suyashshukla911-ops.github.io
```

Do not add:

```text
/PT_26th_Onam
```

CORS uses the origin, not the repository path.

For example:

```text
https://suyashshukla911-ops.github.io
```

---

# 7. Render service setup

## Step 1 — Push the final code to GitHub

Your repository should contain:

```text
app.py
index.html
README.md
```

Do not push:

```text
.env
event_pos.db
MongoDB credentials
```

---

## Step 2 — Create the Render backend

In Render:

```text
New
→ Web Service
```

Connect the GitHub repository that contains `app.py`.

Render's current Web Service setup uses a Build Command and Start Command for application services. citeturn861808search0turn861808search5

### Runtime

Choose:

```text
Python 3
```

### Build Command

Use:

```bash
pip install pymongo
```

### Start Command

Use:

```bash
python app.py
```

The final `app.py` uses the Render `PORT` value and binds to `0.0.0.0`, which is required for a public Render Web Service.

---

# 8. Render environment variables

In:

```text
Render
→ Your Service
→ Environment
```

add:

```text
MONGO_URI
MONGO_DB
EVENT_NAME
EVENT_DATE
CORS_ORIGIN
```

### Example

```text
MONGO_DB=event_pos
EVENT_NAME=Society One-Day Event
EVENT_DATE=2026-09-26
CORS_ORIGIN=https://suyashshukla911-ops.github.io
```

For `MONGO_URI`, paste the real MongoDB connection URI as the secret value.

Do not put it in the GitHub repository.

---

# 9. Deploy the backend

Click:

```text
Deploy Web Service
```

Wait for the build and deployment to finish.

Render provides a public URL similar to:

```text
https://your-service-name.onrender.com
```

Every Render Web Service gets an `onrender.com` URL. citeturn861808search0

---

# 10. Test the backend BEFORE changing GitHub Pages

Open:

```text
https://YOUR-RENDER-SERVICE.onrender.com/api/health
```

The response should look like:

```json
{
  "ok": true,
  "database": "event_pos",
  "message": "MongoDB connected"
}
```

This is the first test.

If `/api/health` does not report a successful MongoDB connection, do not move to the frontend step yet.

---

# 11. Test the backend metadata

Open:

```text
https://YOUR-RENDER-SERVICE.onrender.com/api/meta
```

It should return event metadata such as:

```json
{
  "ok": true,
  "event_name": "Society One-Day Event",
  "event_date": "2026-09-26",
  "currency": "₹",
  "categories": [...]
}
```

---

# 12. Test products

Open:

```text
https://YOUR-RENDER-SERVICE.onrender.com/api/products
```

You should see the product catalog and `current_stock`.

This confirms the application can read MongoDB inventory.

---

# 13. Test the GitHub Pages frontend

After Render is working:

1. Open `index.html`.
2. Find:

```javascript
const RENDER_API_URL = "https://YOUR-RENDER-SERVICE.onrender.com";
```

3. Replace it with your actual Render URL.

Example:

```javascript
const RENDER_API_URL = "https://pt-26th-onam-pos.onrender.com";
```

4. Save the file.
5. Commit and push `index.html` to GitHub.

Then wait for GitHub Pages to publish.

GitHub Pages can publish static files, but it cannot execute the Python backend itself.

---

# 14. Final live flow

After deployment:

```text
User opens:
https://suyashshukla911-ops.github.io/PT_26th_Onam/

              ↓

index.html

              ↓

https://YOUR-RENDER-SERVICE.onrender.com/api/...

              ↓

app.py

              ↓

MongoDB Atlas

              ↓

JSON response

              ↓

Dashboard updates
```

---

# 15. Why this solves the "MongoDB connected locally but not online" problem

When you ran the Python application locally, the browser and Python server were on the same machine.

The browser could call:

```text
/api/dashboard
/api/orders
/api/products
/api/health
```

because `app.py` was serving those routes.

GitHub Pages cannot execute `app.py`. It only serves the HTML/static content.

The final frontend therefore sends API calls to Render.

---

# 16. CORS support

Because GitHub Pages and Render have different origins, the backend needs to allow the frontend origin.

The final Python backend supports:

```text
OPTIONS
GET
POST
PUT
DELETE
```

and sends the appropriate CORS headers.

This is necessary for browser API calls from the GitHub Pages domain to the Render backend.

Recommended production value:

```text
CORS_ORIGIN=https://suyashshukla911-ops.github.io
```

---

# 17. Local testing remains supported

The final code still supports local testing.

Set:

```powershell
$env:MONGO_URI="YOUR_MONGODB_URI"
$env:MONGO_DB="event_pos"
```

Then:

```powershell
py -m pip install pymongo
py app.py
```

Open:

```text
http://127.0.0.1:5000
```

Because the hostname is localhost/127.0.0.1, the frontend automatically uses its local server as the API.

You do not need to change `RENDER_API_URL` for local testing.

---

# 18. Local versus live

## Local

```text
Browser
  ↓
127.0.0.1:5000
  ↓
app.py
  ↓
MongoDB Atlas
```

## Live

```text
GitHub Pages
  ↓
Render
  ↓
app.py
  ↓
MongoDB Atlas
```

Both use the same MongoDB database unless you deliberately configure a different one.

---

# 19. Billing synchronization

A completed bill is processed by the Python backend.

The flow is:

```text
Select products
      ↓
Browser creates cart
      ↓
POST /api/orders
      ↓
Server validates items
      ↓
Server calculates official prices
      ↓
Server validates discount
      ↓
Server validates payment
      ↓
Server creates bill number
      ↓
MongoDB transaction
      ├── save order
      └── deduct inventory
      ↓
JSON response
      ↓
Frontend refreshes all sections
```

The frontend should not be treated as the source of truth.

---

# 20. MongoDB transaction behavior

New bill creation, bill edits, bill voiding, manual inventory updates, and reset are implemented with MongoDB transactions.

For example, creating a sale performs:

```text
orders insert
+
inventory deduction
```

inside the same transaction.

If the transaction fails, MongoDB can roll the operation back rather than leaving a partially completed sale.

---

# 21. Current API endpoints

## Health

```text
GET /api/health
```

## Metadata

```text
GET /api/meta
```

## Products

```text
GET /api/products
```

## Dashboard

```text
GET /api/dashboard?date=YYYY-MM-DD
```

## Orders for a date

```text
GET /api/orders?date=YYYY-MM-DD
```

## One order

```text
GET /api/orders/<ORDER_NO>
```

## Create bill

```text
POST /api/orders
```

## Edit bill

```text
PUT /api/orders/<ORDER_NO>
```

## Void bill

```text
DELETE /api/orders/<ORDER_NO>
```

## Update inventory

```text
POST /api/inventory/<SKU>/stock
```

## Reset event

```text
POST /api/reset
```

## Sales export

```text
GET /api/export/sales.csv
```

## Inventory export

```text
GET /api/export/inventory.csv
```

## Inventory movement export

```text
GET /api/export/inventory-movements.csv
```

## Summary export

```text
GET /api/export/summary.csv
```

## Excel export

```text
GET /api/export/excel.xlsx
```

---

# 22. Inventory behavior

The database stores:

```text
opening_stock
current_stock
low_stock_threshold
```

A sale decreases:

```text
current_stock
```

Editing a bill:

```text
restore old inventory
↓
apply new inventory
```

Voiding a bill:

```text
restore inventory
↓
mark bill voided
```

Manual inventory changes are also recorded in `inventory_movements`.

---

# 23. Reset Event behavior

The red:

```text
RESET EVENT
```

button is for testing.

It performs:

```text
1. Create complete backup
2. Download backup ZIP
3. Delete orders
4. Delete inventory movements
5. Restore opening stock
6. Restore low-stock thresholds
7. Reset bill counter
8. Refresh the complete UI
```

The reset endpoint is:

```text
POST /api/reset
```

Do not expose this endpoint publicly without authentication.

---

# 24. Reset backup contents

Before reset, the browser receives a ZIP containing:

```text
sales.csv
inventory.csv
inventory_movements.csv
summary.csv
event_report.xlsx
backup.json
```

This is the complete pre-reset event snapshot generated by the backend.

---

# 25. Excel export

The Excel report is generated server-side.

The export starts from the supplied workbook template embedded in `app.py`.

The generated workbook contains live POS data in dedicated POS sheets for:

```text
POS Sales Log
POS Inventory
POS Inventory Movements
```

The original workbook structure is retained as the base template.

---

# 26. Current product/pricing rules

The server-side catalog currently includes:

| Product | Price |
|---|---:|
| Mewa Laddus | ₹40 |
| Filter Coffee | ₹60 |
| Instant Coffee | ₹50 |
| Makhana Bhel | ₹99 |
| Bhel Puri | ₹70 |
| Badame Khaas | ₹80 |
| Cashew Delight | ₹80 |
| Choco Pops single | ₹40 |
| Choco Pops 2-for-₹70 combo | ₹70 |
| Water | ₹6 |

Choco Pops combo rules are:

```text
Same:
2 Chana = ₹70
2 Peanuts = ₹70

Different:
1 Chana + 1 Peanuts = ₹70
```

Physical Choco Pops inventory is one shared stock pool.

---

# 27. How to test the complete system

Perform these tests in this order.

## Test A — backend connectivity

Open:

```text
https://YOUR-RENDER-SERVICE.onrender.com/api/health
```

Expected:

```text
MongoDB connected
```

---

## Test B — frontend connectivity

Open the GitHub Pages website.

Expected header:

```text
MongoDB connected
```

---

## Test C — inventory read

Confirm product cards display stock.

---

## Test D — sale

Create:

```text
2 × Mewa Laddus
1 × Filter Coffee
1 × Water
```

Expected subtotal:

```text
₹146
```

Expected:

```text
Bills = 1
Revenue = ₹146
Avg. Bill = ₹146
```

Inventory:

```text
Mewa Laddus -2
Filter Coffee -1
Water -1
```

---

## Test E — recent bill

The newly created bill should immediately appear under:

```text
Recent bills
```

---

## Test F — edit

Edit the same bill.

Confirm:

```text
old stock restored
new stock applied
new total displayed
dashboard refreshed
```

---

## Test G — void

Void the bill.

Confirm:

```text
status = voided
inventory restored
completed revenue excludes bill
```

---

## Test H — inventory edit

Manually change an item's stock.

Confirm:

```text
inventory table changes
low-stock status updates
movement is recorded
```

---

## Test I — export

Download:

```text
Sales CSV
Inventory CSV
Movements CSV
Excel
```

---

## Test J — reset

Create test bills first.

Then click:

```text
RESET EVENT
```

Confirm:

```text
backup ZIP downloads
bills disappear from completed totals
inventory returns to opening quantities
bill numbering resets
```

---

# 28. Troubleshooting

## Frontend says "MongoDB error"

First open:

```text
https://YOUR-RENDER-SERVICE.onrender.com/api/health
```

If it fails:

- check Render service logs
- check `MONGO_URI`
- check MongoDB Atlas network access
- check database-user credentials

If `/api/health` works but GitHub Pages does not:

- verify `RENDER_API_URL` inside `index.html`
- verify `CORS_ORIGIN` on Render
- hard-refresh the browser

---

## Render deployment fails

Check:

```text
Build Command:
pip install pymongo
```

```text
Start Command:
python app.py
```

The backend must bind to:

```text
0.0.0.0
```

and use:

```text
PORT
```

Render requires this for public Web Services.

---

## Local works, Render does not

This normally means one of:

```text
HOST binding
PORT
MONGO_URI
MongoDB network access
Render environment variables
```

The final backend already handles the correct Render host/port pattern.

---

## GitHub Pages shows the website but buttons do nothing

This means the static frontend is loading but its API target is wrong.

Check:

```javascript
const RENDER_API_URL = "https://YOUR-RENDER-SERVICE.onrender.com";
```

and replace it with the actual Render service URL.

---

# 29. Browser cache

After updating `index.html` on GitHub Pages, do a hard refresh:

```text
Ctrl + Shift + R
```

or:

```text
Ctrl + F5
```

---

# 30. Git commands for updating GitHub Pages

From the project folder:

```powershell
git add index.html app.py README.md
git commit -m "Connect POS frontend to Render MongoDB backend"
git push
```

Do not run:

```text
git add .
```

until you are certain the `.env` file is not present.

---

# 31. Final recommended repository contents

```text
PT_26th_Onam/
├── app.py
├── index.html
└── README.md
```

No credentials.

No local database.

No `.env`.

---

# 32. Final deployment order

Follow this exact sequence:

```text
STEP 1
MongoDB Atlas ready
        ↓
STEP 2
Create Render Web Service
        ↓
STEP 3
Deploy app.py
        ↓
STEP 4
Add MongoDB environment variables
        ↓
STEP 5
Open /api/health
        ↓
STEP 6
Confirm "MongoDB connected"
        ↓
STEP 7
Copy Render URL
        ↓
STEP 8
Put Render URL into index.html
        ↓
STEP 9
Push index.html to GitHub
        ↓
STEP 10
Open GitHub Pages
        ↓
STEP 11
Create test bill
        ↓
STEP 12
Verify dashboard + inventory + recent bill
        ↓
STEP 13
Test edit
        ↓
STEP 14
Test void
        ↓
STEP 15
Test exports
        ↓
STEP 16
Test RESET EVENT
```

Do not test the live event before the `/api/health` and bill/inventory synchronization tests pass.

---

# 33. Final source-of-truth rule

MongoDB is the source of truth.

```text
MongoDB
    ↓
app.py
    ↓
API response
    ↓
index.html display
```

Not:

```text
Browser display
    ↓
assume data is saved
```

The browser display is only the user interface.

---

# 34. Deployment checklist

### MongoDB

```text
[ ] Cluster running
[ ] Database user exists
[ ] Network access configured
[ ] URI verified
[ ] Password rotated if old credential was exposed
```

### Render

```text
[ ] Web Service created
[ ] Python selected
[ ] Build = pip install pymongo
[ ] Start = python app.py
[ ] MONGO_URI added
[ ] MONGO_DB added
[ ] EVENT_NAME added
[ ] EVENT_DATE added
[ ] CORS_ORIGIN added
[ ] Deployment successful
```

### Backend

```text
[ ] /api/health works
[ ] /api/meta works
[ ] /api/products works
```

### GitHub Pages

```text
[ ] RENDER_API_URL updated
[ ] index.html committed
[ ] index.html pushed
[ ] GitHub Pages updated
[ ] MongoDB connected shown
```

### POS

```text
[ ] Test sale
[ ] Dashboard update
[ ] Inventory deduction
[ ] Recent bill
[ ] Bill edit
[ ] Bill void
[ ] Inventory edit
[ ] CSV exports
[ ] Excel export
[ ] Reset backup
[ ] Reset inventory
```

---

# 35. Official deployment references

GitHub Pages:

https://docs.github.com/en/pages

GitHub's documentation states that Pages hosts static files and does not support server-side Python.

Render Web Services:

https://render.com/docs/web-services

Render documents the requirement to bind public web services to `0.0.0.0` and use the `PORT` environment variable. citeturn861808search0

Render environment variables:

https://render.com/docs/configure-environment-variables

Render recommends environment variables/secrets for credentials and database connection strings and advises not committing `.env` files.

---

# 36. Final status

The intended production topology is:

```text
GitHub Pages
    ↓
index.html
    ↓
Render Web Service
    ↓
app.py
    ↓
MongoDB Atlas
```

This is the correct model for keeping the current frontend on GitHub Pages while running the Python + MongoDB backend separately.
