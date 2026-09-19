# Cookies

When You Login to some server, Some data has to be stored about user Like his preferences. Theme, fontsize or Login information etc. This makes sense to store this data at users end insted of storing this at server.

as Servers are stateless in this case,they cannot remember and HTTP request is also stateless and sending all the data manually everytime is not possible  

Cookies come into picture so that browser automatically can send cookie (data) when ever request is made again to same origin Solves one but introduces another problem Where CSRF is used.


# Cookies - Extended

When you login to a server, certain user-specific data needs to be stored—such as user preferences (theme, font size) or authentication states. Instead of forcing the server to manage this overhead for millions of users, it makes perfect sense to store this data on the client (user's) end.

## The Problem: Statelessness
Both web servers and the **HTTP protocol are completely stateless**. 
* They treat every single request as a completely isolated transaction. 
* They cannot natively remember who you are from one page click to the next.
* Manually attaching user data or tokens to every single request explicitly is inefficient and prone to developer error.

## The Solution: Cookies
Cookies step in to automate this process. When a server wants a browser to remember data, it sends a specific instruction.

```http
Set-Cookie: session_id=xyz123; Secure; HttpOnly; SameSite=Lax
```

Once stored, the browser **automatically attaches and sends the cookie** back to the server inside the headers whenever a subsequent request is made to that *same origin*.

```http
Cookie: session_id=xyz123
```

## Types of Cookies
* **Session Cookies:** Temporary cookies that are deleted the moment you close your browser tab or window.
* **Persistent Cookies:** Stored on the user's hard drive until a specific expiration date (`Expires` or `Max-Age`) is reached.

## The Security Trade-off (CSRF)
While automatically sending cookies solves the authorization problem, it introduces a severe security vulnerability called **Cross-Site Request Forgery (CSRF)**. 

Because browsers blindly attach cookies to requests pointing to the origin server, a malicious third-party website can trick a user's browser into sending an unauthorized, authenticated request to your server.

### The Modern Fix
To mitigate CSRF, modern web architecture relies on the **`SameSite` attribute** inside the cookie configuration:
* `SameSite=Strict`: The cookie is never sent in cross-site requests (e.g., clicking a link from an external site).
* `SameSite=Lax`: The cookie is withheld on cross-site subrequests (like images), but sent when a user navigates to the origin site normally. This is the modern browser default.




# CSRF (Cross-Site Request Forgery)  

There is a scenario where other origins can cause a request to be made and the request is considered as if you made it because cookies are attached by the browser automatically before sending the request.  

To avoid this we save a CSRF token at the client and the client attaches it manually (similar to JWT).  

If the token matches, the request is accepted, avoiding the vulnerable state.  


## How CSRF Tokens Are Sent (Two Main Ways)

Depending on how your application is built, a CSRF token is sent in one of two ways:


### 1. Automatically (Traditional Form-Based Apps)

In traditional server-rendered applications (like Django, Laravel, or Spring Boot with Thymeleaf), the server embeds the CSRF token into a hidden input field inside an HTML form.

```html
<form action="/transfer-funds" method="POST">

    <!-- The server injects this automatically -->
    <input type="hidden"
           name="csrf_token"
           value="abc123xyz..." />

    <input type="number" name="amount" />

    <button type="submit">Send</button>

</form>
```

When the user clicks submit, the browser sends the form data automatically, including the hidden token field.


### 2. Manually via JavaScript (Modern SPAs)

If you are building a Single Page Application (like React, Vue, or Angular) that talks to a separate API backend, you do send the CSRF token manually, very much like a JWT.

This is often called the **Cookie-to-Header Pattern**:

1. The server sets a standard cookie containing the CSRF token (e.g., `XSRF-TOKEN`).
2. Your JavaScript framework reads that cookie value.
3. Your JavaScript code manually injects that value into a custom HTTP header (usually `X-XSRF-TOKEN`) for every API request.
4. The server validates that the header matches the cookie.

Example:

```http
X-XSRF-TOKEN: abc123xyz...
```

# CORS (Cross Origin Resourse Sharing)

Backend is already capable of Authenticating, Validating and weather to send or not send any response. This CORS is simple browser policy even when backend sends the data it checks for header `Access-control-allow-origin` setted by cors at backend and value is matched with current domain and browser can reject the origin's Javascript to access the response if the header origin is not matched or header is missing. 

Just a browser Policy and nothing to do with security
