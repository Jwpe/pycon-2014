
## pg_stat_statements

- Lets you look at queries that have been run
    - average time it takes
    - number of times it's been run
- You can see the expensive queries and where to optimize

## Indexes

- Which do I use?

### GIN (Generalized Inverted Index)

- Use with multiple values in one column
- Arrays, JSON

### KNN

- Similarity

### SP-GIST

- Good for phone numbers

### VODKA

- Space-partitioned GIN index?

### Functional Index

- Do something to data and make an index on that result

### Combined Conditional/Functional

- Cool

### CREATE INDEX CONCURRENTLY

- Slower
- Non-locking

## Data Types

### JSON

- Currently just text validation
- Gonna be better in 9.4
- JSONB = hstore2 with JSON and indexes

### hstore

- One-deep hash
- Works with indexes

## Pooling

- Application/Framework layer
- Standalone daemon
    - pgbouncer
    - pgpool

## Adding Cache

### Replication

- Wal-e

## Backups

### Logical

- pg_dump
- <50GB
- Has load on DB

### Physical

- Bytes on disk
- Limited system load
- Use above 50GB

## OLTP

- Web apps
- Ensure bulk of data is cached
- pg_bouncer

## Questions

- Heavy write load?
    - Run postgres with unlocked tables
    - Don't overuse indexes