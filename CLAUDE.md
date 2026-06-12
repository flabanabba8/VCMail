# VCMail

Multi-tenant serverless email server, shipped as an npm package/CLI (`npx vcmail`).
One shared AWS Lambda handles SES email ingress + the webmail REST API for many
domains; per-domain config lives in AWS SSM. Storage is split: **S3** = raw bodies
+ attachments, **Firebase Realtime DB** = lightweight metadata for fast inbox lists.
Fork of `oceanseth/VCMail` (Apache-2.0). Node >=18, JavaScript (no TypeScript).

## Commands
- `npm install` — install deps
- `npm run build` — Vite build of the webmail frontend → `dist/`
- `npm run preview` — local preview server for the frontend
- `npm run deploy-rules` — push `database.rules.json` to Firebase
- `npm run vcmail` / `npx vcmail` — setup wizard (see deploy gotcha below)

## ⚠️ Deploy gotcha (do not ignore — see `.cursorrules`)
**Never deploy from this repo.** It is the package. Real deploys happen in a
*consuming* project: `npm link` here, `npm link vcmail` there, then `npx vcmail`
in that project — its wizard copies Terraform (`lib/terraform/`) into that project
and runs it there. Never run `terraform apply` or `npx vcmail` deploy steps here.

## Structure
- `api/api.js` — ~3,600-line Lambda: SES inbound + all REST routes. The core.
- `firebaseInit.js` — per-domain Firebase Admin init from SSM service account.
- `src/email.js` — ~3,600-line webmail frontend (Firebase Auth, inbox, compose).
- `src/googleCalendar.js` — `.ics` import to Google Calendar (real, working).
- `index.html` — frontend shell; `window.VCMAIL_CONFIG` injected at build/deploy.
- `lib/setup.js`, `bin/vcmail.js` — CLI/setup wizard. `lib/terraform/` — IaC templates.
- `serverless.yml`, `database.rules.json` — infra + Firebase security rules.
- `mail-server/` — optional Oracle Cloud IMAP/SMTP/POP3 (not in the Lambda path).

## Conventions
- Backend is one big handler; trace routes from `exports.handler` in `api/api.js`.
- Firebase paths can't contain `. # $ [ ]` — emails are encoded (`user_at_domain_dot_com`).
- Region is hardcoded `us-east-1`. Firebase databaseURL is *constructed* assuming the
  default `-default-rtdb` name — breaks on custom-named DBs.
- No automated test suite. The root `test-*.json` / `response*.json` files are stray
  manual-debug payloads, not tests — don't treat them as fixtures.

## Reality-check (README oversells)
- **WebLLM / on-device AI is NOT implemented** — no mlc-ai/WebLLM code exists.
- **E2E encryption / "VoiceCert" decryption is NOT implemented** — bodies are stored
  plaintext in S3. "VoiceCert" is branding, not a crypto/auth layer here.
- License: real one is **Apache-2.0** (README's MIT badge is wrong).
Treat these as greenfield if a fork goal depends on them, not as existing features.
