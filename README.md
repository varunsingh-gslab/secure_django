# Rails security best practices for developers

Everyone writing code must be responsible for security. :lock:

- Start with the [Rails Security Guide](http://guides.rubyonrails.org/security.html) to see how Rails protects you.

- Identify various application specific config variables, secret tokens, passwords, credentials etc and store them in a location that exist as long as application is running, or keep them served using any service, with Rails 5.1 one can have encrypted secrets file, read more about this [here](https://www.google.co.in/search?q=rails+secrets+encrypted) and [here](https://medium.com/poka-techblog/the-best-way-to-store-secrets-in-your-app-is-not-to-store-secrets-in-your-app-308a6807d3ed)

- Even with ActiveRecord, SQL injection is still possible if misused

  ```ruby
  User.group(params[:column])
  ```

  is vulnerable to injection. [Learn about other methods](https://rails-sqli.org)

- Don’t use standard Ruby interpolation (`#{foo}`) to insert user inputted strings into ActiveRecord or raw SQL queries. Use the `?` character, named bind variables or the [ActiveRecord::Sanitization methods](http://api.rubyonrails.org/classes/ActiveRecord/Sanitization/ClassMethods.html#method-i-sanitize_conditions) to sanitize user input used in DB queries

-  Don't pass user inputted strings to methods capable of evaluating code or running O.S. commands such as `eval`, `system`, `syscall`, `%x()`, `open`, `popen<n>`, `File.read`, `File.write`, `send`, `to_sym` and `exec`

- Sanitize the data/text/html/json that is getting rendered. [Be careful](https://product.reverb.com/2015/08/29/stay-safe-while-using-html_safe-in-rails/) with `html_safe`

- Use `json_escape` when passing variables to JavaScript, or better yet, a library like [Gon](https://github.com/gazay/gon)

  ```erb
  <script>
    var currentUser = <%= raw json_escape(current_user.to_json) %>;
  </script>
  ```

- Set `autocomplete="off"` for sensitive form fields, like credit card number

- Make sure sensitive request parameters are not logged

  ```ruby
  Rails.application.config.filter_parameters += [:credit_card_number, :password, :username, :login]
  ```

- Log all successful and failed login attempts and password reset attempts (check out [Authtrail](https://github.com/ankane/authtrail) if you use Devise)

- Notify users of password changes, notify users of email address changes - send an email to both the old and new address

- Protect sensitive data at rest with a library like [attr_encrypted](https://github.com/attr-encrypted/attr_encrypted) and possibly [KMS Encrypted](https://github.com/ankane/kms_encrypted). Further if necessary, keep rotating the keys/hash/salts used for encryption, keep track of latest encryption algorithms and their implementation libraries 

- Ask search engines not to index pages with secret tokens in the URL

  ```html
  <meta name="robots" content="noindex, nofollow">
  ```

- Ask the browser [not to cache pages](https://stackoverflow.com/a/748646) with sensitive information

  ```ruby
  response.headers["Cache-Control"] = "no-cache, no-store, max-age=0, must-revalidate"
  response.headers["Pragma"] = "no-cache"
  response.headers["Expires"] = "Sat, 01 Jan 2000 00:00:00 GMT"
  ```

- Prevent [host header injection](http://carlos.bueno.org/2008/06/host-header-injection.html) - add the following to `config/environments/production.rb`

  ```ruby
  config.action_controller.default_url_options = {host: "www.yoursite.com"}
  config.action_controller.asset_host = "www.yoursite.com"
  ```

- Use [SecureHeaders](https://github.com/twitter/secureheaders)

- Protect all data in transit with HTTPS - you can get free SSL certificates from [Let’s Encrypt](https://letsencrypt.org/)

  Add the following to `config/environments/production.rb`

  ```ruby
  config.force_ssl = true
  ```

- Add your domain to the [HSTS Preload List](https://hstspreload.org/)

  ```ruby
  config.ssl_options = {hsts: {subdomains: true, preload: true, expires: 1.year}}
  ```

- Rate limit login attempts with [Rack Attack](https://github.com/kickstarter/rack-attack)



## Open Source Tools

- [Brakeman](https://github.com/presidentbeef/brakeman) is a great static analysis tool - it scans your code for vulnerabilities, [please check out what all common security issues are covered by brakeman](https://github.com/presidentbeef/brakeman/tree/master/docs/warning_types)
- [bundler-audit](https://github.com/rubysec/bundler-audit) checks for vulnerable versions of gems

  ```sh
  gem install bundler-audit
  bundle audit check --update
  ```

  To fix `Insecure Source URI` issues with the `github` option, add to the top of your `Gemfile`:

  ```ruby
  git_source(:github) do |repo_name|
    repo_name = "#{repo_name}/#{repo_name}" unless repo_name.include?("/")
    "https://github.com/#{repo_name}.git"
  end
  ```

  And run `bundle install`.

- [nsp](https://github.com/nodesecurity/nsp) checks for vulnerable versions of JavaScript packages (if you use `package.json`)
- [git-secrets](https://github.com/awslabs/git-secrets) prevents you from committing sensitive info

  ```ruby
  brew install git-secrets
  git secrets --register-aws --global
  git secrets --install
  git secrets --scan
  ```

## Mailing Lists

Subscribe to [ruby-security-ann](https://groups.google.com/forum/#!forum/ruby-security-ann) to get security announcements for Ruby, Rails, Rubygems, Bundler, and other Ruby ecosystem projects.

## Services

- [Hakiri](https://hakiri.io/) monitors for dependency and code vulnerabilities
- [CodeClimate](https://codeclimate.com/) provides a hosted version of static analysis
- [HackerOne](https://hackerone.com/) allows you to enlist hackers to surface vulnerabilities

## Additional Reading

- [Rails’ Insecure Defaults](https://codeclimate.com/blog/rails-insecure-defaults/)
- [The Inadequate Guide to Rails Security](https://blog.honeybadger.io/ruby-security-tutorial-and-rails-security-guide/)
- [The Matasano Crypto Challenges](https://cryptopals.com/)
- [Web Application Security Zone by Netsparker](https://www.netsparker.com/blog/web-security/)
- [VERACODE: RUBY ON RAILS SECURITY](https://www.veracode.com/security/ruby-security)
- [OWASP Top10 2017](https://www.owasp.org/index.php/Category:OWASP_Top_Ten_2017_Project)

