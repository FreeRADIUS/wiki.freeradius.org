##Description

Access-Request is defined in RFC 2865.

Access-Request packets are sent to a [[RADIUS]] server, and convey information used to determine whether a user is allowed access to a specific [[NAS]], and any special services requested for that user. An implementation wishing to authenticate a user MUST transmit a [[RADIUS]] packet with the Code field set to 1 (Access-Request).

Upon receipt of an Access-Request from a valid client, an appropriate reply MUST be transmitted.

An Access-Request SHOULD contain a [[User-Name]] attribute.  It MUST contain either a [[NAS-IP-Address]] attribute or a [[NAS-Identifier]] attribute (or both).

An Access-Request MUST contain either a [[User-Password]] or a [[CHAP-Password]] or a [[State]].  An Access-Request MUST NOT contain both a [[User-Password]] and a [[CHAP-Password]].  If future extensions allow other kinds of authentication information to be conveyed, the attribute for that can be used in an Access-Request instead of User-Password or CHAP-Password.

An Access-Request SHOULD contain a [[NAS-Port]] or [[NAS-Port-Type]] attribute or both unless the type of access being requested does not involve a port or the [[NAS]] does not distinguish among its ports.

An Access-Request MAY contain additional attributes as a hint to the server, but the server is not required to honor the hint.

When a [[User-Password]] is present, it is hidden using a method based on the RSA [[Message Digest Algorithm]] [[MD5]].

A summary of the Access-Request packet format is shown below. The fields are transmitted from left to right.

     0                   1                   2                   3
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |     Code      |  Identifier   |            Length             |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                                                               |
    |                     Request Authenticator                     |
    |                                                               |
    |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |  Attributes ...
    +-+-+-+-+-+-+-+-+-+-+-+-+-

##Code

1 for Access-Request.

##Identifier

The Identifier field MUST be changed whenever the content of the Attributes field changes, and whenever a valid reply has been received for a previous request.  For retransmissions, the Identifier MUST remain unchanged.

##Request Authenticator

The Request Authenticator value MUST be changed each time a new Identifier is used.

##Attributes

The Attribute field is variable in length, and contains the list of Attributes that are required for the type of service, as well as any desired optional Attributes.
