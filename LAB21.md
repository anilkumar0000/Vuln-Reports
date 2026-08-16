# Stored Cross-Site Scripting (XSS) - Support Ticket System

## Summary

I identified a Stored Cross-Site Scripting (XSS) vulnerability in the Support Ticket System of the application.

The application accepts user-controlled input through the support ticket creation form without proper sanitization or output encoding. When support staff or other users view the submitted tickets, the injected malicious code is rendered and executed in their browsers. By injecting HTML/JavaScript payloads, arbitrary code execution is achieved.

An attacker can create a support ticket containing malicious payloads that, when viewed by support staff or administrators, executes arbitrary JavaScript in the context of the vulnerable application. The injected payload is persistently stored in the database and affects all users who view the ticket.

## Vulnerability Type

**Cross-Site Scripting (XSS) - Stored (Persistent)**

## Vulnerable Parameters

```text
Subject
Priority
Message / Description
```

## Vulnerable URL

```text
https://kzlabs.in/21.php
```

## Steps to Reproduce

1. Navigate to the vulnerable support ticket page at `https://kzlabs.in/21.php`.

2. Create a new support ticket by filling in the **Subject** field with a payload.

3. Select a **Priority** and fill in the **Message** field with a payload.

4. Submit the support ticket.

5. Observe that the ticket is created and displayed.

6. The browser executes the injected payload and displays a popup.

## Payloads Used

```text
Subject: <SCRIPT>alert(1)</SCRIPT>
Priority: <IMG Src=x OnErRor=confirm(1)>
Message: "><IMG Src=x OnErRor=prompt(1)>
```

URL-encoded versions:
```text
Subject: %3CSCRIPT%3Ealert%281%29%3C%2FSCRIPT%3E
Priority: %3CIMG%20Src%3Dx%20OnErRor%3Dconfirm%281%29%3E
Message: %22%3E%3CIMG%20Src%3Dx%20OnErRor%3Dprompt%281%29%3E
```

## Proof of Concept

The application accepts support ticket submissions and stores them in a database without proper sanitization. When tickets are displayed, the user-controlled input is rendered directly into the page HTML.

The payload works by:
1. Injecting `<SCRIPT>` tags with JavaScript payloads
2. Using `<IMG>` tags with `OnErRor` event handlers (mixed-case variation of `onerror`)
3. Breaking out of existing HTML context using `">`
4. The `OnErRor` event triggers when the image fails to load (due to `Src=x`, where `x` is an invalid source)
5. This executes JavaScript functions like `alert()`, `confirm()`, and `prompt()`

```html
<!-- Injected payload rendered in the page: -->
<script>alert(1)</script>
<img src="x" onerror="confirm(1)">
"><img src="x" onerror="prompt(1)">
```

The payload is persistently stored and executed every time a user views the support ticket, particularly affecting support staff and administrators.

<img width="1596" height="883" alt="Screenshot 2026-08-14 201014" src="https://github.com/user-attachments/assets/eda69ccb-b149-437d-8ce8-de5ad6977165" />


## Impact

Stored XSS vulnerabilities in support ticket systems are particularly dangerous because:

- **Persistent**: Malicious payloads are permanently stored in the database
- **Targeted Impact**: Support staff and administrators who view tickets are affected
- **No User Interaction Required**: Victims don't need to click a malicious link
- **Privilege Escalation**: Can target users with elevated privileges (admins, support staff)
- **Sensitive Data**: Support tickets often contain sensitive information

An attacker can:

- Steal session cookies and hijack accounts of support staff and administrators
- Perform actions on behalf of authenticated users with elevated privileges
- Access sensitive support ticket data
- Deface the support dashboard or manipulate content
- Redirect users to malicious websites
- Capture keystrokes or sensitive information
- Create self-propagating worms
- Conduct phishing attacks within the trusted domain

## Recommendation

To mitigate this vulnerability:

- **Output Encoding**: Use `htmlspecialchars()` (or equivalent) before rendering user-controlled data in HTML.
  ```php
  echo htmlspecialchars($subject, ENT_QUOTES, 'UTF-8');
  echo htmlspecialchars($message, ENT_QUOTES, 'UTF-8');
  ```

- **Input Validation**: Validate and sanitize all user inputs before storing. Use a whitelist approach to allow only safe characters.

- **HTML Sanitization**: Use robust sanitization libraries (DOMPurify, HTML Purifier) to strip dangerous tags and attributes.
  ```php
  $clean_message = HTMLPurifier::sanitize($_POST['message']);
  ```

- **Content Security Policy (CSP)**: Implement a strict CSP restricting inline scripts and event handlers.

- **Context-Aware Escaping**: Use appropriate escaping based on where data is placed (HTML body, attributes, JavaScript).

- **Web Application Firewall (WAF)**: Deploy a WAF to detect and block malicious XSS payloads.

- **Security Headers**: Implement `X-XSS-Protection`, `X-Content-Type-Options`, and other security headers.

- **Case-Insensitive Filtering**: Use case-insensitive pattern matching to filter malicious content.

- **Input Length Limits**: Enforce reasonable length limits on input fields to prevent excessive payload injection.

- **Regular Security Audits**: Conduct regular assessments and code reviews to identify vulnerabilities.

- **Role-Based Access Control**: Ensure support staff and administrators have appropriate access controls to mitigate impact.

---
