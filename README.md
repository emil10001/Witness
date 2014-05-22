# Witness

A simple Java Observer

===============

The idea here is to build a really simple structure that can be used to implement observers easily.

To receive events for a specific datatype, the receiving class needs to implement the `Reporter`
interface, then register for whatever data types the following way:

    Witness.register(EventTypeOne.class, this);
    Witness.register(EventTypeTwo.class, this);

When you're done listening, unregister with the following:

    Witness.remove(EventTypeOne.class, this);
    Witness.remove(EventTypeTwo.class, this);

To publish events to listeners:

    Witness.notify(event);

Handling events in the `Reporter`:

    @Override
    public void notifyEvent(Object o) {
        if (o instanceof SomeObject) {
            objectHandlingMethod(((SomeObject) o));
        }
    }

Android, if you need code run on the main thread, in an Activity or Service:

    public class MyActivity extends Activity implements Reporter {
        private Handler handler = new Handler();
    
        // ...
    
        @Override
        public void notifyEvent(final Object o) {
            if (o instanceof SomeObject) {
                handler.post(new Runnable() {
                    @Override
                    public void run() {
                        objectHandlingMethod(((SomeObject) o));
                    }
                });
            }
        }
    }

Events are published on a background thread, using an `ExecutorService`, backed by a `BlockingQueue`.
This has a few important implications:

* Thread safety
  * You need to be careful about making sure that whatever you're using this for is thread safe
* UI/Main thread
  * All events will be posted to background threads
* Out of order
  * Events are handled in parallel, so it is possible for them to come in out of order

### Note

[+Dietrich Schulten](https://plus.google.com/u/0/115844465204731828022) had the following [comment](https://plus.google.com/u/0/+EJohnFeig/posts/eqprQafTTLc):

> It has the effect that events can be delivered on arbitrary threads. The event handler must be threadsafe, must synchronize access to shared or stateful resources, and be careful not to create deadlocks. Otoh if you use a single event queue, you avoid this kind of complexity in the first place. I'd opt for the latter, threadsafe programming is something you want to avoid if you can.﻿

I should note that my usage is all on Android, where I'm explicitly specifying the thread that the events will run on using `Handler`s. I haven't used this in a non-Android environment, and I'm not entirely sure how to implement the equivalent behavior for regular Java.
