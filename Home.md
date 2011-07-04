## New Wiki

The Wiki is now hosted on a new server, with a new front end, and is backed by "git".  It is now easier to login and make changes.  Please see the [[New Wiki]] Page for details.

A read only version of the wiki can be checked out using git:
    git clone git://git.freeradius.org/wiki.freeradius.org.git

## Overview

[[FreeRADIUS]] is a modular, high performance and feature-rich [[RADIUS]] suite including [[server|radiusd]], [[radius client|radiusclient]], development libraries and numerous additional [[RADIUS]] related utilities.

As the premiere open source RADIUS suite it is included as a [[standard package]] with numerous Operating Systems, has [[binary packages]] for many others and has source available which is known to [[build]] on almost anything. Production deployments include large scale installations comprising multiple [[AAA]] servers with over ten million users and millions of requests per day. It supports request [[proxying|proxy]], with [[fail-over]] and [[load balancing]], as well as the ability to access many types of back-end [[databases]]. Different classes of [[Authentication]] requests can trigger access of different [[Authentication]] and [[Authorization]] [[databases]] (with cascaded fall back), and [[Accounting]] records can be simultaneously recorded in multiple different storage [[databases]] and [[directories]].

[[Other RADIUS Servers]] are available.

We also keep a list of [[Acknowledgements]] of contributions to FreeRADIUS development.

## RADIUS Client Support

[[FreeRADIUS]] comes with:

* Complete support for RFC 2865 and RFC 2866 attributes.
* [[EAP]] with EAP-MD5, EAP-SIM, EAP-TLS, EAP-TTLS, [[EAP-PEAP]], and Cisco [[LEAP]] EAP sub-types
* [[Vendor-Specific Attributes]] for almost one hundred vendors, including BinTec, Foundry, [[Cisco]], Juniper, Lucent/Ascend, [[HP ProCurve|HP]], Microsoft, USR/3Com, Acc/Newbridge and many more.

All known [[RADIUS Clients]] are supported.

* [[RADIUS Clients]] 
* [[EAP Clients]]

## Flexible Configuration

[[FreeRADIUS]] provides a wide range of methods to select user configurations. The server can select a configuration based on any of the following criteria : 

* [[Attributes]] which have a given value
* [[Attributes]] which do not have a given value
* [[Attributes]] which are in the request (independent of their value)
* [[Attributes]] which are not in the request
* String [[attributes]] which match a regular expression
* Integer [[attributes]] which match a range (e.g. , =)
* Source IP address of the request. (This can be different from the NAS-IP-Address)
* Shortname defined for a NAS box. (This can be different from the NAS-Identifier)
* Group of NAS boxes. (These may be grouped based on Source IP address, NAS-IP-Address, or any other configuration)
* User-Name
* DEFAULT template configuration
* multiple cascading DEFAULT template configurations

In addition, FreeRADIUS supports [[virtual server]]s which allow several separate sets of configuration data to coexist inside the same server instance.

## Documentation

This wiki collects a large amount of documentation relating to [[FreeRADIUS]] together in one place. Pages of particular interest to newcomers will be the [[FAQ]] and [[HOWTO]] sections, although the [[Traditional FreeRADIUS docs page|http://www.freeradius.org/doc/]] includes "man" pages, and other documentation.

There are many third-party web sites and HOWTO's that give advice on FreeRADIUS.  They are usually years out of date, and refer to old versions, instead of 2.1. The advice that they give is also wrong.  We **strongly** recommend that you avoid most third-party documentation.

The following is an overview of the types of information available:

* [[Installation]]
* [[Configuration]]
* [[Base Modules|List of Modules]]
* Creating your own [[Modules for FreeRADIUS Version 2|Modules2]]
* Creating your own [[Modules for FreeRADIUS Version 1|Modules1]]
* [[Radius Clients]]
* [[FAQ]]
* [[HOWTO]]
* [[Example Setups]] Cookbook
* [[Glossary]]

For beginners, there is also a short [[Concepts]] page, which can help clarify how the server works.