# Witness

A simple Java Observer

===============

The idea here is to build a really simple structure that can be used to implement observers easily.

To receive events for a specific datatype, the receiving class needs to implement the `Reporter`
interface, then register for whatever data types the following way:

    Witness.register(EventTypeOne.class, this);
    Witness.register(EventTypeTwo.class, this);

To publish events to listeners:

    Witness.notify(event);

Events are published on a background thread, using an `ExecutorService`, backed by a `BlockingQueue`.
This has a few important implications:

* Thread safety
** You need to be careful about making sure that whatever you're using this for is thread safe
* UI/Main thread
** Events will be posted to background threads
* Out of order
** Events are handled in parallel, so it is possible for them to come in out of order