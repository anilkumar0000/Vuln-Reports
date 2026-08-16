# Stored Cross-Site Scripting (XSS) - User Profile Management & Comments System

## Summary

I identified multiple Stored Cross-Site Scripting (XSS) vulnerabilities in the application's User Profile Management and Comments System features.

The application accepts user-controlled input through profile fields (Full Name, Bio) and comment forms without proper sanitization or output encoding. When other users view profiles or comments, the injected malicious code is rendered and executed in their browsers. By injecting HTML/JavaScript payloads using image tags with event handlers and script tags, arbitrary code execution is achieved.

An attacker can update their profile or submit comments containing malicious payloads that, when viewed by other users, execute arbitrary JavaScript in the context of the vulnerable application. The injected payloads are persistently stored in the database and affect all users who view the profiles or comments.

## Vulnerability Type

**Cross-Site Scripting (XSS) - Stored (Persistent)**

## Vulnerable Parameters

```text
Full Name (profile)
Bio (profile)
Comment text input field
```

## Vulnerable URLs

```text
https://kzlabs.in/19.php (User Profile Management)
https://kzlabs.in/18.php (Comments System)
```

## Steps to Reproduce

### Profile Management XSS

1. Navigate to the vulnerable profile page at `https://kzlabs.in/19.php`.

2. Update the **Full Name** field with the following payload:

   ```text
   ">anil<IMg Src=x OnErRor=confirm(2)>
   ```

3. Update the **Bio** field with the following payload:

   ```text
   ">anil<IMg Src=x OnErRor=confirm(3)>
   ```

4. Submit the profile update.

5. Observe that the payloads are stored and reflected in the User Directory section.

6. The browser executes the injected payloads and displays `confirm(2)` and `confirm(3)` popups.

### Comments System XSS

1. Navigate to the vulnerable page at `https://kzlabs.in/18.php`.

2. Enter a malicious payload in the comment field.

3. Submit the comment.

4. Observe that the comment is stored and displayed in the comments section.

5. The browser executes the injected payload and displays popups.

## Payloads Used

```text
" >anil<IMg Src=x OnErRor=confirm(1)>
" >anil<IMg Src=x OnErRor=confirm(2)>
" >anil<IMg Src=x OnErRor=confirm(3)>
tadz1"> <sCrlPT>prompt(1)</sCrlPT>
tadz2"> <sCrlPT>prompt(2)</sCrlPT>
```

## Proof of Concept

The application accepts user input and stores it in a database without proper sanitization. When displayed, the user-controlled input is rendered directly into the page HTML.

The payload works by:
1. Breaking out of the existing HTML context using `">`
2. Injecting `<IMg>` tags with mixed-case formatting to evade case-sensitive filters
3. Using `OnErRor` event handlers (mixed-case variation of `onerror`)
4. The `OnErRor` events trigger when images fail to load (due to `Src=x`, where `x` is an invalid source)
5. This executes JavaScript functions like `confirm()` and `prompt()`

```html
<!-- Injected payload rendered in the page: -->
"><img src="x" onerror="confirm(2)">
"><img src="x" onerror="confirm(3)">
"><script>prompt(1)</script>
```

The payloads are persistently stored and executed every time a user views the profile page or comments section.

<img width="1654" height="582" alt="Screenshot 2026-08-14 200529" src="https://github.com/user-attachments/assets/50158903-e111-498a-aba5-f2536e12a5cc" />


<img width="1919" height="980" alt="Screenshot 2026-08-14 200703" src="https://github.com/user-attachments/assets/23cb9585-0ffc-4796-a0b4-18dba504f72a" />


## Impact

Stored XSS vulnerabilities are more severe than Reflected XSS because:

- **Persistent**: Malicious payloads are permanently stored in the database
- **Widespread**: All users viewing profiles or comments are affected
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
- Escalate privileges by targeting administrators viewing profiles

## Recommendation

To mitigate this vulnerability:

- **Output Encoding**: Use `htmlspecialchars()` (or equivalent) before rendering user-controlled data in HTML.
  ```php
  echo htmlspecialchars($fullName, ENT_QUOTES, 'UTF-8');
  echo htmlspecialchars($bio, ENT_QUOTES, 'UTF-8');
  echo htmlspecialchars($comment, ENT_QUOTES, 'UTF-8');
  ```

- **Input Validation**: Validate and sanitize all user inputs before storing. Use a whitelist approach to allow only safe characters.

- **HTML Sanitization**: Use robust sanitization libraries (DOMPurify, HTML Purifier) to strip dangerous tags and attributes.
  ```php
  $clean_bio = HTMLPurifier::sanitize($_POST['bio']);
  ```

- **Content Security Policy (CSP)**: Implement a strict CSP restricting inline scripts and event handlers.

- **Context-Aware Escaping**: Use appropriate escaping based on where data is placed (HTML body, attributes, JavaScript).

- **Web Application Firewall (WAF)**: Deploy a WAF to detect and block malicious XSS payloads.

- **Security Headers**: Implement `X-XSS-Protection`, `X-Content-Type-Options`, and other security headers.

- **Case-Insensitive Filtering**: Use case-insensitive pattern matching to filter malicious content.

- **Regular Security Audits**: Conduct regular assessments and code reviews to identify vulnerabilities.

---
