## What is authentication?

Authentication refers to the confirmation that a user who is requesting services is a valid user of the network services requested.

Authentication is accomplished via the presentation of an identity and credentials.  Examples of types of credentials are passwords, one-time tokens, digital certificates, and phone numbers (calling/called).

## Authentication methods

The following authentication types are some of the many methods supported by the server:

* Plaintext-Authentication-Protocol (PAP)
* CHAP
* MS-CHAP
* MS-CHAPv2
* Windows Domain Controller Authentication (via [[ntlm_auth]] and winbind)
* [[Proxy]] to another RADIUS server
* Local system authentication (usually through [[unix]] /etc/passwd)
* [[rlm_pam|PAM]] ([[PAM|Pluggable Authentication Modules]])
* [[rlm_ldap|LDAP]] ([[PAP]] only)
* [[rlm_pam|PAM]] ([[PAP]] only)
* [[rlm_cram|CRAM]]
* [[Perl|modules/Rlm_perl]] program
* [[Python|modules/Rlm_python]] program
* [[SIP]] [[Digest]] ([[Cisco]] [[VoIP]] boxes, [http://www.iptel.org/ser/ SER])
* A locally [[exec]]uted program (like a CGI program)
* [[rlm_krb5|Kerberos]] authentication
* X9.9 authentication token (e.g. [http://www.onlineshow.info CRYPTOCard])
* [[rlm_eap|EAP]] wireless with embedded authentication methods 

  * EAP-MD5
  * Cisco LEAP
  * EAP-MSCHAP-V2 (as implemented by Microsoft),
  * EAP-GTC
  * EAP-SIM
  * EAP-TLS
  * EAP-TTLS (with any authentication protocol inside of the TLS tunnel)
  * EAP-PEAP (with tunnelled EAP)

## See also

* RFC 2865
* [[AAA]]
* [[AAAA]]
