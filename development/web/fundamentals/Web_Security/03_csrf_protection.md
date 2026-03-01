# CSRF (Cross-Site Request Forgery) 🎭

**CSRF** (often pronounced "Sea-Surf") is an attack that tricks the victim into submitting a malicious request. It exploits the trust that a web application has in the user's browser (specifically, cookies are sent automatically).

---

## 1. The Attack Scenario 🏴‍☠️

1.  **Context**: You are logged into your bank (`bank.com`). Your session cookie is stored in the browser.
2.  **Trap**: You visit a malicious site (`evil.com`).
3.  **Exploit**: `evil.com` contains a hidden form or image:
    ```html
    <form action="https://bank.com/transfer" method="POST">
      <input type="hidden" name="to" value="hacker">
      <input type="hidden" name="amount" value="1000">
    </form>
    <script>document.forms[0].submit()</script>
    ```
4.  **Execution**: The browser sends the POST request to `bank.com`. Since cookies are sent automatically with requests to their domain, `bank.com` sees the valid session cookie and processes the transfer.

---

## 2. Prevention Strategies 🛡️

### 1. Anti-CSRF Tokens (Synchronizer Token Pattern)
*   **Mechanism**: The server generates a random, unique token for the session and embeds it in every HTML form as a hidden field.
*   **Validation**: When the form is submitted, the server checks if the token in the form matches the token in the session.
*   **Why it works**: `evil.com` cannot read the token from `bank.com` (due to Same-Origin Policy), so it cannot include it in the forged form.

### 2. SameSite Cookie Attribute
A modern browser feature that controls when cookies are sent with cross-site requests.
*   `SameSite=Strict`: Cookie is never sent on cross-site requests (even clicking a link).
*   `SameSite=Lax` (Default): Cookie is sent on top-level navigations (clicking a link) but not on sub-requests (images, frames, XHR). Good balance of security and UX.
*   `SameSite=None`: Cookie is always sent (requires `Secure` flag).

### 3. Check Origin / Referer Headers
Verify that the request originated from your domain. (Not fool-proof, but a good defense-in-depth).
