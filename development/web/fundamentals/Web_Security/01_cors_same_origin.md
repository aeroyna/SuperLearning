# CORS & Same-Origin Policy 🚧

**CORS (Cross-Origin Resource Sharing)** is a mechanism that allows a server to indicate any origins (domain, scheme, or port) other than its own from which a browser should permit loading resources. It acts as a controlled relaxation of the **Same-Origin Policy**.

---

## 1. The Same-Origin Policy (SOP) 🏠

The browser's core security model. It prevents a script on `evil.com` from reading data (like API responses or Cookies) from `bank.com`.

### What defines an "Origin"?
Tuple: `(Protocol, Host, Port)`.

| URL | Origin | Same Origin? |
| :--- | :--- | :--- |
| `https://my-site.com/page` | `https://my-site.com:443` | Base |
| `https://my-site.com/other` | `https://my-site.com:443` | ✅ Yes |
| `http://my-site.com/page` | `http://my-site.com:80` | ❌ No (Diff Protocol) |
| `https://api.my-site.com` | `https://api.my-site.com` | ❌ No (Diff Subdomain) |
| `https://my-site.com:8080` | `https://my-site.com:8080` | ❌ No (Diff Port) |

---

## 2. How CORS Works 🔓

If `frontend.com` makes a request to `api.backend.com`, the browser blocks the response by default. To allow it, `api.backend.com` must send specific headers.

### Simple Requests (GET, POST, HEAD)
The browser sends the request with an `Origin` header.
*   **Request**: `Origin: https://frontend.com`
*   **Response**: `Access-Control-Allow-Origin: https://frontend.com` (or `*`)

If the header matches, the browser allows the JS to read the response.

### Preflight Requests (OPTIONS) ✈️
Triggered by "complex" requests (e.g., `PUT`, `DELETE`, or custom headers like `Authorization`, `Content-Type: application/json`).

1.  **Browser**: Sends an `OPTIONS` request first.
    *   `Access-Control-Request-Method: PUT`
    *   `Access-Control-Request-Headers: Authorization`
2.  **Server**: Responds with permission.
    *   `Access-Control-Allow-Methods: GET, PUT`
    *   `Access-Control-Allow-Headers: Authorization`
3.  **Browser**: If permitted, sends the actual `PUT` request.

---

## 3. Credentials (Cookies) 🍪

By default, CORS requests **do not** send cookies. To send cookies cross-origin:

1.  **Client**: `fetch(url, { credentials: 'include' })`
2.  **Server**: Must respond with:
    *   `Access-Control-Allow-Credentials: true`
    *   `Access-Control-Allow-Origin` cannot be `*` (Must be specific origin).
