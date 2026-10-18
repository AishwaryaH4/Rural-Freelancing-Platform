# Empowering Rural Areas Through Digital Freelancing and Skill-Based Opportunities

A web platform that connects rural clients with nearby skilled workers (tractor operators, electricians,
plumbers, carpenters, harvesting teams and more). Final-year project, Dept. of Information Science & Engineering,
Sapthagiri College of Engineering (VTU), 2025-26.

**Team:** Aishwarya Manjunath Hegde, Bhumika, Chandana H Gowda, Kritika Manani · **Guide:** Prof. Gayathri R

**Stack:** HTML5, CSS3, JavaScript, Bootstrap 5 · Python (Flask) · MySQL · Progressive Web App · pytest

## Features
| Module | What it does |
|---|---|
| Authentication | Register / login, passwords hashed with Werkzeug, session login, deactivated accounts are blocked |
| Roles | Client, Worker and **Admin** roles; role-based access (403 for the wrong role) |
| Worker profile | Skills, bio, experience, availability toggle, photo upload (PNG/JPG, 2 MB max) |
| Services | Workers add/remove services with price per day/hour/acre/job and an optional **service photo** |
| Location matching | Browser Geolocation + **Haversine formula**; filter 1 / 5 / 10 / 25 km |
| Rule-based filtering | Category, keyword, max price, availability, **free on a chosen date**, distance, then sort by distance & rating |
| Scheduling | Workers block days off; no double-booking (a day is reserved when a booking is accepted/confirmed, checked under a row lock) |
| Voice | Voice search (Web Speech API: English/Hindi/Kannada/Tamil/Telugu) + "read aloud" on results |
| Regional languages | Full interface in **English, Hindi and Kannada** (language switcher in the navbar, saved on the profile) |
| Image-based selection | Category tiles use a picture if one is in `static/img/categories/`, else an emoji; services can carry real photos |
| Booking | PENDING → ACCEPTED/REJECTED → CONFIRMED → IN_PROGRESS → COMPLETED, or CANCELLED |
| Negotiation | Either side offers a price, the other accepts; agreed price is saved |
| Chat | In-app chat per booking (auto-refresh every 3 seconds) |
| Masked calling | Real bridged calls through **Twilio** when configured (both sides see only the Twilio number); otherwise a clearly-labelled simulation. Only after the worker accepts; rate-limited |
| Ratings & reviews | Client rates a COMPLETED booking once; average shown on search and profile |
| Complaints | Client or worker can report a problem on a booking |
| Admin module | Dashboard of users, services, bookings and complaints; deactivate users, remove services, resolve complaints |
| Mobile | Installable **PWA** (Add to Home screen on Android/Chrome), offline page, responsive layout |
| Security | Parameterized SQL, CSRF token on every POST (constant-time compare), XSS-safe templates, upload validation, secrets from environment variables |
| Tests | 47 automated pytest tests (security, roles, state machine, scheduling, calling, admin, translations, PWA) |

## Folder structure
```
rural_freelancing/
├── app.py               # Flask routes + logic
├── config.py            # settings read from environment variables
├── translations.py      # English / Hindi / Kannada UI text
├── database.sql         # creates DB, tables, categories (fresh install)
├── migration_v2.sql     # upgrades an existing v1 database in place
├── seed.py              # demo users and services
├── create_admin.py      # creates the administrator account
├── tests/               # pytest suite (uses a separate test database)
├── static/ (css, js, icons, img/categories, uploads, manifest, sw.js)
└── templates/           # HTML pages (Jinja2)
```

## Setup (Windows, step by step)
1. Install **Python 3.10+** (tick "Add to PATH"), **MySQL Server** and **VS Code**.
2. In the project folder:
   ```
   python -m venv venv
   venv\Scripts\activate
   pip install -r requirements.txt
   ```
3. Create the database. PowerShell does not support `<`, so either use MySQL Workbench (File → Open SQL Script →
   run), or:  `cmd /c "mysql -u root -p < database.sql"`.
   *Already have the v1 database?* Run `migration_v2.sql` instead (once), it keeps your data.
4. **Set secrets as environment variables** (nothing secret lives in the code; the app refuses to start without them):
   ```
   python -c "import secrets; print(secrets.token_hex(32))"
   $env:SECRET_KEY="paste-the-long-string"
   $env:DB_PASSWORD="your-mysql-password"
   ```
   These last only for that terminal window. Set them again in every new terminal.
5. Demo data: `python seed.py`  ·  Admin account: `python create_admin.py` (asks for email + password)
6. Start: `python app.py` → http://127.0.0.1:5000  (debug mode is off; for development set `$env:FLASK_DEBUG="1"`)

## Demo credentials (local testing only)
| Role | Email | Password |
|---|---|---|
| Client | client@example.com | Client@123 |
| Worker | worker@example.com | Worker@123 |

Other demo workers: suresh@, manju@, lakshmi@, basu@ `example.com` (all `Worker@123`). The admin account has no
default password: you create it with `create_admin.py`. Demo locations are near Mandya, Karnataka; use **Near me**
or edit the coordinates in `seed.py`. **Delete demo accounts before any real use.**

## Running the tests
```
pip install -r requirements-dev.txt
$env:DB_PASSWORD="your-mysql-password"
python -m pytest -q
```
Tests create and wipe a separate database called `rural_freelancing_test`; your real data is never touched.

## Optional: real masked calling (Twilio)
Set these before `python app.py`; without them the app shows a simulated call instead.
```
$env:TWILIO_ACCOUNT_SID="AC..."; $env:TWILIO_AUTH_TOKEN="..."; $env:TWILIO_NUMBER="+1..."
```
Twilio rings the caller, then bridges to the other person; both only see the Twilio number. A trial account can call
only verified numbers. Calls cost money, so each user is limited to 3 per hour.

## Install on an Android phone (PWA)
Chrome only allows installing over **HTTPS** (or localhost). Deploy it, or tunnel it (for example Cloudflare Tunnel or
ngrok), open the HTTPS address in Chrome on the phone, then menu → **Install app / Add to Home screen**. Voice input
and location also need HTTPS on phones. This is an installable web app, not a native Play-Store APK.

## Category pictures (optional)
Put photos named after the category id in `static/img/categories/` (`1.jpg` = Tractor Operation ... `9.jpg`).
See the README.txt in that folder. Missing files fall back to the emoji.

## Testing checklist (manual, matches the Testing slide)
1. Register a client and a worker → login works, wrong password is rejected.
2. Worker: add a service (with a photo) and block a date. Client: search by tap, typing, voice, and by date.
3. **Near me**, "Within 5 km" → far workers disappear.
4. Client requests with their own price → worker counter-offers → client accepts.
5. Worker accepts → client confirms → worker starts → worker completes. Try booking the same worker on a day that is taken.
6. Chat from both accounts (two browsers). Try **Masked call**.
7. Client reviews the completed booking → rating shows on the worker profile. File a complaint → resolve it as admin.
8. Switch the language to Hindi and Kannada from the navbar.

## How the core algorithm works
`Client request → location & service processing → worker matching → booking → communication → completion → rating`
1. Get the client's GPS location and chosen service. 2. Select workers who offer it, are available and free on the date.
3. Filter by price and keyword. 4. Compute distance with Haversine:
`a = sin²(Δφ/2) + cos φ1 · cos φ2 · sin²(Δλ/2)`, `d = 2R · asin(√a)` with R = 6371 km.
5. Apply the km filter, sort by nearest then best rating, and show the list.

## Known limitations
- Server messages (flash alerts, validation errors) are English only; page text is translated.
- Tamil and Telugu are available for voice input only, not for the interface.
- No payment integration; no email/SMS notifications; chat uses polling, not WebSockets.
- Translations should be reviewed by native speakers before a real launch.

## Troubleshooting
- `RuntimeError: Set the SECRET_KEY and DB_PASSWORD...` → set both environment variables in this terminal.
- `Access denied for user 'root'` → wrong `DB_PASSWORD`.  `Unknown database` → run step 3 again.
- Garbled emoji after loading the SQL → use `mysql --default-character-set=utf8mb4`.
- Voice or location button does nothing → use Chrome/Edge and allow the permission.

## Future enhancements
Secure payments (Razorpay/UPI) · more interface languages (Tamil, Telugu) · native Android app · maps (Leaflet / Google Maps)
· real image recognition for service selection · email/SMS notifications · WebSocket chat.
#   R u r a l - F r e e l a n c i n g - P l a t f o r m  
 