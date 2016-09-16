# Threading

Version 4 should be async by default.  There should be minimal
communication between threads.  All necessary communication should be
through thread-safe queues.

We need a threading API which is separate from everything else.  It
takes care of queueing requests, signalling them, etc.

### create

Takes `CONF_SECTION *` and returns `fr_worker_thread_t *`.

### kill

Takes `fr_worker_thread_t *`, and ???, and signals the thread to exit.
The signals can be NOW, or terminate gracefully.

### enqueue

Takes `fr_thread_t *` and `REQUEST *`, and enqueues the request for the thread to process.

*Maybe we just need one enqueue function, which enqueues a request and a signal?*

And the default is `FR_ACTION_QUEUE` ?

### signal

Takes `fr_thread_t *` and `REQUEST *`, and `fr_action_t *`, and signals the request.

*We need this to be async safe, so we may need a REQUEST from the
network thread, which is then duplicated as a child REQUEST in the
worker thread?*`
