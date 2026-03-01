# Content Security Policy (CSP) 👮

**CSP** is an added layer of security that helps to detect and mitigate certain types of attacks, including Cross-Site Scripting (XSS) and data injection attacks. It works by defining which dynamic resources are allowed to load.

---

## 1. How it Works ⚙️

CSP is implemented via an HTTP response header.

```http
Content-Security-Policy: default-src 'self'; img-src https://*; child-src 'none';
```

### Directives
*   `default-src`: Fallback for other directives.
*   `script-src`: Where JS files can load from.
*   `style-src`: Where CSS files can load from.
*   `img-src`: Where images can load from.
*   `connect-src`: Who we can send XHR/Fetch requests to.

---

## 2. Common Configurations 📝

### Strict (Recommended)
Only allow scripts from own domain.
```http
Content-Security-Policy: default-src 'self'; script-src 'self' https://trusted-cdn.com;
```

### Preventing Inline Scripts
XSS often relies on inline scripts (`<script>...`). CSP blocks inline scripts by default unless `'unsafe-inline'` is specified (NOT recommended).

### Nonces (Number used ONCE)
If you MUST use inline scripts, use a cryptographic nonce.
1.  Server generates random nonce: `abc12345`.
2.  Header: `script-src 'nonce-abc12345'`.
3.  HTML: `<script nonce="abc12345">...</script>`.
Scripts without the matching nonce are blocked.

---

## 3. Reporting Mode 📢

You can test CSP without breaking your site using `Content-Security-Policy-Report-Only`. Violations are reported to a URL but not blocked.

```http
Content-Security-Policy-Report-Only: default-src 'self'; report-uri /csp-violation-report-endpoint/
```
