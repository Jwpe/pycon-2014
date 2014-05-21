# Postgres is Web Scale

## Why NoSQL?

- Let's use files for speed

### What NoSQL Sacrifices

- ACID

## What is PostgreSQL

- Been developed since 1986

### What can you sacrifice in PostgreSQL?

- Atomicity can't be turned off
- Consistency can be
    - No FOREIGN KEYs
- Isolation can't be turned off
- Durability can be turned off
    - asynchronous commit
    - use UNLOGGED tables (super fast, but table disappears after crash)

## Scaling

 - First you have a single DB
 - Then you split functionality (vertical scaling)
 - Then you hit a signle table which is too big
 - Split the table, taking along some related tables (sharding)

