
# Postgres for Python Programmers

## Debian CLI

- `pgcreatecluster` - create a new cluster of databases. Useful if you want to run two different versions of postgres concurrently.

- `service postgresql start` - start postgres

- `service postgresql stop` - stop postgres


## DB Creation Tips

- always create DBs as `UTF8`

## Postgres configuration files

- `pg_hba.conf` and `postgresql.conf` are the main config files. `pg_hba.conf` controls login, `postgresql.conf` controls server config.

### Important Params

#### Logging

- Be generous with logging
- Low impact
- Best source of info for debugging
- `log_destination` controls how to log e.g. `syslog`
- `log_directory` controls the directory to log into e.g. `pg_log`

#### Memory

- `shared_buffers`
    - below 2GB: 20%
    - above 32GB: 8GB

- `work_mem`
    - 32-64 MB
    - look for 'temporary file' lines in logs
    - set for 2-3x the largest temp file you see
    - Huge speed-up if set correctly
    - Remember, each connection can spawn multiple nodes so don't set this too high

- `maintenance_work_mem` = 10% of sys memory up to 1GB

#### Checkpoints

- Everything that is in `shared_buffers` gets flushed to disk
- Potentially a lot of I/O

- `wal_buffers` = 16MB
- `checkpoint_completion_target` = 0.9
- `checkpoint_timeout` = between 10m and 30m
- `checkpoint_segments` = arbitrary, start at 32

- Now look in the log for how often checkpoint entries are occurring. If they are
happening more often than `checkpoint_timeout`, increase the number of
`checkpoint_segments`

#### Planner settings

- `effective_io_concurrency` - Set to the number of I/O channels. Or ignore
if you don't know the hardware config.

- `random_page_cost` - By default 4.0, bring down to 3.0 for raid, EBS or SSD
should be 1.1.

#### DO NOT TOUCH

- `fsync = on`
- `synchronous_commit`

***

- Most settings just need a reload
- Some, like `shared_buffers`, need a full server restart
- Some can be changed on the fly with `set local <param> = <value>;`


## Users and roles

- A 'role' is a db object that can own other objects
- A 'user' is a login
- Don't use the 'postgres' superuser for apps
- However, you will probably have to grant schema-modification privileges to
your app user

### User security
- Turn on SSL

### pg_hba.conf

- Method:
    - trust: if they can log into the system, they can do anything
    - peer: if the unix and postgres user match, let them log in
    - md5: md5-hashed password

## The WAL (Write-Ahead Log)

- Write-ahead log governs replication, crash-recovery, etc.
- When each transaction is committed, it is logged to the WAL
- Pushed to disk *before* the transaction is confirmed
- If the system crashes, the WAL is 'replayed'

### pg_xlog

- WAL is stored in the `pg_xlog` directory
- Don't mess with it
- Postgres recycles them

### Archiving

- `archive_command`
- Runs a command each time a WAL segment completes
- For example: copy and move to someplace safe
- Therefore you can recover from terrible things like a full crash

### synchronous_commit

- 'on': COMMIT doesn't return until WAL flush is done
- 'off' COMMIT returns when WAL flush is queued
- Can lose transactions on a crash, but faster
- Data cannot be **corrupted**, just lost

## Backup + Recovery

### pg_dump

- takes a logical snapshot of the db at the moment it **starts**
- runs concurrently with everything except schema changes
- no locks/writes
- low, non-zero load on the database

### pg_restore

- recreates the DB from a `pg_dump` output
- N.B. *most* of the time is recreating indeces since these aren't copied over


- Back up globals with `pg_dumpall --globals-only`
- Dump with `pg_dump --format=custom`
- Restore in parallel

### PITR backup/recovery

- Snapshot of the data directory...
- ...will not be consistent, but if we have the WAL segments that were created
after it we can restore consistency.

- Check out [WAL-E](http://github.com/wal-e/wal-e)

#### Recovery

- Copy disk image
- Set up recovery.conf to point to WAL files
- Start up Postgres and let it recover
- More WAL files = slower
- Takes 10-20% of the time it took to make the WAL files originally
- N.B. don't **have** to replay the entire WAL. Can stop at a safe/stable point


## Replication

- What if we just sent the WAL to another server?
- That is postgres replication

### WAL Archiving

- Each 16MB segment is sent to the secondary when complete
- Secondary reads and applies it
- Make use WAL is copied atomically (e.g. using `rsync`)

### Streaming Replication

- Pushes WAL down through a TCP/IP connection (postgres connection)
- Secondary continuously applies these
- `recovery.conf` orchestrates this. Lives in `$PGDATA`
- `restore_command` tells postgres what command to restore from e.g. `wal-fetch`

### Disaster Recovery

- What if AWS region goes down?
- Have a recovery plan from a remote site (e.g. Multi-AZ)
- Probably best done with `wal_archive`

### pg_basebackup

- Utility for doing a snapshot of a running server
- Easy way to take a snapshot for a secondary

### The bad parts of replication

- Entire dbs or none
- Can't write *anything* else to the secondary, even temporary tables. Sucks for data analysis

### Trigger-based Replication

- Only want to replicate a certain set of tables
- Installs triggers on tables on the master DB
- A daemon watches these and applies them to secondaries
- E.g. slony
- Good:
    - Highly configurable
- Bad:
    - Really fiddly and complex to set up
    - Schema changes have to be pushed out manuallys
    - Overhead

## Transactions, MVCC and VACCUUM

### Transactions

- An atomic thing to add to the DB
- Once the COMMIT completes, data has been written to storage
- If a database crashes, any transactions will be COMMITed or not - all or nothing
- No transaction can see another
- In postgres, *everything* is a transaction
- Even schema changes (e.g. South)

#### Warnings
- Can only rollback the current transaction
- Resources are held until the end of transactions e.g. locks
- Be aware of IDLE IN TRANSACTION sessions

### MVCC
- How do we deal with concurrency?
    - Lock the database (old Mongo)
    - Lock the table (new Mongo)
    - Lock the tuple

- Actually, postgres creates multiple 'versions' of the DB for each transaction
- Readers don't block readers
- Readers don't block writ theers
- Writers don't block readers
- Writers block writers *to the same tuple*

#### Snapshots

- Can't 'fork' DB and combine two changes to the same thing
- So transactions are blocked if they try to hit the same tuple
- REPEATABLE READ and SERIALIZABLE take snapshots when transaction begins
- Not every conflict can be detected at the single-tuple level.
- Eg INSERTING values

- A consequence of MVCC is that deleted tuples are not immediately freed
because there may be multiple snapshots.
- End up with dead tuples
- So therefore we have: VACUUM

### VACUUM

- Clears out dead tuples.
- Run automatically by AUTOVACUUM.
- AUTOVACUUM also runs ANALYZE, which tells the query planner how to make queries
- VACUUM suffers with high write rates

#### Optimizing
- Reduce autovacuum sleep time
- Increase autovacuum time

## Schema Design

- Keep attributes of entities in their entities
- Consistent plural/singular tables
- lower_case table names
- "Normalization" means not repeating data in places
- "Denormalization" is sometimes useful for storing calculated values
so you don't have to do joins over tables loads. For simple values,
don't copy

### Joins

- PostgreSQL executes joins efficiently
- Don't be afraid!
- Especially don't worry about large sets joining small sets

### Types

- Use the typing system
- Use domains to create custom types
- NO POLYMORPHIC FIELDS

### Constraints

- Use them. Cheap + fast.
- Multi-column constraints are good
- Exclusion constraints for constraints on multiple rows

- Avoid EAV schemas. What even is that?

### Keys

- Use UUIDs instead of serials to avoid merging tables

### Fast/Slow:

- If a table has a frequently-updated section and a slowly-updated section, consider splitting the table. (Impact/Engagement?)

### Arrays:
- Good. Often better than many to many.

### HStore:
- Good for optional variables. (Languages on NewsEntities?)

### JSONB:
- Woooooooooo

### NULL:
- NULL sucks
- **only** use it to mean missing value
- NULL != NULL

### Very Large Objects

- Don't store images/video in the DB or you are an idiot
- Store metadata that points to them
- Database API is not designed for this

### Many-to-many Tables

- These are big
- Consider replacing with array fields, either one or both directions
    - can use a trigger to maintain integrity
    - Up to about a megabyte/250k entries is practical
- M2M = N^2 growth pattern

### Time representation

- `TIMESTAMPTZ` = timestamp in UTC
- `TIMESTAMP` = timestamp in some timezone you hope you know what
it is

## Indexes

- Databases should work identically with or without indexes

### B-Tree
- Standard postgres index
- Single-, multi-column and functional indexes

- Create when query:
    - is going to select <15% of the rows
    - uses comparison operator
    - run frequently
- Could be ascending or descending

### Partial Indexes
- Can add a WHEN clause to limit the size of an index (activity feed?)

### Other index types
- GiST - geometric data
- KNN indexes - nearest neighbour
- GIN - text search

### Creating indexes

- `CREATE INDEX CONCURRENTLY` : slower but non-blocking

### Checking index usage

- `pg_stat_user_indexes`


## Queries

### Cost

- EXPLAIN
- Measured in arbitrary units
- First number is startup cost (get first tuple), second is total cost (get all tuples)
- Costs are inclusive of sub-nodes

- EXPLAIN ANALYZE uses actual ms, but executes query
- Also returns estimated and actual rows - big differences are bad

### Bad things

- JOINS between two very large tables
- Sequential scan on large tables
- SELECT COUNT(*)


## Identifying slowness

- `pg_stat_activity`
- `tail -f` the logs
- `iostat` to check I/O usage
- `pg_locks`
- Bounce between `stat_activity` and `locks` to identify bad transactions

## Python Particulars

### psycopg2 for python2

- The result set of a query is loaded into client memory when the query completes
- Regardless of the size of the result set
- If you want to scroll through results, use a named cursor(?)

### py-postgresql

- Python 3.x

### Django
- If you are running on 1.6+, always use @atomic
- cluster writes into small transactions
- multi-DB is awesome for hot standby (writes to master, reads from secondary)

### South
- Use it.
- Create manual migrations for schema changes that Django can't handle.

## Special Situations

### Minor version upgrades
- Do these
- Could be for security or data corruption
- Only need to install new binaries

### Major version upgrades
- Requires planning
- `pg_upgrade` is now reliable
- Trigger-based replication is another option for zero downtime
- A full `pg_dump`/`pg_restore` is the way to go for safety
- All parts of a replication set need to be upgraded at the same time

### Bulk-loading Data
- Use COPY not INSERT
- VACUUM after

### Very high insert rates
- Reduce shared buffers
- Drop down

### AWS
- Generally, works like any other system
- Always have a good backup solution
- PIOPS are useful
- Scale up and down to meet load on AWS

### RDS
- Overall, not bad
- Automatic failover (AWESOME)
- No reading from the secondary
- Expensive
- Generous set of extensions

### Sharding
- No integrated multi-master solution yet
- `pgpool_2`

## Tools

### Monitoring

- Nagios

### Log Analysis

- `pgbadger`
- `pg_stat_statements`



## Connection Tuning
- MAX_CONNECTIONS = maximum number of simultaneous queries postgres can excecute at that moment
- 150, 200
- pgbouncer: give lots of client connections, MAX_CONNS - 10 database connections
