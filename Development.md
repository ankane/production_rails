# Development Rails

:rocket: Best practices for developing with Rails

## Philosophy

Always, always manually test what you’re building in development. It’s extremely important to have a fast feedback loop. Testing on staging or production is slow. If it’s not easy to test in development, spend time to make it easy.

## Environment

Use [dotenv-rails](https://github.com/bkeepers/dotenv) for environment variables. Do not check this in. Create a `.env.example` without secrets.

Use [Foreman](https://github.com/ddollar/foreman) to manage multiple processes.

## Debugging

Set up debugging tools behind environment variables so you can enable and disable without changing code.

Use [ruby-prof](https://github.com/ruby-prof/ruby-prof) to profile code.

Use [Bullet](https://github.com/flyerhzm/bullet) to find n + 1 queries.

## Console

Enable console history and autocomplete in your `~/.irbrc`.

```ruby
require "irb/completion"
require "irb/ext/save-history"
IRB.conf[:SAVE_HISTORY] = 10000

# and for awesome print
require "awesome_print"
AwesomePrint.irb!
```

Use [Awesome Print](https://github.com/michaeldv/awesome_print) or [pry-rails](https://github.com/rweng/pry-rails) for a friendlier console.

## Logging

Use [Cacheflow](https://github.com/ankane/cacheflow) to instrument caches.

## Email

Use [Letter Opener](https://github.com/ryanb/letter_opener) for email.

## Data

Use a tool like [pgsync](https://github.com/ankane/pgsync) to sync data from another enviroment.

## Aliases

Use aliases for common commands. Here are a few of my favorites:

```sh
# rails
alias rc="bin/rails console"
alias dbm="bin/rails db:migrate"
alias dbr="bin/rails db:rollback"
alias fsw="foreman start -c web=1"

# git
alias gl="git pull -r"
alias gc="git commit"
alias gco="git checkout"
alias gcm="git checkout master"
```
