# Socratic AI — Controlled Activation Guide

**Status:** Live AI remains OFF by default. Guided question mode works without credentials. This document does not imply end-to-end AI requests have passed.

## Access and safety prerequisites

An authorized Vercel project administrator must complete these steps. Do not put secrets in GitHub, the browser, support tickets, or chat.

1. Open the Vercel project `studio33 / strategy-advantage-lab`, then Settings → Environment Variables. Confirm access to manage Production environment variables.
2. Establish AI Gateway access and a **small enforced spending limit** in the relevant Vercel team/project (not merely an email warning). Verify which models and model permissions are available.
3. Establish an Upstash Redis database for rate limiting and obtain its REST URL and REST token.
4. Add these *Production* environment variables in Vercel:
   - `AI_GATEWAY_API_KEY` — secret; AI Gateway credential.
   - `AI_GATEWAY_MODEL` — a **currently verified** provider/model ID; do not guess one.
   - `UPSTASH_REDIS_REST_URL` — endpoint for atomic Redis pipeline.
   - `UPSTASH_REDIS_REST_TOKEN` — secret; database REST token.
   - `ENABLE_SOCRATIC_AI` — keep `false` until all gates pass; then set `true`.
5. Redeploy the current production commit after environment changes. Verify the deployed version's environment, functions, and aliases.

## Limits implemented in `api/socratic.js`

- Requires explicit `ENABLE_SOCRATIC_AI=true`, model, Gateway credential, and Redis credentials.
- Requires same-origin POST, nonempty answer, max 2,000 characters, limits input history, response tokens, and request size.
- Atomic Redis counters: **4 AI requests/IP/minute** and **60 AI requests/day across the whole site**. If Redis is down, billable AI is blocked.
- Short outbound AI timeout; generic errors to visitors, no secrets in responses.
- Only the server calls the AI Gateway. Visitors opt in to send their reflection.
- These are basic usage brakes, **not comprehensive bot prevention**. Shared IPs can affect legitimate users; the global daily cap may be exhausted by anonymous traffic. Production scale calls for managed bot protection, observability and incident response.

## Test plan (do not claim these have run until observed)

1. With `ENABLE_SOCRATIC_AI=false`, open `/socratic.html`: guided questions work; live AI checkbox disabled.
2. GET `/api/socratic`: should return JSON `available:false` when off.
3. POST with AI off: 503 with guidance to continue guided mode.
4. Configure the service privately and enable AI only after cost limits are enforced.
5. GET `/api/socratic`: `available:true` after redeploy.
6. Opt into live AI, send a **fictional, nonconfidential** strategic reflection, and verify that the returned question is relevant rather than generic.
7. Test empty and >2,000 character submissions and origin rejection.
8. Test five rapid requests from one IP; fifth should return 429. Test Redis interruption; must fail closed without AI calls.
9. Confirm a single real provider request in AI Gateway logs and usage dashboard; monitor actual cost and budget enforcement.
10. Rotate test keys if exposed, verify privacy wording, then decide whether public AI access is justified.

## Emergency stop

Set `ENABLE_SOCRATIC_AI=false` in Vercel Production and redeploy. Optionally disable the Gateway key, which prevents provider charges without depending on deployment propagation.

## Current administrative blocker

The connected Vercel credential returned **403 Forbidden** when listing project environment variables. This prevents verifying or adding secrets through the available connector. An authorized administrator must perform environment configuration. No working email address, model access, gateway budget, or live provider response has been independently verified.
