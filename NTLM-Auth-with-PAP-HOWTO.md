If you are already using Samba/ntlm_auth to check EAP/MSCHAP requests, you might also want to authenticate PAP

# FreeRADIUS 2.1.11 and later

Do the following:

1. Edit raddb/modules/ntlm_auth to contain the correct path and domain
2. Add the following to raddb/policy.conf:
<pre>
policy {
  # give the ntlm_auth exec module and "authorize" method that sets Auth-Type to itself
  # but only if it's a valid PAP request, and Auth-Type is not already set to something
  ntlm_auth.authorize {
    if (!control:Auth-Type && User-Password) {
      update control {
        Auth-Type := ntlm_auth
      }
    }
  }
}
</pre>
3. Add the following to raddb/sites-enabled/XXX:
<pre>
authorize {
  ...
  ntlm_auth
}
authenticate {
  Auth-Type ntlm_auth {
    ntlm_auth
  }
 ...
}
</pre>

# Earlier versions

## Unlang-based

Do the following:

1. Edit raddb/modules/ntlm_auth to contain the correct path and domain
2. Add the following to raddb/sites-enabled/XXX:
<pre>
authorize {
  ...
  if (!control:Auth-Type && User-Password) {
    update control {
      Auth-Type := ntlm_auth
    }
  }
}
authenticate {
  Auth-Type ntlm_auth {
    ntlm_auth
  }
 ...
}
</pre>

## Using PAP module

Do the following:

1. Edit raddb/modules/ntlm_auth to contain the correct path and domain
2. Add the following to raddb/sites-enabled/XXX:
<pre>
authorize {
  ...
  pap
}
authenticate {
  Auth-Type PAP {
    ntlm_auth
  }
 ...
}
</pre>