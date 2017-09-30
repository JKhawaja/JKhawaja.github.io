Channels can be a rather stubborn concurrency primitive in Go when first working with them. We first give a quick review of using channels and the blocking scenarios they create, then we move on to patterns for asynchronous concurrency in Go focusing in on the "messengers" pattern.

## Channels

- **Unbuffered** (blocking scenarios)
    + the sender will block on the channel until the receiver receives the data from the channel
    + the receiver will also block on the channel until sender sends data into the channel.
- **Buffered** (blocking scenarios)
    + the sender of buffered channel will block when there is no empty slot of the channel
    + the receiver will block on the channel when it is empty

## Synchronization

- It is preferable to *use channels as state change signalers* between goroutines, and in this way channel blocking becomes useful for *synchronization*.
    + Either signaling the state of an operation (e.g. failed or completed), or the latest value for a variable, etc.
    + The idea is: synchronizing (the) state (of a variable as it) changes.

## Avoiding blocking?

- For both unbuffered and buffered channels: 
    + the receiver will block until there is data to be received. 
    + This is why receiving from a channel should *preferably* be done inside a `select` statement, so a user can control whether the read should be blocking or non-blocking (whereby adding a `default` case in the `select` statement makes it non-blocking).
- *Buffered channels should be preferred* over unbuffered channels so senders do not need to block when sending a message.
    + Often, the `make(chan struct{}, 1)` pattern can be used as only a single state change signal will ever be needed at a given time (required to synchronize two routines before the next signal should be allowed to be sent and read).
    + However, it is not always the case that the capacity for a buffer will be known ahead of time and/or well-defined. In which case, a thread might want to spawn a new goroutine to send specific values onto an unbuffered channel so it can continue asynchronously (messengers pattern) or just allocate arbitrarily-large-capacity buffered channels.

## Asynchronous Patterns

- If synchronization is not a goal for the system then just using arbitrarily large buffered channels, or spawning "messenger" go-routines that are only read from inside non-blocking select statements, should do the trick.
- Since, allocating arbitrarily large buffered channels just *feels* like a bad habit to have ... let's encourage the messengers pattern instead.

## Messengers Pattern

The core idea behind the messengers pattern is to have `worker` goroutines and `messenger` goroutines. The messenger goroutines can be based on messenger objects that are used an re-used from a messengers `sync.Pool`. In this way, any worker can send a message to another go-routine asynchronously via a messenger rather than directly on a channel.

It's almost like creating two separate societal classes of goroutines. So, treat your messengers well, else they revolt and crash the system.

## Example

Run the example here: [https://play.golang.org/p/2-37mqZFCQ](https://play.golang.org/p/2-37mqZFCQ)

Get the example code here: [https://gist.github.com/JKhawaja/8377bcd5d39062acf08ca2d4605a263c](https://gist.github.com/JKhawaja/8377bcd5d39062acf08ca2d4605a263c)