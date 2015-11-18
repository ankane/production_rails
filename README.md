# Production Rails

Best practices for running Rails in production.

Disclaimer: :gem: indicates one of my gems

## Security

Everyone writing code must be responsible for security. [Best practices](https://github.com/ankane/secure_rails)

## Analytics

Use an analytics service like [Google Analytics](http://www.google.com/analytics/) or [Mixpanel](https://mixpanel.com/).

And possibly an open source library like [Ahoy](https://github.com/ankane/ahoy). :gem:

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

## Audits

Use an auditing library like [Audited](https://github.com/collectiveidea/audited).

## Errors

Use an error reporting service like [Rollbar](https://rollbar.com/).

## Monitoring

- There are [two important metrics](https://github.com/ankane/shorts/blob/master/Two-Metrics.md) to track for web servers
- Use an uptime monitoring service like [Pingdom](https://www.pingdom.com/) or [Uptime Robot](https://uptimerobot.com/) - monitor web servers, background jobs, and scheduled tasks
- Use a performance monitoring service like [New Relic](http://newrelic.com/) or [AppSignal](https://appsignal.com/)

### What to Monitor

##### Web Requests

- requests by action - total time, count
- queue time - X-Request-Start header

##### Background Jobs and Rake Tasks

- jobs by type - total time, count

##### Data Stores - Database, Elasticsearch, Redis

- requests by type - total time, count
- CPU usage
- space

##### External Services

- requests by type - total time, count

### Notable Events

Use [Notable](https://github.com/ankane/notable) to track notable requests and background jobs. :gem:

- errors
- slow requests, jobs, and timeouts
- 404s
- validation failures
- CSRF failures
- unpermitted parameters
- blocked and throttled requests

## Timeouts

[Add timeouts](https://github.com/ankane/the-ultimate-guide-to-ruby-timeouts).

Add them to:

- [web requests](https://github.com/ankane/the-ultimate-guide-to-ruby-timeouts#rack-middleware)
- [database connections](https://github.com/ankane/the-ultimate-guide-to-ruby-timeouts#activerecord)
- and more

## Performance

- Use a high performance web server like [Puma](https://devcenter.heroku.com/articles/deploying-rails-applications-with-the-puma-web-server)
- Use [Rack::Deflater](https://robots.thoughtbot.com/content-compression-with-rack-deflater) for compression
- Add [Oj](https://github.com/ohler55/oj) to speed up JSON parsing
- Use [Memcached](https://github.com/mperham/dalli) for caching
- Use a [CDN](https://en.wikipedia.org/wiki/Content_delivery_network) like [Amazon CloudFront](https://aws.amazon.com/cloudfront/) to serve assets

## New Features

Use a feature flipper library like [Rollout](https://github.com/FetLife/rollout) to easily enable and disable new features without pushing code.

## Migrations

[Read this](http://pedro.herokuapp.com/past/2011/7/13/rails_migrations_with_no_downtime/).

## Development Bonus

If you experience [double logging in the Rails console](https://github.com/rails/rails/issues/11415), create `config/initializers/logger.rb` with:

```ruby
ActiveSupport::Logger.class_eval do
  def self.broadcast(logger)
    Module.new do
    end
  end
end
```

## Lastly...

Have suggestions? [Help make this guide better for everyone](https://github.com/ankane/production_rails/issues/new).

If you use Heroku, check out [Rails on Heroku](https://github.com/ankane/shorts/blob/master/Rails-on-Heroku.md).
