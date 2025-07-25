# Production Rails

:rocket: Best practices for running Rails in production

This guide covers different concepts you should be familiar with. Recommendations come from personal experience and work at [Instacart](https://www.instacart.com/opensource). A number of open source projects are ones I’ve created. For a comprehensive list of gems, check out [Awesome Ruby](https://awesome-ruby.com).

## Security

Everyone writing code must be responsible for security. See [best practices](https://github.com/ankane/secure_rails) and [how to secure sensitive data](https://ankane.org/sensitive-data-rails).

## Errors

Use an error reporting service like [Rollbar](https://rollbar.com/).

Use [Safely](https://github.com/ankane/safely) to rescue and report exceptions in non-critical code.

## Logging

Use a centralized logging service like [Mezmo](https://www.mezmo.com/).

Use [Lograge](https://github.com/roidrage/lograge) to reduce volume. Configure it to add `request_id`, `user_id`, and `params`.

```ruby
# config/environments/production.rb
config.lograge.enabled = true
config.lograge.custom_options = lambda do |event|
  options = event.payload.slice(:request_id, :user_id)
  options[:params] = event.payload[:params].except("controller", "action")
  options
end

# app/controllers/application_controller.rb
def append_info_to_payload(payload)
  super
  payload[:request_id] = request.uuid
  payload[:user_id] = current_user.id if current_user
end
```

## Audits

Use an auditing library like [Audited](https://github.com/collectiveidea/audited).

## Migrations

Use [Strong Migrations](https://github.com/ankane/strong_migrations) to catch unsafe migrations at dev time.

## Web Requests

Use compression. If not using a reverse proxy like nginx, use [Rack::Deflater](https://www.schneems.com/2017/11/08/80-smaller-rails-footprint-with-rack-deflate/).

Use a [CDN](https://en.wikipedia.org/wiki/Content_delivery_network) like [Amazon CloudFront](https://aws.amazon.com/cloudfront/) to serve assets.

Use [Slowpoke](https://github.com/ankane/slowpoke) for request timeouts.

## Background Jobs

Use a high performance background processing framework like [Sidekiq](https://github.com/mperham/sidekiq) with Active Job.

```ruby
config.active_job.queue_adapter = :sidekiq
```

Use [ActiveJob::TrafficControl](https://github.com/nickelser/activejob-traffic_control) to:

- quickly disable jobs
- throttle
- limit concurrency

```ruby
BadJob.disable!
```

## Email

Use an email delivery service. You can use one that supports both transactional and marketing email like [Postmark](https://postmarkapp.com/), or separate ones for each.

For styling, use a CSS inliner like [Roadie](https://github.com/Mange/roadie-rails).

```ruby
class ApplicationMailer < ActionMailer::Base
  include Roadie::Rails::Automatic
end
```

Add UTM parameters to links.

## Caching and Performance

Use [Memcached](https://memcached.org/) and [Dalli](https://github.com/petergoldstein/dalli) for caching.

```ruby
config.cache_store = :mem_cache_store
```

Use a library like [MemoWise](https://github.com/panorama-ed/memo_wise) for memoizing.

```ruby
memo_wise :time_consuming_method
```

## Monitoring

### Tracing

Use a performance monitoring service with transaction traces like [New Relic](https://newrelic.com/) or [AppSignal](https://appsignal.com/).

### Uptime

Use an uptime monitoring service like [Pingdom](https://www.pingdom.com/) or [Uptime Robot](https://uptimerobot.com/).

### Database

The database is a common bottleneck for Rails apps and deserves some special monitoring attention. There are some dedicated tools for this:

- If you use Postgres, [PgHero](https://github.com/ankane/pghero) can help identify issues
- Use [Active Record Query Logs](https://api.rubyonrails.org/classes/ActiveRecord/QueryLogs.html) to track the origin of SQL queries

### Notable Events

Use [Notable](https://github.com/ankane/notable) to track notable requests and background jobs.

- errors
- slow requests, jobs, and timeouts
- 404s
- validation failures
- CSRF failures
- unpermitted parameters
- blocked and throttled requests

## Timeouts

[Add timeouts](https://github.com/ankane/the-ultimate-guide-to-ruby-timeouts).

One very important place is Active Record. Add to `config/database.yml` and adjust as needed.

#### PostgreSQL

```yml
production:
  connect_timeout: 2
  checkout_timeout: 5
  variables:
    statement_timeout: 5000 # ms
```

#### MySQL and MariaDB

```yml
production:
  connect_timeout: 1
  read_timeout: 1
  write_timeout: 1
  checkout_timeout: 5
  variables:
    max_execution_time: 5000 # ms
    max_statement_time: 5 # sec
```

## Analytics

Use an analytics tool to measure important events. Consider a first-party library like [Ahoy](https://github.com/ankane/ahoy), or use a third-party service like [Amplitude](https://amplitude.com/) or [Mixpanel](https://mixpanel.com/).

## New Features

Use a feature flipper library like [Rollout](https://github.com/FetLife/rollout) to easily enable and disable new features without pushing code.

To A/B test features, use a library like [Field Test](https://github.com/ankane/field_test).

## Lastly...

Have suggestions? [Help make this guide better for everyone](https://github.com/ankane/rails-best-practices/issues/new).

Also check out [Development Rails](Development.md) and [Scaling Rails](Scaling.md).
