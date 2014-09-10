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

# hack to bubble timeout errors
module ActionView
  class Template

    def handle_render_error_with_timeout(view, e)
      raise e if e.is_a?(Rack::Timeout::Error)
      handle_render_error_without_timeout(view, e)
    end
    alias_method_chain :handle_render_error, :timeout

  end
end
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

## Database Timeouts

Ensure database queries do not run for too long.

For PostgreSQL, create an initializer with:

```ruby
module ActiveRecord
  module ConnectionAdapters
    class PostgreSQLAdapter

      def configure_connection_with_statement_timeout
        configure_connection_without_statement_timeout
        ActiveRecord::Base.logger.silence do
          execute("SET statement_timeout = 5000")
        end
      end
      alias_method_chain :configure_connection, :statement_timeout

    end
  end
end
```

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

## Slow Requests

Keep track of slow requests

```ruby
ActiveSupport::Notifications.subscribe "process_action.action_controller" do |name, start, finish, id, payload|
  duration = finish - start
  if duration > 5.seconds
    # track it
  end
end
```

## Uptime Monitoring

Use an uptime monitoring service like [Pingdom](https://www.pingdom.com/) or [Uptime Robot](https://uptimerobot.com/).

Monitor web servers, background jobs, and scheduled tasks.

## Performance Monitoring

Use a performance monitoring service like [New Relic](http://newrelic.com/) or [AppSignal](https://appsignal.com/).

## Web Server

Use a high performance web server like [Unicorn](http://unicorn.bogomips.org/).

```ruby
gem 'unicorn'
```

## Security

Use SSL to protect your users. Add the following to `config/environments/production.rb`.

```ruby
config.force_ssl = true
```

## Thanks

- [cant_wait gem](https://github.com/CarlosCD/cant_wait) for database timeouts
