# Pilot Jobs – setup guide

A browser-based job tracker and booking checklist for a freelance pilot. It works on a phone and a laptop, puts jobs on a Google Calendar, and keeps its records in a Google Sheet. There are no servers to run and nothing to pay for.

This guide does two jobs: it tells you **what to do**, and for each step it explains **what is happening and why**, so you can understand the setup rather than just follow it.

**Files in this package**

| File | Where it goes |
|---|---|
| `Code.gs` | Google Apps Script (the "backend": talks to Calendar, Sheets and Gmail) |
| `appsscript.json` | Google Apps Script (its settings file, called the "manifest") |
| `index.html` | A web host such as GitHub Pages (the "page" people look at) |
| `manifest.webmanifest`, `icon-192.png`, `icon-512.png`, `icon-maskable-512.png`, `apple-touch-icon.png`, `favicon.svg` | The same web host, in the same folder as `index.html` (the app's icon and name; see "Customising the icon") |
| `logo.svg` | Nowhere: the editable master of the logo |
| `preview.html` | Nowhere – open it to look at the layout with sample data, no setup needed |

---

## 1. How it fits together

```
  His phone / laptop                     Apps Script "backend"                   Google
 ┌─────────────────────┐   HTTPS POST   ┌────────────────────────┐            ┌─────────────┐
 │  index.html         │ ─────────────▶ │  Code.gs               │ ─────────▶ │ Calendar    │
 │  (the page, hosted  │                │  1. checks the passcode│            │ Sheet (data)│
 │   on GitHub Pages)  │ ◀───────────── │  2. runs the request   │ ◀───────── │ Gmail       │
 └─────────────────────┘   JSON reply   │  runs AS THE PILOT     │            └─────────────┘
                                        └────────────────────────┘
```

**Two halves, deliberately separate**

- The **page** (`index.html`) is just a file on ordinary web hosting. It draws the screens and remembers nothing about jobs itself.
- The **backend** (`Code.gs`) is a Google Apps Script "web app". It does everything that needs Google: creating calendar events, saving rows in the Sheet, sending emails. It runs *as the pilot*, using his permissions.
- They talk by sending small messages. The page sends a request like `{"fn": "getAll", "args": [], "key": "<passcode>"}` and the backend answers with data in the same JSON text format.

**Why not let Google serve the page too?** That was the first design, and it breaks for anyone signed in to several Google accounts: Google opens the app as whichever account is first in the browser, and shows "Sorry, unable to open the file at present". When the page lives elsewhere and sends its requests *without* Google cookies, Google never sees which accounts are signed in, so the problem can't occur.

**Why a passcode?** To work without a Google sign-in, the backend has to accept requests from anyone who has its address ("Who has access: Anyone"). Since it runs as the pilot, that would let anyone with the address read and change his calendar. The passcode is the lock: every request must include it, and the backend refuses anything without it. Make it long (see step 5).

**Words you'll meet**

| Term | Meaning |
|---|---|
| **Web app / deployment** | A published copy of an Apps Script project that has a web address and answers requests. |
| **Script property** | A saved setting for a script that isn't written in the code. We keep the passcode there so it never appears in the code you paste around. |
| **Static hosting** | A service that just serves files unchanged (here, `index.html`). No code runs on it. |
| **localStorage** | A small store inside the browser. Each device keeps the server address and passcode there after you connect. |
| **fetch** | The browser's built-in way for a page to send a request and read the reply. |
| **CORS / preflight** | The browser's safety rules for one website calling another. The page sends its requests as plain text so no "permission check" is needed first, which Apps Script can't answer. |

---

## 2. Setup

You'll do three things: **A.** publish the backend, **B.** host the page, **C.** connect each device. Allow about 20 minutes.

> **Which Google account?** The one that will own the calendar and data: the pilot's own account for real use, or your own account while testing. The backend runs as that account.

### A. Publish the backend (Apps Script)

**1. Create the project.** Go to <https://script.google.com> → **New project**, and name it "Pilot Jobs".
*What this does:* a script project is a container for code that runs on Google's servers, tied to your account.

**2. Paste the code.** Replace everything in `Code.gs` with the `Code.gs` from this package. If an earlier version of this project had a file called `Index` (an HTML file), delete it – it isn't used any more.
*What this does:* this is the backend described above. It also no longer serves any page, which matters for security: a page served from here would run as the pilot for anyone who opened it, bypassing the passcode.

**3. Paste the settings file.** Project Settings (gear icon) → tick **Show "appsscript.json" manifest file in editor** → go back to the editor, open `appsscript.json` and replace its contents with the one from this package.
*What this does:* the manifest holds the project's settings. Two lines matter: `"executeAs": "USER_DEPLOYING"` (run as the pilot) and `"access": "ANYONE_ANONYMOUS"` (reachable without a Google sign-in; the passcode protects it). The time zone is set to Europe/London; change it if needed.

**4. Review the settings (optional).** At the top of `Code.gs` is a `CONFIG` block: calendar name, email on/off, reminder times and so on. For now only `APP_URL` matters, and you'll fill that in after step 9.

**5. Set the passcode.** Project Settings → **Script properties** → **Add script property**. Property: `API_PASSCODE`. Value: a long passphrase – at least 16 characters, for example four random words with spaces or hyphens. Save.
*What this does:* the backend compares every request's passcode with this value. Anyone who can guess it gets full access, so don't reuse a password from elsewhere. After 10 wrong guesses in a row, *every* request (even the right passcode) is refused for 5 minutes, so guessing is hopeless against a long passphrase. (The trade-off: someone who knew the address could keep him locked out by spamming wrong guesses. That annoys but doesn't expose anything.)

**6. Run setup once.** In the editor choose the `setup` function in the toolbar and click **Run**. Google asks for permissions (Calendar, Sheets, Gmail). On the "Google hasn't verified this app" screen choose **Advanced → Go to Pilot Jobs (unsafe)** – it's your own script. When it finishes, open **Execution log** and check it says the passcode is set.
*What this does:* it asks you to approve what the script may do, then creates a calendar called "Pilot Jobs" and a spreadsheet called "Pilot Jobs – data" in the account. The warning appears only because you wrote the script yourself and haven't submitted it to Google for review.

**7. Deploy it.** **Deploy → New deployment** → click the gear next to "Select type" → **Web app**. Set *Execute as:* **Me**, *Who has access:* **Anyone**. Click **Deploy**, then **copy the Web app URL** (it ends in `/exec`).
*What this does:* it publishes a copy of the code at a public address. "Execute as: Me" means the code runs with your permissions. "Anyone" means no Google sign-in is needed to call it.
Some Google Workspace (work/school) accounts don't allow "Anyone". If the option is missing, use a personal Gmail account.

**8. Quick check (optional).** Open the URL in a **private window**. You should see a short message beginning `{"ok":true,"app":"Pilot Jobs API"…`. It shows no data. (Use a private window for this check only, because a normal window with several accounts signed in can trigger the "unable to open the file" error on this simple check. The page itself doesn't have that problem.)

### B. Host the page

The page is a single file, so any static host works. GitHub Pages is free:

**9. Create a repository.** On github.com: **New repository**. Name it, for example, `pilot-jobs`. Choose **Public**.
*Why public?* Free GitHub Pages needs it. That's fine here, because `index.html` contains no secrets: no address, no passcode, no data. Each person types those in on their own device.

**10. Upload the page.** **Add file → Upload files** → drag in `index.html` **and the icon files and `manifest.webmanifest`** (you can select them all at once) → **Commit changes**. They must sit in the same place as `index.html`.

**11. Switch on Pages.** **Settings → Pages**. Under *Build and deployment*, Source: **Deploy from a branch**; Branch: **main**, folder **/(root)**; **Save**. After a minute or so the page shows its address, like `https://yourname.github.io/pilot-jobs/`. That is the app's address.
*What this does:* GitHub now serves your `index.html` over HTTPS at that address. HTTPS matters because it encrypts the passcode in transit.

*Alternatives:* Netlify (drag-and-drop a folder containing `index.html`) and Cloudflare Pages work the same way, and both can host from a private repository.

**12. (Optional) Make "Open checklist" links work.** In `Code.gs`, set `APP_URL: 'https://yourname.github.io/pilot-jobs/'`, then redeploy (see "Updating", below). Calendar events and emails will then link straight to the job.

### C. Connect each device

**13.** Open the page address on the phone or laptop. The first time, it shows a **Connect** screen: paste the **Server address** (the `/exec` URL from step 7) and the **Passcode**, then tap **Connect**.
*What this does:* the page saves both in that browser's localStorage and uses them for every request. Anyone who can use that browser can use the app, so don't connect a shared computer. **Disconnect this device** (at the bottom of the job list) forgets them.
The page refuses any address that isn't a `script.google.com … /exec` address, so a mistyped or malicious address can't receive the passcode.

**14.** Bookmark it. On the Pixel in Chrome: ⋮ menu → **Add to Home screen** (or **Install app**, if Chrome offers it) for an app icon that opens full-screen. Repeat step 13 on each device. If you added the shortcut before the icon files were uploaded, remove it and add it again, because phones remember the old icon.

### Updating later

- **Changed `Code.gs`:** paste it in, then **Deploy → Manage deployments → ✏️ → Version: New version → Deploy**. The address stays the same. Without a new version, the live app keeps running the old code – the most common reason a change "doesn't work".
- **Changed `index.html`:** on GitHub open `index.html` → upload the new file over it (**Add file → Upload files**, same name). Wait a minute, then refresh the page (on a phone, close and reopen it or clear the site's data if it still looks old).
- **Changing the passcode:** edit the script property. Each device gets the "passcode wasn't accepted" screen next time and asks again.

**Customising the icon.** The icon is a runway seen on final approach, with an orange sun, on sky blue and green. `logo.svg` is the master: open it in a text editor or a vector tool such as Inkscape or Figma to change the colours (the two sky-blue stops in `<linearGradient>`, `#FF8A1F` for the sun, `#3FA371` for the grass, `#2D3340` for the tarmac) or the shape. Then re-export the PNGs at 512×512, 192×192 and 180×180 (keep the artwork inside the central 80% so Android's circle crop doesn't clip it), keep the same file names, and upload them over the old ones. To change the name shown under the icon, edit `name` and `short_name` in `manifest.webmanifest` (keep `short_name` under about 12 characters). The same two colours appear as `theme_color` and `background_color` there and in `index.html`'s `theme-color` tag.

---

## 3. Troubleshooting

| What you see | Likely cause and fix |
|---|---|
| "The server's reply wasn't what I expected…" | The address is wrong, not ending in `/exec`, or the deployment's access isn't **Anyone**, or the deployment is older than your changes. Check **Manage deployments**. |
| "Couldn't reach the server" | No internet, or the address has a typo. |
| "That passcode wasn't accepted" | Typo, stray space, or the property isn't named exactly `API_PASSCODE`. |
| "Too many wrong attempts" | Wait five minutes. |
| "The server has no passcode yet" | The `API_PASSCODE` script property is missing (step 5). |
| Connect screen says "doesn't look like a Pilot Jobs server address" | Paste the web app URL from step 7 (`https://script.google.com/macros/s/…/exec`), not the editor's address and not a `/dev` address. |
| Changes to `Code.gs` have no effect | You need a **New version** in Manage deployments. |
| Page looks out of date after an update | Refresh; GitHub Pages can take a minute or two. |
| No calendar reminders on the phone | The "Pilot Jobs" calendar must be ticked in the Google Calendar app and its notifications enabled. |
| "Authorization is required" in Apps Script | Run `setup` once (step 6) and approve the permissions. |

---

## 4. What the app does

| Step | What happens |
|---|---|
| **Quick add** | Tap **+ New job**. By default it's the quick tentative form: type whatever is known (notes, route, dates, times) and save; it goes on the calendar in orange. Tick **Confirmed flight** at the top to switch to the full confirmed form (title, route, dates and times, notes, plus outbound/return flights, accommodation, legs with destination and filing rules, and after-job tasks); it's saved straight to green. Nothing is saved until you tap Save. |
| **Calendar** | If there is a start date, an **orange** event ("TENTATIVE: …") is created on a calendar called *Pilot Jobs*, and a summary email is sent to him. Jobs with no date yet are saved and join the calendar as soon as a date is added. |
| **Confirm** | The event turns **green** (and loses the TENTATIVE prefix) and the checklist appears with a **Next up** prompt. |
| **Checklist** | 1. Flights you operate – per leg: GAR, flight plan, handling/fuel/permits, weather & NOTAM briefing · 2. Outbound / return travel – booking and check-in for each commercial flight · 3. Accommodation (as many stays/locations/nights as needed) · 4. Post-flight tasks: invoice, expenses, plus your own tasks. "Next up" and the To do tab's *By task* view follow this order. |
| **To do tab** | Every outstanding checklist item across confirmed jobs, sortable **by date** or **by task** (e.g. all hotel bookings together). Items can be ticked straight from the list. |
| **GAR deadlines** | Each leg has **From** and **To** ICAO airport fields, and the destination country is worked out from the **To** code (**EG…** = UK, **LF…** = France, any other code = anywhere else). You can still override the country by hand; it stays until you change the destination airport. With no airport entered it defaults to **UK**. Then the app applies the rules: **UK** arrival – GAR no earlier than 48 h before flight time; **France, Mon–Fri** – within 24 h of flight time (not earlier); **France, Sat/Sun** – at least 48 h before; **anywhere else** – no rule. The weekday is that of the departure in Paris time. |
| **Flight plan deadlines** | Each leg has a departure time (UTC) and a filing rule: *earliest you can file* and/or *latest you must file*, in hours before departure. "Only within 48 h of flight" = earliest 48; "at least 48 h ahead" = latest 48. |
| **Deadline alerts** | The app shows each GAR / flight plan status live (too early / window open / submit now / overdue) and a red banner on the Active and To do tabs when a deadline is within 24 h or has passed. The calendar gets a red **DEADLINE** event with popups 24 h, 6 h, 1 h and 15 min before (plus one email at 24 h) and, if there's an earliest limit, a yellow "window opens" marker. Ticking the box removes them. |
| **Commercial flights** | Enter a departure time on a flight and it gets its own **blue** calendar event with reminders **24 h, 4 h and 2 h before** (popups, plus emails at 24 h and 2 h). Untick "Commercial flight" for trains/driving. |
| **Import from a spreadsheet** | On the Active tab, tap **Import jobs from a spreadsheet**. Download `pilot-jobs-import-template.xlsx`, fill in one row per flight leg (job title, status, date, departure time in UTC, From and To ICAO codes), then choose the saved .xlsx or .csv. You get a preview (problem rows are listed and skipped, jobs that already exist are unticked) and nothing is saved until you tap Import. Confirmed jobs get the full checklist with the legs filled in, so GAR countries and deadlines follow from the airports; tentative jobs keep the legs in their notes. Google Sheets users download the sheet as .xlsx first. The file is read in the browser (the reader library is loaded from cdnjs when first needed); only the finished jobs are sent to your script. **Upload `pilot-jobs-import-template.xlsx` to GitHub Pages next to `index.html`.** |
| **Check-in task** | Each commercial flight has a tick-box **Checked in** and a matching task in the checklist and the To do tab (departure time shown). Ticking it switches that flight's 24 h / 4 h / 2 h reminders off. Booking and checking in are separate tasks: *Book outbound/return flights* and *Check in for flights*. |
| **Cancel** | Removes the job and its flight/deadline events from the calendar and moves it to the **Archive** tab and *Archive* sheet. It can be restored. |
| **Complete** | Moves a finished job to the Archive; calendar entries stay. |

Everything autosaves. The data lives in the Google Sheet **Pilot Jobs – data** (tabs *Jobs* and *Archive*), so it can be viewed or exported at any time.

### Good to know

- **No inbound calendar email.** Google Calendar has no address that turns emails into events, so the app creates events directly. The email he gets is a summary of each new job with a link back to it (if `APP_URL` is set).
- **Notifications on his Pixel.** Calendar popup reminders arrive as push notifications from the Google Calendar app. Check on the phone: Google Calendar → Settings → *Pilot Jobs* → notifications on; Android Settings → Apps → Calendar → Notifications → allow the reminders channel to alert (and, if wanted, bypass Do Not Disturb). The 15-minute reminder on every deadline is the "very close" alert; change the times in `FLIGHT_PLAN_REMINDER_MINUTES` / `GAR_REMINDER_MINUTES`. Google allows five reminders per event.
- **Check-in reminders staying on screen.** Google Calendar's popup reminders are ordinary Android notifications and, by default, stay in the shade until swiped away. A script cannot force that, because it is a phone setting: on the Pixel, long-press a Calendar notification → *Settings* (or Android Settings → Apps → Calendar → Notifications) and make sure the reminders channel is set to *Alerting* with *Pop on screen*, and that it isn't being auto-cleared by a battery/"clear notifications" tool.
- **The red banner is in-app only.** It shows while the page is open. The calendar notifications are what reach him when it isn't.
- **Flight plans and GARs are tracking tick-boxes only.** The app submits nothing. The GAR rules above are built in; for **flight plans** it knows no country's rules – he enters the rule for each leg and it does the arithmetic. If no rule is entered it prompts him to check, and treats departure as the deadline until he does.
- **Times for GAR and flight plans are UTC.** Enter each leg's departure time in UTC (Zulu). Deadlines are shown in UTC; calendar events show in his own time zone.
- **GAR assumptions to confirm with him:** "flight time" = the leg's departure time; weekday/weekend uses the departure date in Paris time; French public holidays count as weekdays; only the arrival country matters.
- **Deadlines already passed** don't create calendar events (too late for a reminder); the app shows them as overdue.
- **Calendar colours** use Google's named palette (orange = Tangerine, green = Basil, blue = Blueberry); change them in `CONFIG.COLOURS`.
- **All-day vs timed:** with both a start and end time the job is a timed event; otherwise an all-day (multi-day) event with the times in its description.
- Deleting a calendar event by hand is fine – the app recreates it the next time that job is edited.

---

## 5. Security notes

- **What protects the data:** the passcode (long, checked on every request) plus HTTPS. Whoever has both the server address and the passcode has full access to his jobs and calendar.
- **Where it's stored:** the address and passcode are in each device's browser storage. Don't connect shared computers; use **Disconnect this device** if a phone is lost or sold, and change the passcode script property if one is lost.
- **The page is public, and holds nothing secret.** `index.html` has no address, passcode or data.
- **Only a fixed list of backend functions can be called** (read all, create, save, change status). Any other name in a request is refused, so the backend can't be talked into running its internal helpers.
- **What's not covered:** the passcode is a lock, not an account system. There's one passcode and one user, and the page can't tell who is using it.
