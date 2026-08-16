# DOM-Based Cross-Site Scripting (XSS) - HackerOne Careers Page

## Summary

I identified a DOM-Based Cross-Site Scripting (XSS) vulnerability in the HackerOne Careers page of the application.

The application accepts user-controlled input through the `lever` GET parameter and uses it to construct HTML strings dynamically on the client-side without proper sanitization. The `decodeURIComponent()` function is called, which decodes URL-encoded payloads before embedding them into the HTML string. This allows arbitrary JavaScript execution when the victim visits the crafted URL.

An attacker can craft a malicious URL and trick a victim into clicking it. Once the victim visits the page, the injected JavaScript executes in the context of the vulnerable application, triggering an `alert(1)` popup.

## Vulnerability Type

**Cross-Site Scripting (XSS) - DOM-Based**

## Vulnerable Parameter

```text
lever
```

## Vulnerable URL

```text
https://kzlabs.in/65.php?lever='>anil<SCrlpT>alert(1)</SCrlpT>
```

## Steps to Reproduce

1. Open the vulnerable page.

2. Replace the value of the `lever` parameter with the following payload:

   ```text
   '>anil<SCrlpT>alert(1)</SCrlpT>
   ```

3. Visit the crafted URL:

   ```text
   https://kzlabs.in/65.php?lever=%27%3Eanil%3CSCrlpT%3Ealert%281%29%3C%2FSCrlpT%3E
   ```

4. Observe that the browser executes the injected payload and displays an `alert(1)` popup.

## Payload Used

```text
'>anil<SCrlpT>alert(1)</SCrlpT>
```

URL-encoded version:
```text
%27%3Eanil%3CSCrlpT%3Ealert%281%29%3C%2FSCrlpT%3E
```

## Proof of Concept

The vulnerability is DOM-Based, meaning the malicious payload is processed entirely on the client-side rather than being reflected by the server.

The vulnerable JavaScript code:
```javascript
var link = posting.hostedUrl + leverParam;
jQuery("#jobs-container-jobs-list").append('<a href="' + link + "'>. . '</a>');
```

The payload works by:
1. The `leverParam` is taken from the URL parameter
2. `decodeURIComponent()` is called, decoding URL-encoded payloads
3. The decoded value is embedded directly into an HTML string without sanitization
4. The `jQuery.append()` method inserts the HTML into the DOM
5. The injected `<SCrlpT>` tag is parsed and executed

```html
<!-- Injected payload rendered in the page: -->
'><script>alert(1)</script>
```

The source code reveals the vulnerability:
```javascript
// The leverParam is taken from the URL and embedded in the
// HTML string WITHOUT sanitization. decodeURIComponent() is
// called so that URL-encoded payloads also execute.
```

<img width="1622" height="1056" alt="Screenshot 2026-08-14 205918" src="https://github.com/user-attachments/assets/f3b7a6a1-3d40-42c9-a4fc-53c9fb3e6c90" />
<img width="1370" height="393" alt="Screenshot 2026-08-14 210020" src="https://github.com/user-attachments/assets/0c957694-5f3c-4306-8c72-ebc2e70ba4fe" />


## Impact

DOM-Based XSS vulnerabilities are particularly dangerous because:

- **Client-Side Execution**: Payloads execute in the victim's browser context
- **Server-Side Detection Evasion**: The malicious payload may not appear in server logs
- **URL Parameter Injection**: Attackers can craft malicious links and distribute them via email or social media
- **Bypass Server Filters**: Server-side filtering may not detect client-side vulnerabilities

An attacker can:

- Steal session cookies and hijack user accounts
- Perform actions on behalf of the authenticated user
- Deface the webpage or manipulate its content
- Redirect users to malicious websites
- Capture keystrokes or sensitive information entered by the user
- Bypass case-sensitive filtering mechanisms

## Recommendation

To mitigate this vulnerability:

- **Sanitize Client-Side**: Never insert unsanitized user input into HTML strings using `innerHTML`, `jQuery.append()`, `document.write()`, or similar methods.

- **Use Safe Methods**: Use `textContent` or `jQuery.text()` when inserting user-controlled data.
  ```javascript
  // Unsafe
  jQuery("#container").append('<a href="' + link + "'>" + text + '</a>');
  
  // Safe
  jQuery("#container").append($('<a>', { href: link, text: text }));
  ```

- **Input Validation**: Validate and sanitize all user inputs on both the client and server sides.

- **Content Security Policy (CSP)**: Implement a strict CSP that restricts inline scripts and unauthorized script sources.

- **Use Safe DOM APIs**: Use `createElement()` and `setAttribute()` instead of HTML string concatenation.
  ```javascript
  var a = document.createElement('a');
  a.href = link;
  a.textContent = text;
  document.getElementById('container').appendChild(a);
  ```

- **Output Encoding**: Use encoding functions to escape special characters before inserting into HTML.

- **Web Application Firewall (WAF)**: Deploy a WAF to help detect and block malicious XSS payloads.

- **Security Headers**: Implement security headers like `X-XSS-Protection` and `X-Content-Type-Options` for additional protection.

- **Regular Security Audits**: Conduct regular assessments and code reviews to identify vulnerabilities.

---
