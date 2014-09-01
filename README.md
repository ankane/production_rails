# Production Rails

Best practices for running Rails in production

## Error Reporting

Use an error reporting service like [Rollbar](https://rollbar.com/).

```ruby
gem 'rollbar'
```

Run:

```sh
rails generate rollbar
```

And add the following to your environment.

```sh
ROLLBAR_ACCESS_TOKEN=test123
```

## Timeouts

Use [Rack Timeout](https://github.com/heroku/rack-timeout).

```ruby
gem 'rack-timeout', '>= 0.1.0beta3'
```

Add the following to `config/initializers/timeout.rb`.

```ruby
Rack::Timeout.timeout = ENV["RACK_TIMEOUT"] || 5
Rack::Timeout.unregister_state_change_observer(:logger)
```

Show a custom error page for timeouts. Add the following to `app/controllers/application_controller.rb`.

```ruby
# from https://github.com/heroku/rack-timeout/issues/40#issuecomment-51865104
rescue_from Rack::Timeout::Error, with: :handle_request_timeout

private

def handle_request_timeout(e)
  Rollbar.report_exception(e)
  respond_to do |format|
    format.html { render file: Rails.root.join("public/503.html"), status: 503, layout: nil }
    format.all { head 503 }
  end
end
```

And create a `public/503.html`.

## Logging

Use [Lograge](https://github.com/roidrage/lograge).

```ruby
gem 'lograge'
```

Add the following to `config/environments/production.rb`.

```ruby
config.lograge.enabled = true
config.lograge.custom_options = lambda do |event|
  event.payload.slice(:user_id, :visit_id)
end
```

Add the following to `app/controllers/application_controller.rb`.

```ruby
def append_info_to_payload(payload)
  super
  payload[:user_id] = current_user.id if current_user
  payload[:visit_id] = ahoy.visit_id # if you use Ahoy
end
```

## Security

Use SSL to protect your users. Add the following to `config/environments/production.rb`.

```ruby
config.force_ssl = true
```
