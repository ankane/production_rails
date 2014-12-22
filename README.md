# Production Rails

Best practices for running Rails in production

## Errors

Use an error reporting service like [Rollbar](https://rollbar.com/).

## Timeouts

Use [Slowpoke](https://github.com/ankane/slowpoke) for request and database timeouts. :gem: *disclaimer: one of my gems*

## Throttling

Use [Rack Attack](https://github.com/kickstarter/rack-attack) to throttle and block requests.

## Audits

Use an auditing library like [Audited](https://github.com/collectiveidea/audited).

## Logging

Use [Lograge](https://github.com/roidrage/lograge).

```ruby
gem 'lograge'
```

Add the following to `config/environments/production.rb`.

```ruby
config.lograge.enabled = true
config.lograge.custom_options = lambda do |event|
  options = event.payload.slice(:request_id, :user_id, :visit_id)
  options[:params] = event.payload[:params].except("controller", "action")
  # if you use Searchkick
  if event.payload[:searchkick_runtime].to_f > 0
    options[:search] = event.payload[:searchkick_runtime]
  end
  options
end
```

Add the following to `app/controllers/application_controller.rb`.

```ruby
def append_info_to_payload(payload)
  super
  payload[:request_id] = request.uuid
  payload[:user_id] = current_user.id if current_user
  payload[:visit_id] = ahoy.visit_id # if you use Ahoy
end
```

## Uptime Monitoring

Use an uptime monitoring service like [Pingdom](https://www.pingdom.com/) or [Uptime Robot](https://uptimerobot.com/).

Monitor web servers, background jobs, and scheduled tasks.

## Performance Monitoring

Use a performance monitoring service like [New Relic](http://newrelic.com/) or [AppSignal](https://appsignal.com/).

Be sure to monitor:

### Web Requests

- requests by action - total time, count
- queue time - `X-Request-Start` header

### Background Jobs and Rake Tasks

- jobs by type - total time, count

### Data Stores - Database, Elasticsearch, Redis

- requests by type - total time, count
- CPU usage
- space

### External Services

- requests by type - total time, count

## Additional Monitoring

Use [Notable](https://github.com/ankane/notable) to track notable requests and background jobs. :gem: *disclaimer: one of my gems*

- errors
- slow requests, jobs, and timeouts
- 404s
- validation failures
- CSRF failures
- unpermitted parameters
- blocked and throttled requests

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

## Development Bonus

[Fix double logging](https://github.com/rails/rails/issues/11415#issuecomment-57648388) in the Rails console. Create `config/initializers/log_once.rb` with:

```ruby
ActiveSupport::Logger.class_eval do
  def self.broadcast(logger)
    Module.new do
    end
  end
end
```

## TODO

- Redis timeout
- Elasticsearch timeout
- Background jobs
- Scheduled jobs

## Thanks

- [cant_wait gem](https://github.com/CarlosCD/cant_wait) for database timeouts
