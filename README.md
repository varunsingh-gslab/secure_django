# Django security

Everyone writing code must be responsible for security. 🔒  


  * Start with the [Django security guide](https://docs.djangoproject.com/en/2.1/topics/security/) to see how Django protects you and various security related configs available. 

  * Identify various application specific config variables, secret tokens, passwords, credentials etc and store them in a location that exist as long as application is running, or keep them served using any service. For more security, one can have encrypted secrets file, read more about this [here](https://www.google.co.in/search?q=django+encrypted+secrets&oq=django+encrypted+secrets&aqs=chrome..69i57j69i64.5623j0j7&sourceid=chrome&ie=UTF-8) and [here](https://hackernoon.com/4-ways-to-manage-the-configuration-in-python-4623049e841b)

  * Even with Django's ORM, SQL injection is still possible if misused. [Learn about other methods](https://docs.djangoproject.com/en/2.0/topics/db/sql/#)

  * Don't pass user inputted strings to methods capable of evaluating code. Please read [this](https://docs.python.org/2/library/subprocess.html#frequently-used-arguments) for more details.

  * Below points about XSS are taken from [stackoverflow](https://security.stackexchange.com/a/27807). Credit to [D.W](https://security.stackexchange.com/users/971/d-w):

    * Make sure you quote all attributes where dynamic data is inserted. Good: `<img alt="{{foo}}" ...>`. Bad: `<img alt={{foo}} ...>`. Django's auto-escaping isn't sufficient for unquoted attribute values. 

    * For data inserted into CSS (`style` tags and attributes) or Javascript (`script` blocks, event handlers, and `onclick`/etc. attributes), you must manually escape the data using escaping rules that are appropriate for CSS or Javascript. 

    * For data inserted into an attribute where a URL is expected (e.g., `a href`, `img src`), you must manually validate the URL to make sure it is safe. You need to check the protocol against a whitelist of allowed protocols (e.g., `http:`, `https:`, `mailto:`, `ftp:`, etc. -- but definitely not `javascript:`). 

    * For data inserted inside a comment, you have to do something extra to escape `-s`. Actually, better yet, just don't insert dynamic data inside a HTML comment; that's an obscure corner of HTML that's just asking for subtle problems. (Similarly, don't insert dynamic data into attributes where it is crazy for user input to appear (e.g., `foo class=...`).) 

    * You still must avoid client-side XSS (also known as DOM-based XSS) separately; Django doesn't help you with this. YOu could read [Adam Barth's advice on avoiding XSS](http://www.educatedguesswork.org/2011/08/guest_post_adam_barth_on_three.html) for some guidelines that will help you avoid client-side XSS. 

    * If you use `mark_safe` to manually tell that Django that some data has already been escaped and is safe, you'd better know what you're doing, and it really better be safe. It's easy to make mistakes here if you don't know what you are doing. For example, if you generate HTML programmatically, store it into the database, and later send it back to the client (unescaped, obviously), it's very easy to make mistakes; you're on your own, and the auto-escaping won't help you.   


  * Set `autocomplete="off"` for sensitive form fields, like credit card number 

  * Make sure sensitive request parameters are not logged using Django logger. Read [this](https://docs.djangoproject.com/en/1.11/howto/error-reporting/#filtering-sensitive-information) for more information on how to achieve this

  * Use auth mechanisms provided by Django or Django Rest Framework if applicable. Avoid rolling your own Authentication.

  * Expire the session at log out and expire old sessions at every successful login, consider limiting the number of simultaneous sessions per account 

  * Log all successful and failed login attempts and password reset attempts

  * Implement Captcha or Negative Captcha on publicly exposed forms 

  * Encrypt cookies even if an SSL connection is used. This way, an attacker will not be able to view or modify cookies even if they are intercepted, and the user will not be able to read and edit cookies in the browser 

  * Notify users of password changes, notify users of email address changes - send an email to both the old and new address 

  * Protect sensitive data at rest with a library like [django-encrypted-fields](https://github.com/defrex/django-encrypted-fields). Further if necessary, keep rotating the keys/hash/salts used for encryption, keep track of latest encryption algorithms and their implementation libraries 

  * Don't install development/test-related apps such as [django-debug-toolbar](https://django-debug-toolbar.readthedocs.io/en/stable/) in the production environment. Keep the availability of such tools configurable so they are only available in DEBUG mode in development environment

  * Avoid exposing numerical/sequential record IDs in URLs, form HTML source and APIs. Consider using slugs (A.K.A. friendly IDs, vanity URLs) to identify records instead of numerical IDs, as given by the [SlugField](https://docs.djangoproject.com/en/2.1/ref/models/fields/#slugfield) by Django’s ORM. Additional benefits include SEO and better-looking URLs 

  * When using slugs instead of numerical IDs for URLs, consider returning a `404 Not Found` status code instead of `403 Forbidden` for authorization errors. Prevents leakage of attribute values used to generate the slugs. For instance, visiting [www.myapp.com/users/john-doe](http://www.myapp.com/users/john-doe) and getting a `403` return status indicates the application has a user named John Doe.

  * Ask search engines not to index pages with secret tokens in the URL 

```html
<meta name="robots" content="noindex, nofollow">
```

  * Ask the browser [not to cache pages](https://stackoverflow.com/a/748646) with sensitive information 

```html
response.headers["Cache-Control"] = "no-cache, no-store, max-age=0, must-revalidate"
response.headers["Pragma"] = "no-cache"
response.headers["Expires"] = "Sat, 01 Jan 2000 00:00:00 GMT"
```

  * Prevent [host header injection](http://carlos.bueno.org/2008/06/host-header-injection.html) - by whitelisting only certain hosts in ALLOWED_HOST setting 

```python
ALLOW_HOST = [‘192.168.0.1’, ‘192.168.0.2’]
```

  * Use [django-http-security-headers](https://github.com/mazlum/django-http-security-headers)

  * Protect all data in transit with HTTPS - you can get free SSL certificates from [Let’s Encrypt](https://letsencrypt.org/). Read this for more in

  * Add your domain to the [HSTS Preload List](https://hstspreload.org/). Enable HSTS. Read [this](https://docs.djangoproject.com/en/1.11/ref/middleware/#http-strict-transport-security) for more info

  * Rate limit login attempts with [django-axes](https://pypi.org/project/django-axes/) or using [Throttling](http://www.django-rest-framework.org/api-guide/throttling/) if using Django Rest Framework. This is to mitigate the risk of brute-force login attempts

  * You might be a busy person. Security is not a very visible feature in most applications, and so sometimes it’s not a priority. Of course you know it’s important, but how can you fit it into your busy schedule? The answer may be in the power of habits, especially [mini habits](https://www.airpair.com/ruby-on-rails/posts/a-week-with-a-rails-security-strategy)




## Open Source Tools 

  * You can use [static analysis tools](https://www.google.co.in/search?q=django+security+analysis+tools&oq=django+security+analysis+tools&aqs=chrome..69i57j69i64.5334j0j4&sourceid=chrome&ie=UTF-8) to uncover security vulnerabilities. 

  * pyupio checks for installed dependencies for known security vulnerabilities

```bash
pip install safety
safety check
```

  * [nsp](https://github.com/nodesecurity/nsp) checks for vulnerable versions of JavaScript packages (if you use package.json) 

  * [git-secrets](https://github.com/awslabs/git-secrets) prevents you from committing sensitive info 

```bash
brew install git-secrets
git secrets --register-aws --global
git secrets --install
git secrets --scan
```
