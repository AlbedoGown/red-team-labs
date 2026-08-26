# Starting Point (Tier 2): Bike (Very Easy)

> Public portfolio note: flags, passwords, hashes, host addresses and exploitation details have been removed or generalized.

## 1. Recon

### Network scanning

Initial port and service discovery identified two exposed TCP services:

```text
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu
80/tcp open  http    Node.js / Express
```

### Web application observations

The HTTP service exposed a subscription form that submitted an `email` value via a POST request to the application root. Supplying template-like syntax caused a verbose parser error referencing Handlebars and a server-side path within the Node.js application.

This established two important facts for triage:

- User-controlled input appeared to reach template compilation.
- Detailed errors disclosed framework internals and filesystem paths.

## 2. Vulnerability and Misconfiguration Analysis

### Server-Side Template Injection (SSTI)

**Root cause:** Untrusted data from the `email` parameter was treated as template source rather than being rendered as data in a fixed template.

**Impact:** An attacker could potentially alter template evaluation and, depending on the runtime and configuration, access functionality outside the intended template context.

### Template sandbox escape risk

**Root cause:** Handlebars templates and JavaScript runtime objects were exposed in a configuration that allowed dangerous property access patterns.

**Impact:** A successful sandbox escape could enable command execution through the Node.js process. Public exploit payloads are intentionally omitted from this write-up.

### Excessive privileges

**Root cause:** The web application process ran as `root` rather than a dedicated unprivileged service account.

**Impact:** Application-level code execution would immediately become full host compromise, removing the need for a separate local privilege-escalation step.

## 3. Validation in the Lab

The lab was assessed by using benign, non-destructive checks to confirm server-side template processing and to establish the security context of the affected service. No flags, credentials, reusable payloads, or commands targeting sensitive files are included here.

The key lesson is the attack chain:

1. A user-controlled form value reaches template compilation.
2. Verbose errors disclose the template engine and deployment details.
3. Unsafe template behavior can become server-side code execution.
4. Running the application as root turns that application compromise into host compromise.

## 4. Remediation

### Keep templates static

Never compile user input as a template. Compile a trusted, static template and supply user input only as context data:

```javascript
// Vulnerable pattern
const template = Handlebars.compile(req.body.email);

// Safer pattern
const template = Handlebars.compile("We will contact you at: {{email}}");
const result = template({ email: req.body.email });
```

### Reduce information disclosure

- Set `NODE_ENV=production`.
- Configure centralized Express error handling.
- Return generic client-facing errors and retain stack traces only in protected server-side logs.

### Apply least privilege

- Run Node.js under a dedicated non-root account such as `node` or `www-data`.
- Restrict the service account's filesystem permissions.
- Keep sensitive administrator directories inaccessible to the web service.
- Use container or systemd hardening controls where applicable.

### Secure development controls

- Validate input server-side and apply allow-lists appropriate to each field.
- Review template-engine configuration and dependency versions.
- Add SAST rules and code-review checks for dynamic template compilation.
- Test error handling before production deployment.

## 5. SOC Detection Opportunities

| Data source | Suspicious behavior | Detection idea |
| --- | --- | --- |
| WAF / HTTP access logs | Template delimiters or suspicious JavaScript-property terms in POST parameters | Alert on repeated requests containing Handlebars-style blocks, `constructor`, or process-execution indicators. |
| EDR / Auditd | Node.js spawns shells or system utilities | High-severity alert when `node` is the parent process of `/bin/sh`, `whoami`, `cat`, or other unexpected binaries. |
| File auditing | Web-service account reads sensitive administrator paths | Alert when the Node.js service account accesses `/root/` or other protected locations. |
| Application logs | Parser exceptions and recurring malformed template input | Correlate bursts of Handlebars parse errors with the source IP, endpoint, and request body indicators. |

## 6. Key Takeaways

- Treat template source as code and keep it fully trusted and static.
- Debug output can materially shorten an attacker's reconnaissance phase.
- Least privilege limits the blast radius of web-application vulnerabilities.
- SOC monitoring should correlate suspicious web input with process creation and sensitive-file access.
