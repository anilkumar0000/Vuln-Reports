# POST XSS in Input Tag Value

## Summary

I identified a POST-Based Cross-Site Scripting (XSS) vulnerability in the application where user input is reflected in the `value` attribute of an input tag.

The application accepts user-controlled input through the `Name` POST parameter and reflects it directly into the `value` attribute of an HTML input tag without proper sanitization or output encoding. By injecting a `<ScrIpt>` tag with mixed-case formatting, arbitrary code execution is achieved when the victim submits the form.

An attacker can craft a malicious form or trick a victim into submitting the vulnerable form with a malicious payload. Once the victim submits the form, the injected JavaScript executes in the context of the vulnerable application, triggering an `alert(1)` popup.

## Vulnerability Type

**Cross-Site Scripting (XSS) - Reflected (POST-based)**

## Vulnerable Parameter

```text
Name
```

## Vulnerable URL

```text
https://kzlabs.in/52.php
```

## Steps to Reproduce

1. Navigate to the vulnerable page at `https://kzlabs.in/52.php`.

2. Enter the following payload in the **Name** field:

   ```text
   ">anil<ScrIpt>alert(1)</ScrIpt>
   ```

3. Click the **Enter** button to submit the form via POST.

4. Observe that the browser executes the injected payload and displays an `alert(1)` popup.

## Payload Used

```text
">anil<ScrIpt>alert(1)</ScrIpt>
```

URL-encoded version:
```text
%22%3Eanil%3CScrIpt%3Ealert%281%29%3C%2FScrIpt%3E
```

## Proof of Concept

The application accepts the value of the `Name` POST parameter and reflects it directly into the `value` attribute of an input tag without proper sanitization.

The payload works by:
1. Breaking out of the `value` attribute context using `">`
2. Injecting a `<ScrIpt>` tag with mixed-case formatting to evade case-sensitive filters
3. The browser parses and executes the injected script tag regardless of case
4. The `alert(1)` function executes, displaying a popup

```html
<!-- Original HTML structure: -->
<input type="text" name="name" value="[USER INPUT]">

<!-- After injection with payload: -->
<input type="text" name="name" value=""><script>alert(1)</script>">
```

The `">` portion closes the `value` attribute and the input tag, allowing the injected `<script>` tag to be rendered as HTML.

This confirms that arbitrary JavaScript execution is possible through the `Name` POST parameter.

<img width="1458" height="472" alt="Screenshot 2026-08-14 201628" src="https://github.com/user-attachments/assets/61d5097a-f087-46d6-9cf4-832b30a47b74" />

<img width="1319" height="359" alt="Screenshot 2026-08-14 201621" src="https://github.com/user-attachments/assets/469fe9cc-3136-4d22-ba73-b7a6eeb7bc71" />

## Impact

An attacker can craft a malicious form or use social engineering to trick a victim into submitting the vulnerable form. This could allow an attacker to:

- Steal session cookies and hijack user accounts
- Perform actions on behalf of the authenticated user
- Deface the webpage or manipulate its content
- Redirect users to malicious websites
- Capture keystrokes or sensitive information entered by the user
- Bypass case-sensitive filtering mechanisms

## Recommendation

To mitigate this vulnerability:

- **Output Encoding**: Use `htmlspecialchars()` (or an equivalent output-encoding function) before rendering user-controlled data in HTML attributes to convert special characters to their HTML entities.
  ```php
  $name = htmlspecialchars($_POST['name'], ENT_QUOTES, 'UTF-8');
  echo '<input type="text" name="name" value="' . $name . '">';
  ```

- **Input Validation**: Validate and sanitize all user inputs, even those received via POST. Consider using a whitelist approach to allow only safe characters.
  ```php
  $name = filter_input(INPUT_POST, 'name', FILTER_SANITIZE_STRING);
  ```

- **Case-Insensitive Filtering**: If implementing input filtering, ensure it is case-insensitive by converting all input to lowercase or using case-insensitive pattern matching.

- **Content Security Policy (CSP)**: Implement a strict CSP that restricts inline scripts and unauthorized script sources using `unsafe-inline` and `unsafe-eval` directives.

- **Context-Aware Escaping**: Use appropriate escaping based on where the data is placed (HTML body, attributes, JavaScript, etc.). Attribute values require different escaping than HTML body content.

- **Web Application Firewall (WAF)**: Deploy a WAF, such as Cloudflare WAF, to help detect and block malicious XSS payloads.

- **Security Headers**: Implement security headers like `X-XSS-Protection` and `X-Content-Type-Options` for additional protection.

- **HTML Sanitization**: Use a robust HTML sanitization library (like DOMPurify or HTML Purifier) to strip dangerous tags and attributes regardless of case.

- **CSRF Protection**: Implement CSRF tokens to prevent attackers from forging form submissions.

---
