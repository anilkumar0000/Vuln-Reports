# Reflected Cross-Site Scripting (XSS) - Category Parameter Injection

## Summary

I identified a Reflected Cross-Site Scripting (XSS) vulnerability in the `category` parameter of the application.

The application accepts user-controlled input through the `category` GET parameter and reflects it back to the page without proper sanitization or output encoding. By injecting a malicious JavaScript payload, arbitrary code execution is achieved when the victim visits the crafted URL.

An attacker can craft a malicious URL and trick a victim into clicking it. Once the victim visits the page, the injected JavaScript executes in the context of the vulnerable application, triggering an `alert(1)` popup.

## Vulnerability Type

**Cross-Site Scripting (XSS) - Reflected**

## Vulnerable Parameter

```text
category
```

## Vulnerable URL

```text
https://kzlabs.in/14.php?category=javascript:alert(1)
```

## Steps to Reproduce

1. Open the vulnerable page.

2. Replace the value of the `category` parameter with the following payload:

   ```text
   javascript:alert(1)
   ```

3. Visit the crafted URL:

   ```text
   https://kzlabs.in/14.php?category=javascript:alert(1)
   ```

4. Observe that the browser executes the supplied `javascript:` URI and displays an `alert(1)` popup.

## Payload Used

```text
javascript:alert(1)
```

URL-encoded version:
```text
javascript%3Aalert%281%29
```

## Proof of Concept

The application accepts the value of the `category` parameter as a navigation target or page reference without validating the URI scheme.

When the page loads, the browser processes the `javascript:` URI and executes the following JavaScript:

```javascript
alert(1)
```

The `javascript:` URI scheme allows arbitrary JavaScript code to be executed when the browser navigates to or processes the value. This confirms that arbitrary JavaScript execution is possible through the `category` parameter.

![Uploading Screenshot 2026-08-14 200002.png…]()

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

- **URI Scheme Validation**: When accepting URLs or navigation targets, validate the URI scheme and only allow safe protocols such as `http://`, `https://`, and `mailto:`.
  ```php
  $allowed_schemes = ['http', 'https'];
  $parsed_url = parse_url($_GET['category']);
  if (!in_array($parsed_url['scheme'], $allowed_schemes)) {
      // Reject or sanitize
  }
  ```

- **Input Validation**: Validate and sanitize all user inputs. Consider using a whitelist approach to allow only safe characters.

- **Content Security Policy (CSP)**: Implement a strict CSP that restricts the use of `javascript:` URIs and inline scripts.

- **Web Application Firewall (WAF)**: Deploy a WAF, such as Cloudflare WAF, to help detect and block malicious XSS payloads.

- **Security Headers**: Implement security headers like `X-XSS-Protection` and `X-Content-Type-Options` for additional protection.

---
