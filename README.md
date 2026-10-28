# 🎓 CampusFind – Crowdsourced Lost & Found with SOS Voice Guard

**Challenge:** Build What Matters: The AI-Powered Student Innovation Challenge
**Chosen vertical:** Education (campus life and student safety)

A mobile-first, single-page web app that helps students report, search and recover lost items on campus, with an SOS voice guard for personal safety. No build step: one `index.html` (HTML5 + Tailwind CSS via CDN + vanilla JavaScript).

## The problem
Lost items on campus are usually posted in busy chat groups where messages vanish within minutes, and there is no way to check that a claimant is the real owner. Students also lack a quick way to alert someone when they feel unsafe.

## Approach and logic
1. **One shared board:** anyone can post a *Lost* or *Found* item (name, category, location, date, contact).
2. **Trust through verification:** the person posting adds a private security question and answer. A claimant must answer it before the item is marked *Claimed* and the contact details are shown.
3. **Fast discovery:** instant search plus category and Lost/Found filters, with no page reloads.
4. **Safety layer:** an SOS button and a voice guard that listens for a trigger word (default "help") and prepares an emergency message with live location.

## How it works
| Feature | Behaviour |
|---|---|
| Dashboard | Cards with status tags: red *Reported Lost*, amber *Found – Unclaimed*, green *Claimed*; a "Recently recovered" feed |
| Search and filters | Debounced (150 ms) text search; category chips; Lost/Found toggle |
| Report form | Validates name length, location, non-future date, phone or email, and security Q&A; new item appears instantly |
| Claim verification | Security question dialog; answer compared as a SHA-256 hash; 30 s lockout after 3 wrong tries |
| Persistence | Items and settings saved in `localStorage` and restored on reload |
| 3D element | Draggable CSS 3D cube in the header (respects reduced-motion) |
| SOS | Set name, emergency contact and trigger word. Saying the word (or tapping SOS) starts a 5 s cancellable countdown, then offers WhatsApp or SMS with a prefilled message and a map link |

## Security
- All user text is HTML-escaped before rendering (prevents XSS).
- Security answers are never stored in plain text (SHA-256 with a per-item salt).
- Claim attempts are rate limited per item.
- Meta Content-Security-Policy; the nginx config adds `X-Content-Type-Options`, `X-Frame-Options`, `Referrer-Policy` and `Permissions-Policy`.
- No third-party data collection; everything stays in the browser.

## Testing
23 self-tests run automatically on page load and cover validation, filtering, answer hashing, claim lockout, SOS message building, HTML escaping, empty states and storage. Tap the **Tests** badge in the header to see each test with pass or fail.

## Accessibility
Skip link, labelled inputs and dialogs, `aria-live` announcements for toasts and the SOS countdown, `aria-invalid` with focus on form errors, visible focus rings, 44 px touch targets, light and dark themes, `prefers-reduced-motion` support, and an **Aa** button for three text sizes.

## Efficiency
Keyed rendering reuses unchanged cards, `content-visibility` skips off-screen cards, search is debounced, and the cube animation and microphone pause when the tab is hidden.

## Run locally
Open `index.html` in a browser, or serve it with any static server.

## Deploy to Google Cloud Run
```bash
gcloud run deploy campusfind --source . --region asia-south1 --allow-unauthenticated
```
The included `Dockerfile` and `nginx.conf` serve the app on port 8080.

## Assumptions and limitations
- Data is stored per device (`localStorage`); a real deployment would add a shared backend and login.
- Browsers cannot send SMS or WhatsApp messages silently, so SOS opens a prefilled message that the user taps to send.
- Voice guard uses the browser Speech Recognition API (best in Chrome) and matches the trigger word, not the speaker's identity. It works only while the tab is open.
- Client-side hashing deters casual snooping but does not replace server-side verification.
