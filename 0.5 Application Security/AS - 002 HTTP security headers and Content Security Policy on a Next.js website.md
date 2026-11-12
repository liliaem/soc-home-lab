**Date:** 2026-11-10 to 2026-11-12 
**Analyst:** Lilia
**Status:** Completed 
**Type:** Security hardening / misconfiguration
**Related:** AS-001 Dependency vulnerability triage

## Summary

|Control|Before (production)|After (production)|
|---|---|---|
|Strict-Transport-Security (HSTS)|✅ (added by Vercel)|✅ (unchanged)|
|X-Content-Type-Options|❌|✅ `nosniff`|
|X-Frame-Options|❌|✅ `DENY`|
|Referrer-Policy|❌|✅|
|Permissions-Policy|❌|✅|
|Content-Security-Policy|❌|✅|
|X-Powered-By|Not observed|Disabled in config|

Four security headers and a Content Security Policy were added to a public Next.js website, each verified in a staging environment and then in production. The CSP accepts `'unsafe-inline'` as a documented trade-off, with a compensating control. During validation, the CSP blocked two third-party resources, both traced to their source.

## Objective

Harden the browser-side security of a public website with HTTP security headers and a Content Security Policy, verify each control before and after release, and document the trade-offs.

## Environment & scope

|Item|Details|
|---|---|
|Application|Personal Next.js 16 website (meetbible.faith), static pages|
|Hosting|Vercel (production = `main`, staging = `dev` branch)|
|Tools|curl, Chrome DevTools (Network, Console), ESLint|
|Workflow|Feature branch → PR to `dev` (staging) → release PR to `main`|

Scope: HTTP response headers set by the application. DNS records and the apex domain's HSTS settings were out of scope.

## Methodology

### Step 1 - Baseline (production, before any change)

```
curl -sI https://www.meetbible.faith | grep -iE "^(x-content|x-frame|referrer|permissions|strict-transport)"
```

`curl -sI` downloads only the HTTP response headers. Result:

```
strict-transport-security: max-age=63072000
```

Only HSTS was present, added automatically by Vercel. No other security headers.

### Step 2 - Security headers

Added in `next.config.mjs` for every route:

|Header|What it does|Prevents|
|---|---|---|
|`X-Content-Type-Options: nosniff`|The browser trusts the declared file type, no guessing|MIME sniffing|
|`X-Frame-Options: DENY`|No other site can embed the page in a frame|Clickjacking|
|`Referrer-Policy: strict-origin-when-cross-origin`|External sites only learn the domain, not the full URL|Information leakage|
|`Permissions-Policy: camera=(), microphone=(), geolocation=(), payment=(), usb=()`|Turns off device features the site never uses|Abuse of browser features|
|`poweredByHeader: false`|Stops the `X-Powered-By: Next.js` header|Reconnaissance (fingerprinting)|

Note on `X-Powered-By`: it was not observed in the baseline because static pages are served from Vercel's CDN without Next.js adding it. Disabling it in the config means future dynamic pages won't send it either.

### Step 3 - Staged release

1. PR from the feature branch to `dev`. Vercel built a preview deployment.
2. Verified on the preview in DevTools (Network → Response headers): all four headers present, no `X-Powered-By`.
3. Release PR from `dev` to `main`.
4. Verified in production with the Step 1 command:

```
permissions-policy: camera=(), microphone=(), geolocation=(), payment=(), usb=()
referrer-policy: strict-origin-when-cross-origin
strict-transport-security: max-age=63072000
x-content-type-options: nosniff
x-frame-options: DENY
```

### Step 4 - Content Security Policy design

A CSP is an allowlist: it tells the browser where scripts, styles, images and fonts may load from. Anything not listed is blocked. It is the main browser-side defense against Cross-Site Scripting (XSS).

Production policy:

```
default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; img-src 'self'; font-src 'self'; object-src 'none'; base-uri 'self'; form-action 'self'; frame-ancestors 'none'; upgrade-insecure-requests
```

|Directive|Meaning|
|---|---|
|`default-src 'self'`|By default, only load resources from the site's own domain|
|`img-src` / `font-src 'self'`|Images and fonts only from the site (fonts are self-hosted by Next.js)|
|`object-src 'none'`|No plugins (Flash, Java)|
|`base-uri 'self'`|Blocks `<base>` tag hijacking of relative links|
|`form-action 'self'`|Forms can't send data to other sites|
|`frame-ancestors 'none'`|Modern equivalent of `X-Frame-Options: DENY`|
|`upgrade-insecure-requests`|Forces any `http://` resource to `https://` (production only)|

Environment differences:

- `'unsafe-eval'` is added **only in development** (React uses `eval` for debugging; not needed in production).
- `upgrade-insecure-requests` is **only in production**, because it can break `http://localhost` in some browsers.

**Trade-off: `'unsafe-inline'` in `script-src` and `style-src`.** Next.js and the theme switcher inject small inline scripts. The strict alternative, nonces, would force every page to be rendered on the server per request, giving up static delivery. With `'unsafe-inline'`, an injected inline script would **not** be blocked by the CSP.

Accepted because XSS requires an injection point, and none were identified during a review of the codebase: no forms, no database, no user content, no third-party scripts, and no use of `dangerouslySetInnerHTML`.

### Step 5 - Peer review

The CSP was reviewed by me and two AI assistant before release. Changes made from the review:

- `upgrade-insecure-requests` limited to production.
- `blob:` and `data:` removed from `img-src` (the site doesn't use them; least privilege).
- Trade-off and upgrade plan documented in the code.

One review concern was rejected with justification: an SVG loaded from a `data:` URL as an image cannot run scripts (browsers sandbox SVG in `<img>`), so it was not an XSS vector. The values were removed anyway because they were unused.

### Step 6 - Compensating control

ESLint rule `react/no-danger` set to `error`: the lint fails if any code uses `dangerouslySetInnerHTML`, the most common way XSS enters a React application. Since lint runs before every push, this blocks the main injection path that `'unsafe-inline'` leaves uncovered.

### Step 7 - Validation

```
curl -sI https://www.meetbible.faith | grep -i content-security
```

Returned the full policy from Step 4, including `upgrade-insecure-requests`.

## Findings

### Finding 1 - Vercel Toolbar blocked on preview deployments

On the preview deployment, the Console showed two CSP violations:

- Script from `vercel.live/_next-live/feedback/feedback.js`
- Stylesheet from `fonts.googleapis.com` (Inter font)

Neither resource exists in the site's code: the site uses the Geist font, self-hosted by Next.js. Both belong to the Vercel Toolbar, a feedback widget that Vercel injects **only into preview deployments**. Vercel states the toolbar does not support strict CSPs.

**Decision:** left blocked. The feature is not used, and allowing an extra domain only to support it would go against least privilege. The violation does not appear in production.

### Finding 2 - Unknown font request blocked in production

After the release, the production Console still showed one CSP violation: a stylesheet from `fonts.googleapis.com` (Inter).

Investigation:

|Hypothesis|Test|Result|
|---|---|---|
|The site's code requests it|Searched the repository for `googleapis` and `Inter`|Not found|
|The server sends it in the HTML|`curl -s https://www.meetbible.faith \| grep -o "fonts.googleapis[^\"]*"` (no browser involved)|Empty output|
|A browser extension injects it|Loaded the site in Incognito (extensions disabled)|Console: "No issues"|

**Conclusion:** a browser extension on my machine injected an external font into the page, and the CSP blocked it. I identified the extension and removed it.

## Verdict

**Hardened.** All planned headers and the CSP are active in production and verified.

**Risk accepted (documented):** `'unsafe-inline'` for scripts and styles. Acceptable while the site is static and no injection points have been identified. Compensating control in place (`react/no-danger`).

## OWASP / CWE mapping

- **OWASP Top 10:2025 — A02 Security Misconfiguration:** missing security headers and CSP at baseline.
- **CWE-1021:** Improper Restriction of Rendered UI Layers or Frames (clickjacking) — `X-Frame-Options`, `frame-ancestors`.
- **CWE-200:** Exposure of Sensitive Information — `Referrer-Policy`, `X-Powered-By`.
- **CWE-79:** Cross-Site Scripting — partially mitigated by the CSP.

## Analyst notes / limitations

- With `'unsafe-inline'`, the CSP does **not** block injected inline scripts or styles. It blocks external scripts and other resources.
- `style-src 'unsafe-inline'` still allows CSS injection, which can leak data in some cases. Low risk while the page holds no secrets or personal data.
- The production domain's HSTS is `max-age` only, without `includeSubDomains` or `preload`. Not changed in this work.
- The CSP was enforced directly instead of using `Content-Security-Policy-Report-Only` first. The preview deployment served as the observation phase, which was practical for a small static site but would not scale to a large one.

## Recommendations

- **Before adding a database, the Bible API or AI-generated content:** switch to a nonce-based CSP (`proxy.ts`) and sanitize any HTML/Markdown before rendering it. Dynamic pages remove most of the cost of nonces.
- Keep `react/no-danger` enabled.
- Evaluate HSTS `includeSubDomains` and `preload` for the custom domain.
- Optionally disable the Vercel Toolbar for previews to reduce Console noise.
- Re-run the Step 1 and Step 7 checks after any change to `next.config.mjs`.

## Lessons learned

- **A blocked request is not always an attack.** Both violations came from tools (a Vercel widget and a browser extension), not attackers. Investigate before concluding.
- **Separate server from browser.** `curl` shows what the server sends; Incognito shows what the browser adds. Together they located the source.
- **Severity depends on context.** `'unsafe-inline'` is a real weakness, but its risk depends on whether there's an injection point. Document the trade-off and the trigger to revisit it.
- **Peer review improves the result, but not every comment is correct.** Accept what's right, explain what isn't.
- **Test in staging, verify in production.** The same check before and after release turns "it should work" into evidence.