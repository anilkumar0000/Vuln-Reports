# Stored Cross-Site Scripting (XSS) - Admin Panel Settings

## Summary

I identified a Stored Cross-Site Scripting (XSS) vulnerability in the Admin Panel Settings of the application.

The application accepts user-controlled input through the admin settings configuration form without proper sanitization or output encoding. When administrators view the settings page, the injected malicious code is rendered and executed in their browsers. By injecting HTML/JavaScript payloads, arbitrary code execution is achieved.

An attacker with access to the admin panel can inject malicious payloads into configuration settings that, when viewed by other administrators or users, execute arbitrary JavaScript in the context of the vulnerable application. The injected payload is persistently stored in the database and affects all users who view the affected settings page.

## Vulnerability Type

**Cross-Site Scripting (XSS) - Stored (Persistent)**

## Vulnerable Parameters

```text
Site Name
Site Description
Footer Text
Admin Email
Other Configuration Fields
```

## Vulnerable URL

```text
https://kzlabs.in/22.php
```

## Steps to Reproduce

1. Navigate to the vulnerable admin panel settings page at `https://kzlabs.in/22.php`.

2. Modify the configuration settings by injecting payloads into fields such as Site Name, Description, or Footer Text.

3. Save the settings.

4. Observe that the settings are updated and displayed.

5. The browser executes the injected payload and displays a popup.

## Payloads Used

```text
Site Name: <SCRIPT>alert(1)</SCRIPT>
Description: <IMG Src=x OnErRor=confirm(2)>
Footer: "><IMG Src=x OnErRor=prompt(1)>
```

URL-encoded versions:
```text
Site Name: %3CSCRIPT%3Ealert%281%29%3C%2FSCRIPT%3E
Description: %3CIMG%20Src%3Dx%20OnErRor%3Dconfirm%282%29%3E
Footer: %22%3E%3CIMG%20Src%3Dx%20OnErRor%3Dprompt%281%29%3E
```

## Proof of Concept

The application accepts admin settings and stores them in a database without proper sanitization. When settings are displayed, the user-controlled input is rendered directly into the page HTML.

The payload works by:
1. Injecting `<SCRIPT>` tags with JavaScript payloads
2. Using `<IMG>` tags with `OnErRor` event handlers (mixed-case variation of `onerror`)
3. Breaking out of existing HTML context using `">`
4. The `OnErRor` event triggers when the image fails to load (due to `Src=x`, where `x` is an invalid source)
5. This executes JavaScript functions like `alert()`, `confirm()`, and `prompt()`

```html
<!-- Injected payload rendered in the page: -->
<script>alert(1)</script>
<img src="x" onerror="confirm(2)">
"><img src="x" onerror="prompt(1)">
```

The payload is persistently stored and executed every time a user views the admin panel settings page.

<img width="951" height="804" alt="Screenshot 2026-08-14 201115" src="https://github.com/user-attachments/assets/33eb5eb2-220a-451f-8efe-9cad8b89a133" />


<img width="1389" height="415" alt="Screenshot 2026-08-14 201100" src="https://github.com/user-attachments/assets/b51acf75-94b9-4471-8e16-1a0a22911e41" />


## Impact

Stored XSS vulnerabilities in admin panels are extremely critical because:

- **Persistent**: Malicious payloads are permanently stored in the database
- **High Privilege Target**: Administrators with elevated privileges view the settings
- **No User Interaction Required**: Victims don't need to click a malicious link
- **Full Application Control**: Admin settings affect the entire application
- **Widespread Impact**: All users viewing the application may be affected

An attacker with admin access can:

- Steal session cookies and hijack admin accounts
- Perform actions with administrative privileges
- Access and modify sensitive configuration data
- Deface the entire website or manipulate content
- Redirect all users to malicious websites
- Capture keystrokes and sensitive information
- Create backdoors in the application
- Install malware or persistent threats
- Exfiltrate sensitive data from the database

## Recommendation

To mitigate this vulnerability:

- **Output Encoding**: Use `htmlspecialchars()` (or equivalent) before rendering user-controlled data in HTML.
  ```php
  echo htmlspecialchars($siteName, ENT_QUOTES, 'UTF-8');
  echo htmlspecialchars($description, ENT_QUOTES, 'UTF-8');
  echo htmlspecialchars($footerText, ENT_QUOTES, 'UTF-8');
  ```

- **Input Validation**: Strictly validate and sanitize all user inputs before storing. Use a whitelist approach to allow only safe characters.

- **HTML Sanitization**: Use robust sanitization libraries (DOMPurify, HTML Purifier) to strip dangerous tags and attributes.
  ```php
  $clean_description = HTMLPurifier::sanitize($_POST['description']);
  ```

- **Content Security Policy (CSP)**: Implement a strict CSP restricting inline scripts and event handlers.

- **Context-Aware Escaping**: Use appropriate escaping based on where data is placed (HTML body, attributes, JavaScript).

- **Web Application Firewall (WAF)**: Deploy a WAF to detect and block malicious XSS payloads.

- **Security Headers**: Implement `X-XSS-Protection`, `X-Content-Type-Options`, and other security headers.

- **Case-Insensitive Filtering**: Use case-insensitive pattern matching to filter malicious content.

- **Admin Access Control**: Limit admin panel access to trusted users only and implement multi-factor authentication.

- **Input Length Limits**: Enforce reasonable length limits on configuration fields.

- **Regular Security Audits**: Conduct regular assessments and code reviews to identify vulnerabilities.

- **Audit Logging**: Implement logging of all admin actions to detect suspicious activities.

---
