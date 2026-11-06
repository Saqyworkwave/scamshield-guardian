# ScamShield Guardian

**An AI check that runs *before* a PayPal payment is made.**

Many scams work by rushing someone into sending money: a fake emergency, a request for gift cards, a "don't tell anyone" instruction. ScamShield Guardian adds a pause. Before any PayPal checkout opens, an AI reviews the payment request and points out possible warning signs. If it looks like a likely scam, the server refuses to create the payment, and the app helps the user alert a family member.

> **ScamShield is an assistant, not a guarantee. The final decision is yours.**
> Results are AI-based suggestions and can be wrong. Please double-check with someone you trust.

**Live demo:** https://YOUR-APP-NAME.onrender.com  *(replace with your Render URL)*
*The free Render instance sleeps when idle, so the first load can take about a minute.*

## How it works

1. **Intake:** the user enters who they are paying, how much, why, and optionally pastes the message they received.
2. **Verdict:** the AI returns a verdict, an estimated risk score, and a few plain-language reasons.
   - **Looks lower risk (SAFE):** PayPal buttons appear.
   - **Be careful (RISKY):** a warning is shown and a checkbox must be ticked before paying.
   - **May be a scam (SCAM):** no payment button; a "Message my family" button prepares an alert text.
3. **Payment:** the PayPal JS SDK buttons call this server to create and capture a **PayPal Sandbox** order.

The safety gate is enforced **on the server**, not just in the interface. For a SCAM verdict, `POST /api/orders` returns HTTP 403 and no PayPal order is created. The charged amount is taken from the saved verdict on the server, not from the browser.

## How AI and PayPal are used

- **AI:** Google **Gemini** (`gemini-2.5-flash`) via the native Gemini API. A system prompt asks it to look for common scam signals (urgency, gift cards, crypto or wire transfers to strangers, secrecy requests, fake family emergencies, prize or lottery claims, bank or government impersonation) and to answer in cautious language, never with certainty. If the AI call fails or returns invalid JSON, a keyword-based fallback keeps the app working.
- **PayPal:** **Sandbox** REST Orders API v2 (create order and capture order) plus the PayPal JS SDK Buttons. Only sandbox money is used.

## Built with

Node.js, Express, HTML/CSS/JavaScript, Google Gemini API, PayPal Sandbox Orders API v2, PayPal JS SDK, Render, GitHub.

## Run it locally

1. Install Node.js 18 or newer.
2. `npm install`
3. Copy `.env.example` to `.env` and fill in:
   - `PAYPAL_CLIENT_ID` and `PAYPAL_SECRET`: from a **Sandbox** app at developer.paypal.com (Apps & Credentials)
   - `AI_API_KEY`: a Google Gemini API key from Google AI Studio
   - `PORT`: optional, default 3000
4. `npm start`, then open http://localhost:3000

Without `AI_API_KEY` the app still runs using the keyword fallback.
To test a payment, log in at the PayPal popup with a **Sandbox personal** test account (developer.paypal.com > Testing Tools > Sandbox Accounts).

## Deploy on Render (free)

1. Push this repo to GitHub.
2. On render.com choose New > Web Service and connect the repo.
3. Environment: Node. Build command: `npm install`. Start command: `node server.js`. Plan: Free.
4. Add environment variables `PAYPAL_CLIENT_ID`, `PAYPAL_SECRET`, `AI_API_KEY` in Render's settings. Do not set `PORT`; Render provides it.
5. Deploy, then open the `.onrender.com` URL.

## API

| Endpoint | Purpose |
|---|---|
| `POST /api/analyze` | `{payee, amount, reason, message}` returns `{verdict, score, reasons, advice, verdictId}` |
| `GET /api/config` | `{clientId}` (public id only) |
| `POST /api/orders` | `{verdictId, confirmed}` returns `{id}`; 403 for SCAM, or RISKY without `confirmed: true` |
| `POST /api/orders/:id/capture` | Captures an order this server created |

## Security notes

- Keys live only in environment variables (`.env` locally, Render settings online). `.env` is git-ignored, and the PayPal secret is never sent to the browser.
- Verdicts are kept in memory for 15 minutes and expire. A production version would use a database and rate limiting.
- This is a demo built for a hackathon. It uses PayPal Sandbox only and is not a replacement for a bank's or PayPal's own fraud protection.

## What's next

More languages, real alerts to trusted contacts, and integration inside a payment app.

## License

MIT. See [LICENSE](LICENSE).
