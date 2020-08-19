```yaml
> * 原文地址：[10 security tips for frontend developers](https://konstantinlebedev.com/security-for-frontend/)
> * 原文作者：Konstantin Lebedev
> * 译文出自：[Breword](https://www.breword.com)
```


# 10 security tips for frontend developers

![cover picture](https://konstantinlebedev.com/static/854c2db40248a16d9635effcd7c4d8c6/d0742/cover.jpg) 



Web security is a topic that is often overlooked by frontend developers. When we assess the quality of the website, we often look at metrics like performance, SEO-friendliness, and accessibility, while the website’s capacity to withstand malicious attacks often falls under the radar. And even though the sensitive user data is stored server-side and significant measures must be taken by backend developers to protect the servers, in the end, the responsibility for securing that data is shared between both backend and frontend. While sensitive data may be safely locked in a backend warehouse, the frontend holds the keys to its front door, and stealing them is often the easiest way to gain access.

> “The responsibility for securing user’s data is shared between both backend and frontend.”

There are multiple attack vectors that malicious users can take to compromise our frontend applications, but fortunately, we can largely mitigate the risks of such attacks with just a few properly configured response headers and by following good development practices. In this article, we’ll cover 10 easy things that you can do to improve the protection of your web applications.

## Measuring the results

Before we start improving website security, it is important to have some feedback on the effectiveness of the changes that we make. And while quantifying what makes up “good development practice” may be difficult, the strength of security headers can be measured quite accurately. Much like we use Lighthouse to get performance, SEO, and accessibility scores, we can use [SecurityHeaders.com](https://securityheaders.com/) to get a security score based on the present response headers. For imperfect scores, it will also give us some tips on how to improve the score and, as a result, strengthen the security:

 ![cover picture](https://konstantinlebedev.com/static/a8f4b8f35f1d4230c31b3581007b1537/898f6/report.png "SecurityHeaders.com 报告") 

The highest score that SecurityHeaders can give us is “A+”, and we should always try to aim for it.

## Note on response headers

Dealing with response headers used to be a task for the backend, but nowadays we often deploy our web applications to “serverless” cloud platforms like [Zeit](https://zeit.co/) or [Netlify](https://www.netlify.com/), and configuring them to return proper response headers becomes frontend responsibility. Make sure to learn how your cloud hosting provider works with response headers, and configure them accordingly.

## Security measures

### 1\. Use strong content security policy

Sound content security policy (CSP) is the cornerstone of safety in frontend applications. CSP is a standard that was introduced in browsers to detect and mitigate certain types of code injection attacks, including cross-site scripting (XSS) and clickjacking.

Strong CSP can disable potentially harmful inline code execution and restrict the domains from which external resources are loaded. You can enable CSP by setting `Content-Security-Policy` header to a list of semicolon-delimited directives. If your website doesn’t need access to any external resources, a good starting value for the header might look like this:

```text
Content-Security-Policy: default-src 'none'; script-src 'self'; img-src 'self'; style-src 'self'; connect-src 'self';
```

Here we set `script-src`, `img-src`, `style-src`, and `connect-src` directives to self to indicate that all scripts, images, stylesheets, and fetch calls respectively should be limited to the same origin that the HTML document is served from. Any other CSP directive not mentioned explicitly will fallback to the value specified by `default-src` directive. We set it to `none` to indicate that the default behavior is to reject connections to any URL.

However, hardly any web application is self-contained nowadays, so you may want to adjust this header to allow for other trusted domains that you may use, like domains for Google Fonts or AWS S3 buckets for instance, but it’s always better to start with the strictest policy and loosen it later if needed.

You can find a full list of CSP directives on [MDN website](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy).

### 2\. Enable XSS protection mode

If malicious code does get injected from user input, we can instruct the browser to block the response by supplying `"X-XSS-Protection": "1; mode=block"` header.

Although XSS protection mode is enabled by default in most modern browsers, and we can also use content security policy to disable the use of inline JavaScript, it is still recommended to include `X-XSS-Protection` header to ensure better security for older browsers that don’t support CSP headers.

### 3\. Disable iframe embedding to prevent clickjacking attacks

Clickjacking is an attack where the user on website A is tricked into performing some action on website B. To achieve this, malicious user embeds website B into an invisible iframe which is then placed under the unsuspecting user’s cursor on website A, so when the user clicks, or rather thinks they click on the element on website A, they actually click on something on a website B.

We can protect against such attacks by providing `X-Frame-Options` header that prohibits rendering of the website in a frame:

```text
"X-Frame-Options": "DENY"
```

Alternatively, we can use `frame-ancestors` CSP directive, which provides a finer degree of control over which parents can or cannot embed the page in an iframe.

### 4\. Limit access to browser features & APIs

Part of good security practice is restricting access to anything that is not needed for the proper use of our website. We’ve already applied this principle using CSP to limit the number of domains the website is allowed to connect to, but it can also be applied to browser features. We can instruct the browser to deny access to certain features and APIs that our application doesn’t need by using `Feature-Policy` header.

We set `Feature-Policy` to a string of rules separated by a semicolon, where each rule is the name of the feature, followed by its policy name.

```text
"Feature-Policy": "accelerometer 'none'; ambient-light-sensor 'none'; autoplay 'none'; camera 'none'; encrypted-media 'none'; fullscreen 'self'; geolocation 'none'; gyroscope 'none'; magnetometer 'none'; microphone 'none'; midi 'none'; payment 'none';  picture-in-picture 'none'; speaker 'none'; sync-xhr 'none'; usb 'none'; vr 'none';"
```

[Smashing Magazine](https://www.smashingmagazine.com/2018/12/feature-policy/) has a great article explaining `Feature-Policy` in great detail, but most of the time you’ll want to set `none` for all features that you don’t use.

### 5\. Don’t leak referrer value

When you click on a link that navigates away from your website, the destination website will receive the URL of the last location on your website in a `referrer` header. That URL may contain sensitive and semi-sensitive data (like session tokens and user IDs), which should never be exposed.

To prevent leaking of `referrer` value, we set `Referrer-Policy` header to `no-referrer`:

```text
"Referrer-Policy": "no-referrer"
```

This value should be good in most cases, but if your application logic requires you to preserve referrer in some cases, check out [this article by Scott Helme](https://scotthelme.co.uk/a-new-security-header-referrer-policy/) where he breaks down all possible header values and when to apply them.

### 6\. Don’t set innerHTML value based on the user input

Cross-site scripting attack in which malicious code gets injected into a website can happen through a number of different DOM APIs, but the most frequently used is `innerHTML`.

You should never set `innerHTML` based on unfiltered input from the user. Any value that can be directly manipulated by the user -  be that text from an input field, a parameter from URL, or local storage entry - should be escaped and sanitized first. Ideally, use `textContent` instead of `innerHTML` to prevent generating HTML output altogether. If you do need to provide rich-text editing to your users, use well-established libraries that use whitelisting instead of blacklisting to specify allowed HTML tags.

Unfortunately, `innerHTML` is not the only weak point in DOM API, and the code susceptible to XSS injections can still be hard to detect. This is why it is important to always have a strict content security policy that disallows inline code execution.

For the future, you may want to keep an eye on a new [Trusted Types specification](https://developers.google.com/web/updates/2019/02/trusted-types) which aims to prevent all DOM-based cross-site scripting attacks.

### 7\. Use UI frameworks

Modern UI frameworks like React, Vue, and Angular have a good level of security built into them, and can largely eliminate the risks of XSS attacks. They automatically encode HTML output, reduce the need for using XSS-susceptible DOM APIs, and give unambiguous and cautionary names to potentially dangerous methods, like `dangerouslySetInnerHTML`.

### 8\. Keep your dependencies up to date

A quick look inside `node_modules` folder will confirm that our web applications are lego puzzles built out of hundreds (if not thousands) dependencies. Ensuring that these dependencies don’t contain any known security vulnerabilities is very important for the overall security of your website.

The best way to make sure that dependencies stay secure and up-to-date is to make vulnerability checking a part of the development process. To do that, you can integrate tools like [Dependabot](https://dependabot.com/) and [Snyk](https://snyk.io/), which will create pull requests for out-of-date or potentially vulnerable dependencies and help you apply fixes sooner.

### 9\. Think twice before adding third-party services

Third-party services like Google Analytics, Intercom, Mixpanel, and a hundred others can provide a “one line of code” solution to your business needs. At the same time, they can make your website more vulnerable, because if a third-party service gets compromised, then so will be your website.

Should you decide to integrate a third-party service, make sure to set the strongest CSP policy that would still permit that service to work normally. Most of the popular services have documented what CSP directives they require, so make sure to follow their guidelines.

Especial care should be taken when using Google Tag Manager, Segment, or any other tools that allow anyone in your organization to integrate more third-party services. People with access to this tool must understand the security implication of connecting additional services and ideally discuss it with their development team.

### 10\. Use Subresource Integrity for third-party scripts

For all third-party scripts that you use, make sure to include `integrity` attribute when possible. Browsers have [Subresource Integrity](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity) feature that can validate the cryptographic hash of the script that you’re loading and make sure that it hasn’t been tampered with.

This how your `script` tag may look like:

```html
<script src="https://example.com/example-framework.js"
        integrity="sha384-oqVuAfXRKap7fdgcCY5uykM6+R9GqQ8K/ux..."
        crossorigin="anonymous"></script>
```

It’s worth clarifying that this technique is useful for third-party libraries, but to a lesser degree for third-party services. Most of the time, when you add a script for a third-party service, that script is only used to load another dependant script. It’s not possible to check the integrity of the dependant script because it can be modified at any time, so in this case, we have to fall back on a strict content security policy.

## Conclusion

Save browsing experience is an important part of any modern web application, and the users want to be sure that their personal data remains secure and private. And while this data is stored on the backend, the responsibility for securing it extends to client-side applications as well.

There are many variations of UI-first attacks that malicious users can take advantage of, but you can greatly increase your chances of defending against them if you follow the recommendations given in this article.
