# Lab Name
Unprotected Admin Functionality

# Goal
Solve the lab by deleting the user **carlos** through an exposed admin panel.

# Vulnerability
- Broken Access Control
- Vertical Privilege Escalation
- Information Disclosure (`robots.txt`)

# Theory
This lab demonstrates how information disclosure can expose sensitive functionality. If administrative endpoints are publicly discoverable and authorization checks are missing, an attacker may gain access to privileged functionality, resulting in vertical privilege escalation.

# Environment
A vulnerable shopping application provided by PortSwigger.

# Enumeration

I started by checking common publicly accessible files that often reveal useful information during reconnaissance.

Using `ffuf`, I fuzzed for common files and directories:

```bash
ffuf -u <lab_url>/FUZZ -w wordlist.txt
```

One of the discovered files was:

```
robots.txt
```

Its content was:

```
User-agent: *
Disallow: /administrator-panel
```

Although this file only tells search engine crawlers which paths they should avoid, it is publicly accessible to everyone. As a result, it unintentionally revealed the location of the admin panel.

# Exploitation

I visited the exposed endpoint:

```
/administrator-panel
```

The page was accessible without any authentication or authorization checks.

From there, I deleted the user **carlos**. The delete action was also performed without authorization, successfully solving the lab.

# Impact
An attacker can access administrative functionality without proper authorization and perform privileged actions, such as deleting users or modifying sensitive data.

# Why It Worked
The server failed to enforce authorization checks before granting access to administrative functionality.

Additionally, the application exposed the admin endpoint through a public `robots.txt` file. Although `robots.txt` is intended for search engine crawlers, it should never be considered a security mechanism because anyone can access it.

# Common Mistakes
- Missing authorization checks on administrative functionality.
- Exposing sensitive endpoints through publicly accessible files.
- Relying on obscurity instead of proper access control.

# Defenses
- Enforce authorization checks on every privileged endpoint.
- Never rely on `robots.txt` to hide sensitive resources.
- Regularly review publicly accessible files for unintended information disclosure.

# References
- https://portswigger.net/web-security/information-disclosure
- https://www.fortinet.com/resources/cyberglossary/authentication-vs-authorization
- https://www.seerinteractive.com/insights/how-to-read-robots-txt

# Personal Notes

## Authentication vs Authorization

Authentication verifies **who you are**.

Authorization determines **what you are allowed to do** after you have been authenticated.

## Information Disclosure

Information Disclosure occurs when an application unintentionally exposes information that should not be publicly available. Although the disclosed information may not be directly exploitable, it can help attackers discover additional attack surfaces.

## robots.txt

`robots.txt` is a **public** file used to instruct search engine crawlers which parts of a website should not be crawled.

It is **not** a security mechanism and should never be used to protect sensitive resources.

# Public Write-ups

- https://hackerone.com/reports/350432
- https://hackerone.com/reports/1801427
- https://www.anvilsecure.com/blog/vulnerabilities-in-homepage-dashboard.html
https://www.traceable.ai/blog-post/how-1-exposed-honeywell-api-gave-us-control-over-an-internal-engineering-system