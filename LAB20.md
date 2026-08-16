# Stored Cross-Site Scripting (XSS) - Blog Post System

## Summary

I identified a Stored Cross-Site Scripting (XSS) vulnerability in the Blog Post System of the application.

The application accepts user-controlled input through the blog post creation form without proper sanitization or output encoding. When other users view the published blog posts, the injected malicious code is rendered and executed in their browsers. By injecting HTML/JavaScript payloads, arbitrary code execution is achieved.

An attacker can create a blog post containing malicious payloads that, when viewed by other users, executes arbitrary JavaScript in the context of the vulnerable application. The injected payload is persistently stored in the database and affects all users who view the blog post.

## Vulnerability Type

**Cross-Site Scripting (XSS) - Stored (Persistent)**

## Vulnerable Parameters

```text
Post Title
Post Content / Body
```

## Vulnerable URL

```text
https://kzlabs.in/20.php
```

## Steps to Reproduce

1. Navigate to the vulnerable blog page at `https://kzlabs.in/20.php`.

2. Create a new blog post by filling in the **Post Title** field with a payload.

3. Fill in the **Post Content** field with a payload.

4. Submit the blog post.

5. Observe that the blog post is created and displayed.

6. The browser executes the injected payload and displays a popup.

## Payloads Used

```text
Title: <SCRIPT>alert(1)</SCRIPT>
Content: <IMG Src=x OnErRor=confirm(1)>
```

URL-encoded versions:
```text
Title: %3CSCRIPT%3Ealert%281%29%3C%2FSCRIPT%3E
Content: %3CIMG%20Src%3Dx%20OnErRor%3Dconfirm%281%29%3E
```

## Proof of Concept

The application accepts blog post submissions and stores them in a database without proper sanitization. When blog posts are displayed, the user-controlled input is rendered directly into the page HTML.

The payload works by:
1. Injecting `<SCRIPT>` tags with JavaScript payloads
2. Using `<IMG>` tags with `OnErRor` event handlers (mixed-case variation of `onerror`)
3. The `OnErRor` event triggers when the image fails to load (due to `Src=x`, where `x` is an invalid source)
4. This executes JavaScript functions like `alert()` and `confirm()`

```html
<!-- Injected payload rendered in the page: -->
<script>alert(1)</script>
<img src="x" onerror="confirm(1)">
```

The payload is persistently stored and executed every time a user views the blog post.

<img width="1778" height="687" alt="Screenshot 2026-08-14 200838" src="https://github.com/user-attachments/assets/ad663660-f66d-46d9-a827-78664d0813f3" />


## Impact

Stored XSS vulnerabilities are more severe than Reflected XSS because:

- **Persistent**: Malicious payloads are permanently stored in the database
- **Widespread**: All users viewing blog posts are affected
- **No User Interaction Required**: Victims don't need to click a malicious link
- **Potential for Worm**: Payloads can self-propagate through the application

An attacker can:

- Steal session cookies and hijack user accounts of all viewers
- Perform actions on behalf of authenticated users
- Deface webpages or manipulate content for all visitors
- Redirect users to malicious websites
- Capture keystrokes or sensitive information
- Create self-propagating worms
- Conduct phishing attacks within the trusted domain
- Escalate privileges by targeting administrators viewing the blog

## Recommendation

To mitigate this vulnerability:

- **Output Encoding**: Use `htmlspecialchars()` (or equivalent) before rendering user-controlled data in HTML.
  ```php
  echo htmlspecialchars($postTitle, ENT_QUOTES, 'UTF-8');
  echo htmlspecialchars($postContent, ENT_QUOTES, 'UTF-8');
  ```

- **Input Validation**: Validate and sanitize all user inputs before storing. Use a whitelist approach to allow only safe characters.

- **HTML Sanitization**: Use robust sanitization libraries (DOMPurify, HTML Purifier) to strip dangerous tags and attributes.
  ```php
  $clean_content = HTMLPurifier::sanitize($_POST['content']);
  ```

- **Content Security Policy (CSP)**: Implement a strict CSP restricting inline scripts and event handlers.

- **Context-Aware Escaping**: Use appropriate escaping based on where data is placed (HTML body, attributes, JavaScript).

- **Web Application Firewall (WAF)**: Deploy a WAF to detect and block malicious XSS payloads.

- **Security Headers**: Implement `X-XSS-Protection`, `X-Content-Type-Options`, and other security headers.

- **Case-Insensitive Filtering**: Use case-insensitive pattern matching to filter malicious content.

- **Regular Security Audits**: Conduct regular assessments and code reviews to identify vulnerabilities.

---
