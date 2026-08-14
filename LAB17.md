# Reflected Cross-Site Scripting (XSS) in Category Filter

## Summary

I identified a Reflected Cross-Site Scripting (XSS) vulnerability in the `category` parameter of the application.

The application accepts user-controlled input through the `category` GET parameter and reflects it back to the page without proper sanitization or output encoding. By injecting a `<SCRIPT>` tag with malicious JavaScript, arbitrary code execution is achieved when the victim visits the crafted URL.

An attacker can craft a malicious URL and trick a victim into clicking it. Once the victim visits the page, the injected JavaScript executes in the context of the vulnerable application, triggering an `alert(1)` popup.

## Vulnerability Type

**Cross-Site Scripting (XSS) - Reflected**

## Vulnerable Parameter

```text
category
```

## Vulnerable URL

```text
https://kzlabs.in/17.php?category='>anil<SCRIPT>alert(1)</SCRIPT>
```

## Steps to Reproduce

1. Open the vulnerable page.

2. Replace the value of the `category` parameter with the following payload:

   ```text
   '>anil<SCRIPT>alert(1)</SCRIPT>
   ```

3. Visit the crafted URL:

   ```text
   https://kzlabs.in/17.php?category=%27%3Eanil%3CSCRIPT%3Ealert%281%29%3C%2FSCRIPT%3E
   ```

4. Observe that the browser executes the injected payload and displays an `alert(1)` popup.

## Payload Used

```text
'>anil<SCRIPT>alert(1)</SCRIPT>
```

URL-encoded version:
```text
%27%3Eanil%3CSCRIPT%3Ealert%281%29%3C%2FSCRIPT%3E
```

## Proof of Concept

The application accepts the value of the `category` parameter and reflects it directly into the page HTML without proper sanitization.

The payload works by:
1. Breaking out of the existing HTML context using `'>`
2. Injecting a `<SCRIPT>` tag with the `alert(1)` payload
3. The browser parses and executes the injected script tag

```html
<!-- Injected payload rendered in the page: -->
'><script>alert(1)</script>
```

This confirms that arbitrary JavaScript execution is possible through the `category` parameter.

<img width="1367" height="448" alt="Screenshot 2026-08-14 200205" src="https://github.com/user-attachments/assets/c42f26b4-276b-44ba-b7ac-742e82e91260" />


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
  echo htmlspecialchars($_GET['category'], ENT_QUOTES, 'UTF-8');
  ```

- **Input Validation**: Validate and sanitize all user inputs. Consider using a whitelist approach to allow only safe characters.

- **Content Security Policy (CSP)**: Implement a strict CSP that restricts inline scripts and unauthorized script sources using `unsafe-inline` and `unsafe-eval` directives.

- **Context-Aware Escaping**: Use context-aware escaping based on where the data is placed (HTML body, attributes, JavaScript, etc.).

- **Web Application Firewall (WAF)**: Deploy a WAF, such as Cloudflare WAF, to help detect and block malicious XSS payloads.

- **Security Headers**: Implement security headers like `X-XSS-Protection` and `X-Content-Type-Options` for additional protection.

---
