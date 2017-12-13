# Testing EAP-SIM and EAP-AKA

## Introduction
EAP-SIM/EAP-AKA and EAP-AKA' are EAP methods that allow a supplicant to gain access to a resource by using a SIM (Subscriber Identity Module) card. These EAP methods are usually deployed by MNOs (Mobile Network Operators), where the MNO or a partner or the MNO also operate a large scale WiFi network, and the MNO wishes to offload subscribers onto this WiFi network where available.

Interest in EAP-SIM/EAP-AKA/EAP-AKA' has been increasing in recent years, as with the standardization of "WiFi calling" and the release of the HotSpot 2.0 standards, it has become significantly easier to enable WiFi offload in a way that is seamless to the end user.

## Authentication vectors
Authentication vectors are sets of cryptographically generated keys and challenges.  The authentication vectors themselves do not get sent over the wire, only the RAND (Random challenge) component of the vectors is sent between parties.

The authentication vector is the output of an algorithm running on an AuC (Authentication Centre), and on the SIM of the subscriber. 

## High level overview
EAP-SIM, EAP-AKA and EAP-AKA' are very similar. The TLV and packet format are virtually identical. The major differences are the pseudo-random function used to generate sessions keys, and the format of the authentication vectors used.

All the SIM base EAP-Methods function in a similar way.  In each case, an AS (Authentication server - like FreeRADIUS), requesting an authentication vector from an AuC with knowledge of a SIM's Ki.  The AuC generates a random challenge (RAND), feeds it and the Ki into a vector generation algorithm (COMP128-[1234], Milenage).  The vector generation algorithm produces multiple keys, which are passed back to the AS along with the challenge.

The AS then runs a KDF (Key derivation function), which performs additional cryptographic operations on the keys received from the AuC.  The purpose of this KDF is usually to increase the key size, to modify the keys in such a way that they can't be reused, and in the case of AKA' to bind the use of the keys to a particular network.

After the KDF has run, the AS will generate an EAP-[SIM|AKA] Challenge packet, encapsulate it in an EAP-Request, and send it to the peer. The contents of the challenge packet vary, but it always contains one or more random challenges (``AT_RAND``), and a HMAC (``AT_MAC``). The HMAC is the digest of the contents of the entire EAP-Request (and in the case of SIM some extra data), with the ``AT_MAC`` attribute zeroed, and keyed with ``K_aut``, an authentication key output from the KDF.

When the supplicant receives the Challenge packet, it feeds RAND into the same algorithm as used on the AuC, and then feeds to results of that algorithm into the method specific KDF.  The supplicant uses the ``K_aut`` value from the KDF it ran locally to generate its own HMAC digest and compares that to the one received from the AS. If the MACs match, it proves to the supplicant that the challenge packet was not tampered with and that the AS has an identical authentication vector to the one generated locally.

The supplicant then constructs its challenge response, encapsulates it in an EAP-Response, and generates another HMAC over the packet.  In the case of EAP-SIM the key used for the HMAC, is the concatenation of 


## Which method should I use?

In terms of security (key sizes, strength of authentication, MITM attack prevention), EAP-SIM is the weakest, and EAP-AKA' is the strongest.

EAP-SIM should only be used with legacy 2G AuC that can only generate GSM authentication vectors. EAP-AKA should only be used where the supplicant does not support EAP-AKA'.


## Testing
In practical terms, the best ways to develop an EAP-SIM/EAP-AKA['] service and perform testing, is to get eapol_test working with a smart card reader or to use eapol_test's virtual usim functionality. This lets you run the entire EAP-SIM/EAP-AKA protocol against a smart card, with no phones or access points needed.

If you're set on using physical hardware or have particular handsets you want to use for testing you may want to consider purchasing a [sysmocom](http://shop.sysmocom.de) sim, which allows you to set the Ki, and algorithms used.  The SCM SCR3310 CCID reader and the mechanical adapter [sysmocom](http://shop.sysmocom.de) sell both work very nicely out of the box (at least on OSX).

### PS/CS

Smart card interfaces are remarkably well standardized. The PS/CS (Personal Computer/Smart Card) API is present on Windows by default and bundled OSX (as a framework). On Linux, PSCS lite can be installed to provide the API.

This API should provide an abstraction over *all* PS/CS compatible devices.  eapol_test in fact, just links to a PS/CS library and calls a single initialisation function, for all Smart Card readers, on all operating systems. There's very little OS-specific boilerplate.

In addition to a C interface, there's also ``psyscard``, which is a python wrapper around the PSCS API.  Its used by the project osmocom utilities and would provide a good basis for building your own SIM utilities.

## Practical uses of EAP-SIM and EAP-AKA[']
### Stored vectors
Despite being able to theoretically use EAP-SIM with any SIM card in a local environment, it's not recommended.

Yes you *COULD* extract multiple authentication vectors from a SIM as some sort of onboarding process, and you *COULD* store those vectors, and then authenticate the SIM card locally on your wireless network.  But that's going to be quite a small pool of vectors, and you really shouldn't be repeating vectors as it can compromise the protocol.

If you're intent on using EAP-SIM/EAP-AKA to provide local authentication, you should get some programmable SIM cards, for which you know (or can set) the Ki, and distribute those to your users.  On the GSM/UMTS side, they'll be useless (unless you also happen to be running a cell service), but it's conceivable that you could put them in company iPads (if the user doesn't want to use the 3G/4G radio).

Using EAP-SIM/EAP-AKA with an HLR either via M3UA/SUA (Sigtran) or Diameter is the way to go if you want to perform wireless handoff. In that scenario instead of storing vectors, you call out to the HLR with the SIM's IMSI number, the HLR passes this to the AuC, which generates authentication vectors, which it passes back to the HLR, which passes them back to the AAA server.

But that is outside the scope of this page (for now).  

## eapol_test
### OSX

### Linux

### Windows
Should work out of the box