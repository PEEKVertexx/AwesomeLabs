# User role controlled by request parameter

## Goal

This lab has an admin panel at `/admin`, which identifies administrators using a forgeable cookie.

Solve the lab by accessing the admin panel and using it to delete the user `carlos`.

You can log in to your own account using the following credentials: `wiener:peter`

## Vulnerability

* Broken Access Control
* Vertical Privilege Escalation
* Parameter-based Access Control

## Attack surface

* `/admin` path
* `Admin` cookie parameter

## Indicators

* `Admin=false` cookie parameter

## Theory

This lab demonstrates why relying on client-controlled data for authorization can be dangerous.

The application uses a cookie parameter to determine whether the current user should be treated as an administrator. Since the cookie is controlled by the client, its value can be modified before the request reaches the server.

## Environment

A vulnerable shopping web application provided by PortSwigger.

## Exploitation steps

1. I logged in to the website using the lab-provided credentials (`wiener:peter`).

2. Then, I opened Burp Suite and turned on interception.

3. Based on the lab's goal, I accessed the `/admin` path. The request was intercepted by Burp Suite, and I was able to inspect and modify it. The request looked something like this:

```http
GET /admin HTTP/2
Host: ...web-security-academy.net
Cookie: Admin=false; session=...
```

4. As you can see, the `Cookie` header contains a parameter named `Admin`, whose value is `false`.

   At this point, I asked myself:

```text
What security-sensitive value is the server trusting even though I can control it?
```

The answer was the `Admin` cookie parameter. The application was using a client-controlled value to determine whether I was an administrator.

So, I changed the value of the `Admin` parameter from `false` to `true` and sent the request again.

The application accepted the modified value, and I successfully gained access to the admin panel.

5. From the admin panel, I deleted the user `carlos`. After modifying the `Admin` cookie to `true` again, I was able to perform the administrative action successfully.

## Impact

An attacker can gain unauthorized access to administrative functionality, including functionality that allows them to delete users.

## Why it worked

The application makes an authorization decision based on a client-controlled cookie parameter.

The server expects the `Admin` cookie to contain a value such as `false` for a normal user and `true` for an administrator. However, the user can modify this value before sending the request.

In fact, the application effectively assumes that a normal user will not modify:

```text
Admin=false
```

to:

```text
Admin=true
```

This assumption is insecure because the server must not trust client-controlled data when making authorization decisions.

## Common Mistakes

* Relying on client-controlled data for authorization
* Trusting client-side parameters for security-sensitive decisions
* Failing to enforce authorization based on trusted server-side state
* Assuming that users will not modify request parameters

## Defenses

* Enforce authorization server-side.
* Do not use client-controlled parameters to determine a user's privileges.
* Store and validate the user's role or permissions using trusted server-side state.
* Use an appropriate authorization model, such as RBAC, to manage roles and permissions.
* Apply a deny-by-default approach to sensitive functionality.
* Test access controls by attempting to modify security-sensitive request parameters.

## References

* [PortSwigger — Access control vulnerabilities and privilege escalation](https://portswigger.net/web-security/access-control)
* [PortSwigger — Testing for parameter-based access control](https://portswigger.net/burp/documentation/desktop/testing-workflow/vulnerabilities/access-controls/param-based-access-control)
* [MDN — Using HTTP cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Cookies)

## Personal Notes

### What I learned from this lab?

* Client-controlled data should never be blindly trusted for authorization decisions.
* When reviewing an HTTP request, I should pay attention to security-sensitive parameters and ask whether I can modify them.
* A value that looks like a simple boolean, such as `Admin=false`, can have a major impact if the server blindly trusts it.