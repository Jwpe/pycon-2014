# Distributed Computing

## Handling Abusive Tasks

- Use hard/soft timeouts

## Single Points of Failure

Identify your single points of failure

- DB
- Broker

Eliminate the ones you can

- RabbitMQ Cluster
- Database Slaves

**Mitigate** those you can't

- Pre-connection checks

## Limiting Jobs

- Client calls; you're hammering their server
-

## Thundering Herd

- "occurs when a large no of processes waiting for an event are awoken when that event occurs, but only one process is able to proceed at a time"
- Add jitter to retries
- Zookeeper locks
    - arguably more correct

## Distributed Mutex

- allows only one task to run at a time

## Conclusions

- Be diligent
    - If you get careless you can introduce deadlocks
- Ask questions

## Questions