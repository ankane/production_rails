# Scaling Rails

:rocket: Best practices for scaling Rails

**Do not scale prematurely. Many of these add complexity. Also, do not do all of them. Evaluate each of them independently for your application.**

## Codebase

Keep the [majestic monolith](https://m.signalvnoise.com/the-majestic-monolith-29166d022228) as long as possible. Spend a good amount of time fixing pain (app boot times and test times are common sources). [Microservices](https://martinfowler.com/articles/microservices.html) and [self-contained services](http://scs-architecture.org/) add a huge amount of complexity.

Assign different teams as owners for different parts of the code for things like triaging errors. **Do not** discourge other teams from working in that part of the code. [Ownership](https://github.com/ankane/ownership) can help with this.

## Technology

Max out your existing technologies before introducing new ones. Even if your current technology isn’t the perfect fit for the job, there’s a huge cost associated with learning and operating something new.

## Database

The database is often the primary bottleneck when scaling.

- See which queries take the most total time. If you use PostgreSQL, [PgHero](https://github.com/ankane/pghero) can help. [Marginalia](https://github.com/basecamp/marginalia) can help identify their origin.
- Scale reads out by fixing n+1 queries, caching frequent queries, and using read replicas. Use [Distribute Reads](https://github.com/ankane/distribute_reads) for replicas.
- Scale writes with separate databases. Use [Multiverse](https://github.com/ankane/multiverse) for seperate databases.
- Scale space with separate database or partitioning. For Postgres, [pgslice](https://github.com/ankane/pgslice) can help with partitioning.
- Scale connections with a connection pooler. For Postgres, use [PgBouncer](https://github.com/ankane/shorts/blob/master/PgBouncer-Setup.md).

## Background Jobs

If you use [Sidekiq](https://github.com/mperham/sidekiq) and Redis CPU gets too high, try reducing the number of queues each type of worker is dedicated to. The number of Redis calls required to check queues is proportional to the number of queues a worker has to process.

---

:arrow_forward: [Back to home](https://github.com/ankane/rails-best-practices)
