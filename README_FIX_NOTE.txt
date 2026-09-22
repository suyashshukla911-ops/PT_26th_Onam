FINAL FIX NOTE

The previous frontend had a JavaScript regex typo in index.html:
replace(/\\/+$/, "")

It has been corrected to:
replace(/\/+$/, "")

That syntax error prevented all frontend JavaScript from executing, which is why the page stayed on "Loading event..." and "Checking MongoDB..." even while app.py was running and MongoDB was connected.

app.py also no longer requires python-dotenv. It uses MONGO_URI, MONGO_DB, EVENT_NAME, EVENT_DATE, HOST and PORT from the process environment.

Local test:
1. Set MONGO_URI in PowerShell.
2. Run: py app.py
3. Open: http://127.0.0.1:5000
4. The site should load products and show MongoDB connected.
