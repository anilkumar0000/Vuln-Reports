# Reflected Cross-Site Scripting (XSS) in Search Function

## Summary

I identified a Reflected Cross-Site Scripting (XSS) vulnerability in the `search` parameter of the application.

The application accepts user-controlled input through the `search` GET parameter and reflects it back to the page without proper sanitization or output encoding. By injecting a `<SCRiPt>` tag with mixed-case formatting, arbitrary code execution is achieved despite case-sensitive filtering mechanisms.

An attacker can craft a malicious URL and trick a victim into clicking it. Once the victim visits the page, the injected JavaScript executes in the context of the vulnerable application, triggering an `alert(1)` popup.

## Vulnerability Type

**Cross-Site Scripting (XSS) - Reflected**

## Vulnerable Parameter

```text
search
```

## Vulnerable URL

```text
https://kzlabs.in/16.php?search='>anil<SCRiPt>alert(1)</SCRiPt>
```

## Steps to Reproduce

1. Open the vulnerable page.

2. Replace the value of the `search` parameter with the following payload:

   ```text
   '>anil<SCRiPt>alert(1)</SCRiPt>
   ```

3. Visit the crafted URL:

   ```text
   https://kzlabs.in/16.php?search=%27%3Eanil%3CSCRiPt%3Ealert%281%29%3C%2FSCRiPt%3E
   ```

4. Observe that the browser executes the injected payload and displays an `alert(1)` popup.

## Payload Used

```text
'>anil<SCRiPt>alert(1)</SCRiPt>
```

URL-encoded version:
```text
%27%3Eanil%3CSCRiPt%3Ealert%281%29%3C%2FSCRiPt%3E
```

## Proof of Concept

The application accepts the value of the `search` parameter and reflects it directly into the page HTML without proper sanitization.

The payload works by:
1. Breaking out of the existing HTML context using `'>`
2. Injecting a `<SCRiPt>` tag with mixed-case formatting to evade case-sensitive filters
3. The browser parses and executes the injected script tag regardless of case
4. The `alert(1)` function executes, displaying a popup

```html
<!-- Injected payload rendered in the page: -->
'><script>alert(1)</script>
```

HTML is case-insensitive, so `<SCRiPt>`, `<script>`, `<SCRIPT>`, etc., all work the same way. This technique effectively bypasses case-sensitive blacklist filters.

<img width="1463" height="506" alt="Screenshot 2026-08-14 200053" src="https://github.com/user-attachments/assets/b9fb3d49-3bf7-4132-ac57-3f6cc7673d1d" />


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
  echo htmlspecialchars($_GET['search'], ENT_QUOTES, 'UTF-8');
  ```

- **Case-Insensitive Filtering**: If implementing input filtering, ensure it is case-insensitive by converting all input to lowercase or using case-insensitive pattern matching.
  ```php
  $input = strtolower($_GET['search']);
  if (strpos($input, '<script>') !== false) {
      // Block or sanitize
  }
  ```

- **Input Validation**: Validate and sanitize all user inputs. Consider using a whitelist approach to allow only safe characters.

- **Content Security Policy (CSP)**: Implement a strict CSP that restricts inline scripts and unauthorized script sources using `unsafe-inline` and `unsafe-eval` directives.

- **Context-Aware Escaping**: Use context-aware escaping based on where the data is placed (HTML body, attributes, JavaScript, etc.).

- **Web Application Firewall (WAF)**: Deploy a WAF, such as Cloudflare WAF, to help detect and block malicious XSS payloads.

- **Security Headers**: Implement security headers like `X-XSS-Protection` and `X-Content-Type-Options` for additional protection.

- **HTML Sanitization**: Use a robust HTML sanitization library (like DOMPurify or HTML Purifier) to strip dangerous tags and attributes regardless of case.

---
