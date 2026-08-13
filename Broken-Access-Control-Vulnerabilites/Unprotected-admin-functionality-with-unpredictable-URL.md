# Unprotected admin functionality with unpredictable URL

## Goal

This lab has an unprotected admin panel. It's located at an unpredictable URL, but the location is disclosed somewhere in the application.

Solve the lab by accessing the admin panel and using it to delete the user `carlos`.

## Vulnerability

* Broken Access Control
* Vertical Privilege Escalation
* Information Disclosure (admin panel URL disclosed through an inline `<script>` element)

## Theory

This lab demonstrates how **security through obscurity** can allow an attacker to discover sensitive information and access privileged functionality. It also teaches us how JavaScript code can contain valuable information about the behavior of an application, and why blindly trusting client-side controls for authorization can be dangerous.

## Environment

A vulnerable shopping web application provided by PortSwigger.

## Enumeration

I started to gather information about the web application by reading JavaScript files. In contrast with the previous lab, this application doesn't contain a `robots.txt` file. In my enumeration methodology, I usually rely on JavaScript data after checking common files and directories such as `robots.txt`.

## Exploitation Steps

1. I started the enumeration. First, I tried to gain information about the website by fuzzing common web directories, but I didn't find anything valuable.

2. Then I started to read **JavaScript data**. First, I read the `labHeader.js` file, but it didn't contain any special information.

3. After `labHeader.js` didn't contain any valuable information, I started to read the main page's inline `<script>` element. Its content was like this:

```js
var isAdmin = false;
if (isAdmin) {
   var topLinksTag = document.getElementsByClassName("top-links")[0];
   var adminPanelTag = document.createElement('a');
   adminPanelTag.setAttribute('href', '/admin-uvtp1s');
   adminPanelTag.innerText = 'Admin panel';
   topLinksTag.append(adminPanelTag);
   var pTag = document.createElement('p');
   pTag.innerText = '|';
   topLinksTag.appendChild(pTag);
}
```

The important part was the `href` attribute:

```text
/admin-uvtp1s
```

Even though the application hides the Admin Panel link from non-admin users, the URL itself is exposed in the client-side code.

4. I opened `/admin-uvtp1s` directly and then deleted the user `carlos`.

## Impact

An attacker can directly access administrative functionality, such as deleting user accounts, without having the required privileges.

## Why it worked

The server failed to properly enforce authorization for the admin panel and its functionality.

The JavaScript code contains an `isAdmin` variable that determines whether the Admin Panel link should be displayed in the navigation bar. If `isAdmin` is `true`, the application adds the Admin Panel link to the page.

However, this is only a **client-side UI control**. Hiding a link from the user does not prevent the user from accessing the underlying endpoint directly.

In this case, the admin panel's unpredictable URL was exposed in the page's inline JavaScript code. This is where the **Information Disclosure** occurs.

After discovering the URL, I could access the admin panel directly without being an administrator. The server also didn't verify whether the current user had the required privileges to access the admin functionality. This results in **Vertical Privilege Escalation** and is an example of **Broken Access Control**.

## Common Mistakes

* Missing server-side authorization checks on administrative functionality.
* Relying on client-side controls to protect sensitive functionality.
* Security through obscurity.

## Defenses

* Enforce authorization checks on the server side for every administrative endpoint and functionality.
* Do not rely on hiding links or UI elements as an authorization mechanism.
* Avoid exposing sensitive internal information in client-side JavaScript.
* Regularly review publicly accessible files and client-side code for unintended information disclosure.

## References

* PortSwigger — Information Disclosure
* GeeksforGeeks — Authentication vs Authorization
* Medium — Security Through Obscurity
* Simunai Infosec — Privilege Escalation Testing

## Public Write-ups

* Infosec Write-ups — JS Analysis Leads to Information Disclosure
* Infosec Write-ups — JavaScript Secret Hunting
* Infosec Write-ups — From JS File to Jailbreak

## Personal Notes

### Authentication vs Authorization

Authentication verifies **who you are**.

Authorization determines **what you are allowed to do** after you have been authenticated.

### Information Disclosure

Information Disclosure occurs when an application unintentionally exposes information that should not be publicly available. Although the disclosed information may not be directly exploitable, it can help attackers discover additional attack surfaces.

### Vertical vs Horizontal Privilege Escalation

Vertical Privilege Escalation occurs when a low-privileged user gains access to higher-level privileges, such as gaining access to admin functionality.

Horizontal Privilege Escalation occurs when a user accesses resources belonging to another user at the same privilege level, such as when user A can access or modify user B's resources.
