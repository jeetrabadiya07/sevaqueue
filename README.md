# SevaQueue — Government Services
**PS-02 | Team Hustlers**

**લાઈનમાં રાહ નહીં, સીધી સેવા.**

A government-services queue system that predicts waiting time and lets citizens take a **virtual token remotely**, arriving only when their turn is close.

## Features
- **Citizen app**: live office load, wait prediction, virtual token, live status ring, "leave now" alert
- **Best-time forecast**: hourly estimated wait chart based on current queue and typical busy-hour factors
- **Documents checklist**: service-specific checklist for passport, RTO, municipal and Aadhaar services
- **Priority queue**: pregnant women, persons with disabilities, and senior citizens (60+) are called before the regular queue
- **QR token**: citizen token QR can be scanned from the officer panel to call that token
- **Officer analytics**: hourly bookings, average wait, no-show rate and citizen rating
- **Feedback**: citizens can rate completed service from 1 to 5 stars
- **Officer panel**: login, call next, done / no-show, open-counter control, live queue, demo crowd and Autopilot
- **Voice call-out**: opt-in browser speech announces the citizen's turn in English, Hindi, or Gujarati
- **Time saved**: completed tokens show an estimate of queue-standing time avoided
- **WhatsApp / SMS alerts**: token booked, time to leave, your turn (in the citizen's language)
- **Missed-call token**: optional Twilio voice webhook issues a token by SMS for feature-phone users
- **3 languages**: English, Hindi, Gujarati (switch anytime)
- **Self-learning wait time**: average service time updates after every served citizen

## Run locally
```
node server.js
```
Open http://localhost:3000 (needs Node.js 18+). The page connects to Firebase; fill
the Firebase web app values in `public/firebase-config.js` first. The Node server
serves the frontend only; queue data and authentication are handled by Firebase.
The hourly forecast is a demo estimate using the live queue and configured busy-hour factors; it is not trained on historical footfall. The document list is guidance, not an official government checklist.

## Render hosting (optional)
The included Blueprint can host the frontend on Render's Node service. Configure
`public/firebase-config.js` with the same Firebase web app values. Render does not
store the queue; all current queue state is in Firestore.

## Firebase Auth, Firestore, and Hosting
This repository targets the `queueless-hustlers-2026` Firebase project. Before running
or deploying, copy the Web app configuration from **Firebase console → Project settings
→ General → Your apps** into `public/firebase-config.js`. The Firebase web config is
public client configuration; never put service-account keys or Twilio credentials there.

In **Authentication → Sign-in method**, enable Phone and Google. Enable Email/Password
for officers. Anonymous sign-in is optional and is shown only when `demoMode: true` in
`firebase-config.js`. For OTP tests, open **Authentication → Sign-in method → Phone
numbers for testing**, add a test phone number and a six-digit test code, then use
those exact values in the app. In **Authentication → Settings → Authorized domains**,
add `queueless-hustlers-2026.web.app` (and `localhost` for local testing).

Create a Firestore database, then add officer records after creating officer accounts
in Firebase Authentication: for each officer, create `officers/{AUTH_UID}` with
`{ "active": true }`. Rules deliberately prevent clients from adding or changing
officer records. The app initializes the four fixed public office-stat documents.
Deploy rules, indexes, hosting and notification functions with:
```
firebase deploy --only firestore,functions,hosting
```

## Real WhatsApp / SMS (optional)
Firebase notification functions use Twilio, if configured. Put `TWILIO_SID`,
`TWILIO_TOKEN`, `TWILIO_SMS_FROM` and/or `TWILIO_WHATSAPP_FROM` in the ignored
`functions/.env.queueless-hustlers-2026` file; `TWILIO_FROM` is a sender fallback.
Do not commit that file or put provider credentials in frontend config. WhatsApp
senders must be enabled in Twilio, and sandbox recipients must first join the sandbox.
Trial accounts can message only verified recipients. The officer panel distinguishes
not configured, failed and accepted-by-Twilio; accepted does not guarantee delivery.

## Missed-call tokens (optional)
For the Firebase queue, set `TWILIO_WEBHOOK_URL` and optionally
`MISSED_CALL_OFFICE` (`passport`, `rto`, `muni`, or `aadhaar`) in the ignored
`functions/.env.queueless-hustlers-2026` file. Configure the Twilio voice number's
incoming-call webhook to the deployed `twilioVoice` HTTPS function URL using POST.
Set `missedCallNumber` in `public/firebase-config.js` to display that number.
The webhook validates Twilio's signature, creates a guest token in Firestore,
rejects the call, and sends the token by SMS. Twilio charges may apply.

## Demo script for judges (2 minutes)
1. Open the citizen page, switch language to Gujarati, choose a service and inspect the hourly forecast and document checklist.
2. Sign in as a citizen by phone OTP or Google, finish the profile, and book a token; use demo anonymous login only when `demoMode` is enabled.
3. Open the officer panel with an authorized email/password account, scan a token QR or enable Autopilot to run a demo-only queue.
4. Complete service, submit a rating, then show the hourly analytics and alerts.

## Formula
`wait = people ahead x avg service time / open counters x peak-hour factor`

## Tech
Firebase Authentication + Firestore + vanilla JS frontend, with optional Firebase
Functions for Twilio notifications. `server.js` serves the frontend locally; it does
not provide queue storage or authentication.
