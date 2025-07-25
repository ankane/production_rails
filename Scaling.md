# Scaling Rails

:rocket: Best practices for scaling Rails

**Do not scale prematurely. Many of these add complexity. Also, do not do all of them. Evaluate each of them independently for your application.**

## Codebase

Keep the [majestic monolith](https://signalvnoise.com/svn3/the-majestic-monolith/) as long as possible. Spend a good amount of time fixing pain (app boot times and test times are common sources). [Microservices](https://martinfowler.com/articles/microservices.html) or multiple [self-contained systems](https://scs-architecture.org/) add a huge amount of complexity.

Assign different teams as owners for different parts of the code for things like triaging errors. **Do not** discourge other teams from working in that part of the code. [Ownership](https://github.com/ankane/ownership) can help with this.

## Technology

Max out your existing technologies before introducing new ones. Even if your current technology isn’t the perfect fit for the job, there’s a huge cost associated with learning and operating something new.

## Database

The database is often the primary bottleneck when scaling.

- See which queries take the most total time. If you use PostgreSQL, [PgHero](https://github.com/ankane/pghero) can help. [Query logs](https://api.rubyonrails.org/classes/ActiveRecord/QueryLogs.html) can help identify their origin.
- Scale reads out by fixing n+1 queries, caching frequent queries, and using read replicas. Use [Distribute Reads](https://github.com/ankane/distribute_reads) for replicas.
- Scale writes with separate databases.
- Scale space with separate database or partitioning. For Postgres, [pgslice](https://github.com/ankane/pgslice) can help with partitioning.
- Scale connections with a connection pooler. For Postgres, use [PgBouncer](https://ankane.org/pgbouncer-setup).

## Background Jobs

If you use [Sidekiq](https://github.com/mperham/sidekiq) and Redis CPU gets too high, try reducing the number of queues each type of worker is dedicated to. The number of Redis calls required to check queues is proportional to the number of queues a worker has to process.
