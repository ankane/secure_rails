# Secure Rails

Everyone writing code must be responsible for security. :lock:

Start with the [Rails Security Guide](http://guides.rubyonrails.org/security.html) to see how Rails protects you.

## Best Practices

- Keep secret tokens out of your code - `ENV` variables are a good practice

- Even with ActiveRecord, SQL injection is still possible if misused

  ```ruby
  User.group(params[:column])
  ```

  is vulnerable to injection. [Learn about other methods](http://rails-sqli.org)

- Use [SecureHeaders](https://github.com/twitter/secureheaders)

- Protect all data in transit with HTTPS - you can get free SSL certificates from [Let’s Encrypt](https://letsencrypt.org/)

  Add the following to `config/environments/production.rb`

  ```ruby
  config.force_ssl = true
  ```

- Add your domain to the [HSTS Preload List](https://hstspreload.org/)

  Add the following to `config/application.rb`

  ```ruby
  SecureHeaders::Configuration.default do |config|
    config.hsts = "max-age=#{1.year.to_i}; includeSubdomains; preload"
  end
  ```

- Protect sensitive data at rest with a library like [attr_encrypted](https://github.com/attr-encrypted/attr_encrypted)

- Prevent [host header injection](http://carlos.bueno.org/2008/06/host-header-injection.html) - add the following to `config/environments/production.rb`

  ```ruby
  config.action_controller.default_url_options = {host: "www.yoursite.com"}
  config.action_controller.asset_host = "www.yoursite.com"
  ```

- Set `autocomplete="off"` for sensitive form fields, like credit card number

- Make sure sensitive request parameters aren’t logged

  ```ruby
  Rails.application.config.filter_parameters += [:credit_card_number]
  ```

- Use a trusted library like [Devise](https://github.com/plataformatec/devise) for authentication (see [Hardening Devise](https://github.com/ankane/shorts/blob/master/Hardening-Devise.md) if applicable)

- Notify users of password changes

- Notify users of email address changes - send an email to both the old and new address

- Rate limit login attempts with [Rack Attack](https://github.com/kickstarter/rack-attack)

- Log all login attempts (successful and failed) and password reset attempts

- Rails has a number of gems for [authorization](https://www.ruby-toolbox.com/categories/rails_authorization) - we like [Pundit](https://github.com/elabs/pundit)

- Ask search engines not to index pages with secret tokens in the URL

  ```html
  <meta name="robots" content="noindex, nofollow">
  ```

- Ask the browser [not to cache pages](http://stackoverflow.com/a/748646) with sensitive information

  ```ruby
  response.headers["Cache-Control"] = "no-cache, no-store, max-age=0, must-revalidate"
  response.headers["Pragma"] = "no-cache"
  response.headers["Expires"] = "Sat, 01 Jan 2000 00:00:00 GMT"
  ```

- Use `json_escape` when passing variables to JavaScript (or a library like [Gon](https://github.com/gazay/gon))

  ```erb
  <script>
    var currentUser = <%= raw json_escape(current_user.to_json) %>;
  </script>
  ```

- [Be careful](https://product.reverb.com/2015/08/29/stay-safe-while-using-html_safe-in-rails/) with `html_safe`

- If you still use `attr_accessible`, [upgrade to strong_parameters](https://github.com/ankane/shorts/blob/master/Strong-Parameters.md)

## Open Source Tools

- [Brakeman](https://github.com/presidentbeef/brakeman) is a great static analysis tool - it scans your code for vulnerabilities
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

## Mailing Lists

Subscribe to [ruby-security-ann](https://groups.google.com/forum/#!forum/ruby-security-ann) to get security announcements for Ruby, Rails, Rubygems, Bundler, and other Ruby ecosystem projects.

## Services

- [CodeClimate](https://codeclimate.com/) provides a hosted version of static analysis
- [HackerOne](https://hackerone.com/) allows you to enlist hackers to surface vulnerabilities

## Additional Reading

- [Rails’ Insecure Defaults](http://blog.codeclimate.com/blog/2013/03/27/rails-insecure-defaults/)
- [The Inadequate Guide to Rails Security](http://blog.honeybadger.io/ruby-security-tutorial-and-rails-security-guide/)
- [The Matasano Crypto Challenges](http://cryptopals.com/)

## Contributing

Have other good practices? Know of more great tools? [Help make this guide better for everyone](https://github.com/ankane/secure_rails/issues/new).
