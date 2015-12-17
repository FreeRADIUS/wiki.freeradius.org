## Python module for freeradius

### Purpose
To allow module writers to write modules in a high-level language, for implementation or for prototyping.

### Usage
Your Python file should have a number of methods named after the sections where they're called in (like `authorize`). The functions get a single argument: a tuple of the attributes of the request list. Those attributes are given as new tuples (size 2) with the first value the attribute name, and the second value the attribute value. An example of what you could get as input:

    (('User-Name', '"bob"'), ('User-Password', '"hello"'), ('NAS-IP-Address', '127.0.1.1'))

(At the moment (version 3.0.10) all string values have extra quotes added, this might change in the future.)

There are three possible return values: `None` (which is the same as returning `radiusd.RLM_MODULE_OK`), an integer (preferable using the `radiusd.RLM_MODULE_*` constants) and a tuple of size 3: first element is the integer return, second argument is a tuple to update reply, third argument is a tuple to update config. Those tuples use the same format as mentioned above. It is also possible to use a tuple of size 3, the middle argument becomes the operator.

(The extra quotes are not used in the return values)
(Releases before version 3.0.10 support adding the operator, but it was only supported using the integer from `token.h`, and many operators don't work)

### Example
We will be writing a script that allows every user to login with the password "hello", and add a reply message to show the user he has been authenticated using python

#### Setup
Starting with a completely clean config, first enable `rlm_python` by creating a symlink to `mods-available/python` from `mods-enabled/`. Edit this file, make sure it has the following config:

    python {
      module = example
    
      python_path = ${modconfdir}/${.:name}
    
      mod_authorize = ${.module}
      func_authorize = authorize
    }

(At the moment, every function that you want to used has to be enabled manually, this might change in future version).

Next, enable python in `sites-enabled/default` in the section `authorize`. Added a line with just `python` between `auth_log` and `chap`. And last, ensure the file `example.py` is available in `mods-config/python`. When starting the server in debug mode and trying to authenticate using bob/hello, we'll get a failed password, but we should some lines like this in the debug logging

    *** authorize ***
    
    *** radlog call in authorize ***
    
    (('User-Name', '"bob"'), ('User-Password', '"hello"'), ...many more attributes...)

This is the equivalent of "Hello, world" in `rlm_python`. The next step is making the authentication really work

#### Writing the code
Just a quick sidestep. To make the user authenticate with password 'Hello', we need to add the attribute `Cleartext-Password` with value 'Hello' to the config list. The give the user some feedback, we can add the `Reply-Message` attribute to the reply list.

Let's construct the reply tuples first. Below the line `print p`, add the following two lines:

    reply = ( ('Reply-Message', 'Hello from rlm_python'), )
    config = ( ('Cleartext-Password', 'hello'), )

If you're not used to Python: make sure you don't mess up indenting tabs and spaces in your source file, and don't omit the trailing commas (otherwise python decides to flatten the tuples).

Now we have to return these tuples to FreeRADIUS. To do that, replace the line with `return radiusd.RLM_MODULE_OK` with the following line:

    return (radiusd.RLM_MODULE_OK, reply, config)

As discussed before: we return the status, the reply updates and the config updates. Trying again in debug mode should now add the following lines:

    rlm_python:authorize: 'reply:Reply-Message' = 'Hello from rlm_python'
    rlm_python:authorize: 'config:Cleartext-Password' = 'hello'

(FreeRADIUS older than version 3.0.11 shows less debug output, don't worry too much about that)

And your authentication should now be successful, with an added attribute that says hello from python.

From version 3.0.11, you can also add operators to the return tuples. To do this, use a tuple with 3 items:

    reply = ( ('Tmp-String-0', ':=', 'Another hello from rlm_python'), )