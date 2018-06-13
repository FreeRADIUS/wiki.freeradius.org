# TLS Virtual Server

It gets bootstrapped... somehow. when the modules / network sockets start.

## Config

There are three actions, which need to be in a virtual server by themselves.  Currently, they're hacked into `recv` and `send` (or even `authorize`).  That needs fixing.

When a session is created (only on sending Access-Accept)

    new session {
        cache # new session information
    }

When a session is resumed:

    resume session {
        cache # resume a session
    }

When a session is deleted:

    delete session {
        cache # new session information
    }