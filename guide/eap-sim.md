# Introduction

EAP-SIM/EAP-AKA and EAP-AKA' are very similar. The TLV and packet format are virtually identical.  The major differences are the pseudo random function used to generate sessions keys, and the format of the authentication vectors used.

AKA also supports server authentication using the AUTN, a value sent from HLR -> AAA -> Supplicant, derived from the Ki and RAND value.

In practical terms the best way to develop an EAP-SIM/EAP-AKA['] service and peform the testing, is to get eapol_test working with a smart card reader.

This lets you run the entire EAP-SIM/EAP-AKA protcol against a smart card, with no phones or access points needed.

It also lets you get SIM authentication vectors from the SIM card.  Unfortunately you can't get UMTS authentication vectors unless you have a valid AUTN for the SIM, and for that you need access to the HLR.  For testing AKA you may want to consider purchasing a [sysmocom](http://shop.sysmocom.de) programmable sim.

The SCM SCR3310 CCID reader and the mechanical adapter they sell, both work really well too.

## PS/CS

Smart card interfaces are remarkably well standardized. The PS/CS (Personal Computer/Smart Card) API is present on Windows by default, and bundled OSX. On linux PSCS lite can be installed to provide the API.

It appears one of the aims of this interface was to make a single driver that would work with all smart cards.

## eapol_test
### OSX

### Linux

### Windows
Should work out of the box


