# Clients in v4

Clients are really a RADIUS-specific construct, and aren't used in other protocols.  We need an `rlm_client` module to handle dynamic clients (and merge `rlm_dynamic_clients` into `rlm_clients`).  The `dynamic_clients` virtual server needs to be renamed `clients`

## Config

New client:

    new client {
        map sql ... {
        }
    }

Delete a client:

    delete client {
       # stuff
    }

The question is what *current* method is used in these to call module?  i.e. `authorize`, `post-auth`, etc.

The solution to that isn't clear yet.