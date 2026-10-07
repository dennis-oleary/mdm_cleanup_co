# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Single static landing page for "MDM Cleanup Co." — a service pitch for removing unused devices from MDM platforms (Workspace ONE, Jamf, Intune) to cut licensing costs. The page is a lead-capture form, not an app.

Deployed via GitHub Pages at the repo root (`CNAME` → `mdmcleanup.com`). No build step: edit `index.html` / `style.css`, commit, push to `main`, GitHub Pages serves it directly.

## Files

- `index.html` — entire page: markup, inline `<script>` for form handling, Google reCAPTCHA v2 widget.
- `style.css` — all styling, single file, no preprocessor.
- `CNAME` — custom domain for GitHub Pages (`mdmcleanup.com`). Don't remove/edit unless intentionally changing the domain.
- `favicon.ico`

No package.json, no JS framework, no test suite, no linter. There is nothing to build or run beyond opening `index.html` in a browser.

## Form submission flow

The quote-request form posts to a Cloud Function (`submitLead`) in the sibling
`mdm-cleanup-backend` project (`D:\Dev\mdm-cleanup-backend`), not Formspree —
it used to go through Formspree but was swapped over once that backend existed:

1. Submit intercepted by `handleFormSubmit` (inline script in `index.html`) — `e.preventDefault()`.
2. reCAPTCHA v2 token read via `grecaptcha.getResponse()`; blocks submission with an alert if missing.
3. `submitFormWithCaptcha` does a `fetch` POST (JSON body + `g-recaptcha-response`) to
   `https://us-central1-mdm-cleanup-backend.cloudfunctions.net/submitLead`.
4. `submitLead` writes the lead to that project's Firestore (`leads` collection) and emails
   a notification via Resend. The lead shows up in that project's admin inbox UI.
5. On success, the form section hides and `#thanks` section is shown in its place; a "Submit Another Request" button resets `grecaptcha` and swaps the sections back.
6. On failure (non-OK response or network error), shows an `alert`, re-enables the submit button, and resets reCAPTCHA.

There's also a hidden honeypot field (`name="website"`, `display:none` label) for basic bot filtering — `submitLead` silently no-ops if it's filled in.

When editing the form, keep the `_replyto` field name (the backend reads it as the lead's reply email) and the honeypot field intact unless deliberately changing the backend integration. The `submitLead` function only accepts JSON bodies and only allows CORS from `https://mdmcleanup.com` — test against it from that origin, not by opening `index.html` as a local file.

## Editing notes

- The reCAPTCHA site key is hardcoded in `index.html` (`data-sitekey`) — it's a public client-side key, not a secret, but if the domain changes, the key must be reissued in the Google reCAPTCHA admin console and swapped here.
- Two page sections (`#home`/`#form-section` and `#thanks`) are toggled via inline `style.display`, not routing — there's no multi-page structure to preserve.
- Mobile styling is a single media query at the bottom of `style.css` (`max-width: 600px`); keep new styles consistent with that breakpoint rather than adding others.
