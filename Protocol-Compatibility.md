You can find the original version of this page on http://deployingradius.com/documents/protocols/compatibility.html

The following table shows which protocol is compatible with what kind of password.

Protocol    |Clear text|NT hash (ntlm_auth)|MD5 hash|Salted SHA1 hash|SHA1 hash|Salted MD5 hash|Unix Crypt
            |          |                   |        |                |         |               |          
PAP         |         ✓|                  ✓|       ✓|               ✓|        ✓|              ✓|         ✓
CHAP        |         ✓|                  x|       x|               x|        x|              x|         x
Digest      |         ✓|                  x|       x|               x|        x|              x|         x
MS CHAP     |         ✓|                  ✓|       x|               x|        x|              x|         x
PEAP        |         ✓|                  ✓|       x|               x|        x|              x|         x
EAP MSCHAPv2|         ✓|                  ✓|       x|               x|        x|              x|         x
Cisco LEAP  |         ✓|                  ✓|       x|               x|        x|              x|         x
EAP GTC     |         ✓|                  ✓|       ✓|               ✓|        ✓|              ✓|         ✓
EAP MD5     |         ✓|                  x|       x|               x|        x|              x|         x
EAP SIM     |         ✓|                  x|       x|               x|        x|              x|         x


For EAP-TTLS, look up the tunneled protocol in the above table. For the purposes of this table, the tunneled session is just another RADIUS authentication request. So for EAP-TTLS, with tunneled PAP, look up PAP in the above table.

Similarly, PEAP normally contains EAP-MSCHAPv2 in the tunneled session, so its row in the table is identical to the EAP-MSCHAPv2 row, which is in turn identical to the MS-CHAP row.

We do not list EAP-TLS in the above table, because it performs authentication with certificates, and doesn't use passwords.
