# Insecure Direct Object References

## Goal
This lab stores user chat logs directly on the server's file system, and retrieves them using static URLs.

Solve the lab by finding the password for the user carlos, and logging into their account. 

## Vulnerability

* Broken Access Control
* Horizontal Privilege Escalation
* Insecure Direct Object Reference (IDOR)

## Indicators

A numeric identifier for files (`/download-transcript/1.txt`).

## Theory

This lab demonstrates how relying solely on user-supplied data to identify an object, without enforcing proper server-side authorization, can be extremely dangerous.

## Environment

A vulnerable website with an LLM-powered live chat feature provided by PortSwigger.

# Exploitation steps

1. After I launched the lab, I opened the `Live chat` section to chat with the LLM.

2. I started a conversation with the LLM, but after that, I couldn't find any sensitive or valuable endpoints.

3. I started exploring the application's features. Then I discovered a feature for downloading the conversation as a transcript. This was important because if the website provides transcript files from its own storage, it needs to enforce authorization to determine whether a user is accessing their own chat history or someone else's. But the question is: what if it fails?

4. Then I opened Burp Suite and enabled interception. I sent a request to download the transcript file and intercepted it through Burp Suite. The transcript path looked like this:

```text
https://xxx.web-security-academy.net/download-transcript/2.txt
```

The indicator is the numeric identifier of the file, as I mentioned.

5. I asked myself: "What's the developer/backend assumption?" The answer was: "Their assumption is that I'm **only** going to access **my own chat history** and the **2.txt** file."

6. So, to test this assumption, I changed `2` to `1` and resent the request.

7. I gained access to Carlos' conversation with the LLM. In the conversation, Carlos was attempting to determine whether the password he provided matched his account password. The LLM confirmed that the passwords were exactly the same. Since I had access to the conversation, I obtained the password, logged into Carlos' account, and finished the lab.

## Impact

An attacker can gain unauthorized access to users' conversations with the LLM.

## Burp requests

`2.txt` → My own conversation with the LLM

`1.txt` → Carlos' conversation with the LLM

## Root cause

The server failed to enforce proper authorization to determine whether I was allowed to access another user's conversation with the LLM. An administrator may legitimately need access to other users' conversations for debugging, but such access should also be governed by appropriate authorization and privilege boundaries.

## Why it worked

The server blindly returned the requested transcript without performing an authorization check. Because the transcript was accessible through a user-controlled file identifier, I was able to change the identifier and access another user's transcript.

## Common Mistakes

- Assuming that a user is going to **only** access their own data
- Assuming that **authentication** is enough
- Missing authorization checks when returning users' data
- Trusting a user-supplied object identifier as proof of authorization
- Relying solely on user-supplied data without performing server-side authorization

## Defenses

* Enforcing proper server-side authorization
* Checking whether the requested transcript belongs to the authenticated user before returning it
* Denying access when the authenticated user is not authorized to access the requested transcript

## References

* [PortSwigger — Access control vulnerabilities and privilege escalation](https://portswigger.net/web-security/access-control)
* [PortSwigger — Insecure Direct Object References (IDOR)](https://portswigger.net/web-security/access-control/idor)
* [PortSwigger — Testing for IDORs](https://portswigger.net/burp/documentation/desktop/testing-workflow/vulnerabilities/access-controls/testing-for-idors)
* [PortSwigger — Testing horizontal access controls](https://portswigger.net/burp/documentation/desktop/testing-workflow/vulnerabilities/access-controls/horizontal-access-controls)

## Public write-ups

[InfoSecWriteUps - Horizontal Privilege Escalation via IDOR: Viewing, Editing and Deleting](https://scriptjacker.medium.com/horizontal-privilege-escalation-via-idor-viewing-editing-and-deleting-b10936ad4eb1)

[InfoSecWriteUps - 🐛💰🔓🎯 Finding an IDOR in User Profile API: A $15,000 Journey to Critical](https://infosecwriteups.com/finding-an-idor-in-user-profile-api-a-15-000-journey-to-critical-0f05e583c00b)

[InfoSecWriteUps - IDOR and Broken Access Control Risking Private Data Exposure](https://c0nqr0r.medium.com/idor-and-broken-access-control-risking-private-data-exposure-dd808412ed13)

## Personal Notes

### What's Broken Access Control?

* **Broken access control** is a security flaw where an application fails to properly enforce what users are allowed to access or do.

### What's the difference between authentication and authorization?

* **Authentication** verifies **who you are**
* **Authorization** determines **what you are allowed to do**

### What's Horizontal Privilege Escalation?

* **Horizontal Privilege Escalation** occurs when a user accesses the resources or performs actions belonging to another user at the same privilege level.

### What's Insecure Direct Object Reference (IDOR)?

* **Insecure Direct Object Reference (IDOR)** is an access-control vulnerability that occurs when an application uses user-controlled input to reference objects without properly enforcing server-side authorization for the requested object.
