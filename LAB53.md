# POST XSS in Document Title Context

## Summary

I identified a POST-Based Cross-Site Scripting (XSS) vulnerability in the application where user input is reflected in the document title context.

The application accepts user-controlled input through the `Name` POST parameter and reflects it directly into the page title without proper sanitization or output encoding. By injecting a `</title>` tag to break out of the title context and inserting a `<SCrlPt>` tag with mixed-case formatting, arbitrary code execution is achieved when the victim submits the form.

An attacker can craft a malicious form or trick a victim into submitting the vulnerable form with a malicious payload. Once the victim submits the form, the injected JavaScript executes in the context of the vulnerable application, triggering an `alert(1)` popup.

## Vulnerability Type

**Cross-Site Scripting (XSS) - Reflected (POST-based)**

## Vulnerable Parameter

```text
Name
```

## Vulnerable URL

```text
https://kzlabs.in/53.php
```

## Steps to Reproduce

1. Navigate to the vulnerable page at `https://kzlabs.in/53.php`.

2. Enter the following payload in the **Name** field:

   ```text
   ">anil</title><SCrlPt>alert(1)</SCrlPt>
   ```

3. Click the **Enter** button to submit the form via POST.

4. Observe that the browser executes the injected payload and displays an `alert(1)` popup.

## Payload Used

```text
">anil</title><SCrlPt>alert(1)</SCrlPt>
```

URL-encoded version:
```text
%22%3Eanil%3C%2Ftitle%3E%3CSCrlPt%3Ealert%281%29%3C%2FSCrlPt%3E
```

## Proof of Concept

The application accepts the value of the `Name` POST parameter and reflects it directly into the page title without proper sanitization.

The payload works by:
1. Breaking out of the existing HTML context using `">`
2. Closing the `<title>` tag using `</title>`
3. Injecting a `<SCrlPt>` tag with mixed-case formatting to evade case-sensitive filters
4. The browser parses and executes the injected script tag regardless of case
5. The `alert(1)` function executes, displaying a popup

```html
<!-- Original HTML structure: -->
<title>Welcome [USER INPUT]</title>

<!-- After injection with payload: -->
<title>Welcome ">anil</title><script>alert(1)</script>
```

The `</title>` tag closes the title element, and the `<script>` tag is parsed as HTML outside the title context. This technique is effective when the application reflects user input within the `<title>` tag.

This confirms that arbitrary JavaScript execution is possible through the `Name` POST parameter in the document title context.

<img width="1509" height="523" alt="Screenshot 2026-08-14 201711" src="https://github.com/user-attachments/assets/dee1298b-71e8-46f0-b8c3-bdb8dc987086" />

<img width="1380" height="384" alt="Screenshot 2026-08-14 201720" src="https://github.com/user-attachments/assets/6df512d1-5392-4880-a597-0389027a6c9d" />


## Impact

An attacker can craft a malicious form or use social engineering to trick a victim into submitting the vulnerable form. This could allow an attacker to:

- Steal session cookies and hijack user accounts
- Perform actions on behalf of the authenticated user
- Deface the webpage or manipulate its content
- Redirect users to malicious websites
- Capture keystrokes or sensitive information entered by the user
- Bypass case-sensitive filtering mechanisms
- Bypass HTML context restrictions

## Recommendation

To mitigate this vulnerability:

- **Output Encoding**: Use `htmlspecialchars()` (or an equivalent output-encoding function) before rendering user-controlled data in the HTML title to convert special characters to their HTML entities.
  ```php
  $name = htmlspecialchars($_POST['name'], ENT_QUOTES, 'UTF-8');
  echo '<title>Welcome ' . $name . '</title>';
  ```

- **Input Validation**: Validate and sanitize all user inputs, even those received via POST. Consider using a whitelist approach to allow only safe characters.
  ```php
  $name = filter_input(INPUT_POST, 'name', FILTER_SANITIZE_STRING);
  ```

- **Case-Insensitive Filtering**: If implementing input filtering, ensure it is case-insensitive by converting all input to lowercase or using case-insensitive pattern matching.

- **Content Security Policy (CSP)**: Implement a strict CSP that restricts inline scripts and unauthorized script sources using `unsafe-inline` and `unsafe-eval` directives.

- **Context-Aware Escaping**: Use appropriate escaping based on where the data is placed (HTML body, attributes, title, JavaScript, etc.). Different contexts require different escaping strategies.

- **Web Application Firewall (WAF)**: Deploy a WAF, such as Cloudflare WAF, to help detect and block malicious XSS payloads.

- **Security Headers**: Implement security headers like `X-XSS-Protection` and `X-Content-Type-Options` for additional protection.

- **HTML Sanitization**: Use a robust HTML sanitization library (like DOMPurify or HTML Purifier) to strip dangerous tags and attributes regardless of case.

- **CSRF Protection**: Implement CSRF tokens to prevent attackers from forging form submissions.

- **Avoid Direct Reflection**: Consider avoiding the direct reflection of user input in the response. Use server-side validation and sanitization before output.

---
