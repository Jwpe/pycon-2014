# Async

## What is async for?

- Minimises resources for idle connections

## C10K

- [Seminal Paper](kegel.com/c10k.html)

## Why is async hard to do?

- You need to store state
- Needs to remember for a frontend client, what to do when the backend responds
- Storing state has two axes:
    - Memory/connection
    - Coding difficulty
- Callbacks = low mem/connection, difficult to code
- Threads = high mem/connection, easy to use
- Still a live and developing area of research

## What IS async?

- Single-threaded
- Can do I/O concurrency
- Can't do computation concurrently
- Non-blocking sockets (epoll)
- Uses an event loop to continuously process event notifications
- Examples: Node, Tornado, Twisted, Tulip

## asyncio

- AKA "Tulip"
- Made by Guido
- In the Python 3.4 standard lib
- Standard event loop
- Coroutines
- Implements [PEP 3156](http://legacy.python.org/dev/peps/pep-3156/)

## Async App Example

### Layers

- app
- protocol
    -e.g. HTTP
    - example uses autobahn lib
- transport
    -e.g. TCP
- event loop
    - finds out about events and dispatches them to event handlers, which are callbacks
- selectors
    - abstracts away platform specific differences between event notifications
    - *magic*

### Implementation

- `sock.setblocking(False)`: all operations will send or receive instantly, but can't block. Will raise an exception if blocking
- pass the file descriptor of the socket to `_selector` to abstract to the operating system
- also pass in a 'reader' and 'writer' callback - what to do on read, or on
write
- for a listening socket, just pass in a reader

#### _accept_connection

- takes a `protocol_factory` (ChatProtocol) and a socket
- accepts the socket and returns a connected socket
- calls the protocol factory
- calls `_SelectorSocketTransport` with the connected socket and the result
of the protocol factory

#### _SelectorSocketTransport
- takes connected socket and protocol

### BaseEventLoop
- `run_forever` is an infinite loop
- `while True`, recieves an event list from the `_selector`
- list of events that are currently ready on non-blocking sockets
- these contain file descriptors, mask(?) and data
- data contains reader and writer callbacks
- add ready callbacks to a ready queue
- check length of ready queue
- iterate through ready queue for the length of the current ready queue
    - this makes sure that any callbacks that add new callbacks aren't executed until the next iteration of the event loop

## Review

- asyncio uses non-blocking sockets
- Event loop tracks sockets and the callbacks waiting
- Selectors handles abstraction of events on operating system

## When should I use it?

### Yes

- Slow backend
- Websockets
- Many connections

### No

- CPU-bound
- No async driver
- No async expertise

## Resources

- http://initd.org/psycopg/docs/advanced.html#asynchronous-support

## Questions

- Debugging?
    - Callbacks usually lose your traceback
    - Chain of events that led to it being added to
    the runnable queue is gone
    - Coroutines are better than callbacks for debugging
- Asynchronous drivers for other kinds of blocking I/O?
    - gevent monkey-patches stdlib I/O with an async layer
    - theres a stdlib email module, then a non-blocking Twisted version
    - will be a few years before there is a separation between business logic and I/O scheduling

- Why didn't you just untangle existing PyMongo lib?
    - Too intertwined scheduling I/O and business logic
