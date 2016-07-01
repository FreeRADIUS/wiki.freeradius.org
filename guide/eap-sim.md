# Testing EAP-SIM and EAP-AKA

## Introduction
EAP-SIM/EAP-AKA and EAP-AKA' are very similar. The TLV and packet format are virtually identical.  The major differences are the pseudo random function used to generate sessions keys, and the format of the authentication vectors used.

AKA also supports server authentication using the AUTN, a value sent from HLR -> AAA -> Supplicant, derived from the Ki and RAND value.

In practical terms the best way to develop an EAP-SIM/EAP-AKA['] service and peform the testing, is to get eapol_test working with a smart card reader.

This lets you run the entire EAP-SIM/EAP-AKA protcol against a smart card, with no phones or access points needed.

It also lets you get SIM authentication vectors from the SIM card.  Unfortunately you can't get UMTS authentication vectors unless you have a valid AUTN for the SIM, and for that you need access to the HLR or a programmable SIM.  For testing AKA you may want to consider purchasing a [sysmocom](http://shop.sysmocom.de) sim, which allows you to set the Ki, and algorithms used.

The SCM SCR3310 CCID reader and the mechanical adapter they sell, both work really well too.

## PS/CS

Smart card interfaces are remarkably well standardized. The PS/CS (Personal Computer/Smart Card) API is present on Windows by default, and bundled OSX (as a framework). On linux PSCS lite can be installed to provide the API.

This API should provide an abstraction over *all* PS/CS compatible devices.  eapol_test in fact, just links to a PS/CS library and calls a single initialisation function, for all Smart Card readers, on all operating systems. There's very little OS specific boiler plate.

In addition to a C interface, there's also ``psyscard``, which is a python wrapper around the PSCS API.  Its used by the project osmocom utilities, and would provide a good basis for building your own SIM utilities.

## Use of EAP-SIM and EAP-AKA
Despite being able to theoretically use EAP-SIM with any SIM card in a local environment, it's not recommended.

Yes you *COULD* extract multiple authentication vectors from a SIM as some sort of onboarding process, and you *COULD* store those vectors, and then authenticate the SIM card locally on your wireless network.  But that's going to be quite a small pool of vectors, and you really shouldn't be repeating vectors as it can compromise the protocol.

If you're intent on using EAP-SIM/EAP-AKA to provide local authentication, you should get some programmable SIM card, for which you know the Ki, and distribute those to your users.  On the GSM/UMTS side they'll be useless (unless you also happen to be running a cell service), but it's conceivable that you could put them in company iPads (if the user doesn't want to use the 3G/4G radio).

Using EAP-SIM/EAP-AKA with a HLR either via M3UA/SUA (Sigtran) or Diameter is the way to go if you want to perform wireless handoff.  But that is outside the scope of this page (for now).  In that scenario instead of storing vectors, you call out to the HLR with the SIM's IMSI number, the HLR passes this to the AuC, and hands authentication vectors, which are passed back to the AAA server.

## eapol_test
### OSX

### Linux

### Windows
Should work out of the box


