# AgeWell

AgeWell's blue-and-white landing page, a Gemini-powered weekly coordination planner, and a Supabase-backed early-access waitlist.

## Direct browser waitlist

Run `supabase/browser-waitlist.sql` after the base schema. The browser calls the restricted `agewell_join_waitlist` RPC with the Supabase publishable key. No private key is included in the frontend. Anonymous visitors cannot select, update, or delete waitlist entries or care requests. New signups are capped at 100/day across the project. This cap is basic abuse containment, not bot protection; add verified CAPTCHA or authenticated signup before a larger public launch. The AI planner still requires server-only Gemini and database credentials; the publishable key does not make Gemini available.

## What works

- The planner accepts fixed anonymous routine fields. Gemini assigns the existing tasks to family members A and B. Exact medicine timings and task labels come from validated input, not model text.
- Requests, responses, status and actual token usage are saved in `care_requests`.
- The page reads live plan totals and the most common task category from Supabase.
- The waitlist stores email and explicit consent in `waitlist`, with duplicate-safe inserts. Signup does not send a confirmation email.
- Three plan attempts per signed browser visitor, plus daily network and global limits. Database reservation is atomic, so concurrent calls cannot bypass the cap. Clearing cookies can reset the browser identity, but not the network/global daily caps.
- Medical advice, dose changes and symptom-assessment requests are refused. Raw coordination notes are not persisted or sent to Gemini.
- Checklists can be copied or downloaded. This feature does not send reminders, book appointments or staff a live care team. The hero dashboard remains an illustrative preview.

## Setup on the existing Vercel project

1. Run `supabase/schema.sql` in the AgeWell Supabase SQL Editor.
2. In the Vercel project's Environment Variables, set the values named in `.env.example`. Never commit real values.
3. Use the project's Supabase URL and a server-only secret key for `SUPABASE_SERVICE_KEY`. New `sb_secret_` keys and legacy service-role JWTs are both supported.
4. Create a Gemini API key in Google AI Studio. Set `GEMINI_MODEL` to a model actually available to that key. The default is `gemini-3.5-flash-lite`.
5. Set `VISITOR_SECRET` to at least 32 random characters, and `APP_ORIGIN` to `https://agewell-tau.vercel.app`.
6. Push to the existing GitHub repository's production branch. Keep Vercel Framework Preset **Other**, Root Directory **.**, and no static-only build/output override. Vercel deploys `/api/*.js` as server functions.
7. Redeploy after changing environment variables. Test the waitlist, normal/sample plan, unsafe question, empty input, and request cap. Confirm rows and tokens in Supabase, and the live count on the page.

Supabase tables use RLS with no browser access. Only Vercel's server functions use the secret key. Aggregate statistics expose no personal records. The planner is an early-access coordination demonstration and accepts anonymous routines only.

## Local development

Copy `.env.example` to `.env.local`, set values privately, then run `npm run dev`. No external packages are required. `npm test` checks validation, guardrails, fixed schedules, untrusted model output, cookie integrity and cross-site checks.

## Assignment evidence

Capture the live feature, Vercel variable names with values hidden, at least five `care_requests` rows, a typical and an edge-case transcript, and the refusal response. Record measured `input_tokens` and `output_tokens`; do not claim a completed integration until live tests pass.

## Maintenance

Delete unused waitlist entries and request history according to a retention policy before operating this as a public service. For larger traffic, add email verification/CAPTCHA and a dedicated abuse-control layer. Never use this demonstration to store real clinical records.

## Official implementation references

- https://vercel.com/docs/functions/runtimes/node-js
- https://supabase.com/docs/guides/getting-started/api-keys
- https://supabase.com/docs/guides/database/postgres/row-level-security
- https://ai.google.dev/gemini-api/docs/generate-content/structured-output
