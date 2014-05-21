# Designing Django's Migrations

- Two parts `SchemaEditor` and `Migrations` to abstract making schema changes
from migrations themselves

## Two Parts

### SchemaEditor

- Abstracts schema operations across DBs
- Works in terms of Django fields/models
- Contains DB workarounds

### django.db.migrations

- Migration file reader/writer
- Dependency resolver
- Autodetector - takes your migration state, current models file and works out
the differences between them

## New Migration File Format

- More concise
- Declarative
- In-memory running - creates models from migration sets to compare to DB

## DB Peculiarities

### Postgres

- None

### MySQL

- No transactional DDL - if a migration fails part way through, you're fucked. Can't roll back.
- No `CHECK` constraints
- Conflates `UNIQUE` and `INDEX` - if you create and remove unique constraints,
it messes with the indexes

### Oracle

- Different SQL syntax
- Picky about names - can't start names with #, short restriction
- Can't convert to/from TextField

### SQLite

- Dear god

## Backwards Compatibility

- Django/South generally good at this
- Auto-applies first migration if tables exist
- Ignores South-style migrations

