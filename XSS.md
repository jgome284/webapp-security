# Cross-Site Scripting (XSS) Attacks

Cross-Site Scripting (XSS) is a common web application vulnerability that occurs when a web application renders unsanitized input to the front end of an application. An attacker takes advantage of this vulnerability by injecting malicious code, generally in the form of JavaScript, through the browser. They can trick a benign website into executing this code for other users. This enables an attacker to steal information from another user’s client-side data, redirect a user to malicious pages, or take control of their browser!

We will walk through different types of such attacks: Stored XSS, Reflected XSS, and DOM-Based XSS.

## Stored XSS

A stored XSS vulnerability occurs when a web server stores an unsanitized user input and displays it to other users. In a worst-case scenario, an attacker can input a malicious script and store it to the vulnerable website, making the script run for all other users on that page.

![An image showing that in Stored XSS attacker an attacker creates a comment with code, the web server saves the comment, the user requests to see the comments, then the server sends the comment to the victim's browser where the code is executed.](./imgs/Cybersecurity_XSS_Stored_v2-03.svg)

Let’s walk through the example in the image above.

Let’s say a website has a poorly designed comment function where the backend does not sanitize user comments. It may be possible for an attacker to add dangerous JavaScript to their comment. Now, any time another user loads the page and the server displays the comment with the bad code to the HTML, that user’s browser will execute the JavaScript code. This is an example of a Stored XSS attack.

This effect makes Stored XSS attacks some of the most serious XSS attacks.

## Reflected XSS

Reflected XSS occurs when a user’s input is immediately returned back to the user. This return may come in the form of an error message, a popup, or a search term. In these instances, the malicious code is never stored by the server. Rather, it exists as a value in the URL or request.

Despite the bad code not being stored in the database and executed by all victims’ browsers, the attacker can use Reflected XSS to target certain users, forcing them to execute the malicious script. Take a look at the example below where an attacker tricks a user into clicking a script-injected link, set up to steal that user’s credentials!

![An image showing in Reflected XSS that an attacker sends a script injected link to the victim, the victim clicks on it, the victim's browser loads the legitimate site while also executing the malicious script, and then the malicious script sends the user's information to the attacker.](./imgs/Cybersecurity_XSS_Reflected_v2.svg)

## DOM-Based XSS

The DOM, short for Document Object Model, is used to help scripts and the underlying webpage interact. When user input is interpreted by the DOM, an attacker is able to inject malicious code there. These types of vulnerabilities do not cause any changes in how the server responds. Rather, these attacks are completely client-side.

For example, a web page may use client-side Javascript to customize a welcome page, displaying their name based on a value in the URL. Depending on how the Javascript runs, a DOM-Based XSS attack may be able to replace the name value with a malicious script. If a victim loaded the page with the attacker’s code, the vulnerable webpage may execute the code!

![An image with the following steps: 1) Attacker sends link containing malicious code, 2) Victim opens link, 3) Victim's browser sends request to server, 4) Server responds with page 5) Victim's browser loads vulnerable Javascript and executes the attacker's payload.](./imgs/Cybersecurity_XSS_DOM_v2.svg)

## Identifying XSS Vulnerabilities

Let’s look at how we can uncover XSS vulnerabilities in a web application.

As with any vulnerability, it is important that we investigate any potential input areas. When looking at the application, consider all possible fields. Comments, usernames, custom settings, and parameters all provide great starting points.

Once we have identified a potential injection point, we can begin testing various inputs to create a proof-of-concept payload (POC). A POC payload will demonstrate that an issue exists, without causing damage. The most basic POC payload is shown below.

```html
<script>
  alert(1);
</script>
```

If a web server is not properly sanitizing user input, this will return a pop-up box similar to the below image.

![This image shows "<script>alert(1)</script>" was injected into the URL which caused a Javascript alert to show up on the screen.](./imgs/alert.jpg)

If this payload does not work, that does not necessarily mean the system is secure. In fact, many systems will take a flawed approach to protection and block certain words. If a blocklist is in effect your request may be blocked, or your <script> tags could be removed. There are numerous other ways we can execute code, without ever using a <script> tag. Below are some potential workarounds that could be used by an attacker.

```html
<img src="X" onerror="alert(1);" />

<b onmouseover="alert(1)">click me!</b>

<body onload=alert('test1')>
```

## Sanitization

Sanitization is the process of removing/replacing problematic characters with safe versions. Depending on the backend language, there may or may not be built-in functions to aid in this process.

However, if these functions do not exist, we can generally succeed in preventing XSS attacks by removing characters such as <, >, ", =, and potentially dangerous keywords.

Rather than remove characters, we can also replace them with HTML-encoded versions of the characters. This allows us to retain the characters, but remove their capacity to affect the page’s HTML.

For example, the < character would be converted to the “<” string. The browser will render this string as the “<” character, but it will not interpret it as actual HTML, preventing the attack.

It is important to note, however, that depending on how the data is used, this type of escaping may not be enough. It’s important to consider all potential avenues for an attack.

There are also JavaScript packages like [sanitize-html](https://www.npmjs.com/package/sanitize-html) that help sanitizer user inputs!
