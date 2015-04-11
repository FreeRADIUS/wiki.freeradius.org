RADIUS Concepts
===============

![FreeRADIUS flow diagram](/request_flow.svg "Railroad diagram")


Many people have problems configuring FreeRADIUS.  These problems are often due to faulty concepts.  No amount of editing configuration files will correct the faulty concepts.  This page tries to correct that problem.

How things work in RADIUS
=========================

The client sends you a radius authentication request, you don't decide what's in the request, the client does.  The server doesn't decide what's in the request, the client does.  The client is 100% responsible for everything in the request.


Picking an Auth-Type
====================

The radius server looks at the request and says:

>  Hmmm... can I deal with this request?

The answer to that depends on what authentication types you have enabled, what the server can look up in a DB, and what is in the request.

The server will then start querying the modules in the authorize::

>  Unix module, can you handle this one?
  
>  Pap module, can you handle this one?
  
>  Mschap module, can you handle this one?

At some point, one of the modules will say:

>  Yes, I see something in the request I recognize.  I can do something!

The module does this by looking in the request for key attributes, such as MS-CHAP-Challenge (for mschap), or CHAP-Challenge (for chap), or EAP-Message (for eap). Or it may just assume it needs to add something to every request.

If the module thinks it has a shot at authenticating the user it'll say:

>  I can't authenticate this user now (I was just told to authorize them)
>  But my pal in the Authenticate section can!
>  Hey, set the Auth-Type to me!

If the module doesn't see anything it recognizes, or knows it doesn't need to lookup anything, it does nothing.

Authenticating a user
=====================

At the end of authorize, the server will check if anything set the Auth-Type.

If nothing did, it immediately rejects the request.

Lets suppose that the client sends a request with a User-Password attribute, and pap is enabled, the pap module will then have set ``Auth-Type = pap``.

So in authenticate, the server will call the pap module again:

>  I see a User-Password, which is what the user entered.
>  That's nice, but I need to compare it with something.
>  Ah! Another module added the "known good" password for this user in authorize!

So it then compares the local "known good" password to the password as entered by the user.  This is how authentication works.

The "known good" password comes from another module.  The pap module just does PAP authentication, and nothing more.  The benefit of this approach is that the "known good" password can come from the 'users' file, SQL, LDAP, ``/etc/passwd``, external program, etc.  i.e. pretty much anything.

Lets suppose that the ldap module was listed in authorize it'll have run and checked:

>  Hmm... Can I find a "known good" password for this user?

If so, it will have added the "known good" password to the request, so that another module in authenticate can use it.

Insufficient information
========================

But WAIT! What if the client sends a MSCHAP request? What does the radius server say then?
::
>  Well that's a fine kettle of fish! 
>  That client has really really tied my hands on this one!

In this case, the mschap module looks at the request, and finds the MS-CHAP attributes.  It sets the *Auth-Type* to itself (mschap).  A database module (such as LDAP, above) gets the "known good" password, and adds it to the request.  The mschap module is then run for authentication.  It looks for either a clear text password or nt-hash (why? look at the [[protocol table|http://deployingradius.com/documents/protocols/compatibility.html]]). If one of those hasn't been added by a datastore, the mschap module says:

>  Sorry, I can't authenticate the user,
>  because I don't have the information I need to validate MSCHAP

But now the server has run out of options, its only choice was mschap because that's what the client sent in the request.  The mschap module can't do anything because you didn't give it a useful "known good" password . So the server has no choice but to reject the request.  The MSCHAP data might be correct, but the server has no way to know that.  So it replies with a reject.