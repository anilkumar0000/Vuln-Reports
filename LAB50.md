# Stored Cross-Site Scripting (XSS) - Site Settings Management

## Summary

I identified a Stored Cross-Site Scripting (XSS) vulnerability in the Site Settings Management panel of the application.

The application accepts user-controlled input through multiple configuration fields (Site Title, Site Description, Welcome Message, Footer Text) without proper sanitization or output encoding. When administrators or users view the site settings, the injected malicious code is rendered and executed in their browsers. By injecting `<ScriPt>` tags with mixed-case formatting and JavaScript payloads, arbitrary code execution is achieved despite case-sensitive filtering mechanisms.

An attacker with access to the settings panel can inject malicious payloads into configuration fields that, when viewed by other administrators or users, execute arbitrary JavaScript in the context of the vulnerable application. The injected payloads are persistently stored in the database and affect all users who view the affected settings page.

## Vulnerability Type

**Cross-Site Scripting (XSS) - Stored (Persistent)**

## Vulnerable Parameters

```text
Site Title
Site Description
Welcome Message
Footer Text
```

## Vulnerable URL

```text
https://kzlabs.in/22.php
```

## Steps to Reproduce

1. Navigate to the vulnerable site settings management page at `https://kzlabs.in/22.php`.

2. Modify the configuration settings by injecting payloads into the following fields:
   - Site Title
   - Site Description
   - Welcome Message
   - Footer Text

3. Save the settings.

4. Observe that the settings are updated and displayed.

5. The browser executes the injected payload and displays an `alert(2)` popup.

## Payloads Used

```text
Site Title: ">anil<ScriPt>alert(2)</ScriPt>
Site Description: ">anil<ScriPt>alert(2)</ScriPt>
Welcome Message: ">anil<ScriPt>alert(2)</ScriPt>
Footer Text: ">anil<ScriPt>alert(2)</ScriPt>
```

URL-encoded versions:
```text
Site Title: %22%3Eanil%3CScriPt%3Ealert%282%29%3C%2FScriPt%3E
Site Description: %22%3Eanil%3CScriPt%3Ealert%282%29%3C%2FScriPt%3E
Welcome Message: %22%3Eanil%3CScriPt%3Ealert%282%29%3C%2FScriPt%3E
Footer Text: %22%3Eanil%3CScriPt%3Ealert%282%29%3C%2FScriPt%3E
```

## Proof of Concept

The application accepts site configuration settings and stores them in a database without proper sanitization. When settings are displayed, the user-controlled input is rendered directly into the page HTML.

The payload works by:
1. Breaking out of the existing HTML context using `">`
2. Injecting `<ScriPt>` tags with mixed-case formatting to evade case-sensitive filters
3. Including the JavaScript `alert(2)` payload
4. The browser parses and executes the injected script tag regardless of case

```html
<!-- Injected payload rendered in the page: -->
"><script>alert(2)</script>
```

The payload is persistently stored and executed every time a user views the site settings page.

<img width="951" height="804" alt="Screenshot 2026-08-14 201115" src="https://github.com/user-attachments/assets/a22075d4-19a2-4661-91a9-96e0b660c1de" />

<img width="1574" height="526" alt="Screenshot 2026-08-14 201234" src="https://github.com/user-attachments/assets/d6e6b3a9-bddf-459d-811a-ee1c2eb450a4" />

## Impact

Stored XSS vulnerabilities in site settings management are extremely critical because:

- **Persistent**: Malicious payloads are permanently stored in the database
- **High Privilege Target**: Administrators with elevated privileges view the settings
- **No User Interaction Required**: Victims don't need to click a malicious link
- **Full Application Control**: Site settings affect the entire application
- **Widespread Impact**: All users viewing the application may be affected
- **Multiple Injection Points**: Four vulnerable fields provide multiple attack vectors

An attacker with access can:

- Steal session cookies and hijack admin accounts
- Perform actions with administrative privileges
- Access and modify sensitive configuration data
- Deface the entire website or manipulate content
- Redirect all users to malicious websites
- Capture keystrokes and sensitive information
- Create backdoors in the application
- Install malware or persistent threats
- Exfiltrate sensitive data from the database
- Bypass case-sensitive filtering mechanisms

## Recommendation

To mitigate this vulnerability:

- **Output Encoding**: Use `htmlspecialchars()` (or equivalent) before rendering user-controlled data in HTML for ALL fields.
  ```php
  echo htmlspecialchars($siteTitle, ENT_QUOTES, 'UTF-8');
  echo htmlspecialchars($siteDescription, ENT_QUOTES, 'UTF-8');
  echo htmlspecialchars($welcomeMessage, ENT_QUOTES, 'UTF-8');
  echo htmlspecialchars($footerText, ENT_QUOTES, 'UTF-8');
  ```

- **Case-Insensitive Filtering**: If implementing input filtering, ensure it is case-insensitive by converting all input to lowercase or using case-insensitive pattern matching.
  ```php
  $input = strtolower($_POST['siteTitle']);
  if (strpos($input, '<script>') !== false) {
      // Block or sanitize
  }
  ```

- **Input Validation**: Strictly validate and sanitize all user inputs before storing. Use a whitelist approach to allow only safe characters.

- **HTML Sanitization**: Use robust sanitization libraries (DOMPurify, HTML Purifier) to strip dangerous tags and attributes.
  ```php
  $clean_title = HTMLPurifier::sanitize($_POST['siteTitle']);
  $clean_description = HTMLPurifier::sanitize($_POST['siteDescription']);
  ```

- **Content Security Policy (CSP)**: Implement a strict CSP restricting inline scripts and event handlers.

- **Context-Aware Escaping**: Use appropriate escaping based on where data is placed (HTML body, attributes, JavaScript).

- **Web Application Firewall (WAF)**: Deploy a WAF to detect and block malicious XSS payloads.

- **Security Headers**: Implement `X-XSS-Protection`, `X-Content-Type-Options`, and other security headers.

- **Admin Access Control**: Limit settings panel access to trusted users only and implement multi-factor authentication.

- **Input Length Limits**: Enforce reasonable length limits on configuration fields.

- **Regular Security Audits**: Conduct regular assessments and code reviews to identify vulnerabilities.

- **Audit Logging**: Implement logging of all admin actions to detect suspicious activities.

---
