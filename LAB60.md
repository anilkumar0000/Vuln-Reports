# Stored Cross-Site Scripting (XSS) - AdPulse Network Reports

## Summary

I identified a Stored Cross-Site Scripting (XSS) vulnerability in the Network Reports functionality of the AdPulse application.

The application accepts user-controlled input through the **Report Name** field when creating a new network report. This input is stored in the database and later displayed on the reports list page without proper sanitization or output encoding. By injecting a malicious payload using an `<img>` tag with an `onerror` event handler, arbitrary code execution is achieved when any authenticated user views the reports list.

An attacker can create a network report with a malicious name containing JavaScript that, when viewed by other authenticated users, executes arbitrary JavaScript in the context of the vulnerable application. The injected payload is persistently stored and affects all users with access to the reports list.

## Vulnerability Type

**Cross-Site Scripting (XSS) - Stored (Persistent)**

## Vulnerable Parameter

```text
Report Name
```

## Vulnerable URL

```text
https://kzlabs.in/60.php
```

## Steps to Reproduce

1. Navigate to the vulnerable page at `https://kzlabs.in/60.php`.

2. Click the **+ New Network Report** button to create a new report.

3. Enter the following payload in the **Report Name** field:

   ```text
   ">anil<img Src=x ONeRroR=confirm(100)>
   ```

4. Submit the form to save the network report.

5. Navigate back to the list of network reports where the saved report is displayed.

6. Observe that the browser executes the injected payload and displays a `confirm(100)` popup.

## Payload Used

```text
">anil<img Src=x ONeRroR=confirm(100)>
```

URL-encoded version:
```text
%22%3Eanil%3Cimg%20Src%3Dx%20ONeRroR%3Dconfirm%28100%29%3E
```

## Proof of Concept

The application accepts the value of the Report Name field and stores it in a database without proper sanitization. When the reports list is loaded, the user-controlled input is rendered directly into the page HTML.

The payload works by:
1. Breaking out of the existing HTML context using `">`
2. Injecting an `<img>` tag with an `ONeRroR` event handler (mixed-case variation of `onerror`)
3. The `ONeRroR` event triggers when the image fails to load (due to `Src=x`, where `x` is an invalid source)
4. This executes the JavaScript `confirm(100)`, displaying a popup with the number 100

```html
<!-- Injected payload rendered in the page: -->
"><img src="x" onerror="confirm(100)">
```

The payload is persistently stored in the database and executed every time a user views the network reports list.

<img width="1607" height="942" alt="Screenshot 2026-08-14 204203" src="https://github.com/user-attachments/assets/602f49fa-079c-4b0d-b3b9-fc482dd3e0a0" />

<img width="1362" height="372" alt="Screenshot 2026-08-14 204122" src="https://github.com/user-attachments/assets/7e386aa1-6392-4b30-9c27-9c0d7df2ef50" />

## Impact

Stored XSS vulnerabilities in reporting and admin panels are highly critical because:

- **Persistent**: Malicious payloads are permanently stored in the database
- **Widespread Impact**: All authenticated users viewing the reports list are affected
- **No User Interaction Required**: Victims don't need to click a malicious link
- **Sensitive Context**: Reports often contain sensitive business data
- **Elevated Privilege Target**: Report management features are often used by administrators

An attacker can:

- Steal session cookies and hijack user accounts of all viewers
- Perform actions on behalf of authenticated users
- Access and exfiltrate sensitive report data
- Deface the reports dashboard
- Redirect users to malicious websites
- Capture keystrokes or sensitive information
- Escalate privileges by targeting administrators viewing reports

## Recommendation

To mitigate this vulnerability:

- **Output Encoding**: Use `htmlspecialchars()` (or an equivalent output-encoding function) before rendering user-controlled data in HTML to convert special characters to their HTML entities.
  ```php
  echo htmlspecialchars($reportName, ENT_QUOTES, 'UTF-8');
  ```

- **Input Validation**: Validate and sanitize all user inputs before storing in the database. Consider using a whitelist approach to allow only safe characters.
  ```php
  $reportName = preg_replace('/[^a-zA-Z0-9\s\-_]/', '', $_POST['reportName']);
  ```

- **HTML Sanitization**: Use a robust HTML sanitization library (like DOMPurify or HTML Purifier) to strip dangerous tags and attributes.

- **Content Security Policy (CSP)**: Implement a strict CSP that restricts inline scripts, event handlers, and unauthorized script sources.

- **Context-Aware Escaping**: Use context-aware escaping based on where the data is placed (HTML body, attributes, JavaScript, etc.).

- **Web Application Firewall (WAF)**: Deploy a WAF, such as Cloudflare WAF, to help detect and block malicious XSS payloads.

- **Security Headers**: Implement security headers like `X-XSS-Protection` and `X-Content-Type-Options` for additional protection.

- **Case-Insensitive Filtering**: Use case-insensitive pattern matching to filter malicious content.

- **Regular Security Audits**: Conduct regular assessments and code reviews to identify vulnerabilities.

- **Input Length Limits**: Enforce reasonable length limits on input fields to prevent excessive payload injection.

---
