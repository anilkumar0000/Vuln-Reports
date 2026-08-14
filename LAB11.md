# Reflected Cross-Site Scripting (XSS) - Category ID Parameter Injection

## Summary

I identified a Reflected Cross-Site Scripting (XSS) vulnerability in the `categoryid` parameter of the application.

The application accepts user-controlled input through the `categoryid` GET parameter and reflects it back to the page without proper sanitization or output encoding. By injecting a `<ScRiPt>` tag with mixed-case formatting and using Unicode escape sequences (`\u0074`) to obfuscate function names, arbitrary code execution is achieved despite function name filtering mechanisms.

An attacker can craft a malicious URL and trick a victim into clicking it. Once the victim visits the page, the injected JavaScript executes in the context of the vulnerable application, triggering an `alert(1)` popup.

## Vulnerability Type

**Cross-Site Scripting (XSS) - Reflected**

## Vulnerable Parameter

```text
categoryid
```

## Vulnerable URL

```text
https://kzlabs.in/11.php?categoryid='>anil<ScRiPt>aler\u0074(1)</ScRiPt>
```

## Steps to Reproduce

1. Open the vulnerable page.

2. Replace the value of the `categoryid` parameter with the following payload:

   ```text
   '>anil<ScRiPt>aler\u0074(1)</ScRiPt>
   ```

3. Visit the crafted URL:

   ```text
   https://kzlabs.in/11.php?categoryid=%27%3Eanil%3CScRiPt%3Ealer%5Cu0074%281%29%3C%2FScRiPt%3E
   ```

4. Observe that the browser executes the injected payload and displays an `alert(1)` popup.

## Payload Used

```text
'>anil<ScRiPt>aler\u0074(1)</ScRiPt>
```

URL-encoded version:
```text
%27%3Eanil%3CScRiPt%3Ealer%5Cu0074%281%29%3C%2FScRiPt%3E
```

## Proof of Concept

The application accepts the value of the `categoryid` parameter and reflects it directly into the page HTML without proper sanitization.

The payload works by:
1. Breaking out of the existing HTML context using `'>`
2. Injecting a `<ScRiPt>` tag with mixed-case formatting to evade case-sensitive filters
3. Using the Unicode escape sequence `\u0074` which represents the letter 't'
4. When the browser decodes the string, `aler\u0074` becomes `alert`
5. The `alert(1)` function executes, displaying a popup

```html
<!-- Original payload: -->
<script>aler\u0074(1)</script>

<!-- After Unicode decoding: -->
<script>alert(1)</script>
```

This technique effectively bypasses blacklist-based filtering that searches for exact function names like "alert" while allowing encoded representations.

<img width="1486" height="367" alt="Screenshot 2026-08-14 191753" src="https://github.com/user-attachments/assets/ad03de99-92d7-4f28-888c-afe2cbea0888" />


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
  echo htmlspecialchars($_GET['categoryid'], ENT_QUOTES, 'UTF-8');
  ```

- **Input Validation**: Validate and sanitize all user inputs. For numeric parameters like `categoryid`, ensure the value is an integer.
  ```php
  $categoryid = filter_input(INPUT_GET, 'categoryid', FILTER_VALIDATE_INT);
  if ($categoryid === false) {
      // Reject invalid input
  }
  ```

- **Normalize Input**: Before filtering, normalize Unicode escape sequences and other encoding variants.
  ```php
  $input = json_decode('"' . addslashes($_GET['categoryid']) . '"');
  ```

- **Content Security Policy (CSP)**: Implement a strict CSP that restricts inline scripts and unauthorized script sources using `unsafe-inline` and `unsafe-eval` directives.

- **Context-Aware Escaping**: Use context-aware escaping based on where the data is placed (HTML body, attributes, JavaScript, etc.).

- **Web Application Firewall (WAF)**: Deploy a WAF, such as Cloudflare WAF, to help detect and block malicious XSS payloads.

- **Security Headers**: Implement security headers like `X-XSS-Protection` and `X-Content-Type-Options` for additional protection.

- **HTML Sanitization**: Use a robust HTML sanitization library (like DOMPurify or HTML Purifier) to strip dangerous tags and attributes.

- **Avoid Direct Reflection**: Consider avoiding the direct reflection of user input in the response. Use server-side validation and sanitization before output.

---
