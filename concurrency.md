# Fan-in and Fan-out: The crucial components of concurrency

## Why do we need asyncio?

- It's easy to do hard things in Python
- It's hard to do a lot of different kinds of work at the same time in one Python program

## Definitions

### Fan-out

- When one thread of control spawns one or more new threads of control
- E.g. spawn sub-processes

### Fan-in

- When one thread of control gathers results from one or more separate threads of control

### Web caller using threads

- Can append results from multiple threads to a list thanks to the GIL. Good and bad.

### Web crawler using asyncio

- replace I/O with `yield from`
- Generators. Generators everywhere.
- `future`s

## Fan-out/Fan-in is everywhere
- SQL
- Map/Reduce

## Resources

- ["Concurrency is not Parallelism"](http://blog.golang.org/concurrency-is-not-parallelism)
    - all about Go concurrency

## Questions

- Can get around computational concurrency by firing off sub-processes, like in David Beazely's talk
