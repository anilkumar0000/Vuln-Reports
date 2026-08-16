# Stored Cross-Site Scripting (XSS) - Quill Blog Platform

## Summary

I identified a Stored Cross-Site Scripting (XSS) vulnerability in the Quill blog platform's article creation functionality.

The application allows users to write and publish articles with support for raw HTML input. The application accepts user-controlled input through the **Body** field in HTML mode and stores it in the database without proper sanitization or output encoding. When other users view the published articles, the injected malicious code is rendered and executed in their browsers. By injecting an `<IMG>` tag with an `OnError` event handler, arbitrary code execution is achieved.

An attacker can create an article containing malicious HTML that, when viewed by other users, executes arbitrary JavaScript in the context of the vulnerable application. The injected payload is persistently stored in the database and affects all users who view the article.

## Vulnerability Type

**Cross-Site Scripting (XSS) - Stored (Persistent)**

## Vulnerable Parameters

```text
Title
Tags
Body (HTML content)
```

## Vulnerable URL

```text
https://kzlabs.in/61.php
```

## Steps to Reproduce

1. Navigate to the vulnerable Quill platform at `https://kzlabs.in/61.php`.

2. Click the **Write article** button to create a new article.

3. Switch to the **HTML** tab in the Body editor.

4. Enter the following payload in the **Body** field:

   ```text
   "">anil<IMG SRC=x OnError=confirm(100)">
   ```

5. Set the **Status** to **Published**.

6. Click the **Publish** button to save the article.

7. Navigate to the **My Articles** page where the published article is displayed.

8. Observe that the browser executes the injected payload and displays a `confirm(100)` popup.

## Payloads Used

```text
Body: "">anil<IMG SRC=x OnError=confirm(100)">
Body: "">anil<IMG SRC=x OnError=confirm(200)">
```

URL-encoded versions:
```text
%22%22%3Eanil%3CIMG%20SRC%3Dx%20OnError%3Dconfirm%28100%29%22%3E
%22%22%3Eanil%3CIMG%20SRC%3Dx%20OnError%3Dconfirm%28200%29%22%3E
```

## Proof of Concept

The Quill platform allows users to write articles with raw HTML support. The application accepts HTML input in the Body field and stores it in the database without proper sanitization. When articles are displayed, the HTML content is rendered directly into the page using `innerHTML`.

The payload works by:
1. Breaking out of the existing HTML context using `">`
2. Injecting an `<IMG>` tag with an `OnError` event handler (mixed-case variation of `onerror`)
3. The `OnError` event triggers when the image fails to load (due to `SRC=x`, where `x` is an invalid source)
4. This executes the JavaScript `confirm(100)` and `confirm(200)`, displaying popups

```html
<!-- Injected payload rendered in the page: -->
"><img src="x" onerror="confirm(100)">
"><img src="x" onerror="confirm(200)">
```

The payload is persistently stored in the database and executed every time a user views the article.

<img width="1598" height="1079" alt="Screenshot 2026-08-14 204351" src="https://github.com/user-attachments/assets/ac98f383-2fe9-4eab-8a7a-37b558edb707" />

<img width="1427" height="415" alt="Screenshot 2026-08-14 204409" src="https://github.com/user-attachments/assets/135c2764-7b0c-46e2-8e39-acf504f90cd0" />

## Impact

Stored XSS vulnerabilities in blogging platforms are highly critical because:

- **Persistent**: Malicious payloads are permanently stored in the database
- **Widespread Impact**: All users viewing the article are affected
- **No User Interaction Required**: Victims don't need to click a malicious link
- **High Visibility**: Articles are often the most viewed content on the platform
- **Trusted Context**: Articles are rendered as trusted content by the browser

An attacker can:

- Steal session cookies and hijack user accounts of all readers
- Perform actions on behalf of authenticated users
- Deface articles or manipulate content for all visitors
- Redirect users to malicious websites
- Capture keystrokes or sensitive information
- Create self-propagating worms through comments or articles
- Conduct phishing attacks within the trusted domain
- Escalate privileges by targeting administrators reviewing articles

## Recommendation

To mitigate this vulnerability:

- **Output Encoding**: Use `htmlspecialchars()` (or an equivalent output-encoding function) before rendering user-controlled data in HTML to convert special characters to their HTML entities.
  ```php
  echo htmlspecialchars($body, ENT_QUOTES, 'UTF-8');
  ```

- **HTML Sanitization**: Use a robust HTML sanitization library (like DOMPurify, HTML Purifier, or Sanitize-html) to strip dangerous tags and attributes while preserving safe content.
  ```php
  $clean_body = HTMLPurifier::sanitize($_POST['body']);
  ```

- **Content Security Policy (CSP)**: Implement a strict CSP that restricts inline scripts, event handlers, and unauthorized script sources.

- **Context-Aware Escaping**: Use context-aware escaping based on where the data is placed (HTML body, attributes, JavaScript, etc.).

- **Input Validation**: Validate and sanitize all user inputs before storing. Consider using a whitelist approach to allow only safe content.

- **Web Application Firewall (WAF)**: Deploy a WAF to detect and block malicious XSS payloads.

- **Security Headers**: Implement security headers like `X-XSS-Protection` and `X-Content-Type-Options` for additional protection.

- **Case-Insensitive Filtering**: Use case-insensitive pattern matching to filter malicious content.

- **Regular Security Audits**: Conduct regular assessments and code reviews to identify vulnerabilities.

- **User Education**: Educate users about the risks of pasting untrusted HTML content.

---
