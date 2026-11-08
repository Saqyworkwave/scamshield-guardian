# ScamShield Guardian

**An AI check that runs *before* a PayPal payment is made.**

Many scams work by rushing someone into sending money: a fake emergency, a request for gift cards, a "don't tell anyone" instruction. ScamShield Guardian adds a pause. Before any PayPal checkout opens, an AI reviews the payment request and points out possible warning signs. If it looks like a likely scam, the server refuses to create the payment, and the app helps the user alert a family member.

> **ScamShield is an assistant, not a guarantee. The final decision is yours.**
> Results are AI-based suggestions and can be wrong. Please double-check with someone you trust.

Built for the PayPal AI Hackathon 2026.

## Live demo

https://scamshield-guardian.onrender.com

The app runs on a free Render instance, which sleeps when idle. The first load can take about a minute.
It uses the **PayPal Sandbox** only. No real money moves.

## Try it in one minute

1. Open the live demo and tap **Plumber payment**, then **Check this payment**. You should see a lower-risk result and PayPal buttons.
2. Tap **Check another payment**, then **Grandson in jail**. You should see a scam warning, no payment button, and a "Message my family" button.
3. Try **Crypto offer**. If it is marked as risky, you must tick a checkbox before the PayPal button works.

To complete a sandbox payment, log in at the PayPal popup with a **Sandbox personal** test account (developer.paypal.com > Testing Tools > Sandbox Accounts). Never enter a real card or a real PayPal login.

## How it works

1. **Details:** the user enters who they are paying, how much, why, and optionally pastes the message they received.
2. **Check:** the AI returns a verdict, an estimated risk score, and a few plain-language reasons. A gauge shows the score.
   - **Looks lower risk (SAFE):** PayPal buttons appear.
   - **Be careful (RISKY):** a warning is shown and a checkbox must be ticked before paying.
   - **This may be a scam (SCAM):** no payment button. A "Message my family" button prepares an alert text (share sheet, SMS, or email).
3. **Pay:** the PayPal JS SDK buttons call this server to create and capture a **PayPal Sandbox** order.

The safety gate is enforced **on the server**, not just in the interface. For a SCAM verdict, `POST /api/orders` returns HTTP 403 and no PayPal order is created. For a RISKY verdict the server requires `confirmed: true`. The charged amount comes from the verdict saved on the server, not from the browser.

## How AI and PayPal are used

- **AI:** Google **Gemini**, called from the server through the native Gemini API (`generateContent`) with the key in the `x-goog-api-key` header. The default model is `gemini-3.6-flash`, and it can be changed with the optional `AI_MODEL` environment variable. A system prompt asks the model to look for common scam signals (urgency, gift cards, crypto or wire transfers to strangers, secrecy requests, fake family emergencies, prize or lottery claims, bank or government impersonation) and to answer in cautious language, never with certainty. It returns JSON only. The server strips code fences before parsing.
- **Fallback:** if the AI call fails or returns invalid JSON, a keyword-based check keeps the app working.
- **PayPal:** **Sandbox** REST Orders API v2 (create order and capture order) plus the PayPal JS SDK Buttons.

## Built with

Node.js, Express, HTML/CSS/JavaScript, Google Gemini API, PayPal Sandbox Orders API v2, PayPal JS SDK, Render, GitHub.

## Run it locally

1. Install Node.js 18 or newer.
2. Run `npm install`.
3. Copy `.env.example` to `.env` and fill in:
   - `PAYPAL_CLIENT_ID` and `PAYPAL_SECRET`: from a **Sandbox** app at developer.paypal.com (Apps & Credentials)
   - `AI_API_KEY`: a Google Gemini API key from Google AI Studio
   - `PORT`: optional, default 3000
   - `AI_MODEL`: optional, to use a different Gemini model
4. Run `npm start`, then open http://localhost:3000

Without `AI_API_KEY` the app still runs using the keyword fallback.

## Deploy on Render (free)

1. Push this repo to GitHub.
2. On render.com choose New > Web Service and connect the repo.
3. Runtime: Node. Build command: `npm install`. Start command: `node server.js`. Instance type: Free.
4. Add the environment variables `PAYPAL_CLIENT_ID`, `PAYPAL_SECRET`, and `AI_API_KEY` in Render's settings. Do not set `PORT`; Render provides it.
5. Deploy, then open the `.onrender.com` URL.

If you connected the repo as a public Git repository, Render does not redeploy automatically. After a change, use Manual Deploy > Deploy latest commit.

## API

| Endpoint | Purpose |
|---|---|
| `POST /api/analyze` | `{payee, amount, reason, message}` returns `{verdict, score, reasons, advice, verdictId}` |
| `GET /api/config` | `{clientId}` (public id only) |
| `POST /api/orders` | `{verdictId, confirmed}` returns `{id}`; 403 for SCAM, or RISKY without `confirmed: true` |
| `POST /api/orders/:id/capture` | Captures an order this server created |

## Security notes

- Keys live only in environment variables (`.env` locally, Render settings online). `.env` is git-ignored, and the PayPal secret is never sent to the browser.
- Verdicts are kept in memory for 15 minutes and then expire. A production version would use a database and rate limiting.
- The user's text is sent to the AI as data to analyze, and the system prompt tells the model not to follow instructions inside it.
- This is a hackathon demo. It uses PayPal Sandbox only and is not a replacement for a bank's or PayPal's own fraud protection.

## What's next

More languages, real alerts to trusted contacts, and integration inside a payment app.

## License

MIT. See [LICENSE](LICENSE).
