# Reflected Cross-Site Scripting (XSS) - Encoding Bypass

## Summary

I identified a Reflected Cross-Site Scripting (XSS) vulnerability in the `cat` parameter of the application.

The application accepts user-controlled input through the `cat` GET parameter and reflects it back to the page without proper sanitization or output encoding. By injecting an `<IMG>` tag with mixed-case formatting and using Unicode escape sequences (`\u0061\u006c\u0065\u0072\u0074`) to obfuscate function names, arbitrary code execution is achieved despite multiple filtering mechanisms.

An attacker can craft a malicious URL and trick a victim into clicking it. Once the victim visits the page, the injected JavaScript executes in the context of the vulnerable application, triggering an `alert(1)` popup.

## Vulnerability Type

**Cross-Site Scripting (XSS) - Reflected**

## Vulnerable Parameter

```text
cat
```

## Vulnerable URL

```text
https://kzlabs.in/12.php?cat='>anil<IMG SRc=x OnErrOR=u0061\u006c\u0065\u0072\u0074(1)>
```

## Steps to Reproduce

1. Open the vulnerable page.

2. Replace the value of the `cat` parameter with the following payload:

   ```text
   '>anil<IMG SRc=x OnErrOR=u0061\u006c\u0065\u0072\u0074(1)>
   ```

3. Visit the crafted URL:

   ```text
   https://kzlabs.in/12.php?cat=%27%3Eanil%3CIMG%20SRc%3Dx%20OnErrOR%3Du0061%5Cu006c%5Cu0065%5Cu0072%5Cu0074%281%29%3E
   ```

4. Observe that the browser executes the injected payload and displays an `alert(1)` popup.

## Payload Used

```text
'>anil<IMG SRc=x OnErrOR=u0061\u006c\u0065\u0072\u0074(1)>
```

URL-encoded version:
```text
%27%3Eanil%3CIMG%20SRc%3Dx%20OnErrOR%3Du0061%5Cu006c%5Cu0065%5Cu0072%5Cu0074%281%29%3E
```

## Proof of Concept

The application accepts the value of the `cat` parameter and reflects it directly into the page HTML without proper sanitization.

The payload works by:
1. Breaking out of the existing HTML context using `'>`
2. Injecting an `<IMG>` tag with mixed-case formatting to evade case-sensitive filters (`SRc`, `OnErrOR`)
3. Using Unicode escape sequences to obfuscate the function name:
   - `\u0061` = 'a'
   - `\u006c` = 'l'
   - `\u0065` = 'e'
   - `\u0072` = 'r'
   - `\u0074` = 't'
4. When the browser decodes the string, `u0061\u006c\u0065\u0072\u0074` becomes `alert`
5. The `OnErrOR` event triggers when the image fails to load (due to `SRc=x`, where `x` is an invalid source)
6. The `alert(1)` function executes, displaying a popup

```html
<!-- Original payload: -->
<img src="x" onerror="u0061\u006c\u0065\u0072\u0074(1)">

<!-- After Unicode decoding: -->
<img src="x" onerror="alert(1)">
```

This technique effectively bypasses:
- Case-sensitive filters (using `OnErrOR` instead of `onerror`)
- Function name blacklists (using Unicode escape sequences)
- Simple string-based filtering

<img width="1470" height="498" alt="Screenshot 2026-08-14 195121" src="https://github.com/user-attachments/assets/bcd5a9fd-0e0f-48d1-b03f-e412947c3552" />


## Impact

An attacker can craft a malicious link that, when opened by a victim, executes arbitrary JavaScript in the context of the vulnerable website. This could allow an attacker to:

- Steal session cookies and hijack user accounts
- Perform actions on behalf of the authenticated user
- Deface the webpage or manipulate its content
- Redirect users to malicious websites
- Capture keystrokes or sensitive information entered by the user
- Bypass multiple filtering mechanisms (case-sensitive, function name blacklists, encoding filters)

## Recommendation

To mitigate this vulnerability:

- **Output Encoding**: Use `htmlspecialchars()` (or an equivalent output-encoding function) before rendering user-controlled data in HTML to convert special characters to their HTML entities.
  ```php
  echo htmlspecialchars($_GET['cat'], ENT_QUOTES, 'UTF-8');
  ```

- **Normalize Input**: Before filtering, normalize Unicode escape sequences and other encoding variants.
  ```php
  $input = json_decode('"' . addslashes($_GET['cat']) . '"');
  ```

- **Case-Insensitive Filtering**: If implementing input filtering, ensure it is case-insensitive by converting all input to lowercase or using case-insensitive pattern matching.

- **Content Security Policy (CSP)**: Implement a strict CSP that restricts inline scripts, event handlers, and unauthorized script sources using `unsafe-inline` and `unsafe-eval` directives.

- **Context-Aware Escaping**: Use context-aware escaping based on where the data is placed (HTML body, attributes, JavaScript, etc.).

- **Web Application Firewall (WAF)**: Deploy a WAF, such as Cloudflare WAF, to help detect and block malicious XSS payloads.

- **Security Headers**: Implement security headers like `X-XSS-Protection` and `X-Content-Type-Options` for additional protection.

- **HTML Sanitization**: Use a robust HTML sanitization library (like DOMPurify or HTML Purifier) to strip dangerous tags and attributes.

- **Avoid Direct Reflection**: Consider avoiding the direct reflection of user input in the response. Use server-side validation and sanitization before output.

---
