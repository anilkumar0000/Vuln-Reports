# Stored Cross-Site Scripting (XSS) - User Comments System

## Summary

I identified a Stored Cross-Site Scripting (XSS) vulnerability in the comment submission functionality of the application.

The application accepts user-controlled input through the comment form and stores it in the database without proper sanitization or output encoding. When other users view the comments section, the injected malicious code is rendered and executed in their browsers. By injecting HTML/JavaScript payloads using image tags with event handlers and script tags, arbitrary code execution is achieved.

An attacker can submit a malicious comment that, when viewed by other users, executes arbitrary JavaScript in the context of the vulnerable application. The injected payload is persistently stored and affects all users who view the comments section.

## Vulnerability Type

**Cross-Site Scripting (XSS) - Stored (Persistent)**

## Vulnerable Parameter

```text
Comment text input field
```

## Vulnerable URL

```text
https://kzlabs.in/18.php
```

## Steps to Reproduce

1. Navigate to the vulnerable page at `https://kzlabs.in/18.php`.

2. Locate the comment submission form.

3. Enter a malicious payload in the comment field:

   ```text
   ">anil<IMg Src=x OnErRor=confirm(1)>
   ```

4. Submit the comment.

5. Observe that the comment is stored and displayed in the comments section.

6. The browser executes the injected payload and displays a `confirm(1)` popup.

## Payloads Used

```text
">anil<IMg Src=x OnErRor=confirm(1)>
">anil<IMg Src=x OnErRor=confirm(2)>
tadz1"> <sCrlPT>prompt(1)</sCrlPT>
tadz2"> <sCrlPT>prompt(2)</sCrlPT>
```

## Proof of Concept

The application accepts comment submissions and stores them in a database without proper sanitization. When the comments are displayed, the user-controlled input is rendered directly into the page HTML.

The payload works by:
1. Breaking out of the existing HTML context using `">`
2. Injecting an `<IMg>` tag with mixed-case formatting to evade case-sensitive filters
3. Using the `OnErRor` event handler (mixed-case variation of `onerror`)
4. The `OnErRor` event triggers when the image fails to load (due to `Src=x`, where `x` is an invalid source)
5. This executes the JavaScript `confirm(1)` or `prompt(1)`

```html
<!-- Injected payload rendered in the page: -->
"><img src="x" onerror="confirm(1)">
"><img src="x" onerror="confirm(2)">
"><script>prompt(1)</script>
```

The payload is persistently stored and executed every time a user views the comments page.

<img width="1654" height="582" alt="Screenshot 2026-08-14 200529" src="https://github.com/user-attachments/assets/31272846-e329-4ac2-8f2c-f7af5d94a388" />

<img width="1425" height="434" alt="Screenshot 2026-08-14 200418" src="https://github.com/user-attachments/assets/96e0adfc-3188-4a72-ba79-872ca41cd744" />

## Impact

A Stored XSS vulnerability is more severe than Reflected XSS because:

- **Persistent**: The malicious payload is permanently stored in the database
- **Widespread**: All users viewing the comments section are affected
- **No User Interaction Required**: Unlike reflected XSS, victims don't need to click a malicious link
- **Potential for Worm**: The payload can self-propagate if crafted to spread through comments

An attacker can:

- Steal session cookies and hijack user accounts of all viewers
- Perform actions on behalf of authenticated users
- Deface the webpage or manipulate its content for all visitors
- Redirect users to malicious websites
- Capture keystrokes or sensitive information entered by users
- Create a self-propagating worm that spreads through the comment system
- Conduct phishing attacks within the trusted domain

## Recommendation

To mitigate this vulnerability:

- **Output Encoding**: Use `htmlspecialchars()` (or an equivalent output-encoding function) before rendering user-controlled data in HTML to convert special characters to their HTML entities.
  ```php
  echo htmlspecialchars($comment, ENT_QUOTES, 'UTF-8');
  ```

- **Input Validation**: Validate and sanitize all user inputs before storing in the database. Consider using a whitelist approach to allow only safe characters.

- **HTML Sanitization**: Use a robust HTML sanitization library (like DOMPurify or HTML Purifier) to strip dangerous tags and attributes while preserving safe content.
  ```php
  $clean_comment = HTMLPurifier::sanitize($_POST['comment']);
  ```

- **Content Security Policy (CSP)**: Implement a strict CSP that restricts inline scripts, event handlers, and unauthorized script sources using `unsafe-inline` and `unsafe-eval` directives.

- **Context-Aware Escaping**: Use context-aware escaping based on where the data is placed (HTML body, attributes, JavaScript, etc.).

- **Web Application Firewall (WAF)**: Deploy a WAF, such as Cloudflare WAF, to help detect and block malicious XSS payloads.

- **Security Headers**: Implement security headers like `X-XSS-Protection` and `X-Content-Type-Options` for additional protection.

- **Case-Insensitive Filtering**: If implementing input filtering, ensure it is case-insensitive by converting all input to lowercase or using case-insensitive pattern matching.

- **Regular Security Audits**: Conduct regular security assessments and code reviews to identify and fix vulnerabilities.

---
