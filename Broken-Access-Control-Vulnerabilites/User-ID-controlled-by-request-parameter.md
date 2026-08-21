## User ID controlled by request parameter

## Goal
This lab has a horizontal privilege escalation vulnerability on the user account page.

To solve the lab, obtain the API key for the user carlos and submit it as the solution.

You can log in to your own account using the following credentials: `wiener:peter`

## Vulnerability
- Broken Access Control
- Horizontal Privilege Escalation
- Insecure Direct Object Reference (IDOR)

## Attack surface
- `/my-account` page

## Indicators
- `id` parameter

## Theory

This lab conceptually demonstrates how a horizontal privilege escalation occurs.

I solved this lab by obtaining the API key of user **carlos**, Though my user was **wiener** and submitting it as a solution.

I accessed data belonging to another user without authorization. Which makes it an **horizontal privilege escalation**

## Environment

A vulnerable shopping application provided by PortSwigger.

## Exploitation steps
1. Logged in with my login credentials provided by PortSwigger (username `wiener` and password `peter`)

2. I got redirected to `/my-account` page , The URL's pattern was something like this:

```
https://***.web-security-academy.net/my-account?id=wiener
```

Then I asked myself:

```
What developer/backend assumed that I'm not going to do?
```

The answer was obvious:

```
Developer/Backend assumed that I'm only going to access my own data and entering my own username on id parameter's value
```

3. Then I tried to do against the assumption by accessing another user's data , I modified the id parameter's value from my own user (wiener) to carlos (The user that lab's goal is to access its API key and submitting the API key as a solution) And I sent the request

4. I successfully gained access to carlos' API key. And then I submitted it as a solution

## Impact

Gaining access to another user's API key , It could be extermely dangerous in real-world because modern web applications allows us to access to a lot of functionalities through our API key

## Root cause

- Assuming that a user is only going to access its own data

- Missing server-side authorization while returning user's API key. It might be an attacker or anybody else

- Assuming that successful authentication is sufficient to authorize access to user-specific resources.

- The server uses the user-controlled id parameter to select the requested account without verifying that the authenticated user is authorized to access that account.

## Why it worked

Developer/Backend assumed that I'm only going to access my own data through keeping id parameter's value to my own username (wiener)

**Which was false and I gained access to another user's API key by simply modifying the id parameter**

## Common mistakes

- Assuming that a user is **only** going to access its own data

- Missing server-side authorization while returning user's data

- Assuming that **authentication** is **enough**

- Relying on user provided data for authorization

## Defenses

- Providing a server-side authorization while returning user's API key

## References
- [PortSwigger — Access control vulnerabilities and privilege escalation](https://portswigger.net/web-security/access-control) 
- [PortSwigger — Insecure Direct Object References (IDOR)](https://portswigger.net/web-security/access-control/idor)
- [PortSwigger — Testing for IDORs](https://portswigger.net/burp/documentation/desktop/testing-workflow/vulnerabilities/access-controls/testing-for-idors)
- [PortSwigger — Testing horizontal access controls](https://portswigger.net/burp/documentation/desktop/testing-workflow/vulnerabilities/access-controls/horizontal-access-controls)

## Public write-ups

[InfoSecWriteUps - Horizontal Privilege Escalation via IDOR: Viewing, Editing and Deleting](https://scriptjacker.medium.com/horizontal-privilege-escalation-via-idor-viewing-editing-and-deleting-b10936ad4eb1)

[InfoSecWriteUps - 🐛💰🔓🎯 Finding an IDOR in User Profile API: A $15,000 Journey to Critical](https://infosecwriteups.com/finding-an-idor-in-user-profile-api-a-15-000-journey-to-critical-0f05e583c00b)

[InfoSecWriteUps - IDOR and Broken Access Control Risking Private Data Exposure](https://c0nqr0r.medium.com/idor-and-broken-access-control-risking-private-data-exposure-dd808412ed13)

## Personal Notes

### What's Broken Access Control?

- **Broken access control** is a security flaw where an application fails to properly enforce what users are allowed to access or do.

### What's Horizontal Privilege Escalation?

- **Horizontal Privilege Escalation** occurs when a user accesses the resources or actions of a peer at the same privilege level

### What's Insecure Direct Object Reference (IDOR)?

- **Insecure Direct Object Reference (IDOR)** is a type of access control vulnerability that arises when an application uses user-supplied input to access objects directly.