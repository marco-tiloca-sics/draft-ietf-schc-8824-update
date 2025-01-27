---
v: 3

title: Static Context Header Compression (SCHC) for the Constrained Application Protocol (CoAP)
abbrev: SCHC for CoAP
docname: draft-ietf-schc-8824-update-latest

ipr: trust200902
area: Internet
wg: SCHC Working Group
kw: Internet-Draft
cat: std
submissiontype: IETF
obsoletes: 8824

coding: utf-8

author:
      -
        ins: M. Tiloca
        name: Marco Tiloca
        org: RISE AB
        street: Isafjordsgatan 22
        city: Kista
        code: SE-16440
        country: Sweden
        email: marco.tiloca@ri.se
      -
        ins: L. Toutain
        name: Laurent Toutain
        org: IMT Atlantique
        street: CS 17607, 2 rue de la Chataigneraie
        city: Cesson-Sevigne Cedex
        code: 35576
        country: France
        email: Laurent.Toutain@imt-atlantique.fr
      -
        ins: I. Martinez
        name: Ivan Martinez
        org: Nokia Bell Labs
        street: 12 Rue Jean Bart
        city: Massy
        code: 91300
        country: France
        email: ivan.martinez_bolivar@nokia-bell-labs.com
      -
        ins: A. Minaburo
        name: Ana Minaburo
        org: Consultant
        street: Rue de Rennes
        city: Cesson-Sevigne
        code: 35510
        country: France
        email: anaminaburo@gmail.com

normative:
  RFC3688:
  RFC5116:
  RFC6020:
  RFC7252:
  RFC7641:
  RFC7959:
  RFC7967:
  RFC8126:
  RFC8323:
  RFC8407:
  RFC8613:
  RFC8724:
  RFC8768:
  RFC8949:
  RFC8974:
  RFC9175:
  RFC9177:
  RFC9363:
  RFC9668:
  I-D.ietf-core-oscore-groupcomm:
  I-D.ietf-core-oscore-key-update:
  I-D.ietf-core-href:

informative:
  RFC8824:
  RFC9147:
  RFC9528:
  I-D.ietf-core-groupcomm-bis:

entity:
  SELF: "[RFC-XXXX]"

--- abstract

This document defines how to compress Constrained Application Protocol (CoAP) headers using the Static Context Header Compression and fragmentation (SCHC) framework. SCHC defines a header compression mechanism adapted for constrained devices. SCHC uses a static description of the header to reduce the header's redundancy and size. While RFC 8724 describes the SCHC compression and fragmentation framework, and its application for IPv6/UDP headers, this document applies SCHC to CoAP headers. The CoAP header structure differs from IPv6 and UDP, since CoAP uses a flexible header with a variable number of options, themselves of variable length. The CoAP message format is asymmetric: the request messages have a header format different from the format in the response messages. This specification gives guidance on applying SCHC to flexible headers and how to leverage the asymmetry for more efficient compression Rules. This document replaces and obsoletes RFC 8824.

--- middle

# Introduction # {#intro}

The Constrained Application Protocol (CoAP) {{RFC7252}} is a command/response protocol designed for microcontrollers with small RAM and ROM, and optimized for services based on REST (Representational State Transfer). Although the constrained devices are a leading factor in the design of CoAP, a CoAP header's size is still too large for LPWANs (Low-Power Wide-Area Networks). Static Context Header Compression and fragmentation (SCHC) over CoAP headers is required to increase performance or to use CoAP over LPWAN technologies.

{{RFC8724}} defines the SCHC framework, which includes a header compression mechanism for LPWANs that is based on a static context. {{Section 5 of RFC8724}} explains where compression and decompression occur in the architecture. The SCHC compression scheme assumes as a prerequisite that both endpoints know the static context before transmission. The way the context is configured, provisioned, or exchanged is out of this document's scope.

CoAP is an application protocol, so CoAP compression requires installing common Rules between the two SCHC instances. SCHC compression may apply at two different levels: at the IP and UDP level in the LPWAN, as well as at the application level for CoAP. These two compression techniques may be independent. Both follow the same principle as that described in {{RFC8724}}. As different entities manage the CoAP compression process at different levels, the SCHC Rules driving the compression/decompression are also different. {{RFC8724}} describes how to use SCHC for IP and UDP headers. This document specifies how to apply SCHC compression to CoAP headers.

SCHC compresses and decompresses headers based on common contexts between Devices. The SCHC context includes multiple Rules. Each Rule can match the header fields to specific values or ranges of values. If a Rule matches, the matched header fields are replaced by the RuleID and the Compression Residue that contains the residual bits of the compression. Thus, different Rules may correspond to different protocol headers in the packet that a Device expects to send or receive.

A Rule describes the packets' entire header with an ordered list of Field Descriptors (see {{Section 7 of RFC8724}}). In turn, each Field Descriptor contains the Field ID (FID), Field Length (FL), and Field Position (FP), as well as a Direction Indicator (DI) (upstream, downstream, or bidirectional) and some associated Target Values (TVs). The DI allows the compression to be based on the best TV for the Field Descriptor, when the TV to consider is different for the different transmission directions. Therefore, a field may be described several times in the same Rule.

Furthermore, a Matching Operator (MO) is associated with each header Field Descriptor. The Rule is selected if all the MOs fit the TVs for all the fields of the header. A Rule cannot be selected if the message contains a field that is unknown to the SCHC compressor.

In that case, a Compression/Decompression Action (CDA) associated with each field specifies the method to compress and decompress that field. Compression mainly results in one of four actions:

* send the field value (value-sent),

* send nothing (not-sent),

* send some Least Significant Bits (LSBs) of the field, or

* send an index (mapping-sent).

After applying the compression, there may be some bits to be sent. These values are called "Compression Residue".

SCHC is a general mechanism applied to different protocols, with the exact Rules to be used depending on the protocol and the application. {{Section 10 of RFC8724}} describes the compression scheme for IPv6 and UDP headers. This document targets CoAP header compression using SCHC.

The use of SCHC compression applied to CoAP headers was originally defined in {{RFC8824}}. While this document does not alter the core approach, design choices, and features specified therein, this document clarifies, updates, and extends the SCHC compression of CoAP headers defined in {{RFC8824}}.

In particular, this documents replaces and obsoletes {{RFC8824}} as follows.

* It provides clarifications and amendments to the original specification text, based on collected feedback and reported errata.

* It clarifies how the SCHC compression handles CoAP options in general (see {{sec-coap-options}}).

* It clarifies the SCHC compression for the CoAP options: Size1, Size2, Proxy-Uri, and Proxy-Scheme (see {{ssec-size1-size2-proxy-uri-proxy-scheme-option}}); ETag and If-Match (see {{ssec-etag-if-match-option}}); and If-None-Match (see {{ssec-if-none-match}}).

* It defines the SCHC compression for the recently defined CoAP options Proxy-Cri and Proxy-Scheme-Number (see {{ssec-proxy-cri-proxy-scheme-number-option}}).

* It defines the SCHC compression for the CoAP option Hop-Limit (see {{coap-options-hop-limit}}).

* It defines the SCHC compression for the recently defined CoAP options Echo (see {{coap-options-echo}}), Request-Tag (see {{coap-options-request-tag}}), EDHOC (see {{coap-options-edhoc}}), as well as Q-Block1 and Q-Block2 (see {{ssec-coap-extensions-block}}).

* It updates the SCHC compression processing for the CoAP option OSCORE (see {{ssec-coap-extensions-oscore}}), in the light of recent developments related to the security protocol OSCORE as defined in {{I-D.ietf-core-oscore-key-update}} and {{I-D.ietf-core-oscore-groupcomm}}.

* It clarifies how the SCHC compression handles the CoAP payload marker (see {{payload-marker}}).

* It defines the SCHC compression of CoAP headers in the presence of CoAP proxies (see {{compression-with-proxies}}), for which examples are provided (see {{examples}}).

## Terminology ## {#terminology}

{::boilerplate bcp14-tagged}

Readers are expected to be familiar with the terms and concepts related to the SCHC framework {{RFC8724}}, the web-transfer protocol CoAP {{RFC7252}}, and the security protocols OSCORE {{RFC8613}} and Group OSCORE {{I-D.ietf-core-oscore-groupcomm}}.

# SCHC Applicability to CoAP # {#sec-applicability-to-coap}

SCHC compression for CoAP headers MAY be done in conjunction with the lower layers (IPv6/UDP) or independently. The SCHC adaptation layers, described in {{Section 5 of RFC8724}}, may be used as shown in {{fig-applicability-to-coap-1}}, {{fig-applicability-to-coap-2}}, and {{fig-applicability-to-coap-3}}.

In the first example depicted in {{fig-applicability-to-coap-1}}, a Rule compresses the complete header stack from IPv6 to CoAP. In this case, the Device and the Network Gateway (NGW) perform SCHC C/D (SCHC Compression/Decompression, see {{RFC8724}}). The application communicating with the Device does not implement SCHC C/D.

~~~~~~~~~~~ aasvg
 (Device)             (NGW)                            (App)

+--------+                                           +--------+
|  CoAP  |                                           |  CoAP  |
+--------+                                           +--------+
|  UDP   |                                           |  UDP   |
+--------+     +----------------+                    +--------+
|  IPv6  |     |      IPv6      |                    |  IPv6  |
+--------+     +--------+-------+                    +--------+
|  SCHC  |     |  SCHC  |       |                    |        |
+--------+     +--------+       +                    +        +
|  LPWAN |     | LPWAN  |       |                    |        |
+--------+     +--------+-------+                    +--------+
      ((((LPWAN))))           ------   Internet  -------
~~~~~~~~~~~
{: #fig-applicability-to-coap-1 title="Compression/Decompression at the LPWAN Boundary" artwork-align="center"}

{{fig-applicability-to-coap-1}} shows the use of SCHC header compression above Layer 2 in the Device and the NGW. The SCHC layer receives non-encrypted packets and can apply compression Rules to all the headers in the stack. On the other end, the NGW receives the SCHC packet and reconstructs the headers using the Rule and the Compression Residue. After the decompression, the NGW forwards the IPv6 packet toward the destination. The same process applies in the other direction when a non-encrypted packet arrives at the NGW. Thanks to the IP forwarding based on the IPv6 prefix, the NGW identifies the Device and compresses headers using the Device's Rules.

In the second example depicted in {{fig-applicability-to-coap-2}}, SCHC compression is applied in the CoAP layer, compressing the CoAP header independently of the other layers. The RuleID, Compression Residue, and CoAP payload are encrypted using a mechanism such as DTLS {{RFC9147}}. Only the other end (App) can decipher the information. If needed, layers below use SCHC to compress the header as defined in {{RFC8724}} (represented by dotted lines in the figure).

This use case needs an end-to-end context initialization between the Device and the application. The context initialization is out of scope for this document.

~~~~~~~~~~~ aasvg
 (Device)             (NGW)                            (App)

+--------+                                           +--------+
|  CoAP  |                                           |  CoAP  |
+--------+                                           +--------+
|  SCHC  |                                           |  SCHC  |
+--------+                                           +--------+
|  DTLS  |                                           |  DTLS  |
+--------+                                           +--------+
.  udp   .                                           .  udp   .
..........     ..................                    ..........
.  ipv6  .     .      ipv6      .                    .  ipv6  .
..........     ..................                    ..........
.  schc  .     .  schc  .       .                    .        .
..........     ..........       .                    .        .
.  lpwan .     . lpwan  .       .                    .        .
..........     ..................                    ..........
      ((((LPWAN))))           ------   Internet  -------
~~~~~~~~~~~
{: #fig-applicability-to-coap-2 title="Standalone CoAP End-to-End Compression/Decompression" artwork-align="center"}

The third example depicted in {{fig-applicability-to-coap-3}} shows the use of Object Security for Constrained RESTful Environments (OSCORE) {{RFC8613}}. In this case, SCHC needs two Rules to compress the CoAP header. A first Rule focuses on the Inner header. The result of this first compression is encrypted using the OSCORE mechanism. Then, a second Rule compresses the Outer header, including the CoAP Option OSCORE.

~~~~~~~~~~~ aasvg
 (Device)             (NGW)                            (App)

+--------+                                           +--------+
|  CoAP  |                                           |  CoAP  |
|  Inner |                                           |  Inner |
+--------+                                           +--------+
|  SCHC  |                                           |  SCHC  |
|  Inner |                                           |  Inner |
+--------+                                           +--------+
|  CoAP  |                                           |  CoAP  |
|  Outer |                                           |  Outer |
+--------+                                           +--------+
|  SCHC  |                                           |  SCHC  |
|  Outer |                                           |  Outer |
+--------+                                           +--------+
.  udp   .                                           .  udp   .
..........     ..................                    ..........
.  ipv6  .     .      ipv6      .                    .  ipv6  .
..........     ..................                    ..........
.  schc  .     .  schc  .       .                    .        .
..........     ..........       .                    .        .
.  lpwan .     . lpwan  .       .                    .        .
..........     ..................                    ..........
      ((((LPWAN))))           ------   Internet  -------
~~~~~~~~~~~
{: #fig-applicability-to-coap-3 title="OSCORE Compression/Decompression" artwork-align="center"}

In the case of several SCHC instances, as shown in {{fig-applicability-to-coap-2}} and {{fig-applicability-to-coap-3}}, the Rules may come from different provisioning domains.

This document focuses on CoAP compression, as represented by the dashed boxes in the previous figures.

# CoAP Headers Compressed with SCHC # {#sec-coap-header-compression}

The use of SCHC over the CoAP header applies the same description and compression/decompression techniques as the technique used for IP and UDP, as explained in {{RFC8724}}. For CoAP, the SCHC Rules description uses the direction information to optimize the compression by reducing the number of Rules needed to compress headers. The Field Descriptor MAY define both request/response headers and TVs in the same Rule, using the DI to indicate the header type.

As for other header compression protocols, when the compressor does not find a correct Rule to compress the header, the packet MUST be sent uncompressed using the RuleID dedicated to this purpose, and where the Compression Residue is the complete header of the packet (see {{Section 6 of RFC8724}}).

## Differences between CoAP and UDP/IP Compression  # {#ssec-differences-with-udp-ip}

CoAP compression differs from IPv6 and UDP compression in the following aspects:

* The CoAP message format is asymmetric, i.e., the headers are different for a request and a response.

  For example, the Uri-Path Option can be used in a request, while it is not used in a response. A request might contain an Accept Option, while both a request and a response might include a Content-Format Option. In comparison, the IPv6 and UDP returning path swaps the value of some fields in the header. However, all the directions have the same fields (e.g., source and destination address fields).

  {{RFC8724}} defines the use of a DI in the Field Descriptor, which allows a single Rule to process a message header differently, depending on the direction.

* Even when a field is "symmetric" (i.e., found in both directions), the values carried in each direction are different. The compression may use a "match-mapping" MO to limit the range of expected values in a particular direction and reduce the Compression Residue's size. Through the DI, a Field Descriptor in the Rules splits the possible field value into two parts, one for each direction.

  For instance, if a client sends only Confirmable (CON) requests {{RFC7252}}, the Type can be elided by compression, and the reply from the server may use one single bit to carry either the Acknowledgement (ACK) or Reset (RST) type. The field Code has the same behavior: the 0.0X code format value in a request and the Y.ZZ code format in a response.

* In SCHC, the Rule defines the different header fields' length, so SCHC does not need to send it. In IPv6 and UDP headers, the fields have a fixed size, known by definition.

  On the other hand, some CoAP header fields have variable lengths, and the Rule description specifies it. For example, the size of the Token field may vary from 0 to 8 bytes, and the CoAP options rely on the Type-Length-Value encoding format to specify the size of the actual option value in bytes.

  When doing SCHC compression of a variable-length field, {{Section 7.4.2 of RFC8724}} makes it possible to define a function for the Field Length in the Field Descriptor, in order to determine the length before compression. If the Field Length is unknown, the Rule will set it as a variable, and SCHC will send the compressed field's length in the Compression Residue.

* A field can appear several times in the CoAP headers. This is typically the case for elements of a URI (i.e., path segments or query arguments). The SCHC specification {{RFC8724}} allows a FID to appear several times in the Rule and uses the Field Position (FP) to identify the correct instance, thereby removing the MO's ambiguity.

* Field Lengths defined in CoAP can be too large when it comes to LPWAN traffic constraints. For instance, this is particularly true for the Message ID field and the Token field. SCHC uses different MOs to perform the compression (see {{Section 7.4 of RFC8724}}). In this case, SCHC can apply the Most Significant Bits (MSBs) MO to reduce the information carried on LPWANs.

# Compression of CoAP Header Fields # {#sec-coap-fields-compression}

This section discusses the SCHC compression of the CoAP header fields (see {{Section 3 of RFC7252}}), building on what is specified in {{Section 7.1 of RFC8724}}.

## CoAP Version Field # {#ssec-coap-version-field}

The Version field is described as bidirectional in a SCHC Rule, and it MUST be elided during SCHC compression, since it always contains the same value. In the future, or if a new version of CoAP is defined, new Rules will be needed to avoid ambiguities between versions.

## CoAP Type Field # {#ssec-coap-type-field}

The Type field specifies one of the four types of CoAP messages, encoded as specified in {{Section 3 of RFC7252}}: Confirmable (CON), Non-confirmable (NON), Acknowledgement (ACK), and Reset (RST).

The SCHC compression scheme SHOULD elide this field if, for instance, a client is sending only NON messages or only CON messages. For RST messages, SCHC may use a dedicated Rule. For other usages, SCHC can use a "match-mapping" MO.

## CoAP Token Length (TKL) Field # {#ssec-coap-tkl-field}

The Token Length (TKL) field specifies the size in bytes of the later Token field (see {{ssec-coap-token-field}}), and is described as bidirectional in a SCHC Rule.

If the field value does not change over time, the SCHC Rule describes the TV set to that value, the MO set to "equal", and the CDA set to "not-sent", thereby eliding the field.

Otherwise, if the field value changes over time, the SCHC Rule does not set the TV, while setting the MO to "ignore" and the CDA to "value-sent". The Rule may also use a "match-mapping" MO to compress the value.

## CoAP Code Field # {#ssec-coap-code-field}

The Code field takes value from the "Code" column of the "CoAP Codes" IANA registry, encoded as specified in {{Section 3 of RFC7252}}. This field indicates the Method Code of a CoAP request or the Response Code of a CoAP Response, while the value 0.00 indicates an Empty message. The compression of the CoAP Code field follows the same principle as that of the CoAP Type field.

If the Device plays a specific role, SCHC may split the code values into two Field Descriptors: (1) the Method Codes with the 0 class and (2) the Response Codes. SCHC will use the DI to identify the correct value in the packet. If the Device only implements a CoAP client, SCHC compression may focus only on the Method Codes that the client uses in its outgoing requests.

For known values, SCHC can use a "match-mapping" MO. If SCHC cannot compress the Code field, it will send the values in the Compression Residue.

## CoAP Message ID Field # {#ssec-coap-message-id-field}

SCHC can compress the Message ID field with the MSB MO and the LSB CDA (see {{Section 7.4 of RFC8724}}).

## CoAP Token Field # {#ssec-coap-token-field}

A CoAP message fully specifies the Token by using two CoAP fields: the Token Length (TKL) field in the mandatory header (see {{ssec-coap-tkl-field}}) and the variable-length Token field that directly follows the mandatory CoAP header and specifies the Token value.

For the Token field, SCHC MUST NOT send it as variable-size data in the Compression Residue, to avoid ambiguity with the Token Length field. Therefore, SCHC MUST use the value of the Token Length field to define the size of the Token field in the Compression Residue.

To this end, SCHC designates a specific function, "tkl", that the Rule MUST use to complete the Field Descriptor. During the decompression, this function returns the value contained in the Token Length field, hence the length of the Token field.

# Compression of CoAP Options # {#sec-coap-options}

CoAP defines options placed after the mandatory header and the Token field, ordered by option number (see {{Section 3 of RFC7252}}). As per {{Section 3.1 of RFC7252}}, each option instance in a message uses the format Option Delta (D), Option Length (L), Option Value (V).

In particular, the Option Delta is used to express the option number of a CoAP option within a CoAP message, as the difference between the Option Number of that option and the Option Number of the previous option in that message (or zero for the first option). Also, the Option Length specifies the length of the Option Value, in bytes.

In a SCHC Rule, the Field Descriptor related to a CoAP option is as follows:

* the FID is set to an unambiguous identifier of the CoAP option associated with the corresponding option number;

* the FL is set to the Option Length L of the CoAP option, encoded as per {{Section 7.1 of RFC8724}}; and

* the TV is set to the Option Value V of the CoAP option.

Note that the MO and the CDA specified in the Field Descriptor operates only on the Option Value V. That is, SCHC compression produces a residue from the Option Value V, while ignoring the option number, the Option Delta, and the Option Length. Therefore, the residue of a SCHC packet conveying a compressed CoAP header does not include the option number, the Option Delta, and the Option Length, which the recipient will be able to reconstruct by performing SCHC Decompression.

When the Option Length has a well-known value, the Rule may specify the Option Length value in the FL of the Field Descriptor (see above). In such a case, SCHC compression treats the Option Value as a fixed-length field (see {{Section 7.4.1 of RFC8724}}).

Otherwise, the Rule specifies the FL of the Field Descriptor as indicating a variable length, and SCHC compression treats the Option Value as a variable-length field (see {{Section 7.4.2 of RFC8724}}). In such a case, when the CDA specified in the Field Descriptor is "value-sent" or LSB, then SCHC compression additionally carries the length of the Compression Residue, as prepended to the Compression Residue value. Note that the length coding differs between CoAP options and the Compression Residue of SCHC variable-length fields.

CoAP requests and responses do not include the same options. Compression Rules may reflect this asymmetry by using the DI.

The following sections present how SCHC compresses some specific CoAP options. Unless otherwise indicated, the referred CoAP options are specified in {{RFC7252}}.

If the use of an additional CoAP option is later introduced, the SCHC Rules MAY be updated, in which case a new FID description MUST be assigned to perform the compression of the CoAP option. Otherwise, if no Rule describes that CoAP option, SCHC compression is not achieved, and SCHC sends the CoAP header without compression.

## CoAP Option Content-Format and Accept Fields # {#ssec-content-format-accept-option}

If the client expects a single specific value, SCHC can elide these fields, by specifying the value in the TV of a Rule description with an "equal" MO and a "not-sent" CDA.

Otherwise, if the client expects several possible values, a "match-mapping" MO SHOULD be used to limit the Compression Residue's size. If not, SCHC has to send the option value in the Compression Residue (with fixed or variable length).

## CoAP Option Max-Age, Uri-Host, and Uri-Port Fields # {#ssec-max-age-uri-host-uri-port-option}

SCHC compresses these three fields in the same way. When the values of these options are known, SCHC can elide these fields. If the option uses well-known values, SCHC can use a "match-mapping" MO.

Otherwise, these options' values will be sent in the Compression Residue, i.e., the SCHC Rule description does not set the TV, while setting the MO to "ignore" and the CDA to "value-sent".

## CoAP Option Uri-Path and Uri-Query Fields # {#ssec-uri-path-uri-query-option}

The Uri-Path and Uri-Query fields are repeatable options, i.e., the CoAP header may include them several times and with different values. The SCHC Rule description uses the FP to distinguish the different instances of such options.

To compress these repeatable field values, SCHC can use a "match-mapping" MO to reduce the size of variable paths or queries. When doing so, several elements can be regrouped into a single entry in order to optimize the compression. The numbering of elements does not change, and the first matching element sets the MO comparison.

For example, as per the Rule descriptions shown in {{table-complex-path}}, SCHC can use a single bit in the Compression Residue to code the path segments "/a/b" or the path segments "/c/d". If regrouping were not allowed, then 2 bits in the Compression Residue would be needed. At the same time, SCHC sends the third path element following "/a/b" or "/c/d" as a variable-size field in the Compression Residue.

| Field    | FL           | FP | DI | TV                        | MO            | CDA          |
|----------|--------------|----|----|---------------------------|---------------|--------------|
| Uri-Path |              | 1  | Up | \["/a/b", <br> "/c/d"\]   | match-mapping | mapping-sent |
| Uri-Path | var <br> (B) | 3  | Up |                           | ignore        | value-sent   |
{: #table-complex-path title="Complex Path Example" align="center"}

The length of the Uri-Path and Uri-Query Options may be known when the Rule is defined. In any other case, SCHC MUST set the Field Length (FL) to a variable value. The unit of the variable length is bytes, hence the Compression Residue size is expressed in bytes, encoded as defined in {{Section 7.4.2 of RFC8724}}.

SCHC compression can use the MSB MO for a Uri-Path or Uri-Query element. In such a case, care must be taken when specifying the MSB parameter value in bits, which MUST be a multiple of 8. The length sent at the beginning of the variable-size field Compression Residue indicates the LSB's size in bytes, consistent with the unit of the variable length in the Rule description.

For instance, for a CORECONF path /c/X6?k=eth0, the Rule description can be as shown in {{table-CoMicompress}}. That is, SCHC compresses the first part of the Uri-Path with a "not-sent" CDA. Also, SCHC will send the second element of the Uri-Path preceded with the length (i.e., 0b0010 "X6"), which is followed by the query option preceded with the length (i.e., 0b0100 "eth0").


| Field     | FL           | FP | DI | TV   | MO      | CDA        |
|-----------|--------------|----|----|------|---------|------------|
| Uri-Path  |              | 1  | Up | "c"  | equal   | not-sent   |
| Uri-Path  | var <br> (B) | 2  | Up |      | ignore  | value-sent |
| Uri-Query | var <br> (B) | 1  | Up | "k=" | MSB(16) | LSB        |
{: #table-CoMicompress title="CORECONF URI compression" align="center"}

### Variable Number of Path or Query Elements

SCHC fixes the number of Uri-Path or Uri-Query elements in a Rule at Rule creation time. If the number of such elements varies, SCHC SHOULD either:

* create several Rules to cover all possibilities; or

* create a Rule that defines several entries for Uri-Path to cover the longest path, and send a Compression Residue with a length of 0 to indicate that a Uri-Path entry is empty.

   However, this adds 4 bits to the variable Compression Residue size (see {{Section 7.4.2 of RFC8724}}).

## CoAP Option Size1, Size2, Proxy-Uri, and Proxy-Scheme Fields # {#ssec-size1-size2-proxy-uri-proxy-scheme-option}

The Size2 field is an option defined in {{RFC7959}}.

The SCHC Rule description MAY define sending some field values by not setting the TV, while setting the MO to "ignore" and the CDA to "value-sent". A Rule MAY also use a "match-mapping" MO when there are different alternatives for the same FID. Otherwise, the Rule sets the TV to a specific value, the MO to "equal", and the CDA to "not-sent".

## CoAP Option Proxy-Cri and Proxy-Scheme-Number Fields # {#ssec-proxy-cri-proxy-scheme-number-option}

The Proxy-Cri field is an option defined in {{I-D.ietf-core-href}}. The option carries an encoded CBOR data item {{RFC8949}} that represents an absolute CRI reference (see {{Section 5 of I-D.ietf-core-href}}). The option is used analogously to the Proxy-Uri option as defined in {{Section 5.10.2 of RFC7252}}.

The Proxy-Scheme-Number field is an option defined in {{I-D.ietf-core-href}}. The option carries a CRI Scheme Number represented as a CoAP unsigned integer (see {{Sections 5.1.1 and 8.1 of I-D.ietf-core-href}}). The option is used analogously to the Proxy-Scheme option as defined in {{Section 5.10.2 of RFC7252}}.

The SCHC Rule description MAY define sending some field values by not setting the TV, while setting the MO to "ignore" and the CDA to "value-sent". A Rule MAY also use a "match-mapping" MO when there are different alternatives for the same FID. Otherwise, the Rule sets the TV to a specific value, the MO to "equal", and the CDA to "not-sent".

## CoAP Location-Path and Location-Query Fields # {#ssec-location-path-location-query-option}

A Rule entry cannot store these fields' values. Therefore, SCHC compression MUST always send these values in the Compression Residue. That is, in the SCHC Rule, the TV is not set, while the MO is set to "ignore" and the CDA is set to "value-sent".

## CoAP Option ETag and If-Match Fields # {#ssec-etag-if-match-option}

When a CoAP message uses the ETag Option or the If-Match Option, SCHC compression MAY send its content in the Compression Residue. That is, in the SCHC Rule, the TV is not set, while the MO is set to "ignore" and the CDA is set to "value-sent". Alternatively, if a pre-defined set of values determined by the server is known and is used by the client as ETag values or If-Match values, then a Rule MAY use a "match-mapping" MO when there are different alternatives for the same FID.

## CoAP Option If-None-Match # {#ssec-if-none-match}

The If-None-Match Option occurs at most once and is always empty. The SCHC Rule MUST describe an empty TV, with the MO set to "equal" and the CDA set to "not-sent".

## CoAP Option Hop-Limit Field ## {#coap-options-hop-limit}

The Hop-Limit field is an option defined in {{RFC8768}} that can be used to detect forwarding loops through a chain of CoAP proxies. The first proxy in the chain that understands the option includes it in a received request with a proper value set, before forwarding the request. Any following proxy that understands the option decrements the option value and forwards the request if the new value is different than zero, or returns a 5.08 (Hop Limit Reached) error response otherwise.

When a CoAP message uses the Hop-Limit Option, SCHC compression SHOULD send its content in the Compression Residue. That is, in the SCHC Rule, the TV is not set, while the MO is set to "ignore" and the CDA is set to "value-sent". As an exception, and consistently with the default value 16 defined for the Hop-Limit Option in {{Section 3 of RFC8768}}, a Rule MAY describe a TV with value 16, with the MO set to "equal" and the CDA set to "not-sent".

## CoAP Option Echo Field ## {#coap-options-echo}

The Echo field is an option defined in {{RFC9175}} that a server can include in a CoAP response as a challenge to the client, and that the client echoes back to the server in one or more CoAP requests. This enables the server to verify the freshness of a request and to cryptographically verify the aliveness of the client. Also, it forces the client to demonstrate reachability at its claimed network address.

When a CoAP message uses the Echo Option, SCHC compression SHOULD send its content in the Compression Residue. That is, in the SCHC Rule, the TV is not set, while the MO is set to "ignore" and the CDA is set to "value-sent". An exception applies in case the server generates the values to use for the Echo Option by means of a persistent counter (see {{Section A of RFC9175}}). In such a case, a Rule MAY use the MSB MO and the LSB CDA. This would be effectively applicable until the persistent counter at the server becomes greater than the maximum threshold value that produces an MSB-matching.

## CoAP Option Request-Tag Field ## {#coap-options-request-tag}

The Request-Tag field is an option defined in {{RFC9175}} that the client can set in CoAP requests throughout block-wise operations, with value an ephemeral short-lived identifier of the specific block-wise operation in question. This allows the server to match message fragments belonging to the same request operation and, if the server supports it, to reliably process simultaneous block-wise request operations on a single resource. If requests are integrity protected, this also protects against interchange of fragments between different block-wise request operations.

When a CoAP message uses the Request-Tag Option, SCHC compression MAY send its content in the Compression Residue. That is, in the SCHC Rule, the TV is not set, while the MO is set to "ignore" and the CDA is set to "value-sent". Alternatively, if a pre-defined set of Request-Tag values used by the client is known, a Rule MAY use a "match-mapping" MO when there are different alternatives for the same FID.

## CoAP Option EDHOC Field ## {#coap-options-edhoc}

The EDHOC field is an option defined in {{RFC9668}} that a client can include in a CoAP request, in order to perform an optimized, shortened execution of the authenticated key exchange protocol EDHOC {{RFC9528}}. Such a request conveys both the final EDHOC message and actual application data, where the latter is protected with OSCORE {{RFC8613}} using a Security Context derived from the result of the current EDHOC execution.

The EDHOC Option occurs at most once and is always empty. The SCHC Rule MUST describe an empty TV, with the MO set to "equal" and the CDA set to "not-sent".

# Compression of CoAP Extensions # {#sec-coap-extensions}

## Block-Wise Transfers # {#ssec-coap-extensions-block}

When a CoAP message uses a Block1 or Block2 Option {{RFC7959}} or a Q-Block1 or Q-Block2 Option {{RFC9177}}, SCHC compression MUST send its content in the Compression Residue. In the SCHC Rule, the TV is not set, while the MO is set to "ignore" and the CDA is set to "value-sent".

The Block1, Block2, Q-Block1, and Q-Block2 options allow fragmentation at the CoAP level that is compatible with SCHC fragmentation. Both fragmentation mechanisms are complementary, and the node may use them for the same packet as needed.

## Observe # {#ssec-coap-extensions-observe}

{{RFC7641}} defines the Observe Option. The SCHC Rule description does not set the TV, while setting the MO to "ignore" and the CDA to "value-sent". SCHC does not limit the maximum size for this option (3 bytes). To reduce the transmission size, either the Device implementation MAY limit the delta between two consecutive values or a proxy can modify the increment.

Since the client MAY use a RST message to inform a server that the Observe response is not required, a specific SCHC Rule SHOULD exist to allow the compression of a RST message.

## No-Response # {#ssec-coap-extensions-no-response}

{{RFC7967}} defines a No-Response Option limiting the CoAP responses made by a server to a CoAP request. Different behaviors exist while using this option to limit the responses made by a server to a request. If both ends know the specific value, then the SCHC Rule describes the TV set to that value, the MO set to "equal", and the CDA set to "not-sent".

Otherwise, if the value changes over time, the SCHC Rule does not set the TV, while setting the MO to "ignore" and the CDA to "value-sent". The Rule may also use a "match-mapping" MO to compress the value.

## OSCORE # {#ssec-coap-extensions-oscore}

The security protocol OSCORE {{RFC8613}} provides end-to-end protection for CoAP messages. Group OSCORE {{I-D.ietf-core-oscore-groupcomm}} builds on OSCORE and defines end-to-end protection of CoAP messages in group communication {{I-D.ietf-core-groupcomm-bis}}. This section describes how SCHC Rules can be applied to compress messages protected with OSCORE or Group OSCORE.

{{fig-oscore-option}} shows the OSCORE Option value encoding, as it was originally defined in {{Section 6.1 of RFC8613}}. As explained later in this section, this has been extended in {{I-D.ietf-core-oscore-key-update}} and {{I-D.ietf-core-oscore-groupcomm}}. The first byte of the OSCORE Option value specifies information to parse the rest of the value by using flags, as described below.

* As defined in {{Section 4.1 of I-D.ietf-core-oscore-key-update}}, the eight least significant bit, when set, indicates that the OSCORE Option includes a second byte of flags. The seventh least significant bit is currently unassigned.

* As defined in {{Section 5 of I-D.ietf-core-oscore-groupcomm}}, the sixth least significant bit, when set, indicates that the message including the OSCORE option is protected with the group mode of Group OSCORE (see {{Section 8 of I-D.ietf-core-oscore-groupcomm}}). When not set, the bit indicates that the message is protected either with OSCORE, or with the pairwise mode of Group OSCORE (see {{Section 9 of I-D.ietf-core-oscore-groupcomm}}), while the specific OSCORE Security Context used to protect the message determines which of the two cases applies.

* As defined in {{Section 6.1 of RFC8613}}, bit h, when set, indicates the presence of the kid context field in the option. Also, bit k, when set, indicates the presence of the kid field. Finally, the three least significant bits form the n field, which indicates the length of the piv (Partial Initialization Vector) field in bytes. When n = 0, no piv is present.

Assuming the presence of a single flag byte, this is followed by the piv field. After that, if the h bit is set, the kid context field is present, preceded by one byte "s" indicating its length in bytes. After that, if the k bit is set, the kid field is present, and it ends where the OSCORE Option value ends.

~~~~~~~~~~~
 0 1 2 3 4 5 6 7 <--------- n bytes ------------->
+-+-+-+-+-+-+-+-+---------------------------------+
|0 0 0|h|k|  n  |        Partial IV (if any)      |
+-+-+-+-+-+-+-+-+---------------------------------+
|               |                                 |
|<--   CoAP  -->|<------- CoAP OSCORE_piv ------> |
   OSCORE_flags

 <-- 1 byte --> <------ s bytes ----->
+--------------+----------------------+-----------------------+
|  s (if any)  | kid context (if any) | kid (if any)      ... |
+--------------+----------------------+-----------------------+
|                                     |                       |
|<-------- CoAP OSCORE_kidctx ------->|<-- CoAP OSCORE_kid -->|
~~~~~~~~~~~
{: #fig-oscore-option title="OSCORE Option" artwork-align="center"}

{{fig-oscore-option-kudos}} shows the extended OSCORE Option value encoding, with the second byte of flags also present. As defined in {{Section 4.1 of I-D.ietf-core-oscore-key-update}}, the least significant bit d of this byte, when set, indicates that two additional fields are included in the option, following the kid context field (if any).

These two fields, namely x and nonce, are used when running the key update protocol KUDOS defined in {{I-D.ietf-core-oscore-key-update}}, with x specifying the length of the nonce field in bytes as well as the specific behavior to adopt during the KUDOS execution.

If the seventh least significant bit z of the x field is set, it indicates that two additional fields are included in the option, following the x and nonce fields. These two fields, namely y and old_nonce, are also used when running the key update protocol KUDOS, with y specifying the length of the old_nonce field in bytes.

{{fig-oscore-option-kudos}} provides the breakdown of the x field, where its four least significant bits form the sub-field m, which specifies the size of nonce in bytes, minus 1. Also, the figure provides the breakdown of the y field, where its four least significant bits form the sub-field w, which specifies the size of old_nonce in bytes, minus 1.

~~~~~~~~~~~
 0 1 2 3 4 5 6 7  8   9   10  11  12  13  14  15 <----- n bytes ----->
+-+-+-+-+-+-+-+-+---+---+---+---+---+---+---+---+---------------------+
|1|0|0|h|k|  n  | 0 | 0 | 0 | 0 | 0 | 0 | 0 | d | Partial IV (if any) |
+-+-+-+-+-+-+-+-+---+---+---+---+---+---+---+---+---------------------+
|                                               |                     |
|<------------------- CoAP -------------------->|<- CoAP OSCORE_piv ->|
                   OSCORE_flags

 <- 1 byte -> <----------- s bytes ------------>
+------------+----------------------------------+
| s (if any) |       kid context (if any)       |
+------------+----------------------------------+
|                                               |
|<------------- CoAP OSCORE_kidctx ------------>|


 <------ 1 byte -----> <----- m + 1 bytes ----->
+---------------------+-------------------------+
|     x (if any)      |      nonce (if any)     |
+---------------------+-------------------------+
|<-- CoAP OSCORE_x -->|<-- CoAP OSCORE_nonce -->|
|                     |
|   0 1 2 3 4 5 6 7   |
|  +-+-+-+-+-+-+-+-+  |
|  |0|z|b|p|   m   |  |
|  +-+-+-+-+-+-+-+-+  |


 <------ 1 byte ----->  <------ w + 1 bytes ------>
+---------------------+----------------------------+
|     y (if any)      |     old_nonce (if any)     |
+---------------------+----------------------------+
|<-- CoAP OSCORE_y -->|<-- CoAP OSCORE_oldnonce -->|
|                     |
|   0 1 2 3 4 5 6 7   |
|  +-+-+-+-+-+-+-+-+  |
|  |0|0|0|0|   w   |  |
|  +-+-+-+-+-+-+-+-+  |


+-----------------------+
|    kid (if any) ...   |
+-----------------------+
|                       |
|<-- CoAP OSCORE_kid -->|
~~~~~~~~~~~
{: #fig-oscore-option-kudos title="OSCORE Option extended to support a KUDOS execution" artwork-align="center"}

To better perform OSCORE SCHC compression, the Rule description needs to identify the OSCORE Option and the fields it contains.

Conceptually, it discerns up to eight distinct pieces of information within the OSCORE Option: the flag bits, the piv, the kid context prepended by its size, the x byte, the nonce, the y byte, the old_nonce, and the kid. The SCHC Rule splits the OSCORE Option into eight corresponding Field Descriptors in order to compress those pieces of information:

* CoAP OSCORE_flags
* CoAP OSCORE_piv
* CoAP OSCORE_kidctx
* CoAP OSCORE_x
* CoAP OSCORE_nonce
* CoAP OSCORE_y
* CoAP OSCORE_oldnonce
* CoAP OSCORE_kid

{{fig-oscore-option}} shows the original format of the OSCORE Option with the four fields OSCORE_flags, OSCORE_piv, OSCORE_kidctx, and OSCORE_kid superimposed on it. Also, {{fig-oscore-option-kudos}} shows the extended format of the OSCORE option with all the eight fields superimposed on it.

If a field is not present, then the corresponding Field Descriptor in the SCHC Rule describes the TV set to b'', with the MO set to "equal" and the CDA set to "not-sent". Note that, if the field kid context is present, it directly includes the size octet, s.

In addition, the following applies.

* For the x field, if both endpoints know the value, then the corresponding Field Descriptor in the SCHC Rule describes the TV set to that value, with the MO set to "equal" and the CDA set to "not-sent". This models: the case where the x field is not present, and thus TV is set to b''; and the case where the two endpoints run KUDOS with a pre-agreed size of the nonce field as per the m sub-field of the x field, as well as with a pre-agreed combination of its modes of operation, as per the bits b and p of the x field.

   Otherwise, if the value changes over time, then the corresponding Field Descriptor in the SCHC Rule does not set the TV, while it sets the MO to "ignore" and the CDA to "value-sent". The Rule may also use a "match-mapping" MO to compress this field, in case the two endpoints pre-agree on a set of alternative ways to run KUDOS, with respect to the size of the nonce field and the combination of the KUDOS modes of operation to use.

* For the y field, if both endpoints know the value, then the corresponding Field Descriptor in the SCHC Rule describes the TV set to that value, with the MO set to "equal" and the CDA set to "not-sent". This models: the case where the y field is not present, and thus TV is set to b''; and the case where the two endpoints run KUDOS with a pre-agreed size of the old_nonce field as per the w sub-field of the y field.

   Otherwise, if the value changes over time, then the corresponding Field Descriptor in the SCHC Rule does not set the TV, while it sets the MO to "ignore" and the CDA to "value-sent". The Rule may also use a "match-mapping" MO to compress this field, in case the two endpoints pre-agree on a set of sizes of the old_nonce field.

* For the nonce field, if it is not present (i.e., the x field is not present in the first place), then the corresponding Field Descriptor in the SCHC Rule describes the TV set to b'', with the MO set to "equal" and the CDA set to "not-sent".

   Otherwise, if the nonce field is present, then the corresponding Field Descriptor in the SCHC Rule has the TV not set, while the MO is set to "ignore" and the CDA is set to "value-sent". In such a case, for the value of the nonce field, SCHC MUST NOT send it as variable-length data in the Compression Residue, to avoid ambiguity with the length of the nonce field encoded in the x field. Therefore, SCHC MUST use the m sub-field of the x field to define the size of the Compression Residue. SCHC designates a specific function, "osc.x.m", that the Rule MUST use to complete the Field Descriptor. During the decompression, this function returns the length of the nonce field in bytes, as the value of the m sub-field of the x field, plus 1.

* For the old_nonce field, if it is not present (i.e., the y field is not present in the first place), then the corresponding Field Descriptor in the SCHC Rule describes the TV set to b'', with the MO set to "equal" and the CDA set to "not-sent".

   Otherwise, if the old_nonce field is present, then the corresponding Field Descriptor in the SCHC Rule has the TV not set, while the MO is set to "ignore" and the CDA is set to "value-sent". In such a case, for the value of the old_nonce field, SCHC MUST NOT send it as variable-length data in the Compression Residue, to avoid ambiguity with the length of the old_nonce field encoded in the y field. Therefore, SCHC MUST use the w sub-field of the y field to define the size of the Compression Residue. SCHC designates a specific function, "osc.y.w", that the Rule MUST use to complete the Field Descriptor. During the decompression, this function returns the length of the old_nonce field in bytes, as the value of the w sub-field of the y field, plus 1.

# Compression of the CoAP Payload Marker {#payload-marker}

The following applies with respect to the 0xFF payload marker. A SCHC compression Rule for CoAP includes all the expected CoAP options, therefore the payload marker does not have to be specified in a SCHC Rule description.

If the CoAP message to compress with SCHC is not going to be protected with OSCORE {{RFC8613}} and includes a payload, then the 0xFF payload marker MUST NOT be included in the compressed message, which is composed of the Compression RuleID, the Compression Residue (if any), and the CoAP payload.

After having decompressed an incoming message, the recipient endpoint MUST prepend the 0xFF payload marker to the CoAP payload, if any was present after the consumed Compression Residue.

If the CoAP message to compress with SCHC is going to be protected with OSCORE, the 0xFF payload marker is compressed as specified later in {{ssec-examples-oscore}}.

# Examples of CoAP Header Compression # {#sec-examples}

## Mandatory Header with CON Message # {#ssec-examples-con-message}

In this first scenario, the SCHC compressor on the NGW side receives a POST message from an Internet client, which is immediately acknowledged by the Device. {{table-CoAP-header-1}} describes the SCHC Rule descriptions for this scenario.

~~~~
+----------+
| RuleID 1 |
+----------+
~~~~

| Field              | FL  | FP | DI | TV                           | MO                  | CDA                | Sent <br> \[bits\] |
|--------------------|-----|----|----|------------------------------|---------------------|--------------------|--------------------|
| CoAP <br> Version  | 2   | 1  | Bi | 1                            | equal               | not-sent           |                    |
| CoAP Type          | 2   | 1  | Dw | CON                          | equal               | not-sent           |                    |
| CoAP Type          | 2   | 1  | Up | \[ACK, <br> RST\]            | match- <br> mapping | mapping- <br> sent | T                  |
| CoAP <br> TKL      | 4   | 1  | Bi | 0                            | equal               | not-sent           |                    |
| CoAP Code          | 8   | 1  | Bi | \[0.00, <br> ... <br> 5.05\] | match- <br> mapping | mapping- <br> sent | CC CCC             |
| CoAP <br> MID      | 16  | 1  | Bi | 0000                         | MSB(7)              | LSB                | MID                |
| CoAP <br> Uri-Path | var | 1  | Dw | "status"                     | equal               | not-sent           |                    |
{: #table-CoAP-header-1 title="CoAP Context to compress header without Token" align="center"}

In this example, SCHC compression elides the version and Token Length fields. The 25 Method and Response Codes defined in {{RFC7252}} have been shrunk to 5 bits using a "match-mapping" MO. The Uri-Path contains a single element with the TV set to "status" and the CDA set to "not-sent", thereby eliding the single occurrence of the Uri-Path Option with value "status".

SCHC compression reduces the header, sending only a mapped Type (and only for uplink messages), a mapped code, and the least significant bits of the Message ID (9 bits in the example above).

Note that, if a client is located in an Application Server and sends a request to a server located in the Device, then the request may not be compressed through this Rule, since the MID might not start with 7 bits equal to 0. A CoAP proxy placed before SCHC C/D can rewrite the Message ID to fit the value and match the Rule.

## OSCORE Compression # {#ssec-examples-oscore}

OSCORE aims to solve the problem of end-to-end protection for CoAP messages. Therefore, the goal is to hide as much as possible of the CoAP message, while still enabling proxy operations.

Conceptually, this is achieved by splitting the CoAP message into an Inner Plaintext and an Outer OSCORE message. The Inner Plaintext contains (sensitive) information that is not necessary for performing proxy operations. Therefore, that information can be encrypted end-to-end, until it reaches the other origin endpoint as its final destination. The Outer Message acts as a shell matching the regular CoAP message format, and includes all the CoAP options and information needed for performing proxy operations and caching. This is summarized in {{fig-inner-outer}}.

In particular, the CoAP options are arranged into three classes, each of which is granted a specific type of protection by the OSCORE protocol:

* Class E: Encrypted options moved to the Inner Plaintext.

* Class I: Integrity-protected options, included in the Additional Authenticated Data (AAD) when protecting the Plaintext, but otherwise left untouched in the Outer Message.

* Class U: Unprotected options, left untouched in the Outer Message.

As per these classes, the Outer options comprise the OSCORE Option, which indicates that the message is protected with OSCORE and carries the information necessary for the recipient endpoint to retrieve the Security Context for decrypting the message.

~~~~~~~~~~~ aasvg
                    Original CoAP Message
                 +-+-+---+-------+---------------+
                 |v|t|TKL| code  | Message ID    |
                 +-+-+---+-------+---------------+....+
                 | Token                              |
                 +-------------------------------.....+
                 | Options (IEU)            |
                 .                          .
                 .                          .
                 +------+-------------------+
                 | 0xFF |
                 +------+------------------------+
                 |                               |
                 |     Payload                   |
                 |                               |
                 +-------------------------------+
                        /                \
                       /                  \
                      /                    \
                     /                      \
   Outer Header     v                        v  Plaintext
+-+-+---+--------+---------------+          +-------+
|v|t|TKL|new code| Message ID    |          | code  |
+-+-+---+--------+---------------+....+     +-------+-----......+
| Token                               |     | Options (E)       |
+--------------------------------.....+     +-------+------.....+
| Options (IU)             |                | 0xFF  |
.                          .                +-------+-----------+
. OSCORE Option            .                |                   |
+------+-------------------+                | Payload           |
| 0xFF |                                    |                   |
+------+                                    +-------------------+

~~~~~~~~~~~
{: #fig-inner-outer title="CoAP Message Split into OSCORE Outer Header and Plaintext" artwork-align="center"}

{{fig-inner-outer}} shows the packet format for the OSCORE Outer header and Plaintext.

In the Outer header, the original header code is hidden and replaced by a well-known value. As specified in {{Sections 4.1.3.5 and 4.2 of RFC8613}}, the original header code is replaced with POST for requests and Changed for responses, when the message does not include the Observe Option. Otherwise, the original header code is replaced with FETCH for requests and Content for responses.

The first byte of the Plaintext contains the original header code, the class E options, and, if present, the original message payload preceded by the payload marker.

After that, an Authenticated Encryption with Associated Data (AEAD) algorithm encrypts the Plaintext. This also integrity-protects the Security Context parameters and, if present, any class I options from the Outer header. The resulting ciphertext becomes the new payload of the OSCORE message, as illustrated in {{fig-full-oscore}}.

As defined in {{RFC5116}}, this ciphertext is the encrypted Plaintext's concatenation with the Authentication Tag. Note that Inner Compression only affects the Plaintext before encryption. The Authentication Tag, fixed in length and uncompressed, is considered part of the cost of protection.

~~~~
   Outer Header
+-+-+---+--------+---------------+
|v|t|TKL|new code| Message ID    |
+-+-+---+--------+---------------+....+
| Token                               |
+--------------------------------.....+
| Options (IU)             |
.                          .
. OSCORE Option            .
+------+-------------------+
| 0xFF |
+------+---------------------------+
|                                  |
| Ciphertext: Encrypted Inner      |
|             Header and Payload   |
|             + Authentication Tag |
|                                  |
+----------------------------------+
~~~~
{: #fig-full-oscore title="OSCORE Message" artwork-align="center"}

The SCHC compression scheme consists of compressing both the Plaintext before encryption and the resulting OSCORE message after encryption, as shown in {{fig-OSCORE-Compression}}. Note that, since the recipient endpoint can only decrypt the Inner part of the message, that endpoint will also have to implement Inner SCHC Compression/Decompression.

~~~~~~~~~~~ aasvg
   Outer Message                               OSCORE Plaintext
+-+-+---+--------+---------------+            +-------+
|v|t|TKL|new code| Message ID    |            | code  |
+-+-+---+--------+---------------+....+       +-------+-------......+
| Token                               |       | Options (E)         |
+--------------------------------.....+       +-------+--------.....+
| Options (IU)             |                  | OxFF  |
.                          .                  +-------+-------------+
. OSCORE Option            .                  |                     |
+------+-------------------+                  | Payload             |
| 0xFF |                                      |                     |
+------+------------+                         +---------------------+
|                   |                                 |
| Ciphertext        |<---------+                      |
|                   |          |                      v
+-------------------+          |              +-----------------+
        |                      |              |   Inner SCHC    |
        |                      |              |   Compression   |
        v                      |              +-----------------+
  +-----------------+          |                      |
  |   Outer SCHC    |          |                      |
  |   Compression   |          |                      v
  +-----------------+          |                +----------+
        |                      |                | RuleID   |
        |                      |                +----------+----------+
        v                      |                | Compression Residue |
  +---------+               +------------+      +---------------------+
  | RuleID' |               | Encryption |<-----|                     |
  +---------+------------+  +------------+      | Payload             |
  | Compression Residue' |                      |                     |
  +----------------------+                      +---------------------+
  |                      |
  | Ciphertext           |
  |                      |
  +----------------------+
~~~~~~~~~~~
{: #fig-OSCORE-Compression title="OSCORE Compression Diagram" artwork-align="center"}

The OSCORE message translates into a segmented process where SCHC compression is applied independently in two stages, each with its corresponding set of Rules, i.e., the Inner SCHC Rules and the Outer SCHC Rules. By doing so, compression is applied to all the fields of the original CoAP message.

As to the compression of the 0xFF payload marker, the same rationale described in {{payload-marker}} applies to both the Inner SCHC Compression and the Outer SCHC Compression. That is:

* After the Inner SCHC Compression of a CoAP message including a payload, the payload marker MUST NOT be included in the input to the AEAD Encryption, which is composed of the Inner Compression RuleID, the Inner Compression Residue (if any), and the CoAP payload.

* The Outer SCHC Compression takes as input the OSCORE-protected message, which always includes a payload (i.e., the OSCORE Ciphertext) preceded by the payload marker.

* After the Outer SCHC Compression, the payload marker MUST NOT be included in the final compressed message, which is composed of the Outer Compression RuleID, the Outer Compression Residue (if any), and the OSCORE Ciphertext.

After having completed the Outer SCHC Decompression of an incoming message, the recipient endpoint MUST prepend the 0xFF payload marker to the OSCORE Ciphertext.

After having completed the Inner SCHC Decompression of an incoming message, the recipient endpoint MUST prepend the 0xFF payload marker to the CoAP payload, if any was present after the consumed Compression Residue.

## Example OSCORE Compression

This section provides an example with a GET request and a corresponding Content response, exchanged between a Device-based CoAP client and a cloud-based CoAP server. The example also describes a possible set of Rules for Inner SCHC Compression and Outer SCHC Compression. A dump of the results and a contrast between SCHC + OSCORE performance and SCHC + CoAP performance are also listed. This example shows an estimate of the cost of security with SCHC-OSCORE.

Our first CoAP message is the GET request in {{fig-GET-temp}}.

~~~~~~~~~~~
Original message:
=================
0x4101000182bb74656d7065726174757265

Header:
0x4101
01   Ver
  00   CON
    0001   TKL
        00000001   Request Code 1 "GET"

0x0001 = mid
0x82 = token

Options:

0xbb74656d7065726174757265
Option 11: Uri-Path
Value = temperature

Original message length: 17 bytes
~~~~~~~~~~~
{: #fig-GET-temp title="CoAP GET Request"}

Its corresponding response is the Content response in {{fig-CONTENT-temp}}.

~~~~~~~~~~~
Original message:
=================
0x6145000182ff32332043

Header:
0x6145
01   Ver
  10   ACK
    0001   TKL
        01000101 Successful Response Code 69 "2.05 Content"

0x0001 = mid
0x82 = token

0xFF  Payload marker

Payload:
0x32332043

Original message length: 10 bytes
~~~~~~~~~~~
{: #fig-CONTENT-temp title="CoAP Content Response"}

The SCHC Rules for the Inner Compression include all the fields present in a regular CoAP message. The methods described in {{sec-coap-fields-compression}} apply to these fields. Table 4 provides an example.

~~~~
 +----------+
 | RuleID 0 |
 +----------+
~~~~

| Field              | FL | FP | DI | TV            | MO                  | CDA                | Sent <br> \[bits\] |
|--------------------|----|----|----|---------------|---------------------|--------------------|--------------------|
| CoAP Code          | 8  | 1  | Up | 1             | equal               | not-sent           |                    |
| CoAP Code          | 8  | 1  | Dw | \[69, 132\]   | match- <br> mapping | mapping- <br> sent | C                  |
| CoAP <br> Uri-Path |    | 1  | Up | "temperature" | equal               | not-sent           |                    |
{: #table-Inner-Rules title="Inner SCHC Rule" align="center"}

{{fig-Inner-Compression-GET}} shows the Plaintext obtained for the example GET request. The packet follows the process of Inner Compression and encryption until the payload. The Outer OSCORE message adds the result of the Inner process.

~~~~~~~~~~~ aasvg
+---------------------------------------------------+
|                                                   |
| OSCORE Plaintext                                  |
|                                                   |
| 0x01bb74656d7065726174757265  (13 bytes)          |
|                                                   |
| 0x01 Request Code GET                             |
|                                                   |
|      bb74656d7065726174757265 Option 11: Uri-Path |
|                               Value = temperature |
|                                                   |
+---------------------------------------------------+
                            |
                            | Inner SCHC Compression
                            |
                            v
              +--------------------------+
              |                          |
              | Compressed Plaintext     |
              |                          |
              | 0x00                     |
              |                          |
              | RuleID = 0x00 (1 byte)   |
              | (No Compression Residue) |
              |                          |
              +--------------------------+
                            |
                            | AEAD Encryption
                            |  (piv = 0x04)
                            |
                            v
     +----------------------------------------------+
     |                                              |
     |  encrypted_plaintext = 0xa2 (1 byte)         |
     |  tag = 0xc54fe1b434297b62 (8 bytes)          |
     |                                              |
     |  ciphertext = 0xa2c54fe1b434297b62 (9 bytes) |
     |                                              |
     +----------------------------------------------+
~~~~~~~~~~~
{: #fig-Inner-Compression-GET title="Plaintext Compression and Encryption for GET Request" artwork-align="center"}

In this case, the original message has no payload, and its resulting Plaintext is compressed up to only 1 byte (the size of the RuleID). The AEAD algorithm preserves this length in its first output and yields a fixed-size tag. SCHC cannot compress the tag, and the OSCORE message must include it without compression. The use of integrity protection translates into an overhead on the total message length, thus limiting the amount of compression that can be achieved and contributing to the cost of adding security to the exchange.

{{fig-Inner-Compression-CONTENT}} shows the process for the example Content response. The Compression Residue is 1 bit long. Note that since SCHC adds padding after the payload, this misalignment causes the hexadecimal code from the payload to differ from the original, even if SCHC cannot compress the tag. The overhead for the tag bytes limits SCHC's performance but adds security to the exchange.

~~~~~~~~~~~ aasvg
+-------------------------------------------------+
|                                                 |
| OSCORE Plaintext                                |
|                                                 |
| 0x45ff32332043  (6 bytes)                       |
|                                                 |
| 0x45 Successful Response Code 69 "2.05 Content" |
|                                                 |
|     ff Payload marker                           |
|                                                 |
|       32332043 Payload                          |
|                                                 |
+-------------------------------------------------+
                            |
                            | Inner SCHC Compression
                            |
                            v
 +------------------------------------------------+
 |                                                |
 | Compressed Plaintext                           |
 |                                                |
 | 0x001919902180 (6 bytes)                       |
 |                                                |
 |   00 RuleID                                    |
 |                                                |
 |  0b0 (1 bit match-mapping Compression Residue) |
 |       0x32332043 >> 1 (shifted payload)        |
 |                        0b0000000 Padding       |
 |                                                |
 +------------------------------------------------+
                            |
                            | AEAD Encryption
                            |  (piv = 0x04)
                            |
                            v
 +---------------------------------------------------------+
 |                                                         |
 |  encrypted_plaintext = 0x10c6d7c26cc1 (6 bytes)         |
 |  tag = 0xe9aef3f2461e0c29 (8 bytes)                     |
 |                                                         |
 |  ciphertext = 0x10c6d7c26cc1e9aef3f2461e0c29 (14 bytes) |
 |                                                         |
 +---------------------------------------------------------+
~~~~~~~~~~~
{: #fig-Inner-Compression-CONTENT title="Plaintext Compression and Encryption for Content Response" artwork-align="center"}

The Outer SCHC Rule shown in {{table-Outer-Rules}} is used, also to process the OSCORE Option fields. {{fig-Protected-Compressed-GET}} and {{fig-Protected-Compressed-CONTENT}} show a dump of the OSCORE messages generated from the example messages, also including the Inner Compressed ciphertext in the payload. These are the messages that have to be compressed via the Outer SCHC Compression scheme.

{{table-Outer-Rules}} shows a possible set of Outer Rule items to compress the Outer header.

~~~~
+----------+
| RuleID 1 |
+----------+
~~~~

| Field                     | FL             | FP | DI | TV                   | MO      | CDA            | Sent <br> \[bits\] |
|---------------------------|----------------|----|----|----------------------|---------|----------------|--------------------|
| CoAP <br> Version         | 2              | 1  | Bi | 1                    | equal   | not- <br> sent |                    |
| CoAP <br> Type            | 2              | 1  | Up | 0                    | equal   | not- <br> sent |                    |
| CoAP <br> Type            | 2              | 1  | Dw | 2                    | equal   | not- <br> sent |                    |
| CoAP <br> TKL             | 4              | 1  | Bi | 1                    | equal   | not- <br> sent |                    |
| CoAP <br> Code            | 8              | 1  | Up | 2                    | equal   | not- <br> sent |                    |
| CoAP <br> Code            | 8              | 1  | Dw | 68                   | equal   | not- <br> sent |                    |
| CoAP <br> MID             | 16             | 1  | Bi | 0000                 | MSB(12) | LSB            | MMMM               |
| CoAP <br> Token           | tkl            | 1  | Bi | 0x80                 | MSB(5)  | LSB            | TTT                |
| CoAP <br> OSCORE_flags    | var            | 1  | Up | 0x09                 | equal   | not- <br> sent |                    |
| CoAP <br> OSCORE_piv      | var <br> (bit) | 1  | Up | 0x00                 | MSB(4)  | LSB            | PPPP               |
| CoAP <br> OSCORE_kid      | var <br> (bit) | 1  | Up | 0x636c69 <br> 656e70 | MSB(44) | LSB            | KKKK               |
| CoAP <br> OSCORE_kidctx   | var            | 1  | Bi | b''                  | equal   | not- <br> sent |                    |
| CoAP <br> OSCORE_x        | 8              | 1  | Bi | b''                  | equal   | not- <br> sent |                    |
| CoAP <br> OSCORE_nonce    | osc.x.m        | 1  | Bi | b''                  | equal   | not- <br> sent |                    |
| CoAP <br> OSCORE_y        | 8              | 1  | Bi | b''                  | equal   | not- <br> sent |                    |
| CoAP <br> OSCORE_oldnonce | osc.y.w        | 1  | Bi | b''                  | equal   | not- <br> sent |                    |
| CoAP <br> OSCORE_flags    | var            | 1  | Dw | b''                  | equal   | not- <br> sent |                    |
| CoAP <br> OSCORE_piv      | var            | 1  | Dw | b''                  | equal   | not- <br> sent |                    |
| CoAP <br> OSCORE_kid      | var            | 1  | Dw | b''                  | equal   | not- <br> sent |                    |
{: #table-Outer-Rules title="Outer SCHC Rule" align="center"}

~~~~~~~~~~~
Protected message:
==================
0x4102000182980904636c69656e74ffa2c54fe1b434297b62
(24 bytes)

Header:
0x4102
01   Ver
  00   CON
    0001   TKL
        00000010   Request Code 2 "POST"

0x0001 = mid
0x82 = token

Options:

0x98 0904636c69656e74 (9 bytes)
Option 9: OSCORE
Value = 0x0904636c69656e74
          09 = 000 0 1 001 flag byte
                   h k  n
            04 piv
              636c69656e74 kid

0xFF  Payload marker

Payload:
0xa2c54fe1b434297b62 (9 bytes)
~~~~~~~~~~~
{: #fig-Protected-Compressed-GET title="Protected and Inner SCHC Compressed GET Request" align="center"}

~~~~~~~~~~~
Protected message:
==================
0x614400018290ff10c6d7c26cc1e9aef3f2461e0c29
(21 bytes)

Header:
0x6144
01   Ver
  10   ACK
    0001   TKL
        01000100   Successful Response Code 68 "2.04 Changed"

0x0001 = mid
0x82 = token

Options:

0x90 (1 byte)
Option 9: OSCORE
Value = b''

0xFF  Payload marker

Payload:
0x10c6d7c26cc1e9aef3f2461e0c29 (14 bytes)
~~~~~~~~~~~
{: #fig-Protected-Compressed-CONTENT title="Protected and Inner SCHC Compressed Content Response" align="center"}

For the flag bits, some SCHC compression methods are useful, depending on the application. The most straightforward alternative is to provide a fixed value for the flags, combining an "equal" MO and a "not-sent" CDA. This SCHC description saves most bits but could prevent flexibility. Otherwise, SCHC could use a "match-mapping" MO to choose from several configurations for the exchange. If not, the SCHC description may use an MSB MO to mask off the three hard-coded most significant bits.

Note that fixing a flag bit will limit the possible OSCORE options that can be used in the exchange, since the value of the flag bits plays a role in determining a specific OSCORE option.

The piv field lends itself to having some bits masked off with an MSB MO and an LSB CDA. This SCHC description could be useful in applications where the message transmission frequency is low, such as with LPWAN technologies. Note that compressing the piv field may reduce the maximum number of sequence numbers that can be used in an exchange. Once the sequence number exceeds the maximum value, the OSCORE keys need to be re-established.

The size, s, that is included in the OSCORE_kidctx field MAY be masked off with an LSB CDA. The rest of the OSCORE_kidctx field could have additional bits masked off, or the whole field could be fixed in accordance with an "equal" MO and a "not-sent" CDA. The same holds for the OSCORE_kid field.

The Outer Rule of {{table-Outer-Rules}} is applied to the example GET request and Content response. {{fig-Compressed-GET}} and {{fig-Compressed-CONTENT}} show the resulting messages.

~~~~~~~~~~~
Compressed message:
==================
0x01148889458a9fc3686852f6c4 (13 bytes)
0x01 RuleID
    148889 compression residue
          458a9fc3686852f6c4 padded payload

Compression Residue:
0b 0001 010
    mid tkn

   0100 0100
         piv (residue size and residue)

   0100 0100
         kid (residue size and residue)

   (23 bits -> 3 bytes with padding)

Payload
0xa2c54fe1b434297b62 (9 bytes)

Compressed message length: 13 bytes
~~~~~~~~~~~
{: #fig-Compressed-GET title="SCHC-OSCORE Compressed GET Request"}

~~~~~~~~~~~
Compressed message:
==================
0x0114218daf84d983d35de7e48c3c1852 (16 bytes)
0x01 RuleID
    14 Compression Residue
      218daf84d983d35de7e48c3c1852 Padded payload

Compression Residue:
0b0001 010 (7 bits -> 1 byte with padding)
   mid tkn

Payload
0x10c6d7c26cc1e9aef3f2461e0c29 (14 bytes)
~~~~~~~~~~~
{: #fig-Compressed-CONTENT title="SCHC-OSCORE Compressed Content Response"}

In contrast, the following compares these results with what would be obtained by SCHC compressing the original CoAP messages without protecting them with OSCORE, according to the SCHC Rule in {{table-NoOsc-Rules}}.

~~~~
+----------+
| RuleID 2 |
+----------+
~~~~

| Field              | FL  | FP | DI | TV            | MO                  | CDA                | Sent <br> \[bits\] |
|--------------------|-----|----|----|---------------|---------------------|--------------------|--------------------|
| CoAP <br> Version  | 2   | 1  | Bi | 1             | equal               | not-sent           |                    |
| CoAP Type          | 2   | 1  | Up | 0             | equal               | not-sent           |                    |
| CoAP Type          | 2   | 1  | Dw | 2             | equal               | not-sent           |                    |
| CoAP <br> TKL      | 4   | 1  | Bi | 1             | equal               | not-sent           |                    |
| CoAP Code          | 8   | 1  | Up | 2             | equal               | not-sent           |                    |
| CoAP Code          | 8   | 1  | Dw | \[69, 132\]   | match- <br> mapping | mapping- <br> sent | C                  |
| CoAP <br> MID      | 16  | 1  | Bi | 0000          | MSB(12)             | LSB                | MMMM               |
| CoAP Token         | tkl | 1  | Bi | 0x80          | MSB(5)              | LSB                | TTT                |
| CoAP <br> Uri-Path |     | 1  | Up | "temperature" | equal               | not-sent           |                    |
{: #table-NoOsc-Rules title="SCHC-CoAP Rule (No OSCORE)" align="center"}

The Rule in {{table-NoOsc-Rules}} yields the SCHC compression results shown in {{fig-GET-temp-no-oscore}} for the request and in {{fig-CONTENT-temp-no-oscore}} for the response.

~~~~~~~~~~~
Compressed message:
==================
0x0214
0x02 = RuleID

Compression Residue:
0b00010100 (1 byte)

Compressed message length: 2 bytes
~~~~~~~~~~~
{: #fig-GET-temp-no-oscore title="CoAP GET Compressed without OSCORE"}

~~~~~~~~~~~
Compressed message:
==================
0x020a32332043
0x02 = RuleID

Compression Residue:
0b00001010 (1 byte)

Payload
0x32332043

Compressed message length: 6 bytes
~~~~~~~~~~~
{: #fig-CONTENT-temp-no-oscore title="CoAP Content Compressed without OSCORE"}

As can be seen, the difference between applying SCHC + OSCORE as compared to regular SCHC + CoAP is about 10 bytes.

# CoAP Header Compression with Proxies ## {#compression-with-proxies}

This section defines how SCHC Compression/Decompression is performed when CoAP proxies are deployed. The following refers to the origin client and origin server as application endpoints.

Note that SCHC Compression/Decompression of CoAP headers is not necessarily used between each pair of hops in the communication chain. For example, if a proxy is deployed between an origin client and an origin server, SCHC might be used on the communication leg between the origin client and the proxy, but not on the communication leg between the proxy and the origin server.

## Without End-to-End Security ## {#compression-with-proxies-without-oscore}

In case OSCORE is not used end-to-end between client and server, the SCHC processing occurs hop-by-hop, by relying on SCHC Rules that are consistently shared between two adjacent hops.

In particular, SCHC is used as defined below.

* The sender application endpoint compresses the CoAP message, by using the SCHC Rules that it shares with the next hop towards the recipient application endpoint. The resulting, compressed message is sent to the next hop towards the recipient application endpoint.

* Each proxy decompresses the incoming compressed message, by using the SCHC Rules that it shares with the (previous hop towards the) sender application endpoint.

   Then, the proxy compresses the CoAP message to be forwarded, by using the SCHC Rules that it shares with the (next hop towards the) recipient application endpoint.

   The resulting, compressed message is sent to the (next hop towards the) recipient application endpoint.

* The recipient application endpoint decompresses the incoming compressed message, by using the SCHC Rules that it shares with the previous hop towards the sender application endpoint.

## With End-to-End Security ## {#compression-with-proxies-with-oscore}

In case OSCORE is used end-to-end between client and server (see {{ssec-examples-oscore}}), the following applies.

The SCHC processing occurs end-to-end as to the Inner SCHC Compression/Decompression, by relying on Inner SCHC Rules that are consistently shared between the two application endpoints acting as OSCORE endpoints and sharing the used OSCORE Security Context.

Instead, the SCHC processing occurs hop-by-hop as to the Outer SCHC Compression/Decompression, by relying on Outer SCHC Rules that are consistently shared between two adjacent hops.

In particular, SCHC is used as defined below.

* The sender application endpoint performs the Inner SCHC Compression on the original CoAP message, by using the Inner SCHC Rules that it shares with the recipient application endpoint.

   Following the AEAD Encryption of the compressed input obtained from the previous step, the sender application endpoint performs the Outer SCHC Compression on the resulting OSCORE-protected message, by using the Outer SCHC Rules that it shares with the next hop towards the recipient application endpoint.

   The resulting, compressed message is sent to the next hop towards the recipient application endpoint.

* Each proxy performs the Outer SCHC Decompression on the incoming compressed message, by using the SCHC Rules that it shares with the (previous hop towards the) sender application endpoint.

   Then, the proxy performs the Outer SCHC Compression of the OSCORE-protected message to be forwarded, by using the SCHC Rules that it shares with the (next hop towards the) recipient application endpoint.

   The resulting, compressed message is sent to the (next hop towards the) recipient application endpoint.

* The recipient application endpoint performs the Outer SCHC Decompression on the incoming compressed message, by using the Outer SCHC Rules that it shares with the previous hop towards the sender application endpoint.

   Then, the recipient application endpoint performs the AEAD Decryption of the OSCORE-protected message obtained from the previous step.

   Finally, the recipient application endpoint performs the Inner SCHC Decompression on the compressed input obtained from the previous step, by using the Inner SCHC Rules that it shares with the sender application endpoint. The result is the original CoAP message produced by the sender application endpoint.

# Examples of CoAP Header Compression with Proxies # {#examples}

This section provides examples of SCHC Compression/Decompression in the presence of a CoAP proxy.

The presented examples refer to the same deployment considered in {{sec-applicability-to-coap}}, including a Device communicating over LPWAN with a Network Gateway (NGW), which in turn communicates with an Application Server over the Internet. The Application Server and the Device exchange CoAP messages through the NGW.

The following also applies in the presented examples.

* CoAP request messages are sent only by the Device as targeting the Application Server (uplink traffic), which replies to the Device with corresponding CoAP response messages (downlink traffic). That is, the Device acts as CoAP client, while the Application Server acts as CoAP server.

* A CoAP proxy is co-located on the Network Gateway (NGW) deployed between the Application Server and the Device.

* SCHC is used also on the communication leg between the Application Server and the proxy.

Like in {{sec-applicability-to-coap}}, the presented examples focus on SCHC Compression/Decompression of CoAP headers, i.e., irrespective of possible SCHC Compression/Decompression applied to further protocol headers.

The example in {{examples-without-oscore}} considers an exchange of two unprotected messages, while the example in {{examples-with-oscore}} considers an exchange of two messages protected end-to-end with OSCORE. In the examples, the character \| denotes bit concatenation.

{{fig-example-req}} and {{fig-example-resp}} show the two CoAP messages exchanged between the Device and the Application Server, via the proxy. The figures show the two messages as originally generated by the application at the two origin endpoints, i.e., before they are possibly protected end-to-end with OSCORE as considered by the example in {{examples-with-oscore}}.

In particular, note that:

* On the communication leg between the Device and the proxy, the CoAP Message ID has value 0x0001 and the CoAP Token has value 0x82.

* On the communication leg between the proxy and the Application Server, the CoAP Message ID has value 0x0004 and the CoAP Token has value 0x75.

~~~~~~~~~~~

Original message:
=================
0x41010001823b6578616d706c652e636f6d
  8b74656d7065726174757265d40f636f6170

Header:
0x4101
01   Ver
  00   CON
    0001   TKL
        00000001   Request Code 1 "GET"

0x0001 = mid
0x82 = token

Options:

0x3b6578616d706c652e636f6d
Option 3: Uri-Host
Value = example.com

0x8b74656d7065726174757265
Option 11: Uri-Path
Value = temperature

0xd40f636f6170
Option 39: Proxy-Scheme
Value = coap

Original message length: 35 bytes

~~~~~~~~~~~
{: #fig-example-req title="CoAP GET Request" artwork-align="left"}

~~~~~~~~~~~

Original message:
=================
0x6145000475ff32332043

Header:
0x6145
01   Ver
  10   ACK
    0001   TKL
        01000101 Successful Response Code 69 "2.05 Content"

0x0004 = mid
0x75 = token


0xFF Payload marker

Payload:
0x32332043

Original message length: 10 bytes

~~~~~~~~~~~
{: #fig-example-resp title="CoAP Content Response" artwork-align="left"}

## Without End-to-End Security ## {#examples-without-oscore}

In case OSCORE is not used end-to-end between the Device and the Application Server, the following SCHC Rules are shared between the different entities. Based on those Rules, the SCHC Compression/Decompression is performed as per {{compression-with-proxies-without-oscore}}.

The Device and the proxy share the SCHC Rule shown in {{fig-rules-device-proxy}}, with RuleID 0.


~~~~
+----------+
| RuleID 0 |
+----------+
~~~~

| Field                  | FL           | FP | DI | TV                       | MO                  | CDA                | Sent <br> \[bits\] |
|------------------------|--------------|----|----|--------------------------|---------------------|--------------------|--------------------|
| CoAP <br> Version      | 2            | 1  | Bi | 1                        | equal               | not-sent           |                    |
| CoAP <br> Type         | 2            | 1  | Up | 0                        | equal               | not-sent           |                    |
| CoAP <br> Type         | 2            | 1  | Dw | \[0, 2\]                 | match- <br> mapping | mapping- <br> sent | T                  |
| CoAP <br> TKL          | 4            | 1  | Bi | 1                        | equal               | not-sent           |                    |
| CoAP <br> Code         | 8            | 1  | Up | \[1, 2, <br> 3, 4\]      | match- <br> mapping | mapping- <br> sent | CC                 |
| CoAP <br> Code         | 8            | 1  | Dw | \[65, 68, <br> 69, 132\] | match- <br> mapping | mapping- <br> sent | CC                 |
| CoAP <br> MID          | 16           | 1  | Bi | 0x00                     | MSB(12)             | LSB                | MMMM               |
| CoAP <br> Token        | tkl          | 1  | Bi | 0x80                     | MSB(5)              | LSB                | TTT                |
| CoAP <br> Uri-Host     | var <br> (B) | 1  | Up |                          | ignore              | value- <br> sent   |                    |
| CoAP <br> Uri-Path     | var          | 1  | Up | "temperature"            | equal               | not-sent           |                    |
| CoAP <br> Proxy-Scheme | var          | 1  | Up | "coap"                   | equal               | not-sent           |                    |
{: #fig-rules-device-proxy title="SCHC Rule between the Device and the Proxy" align="center"}

Instead, the proxy and the Application Server share the SCHC Rule shown in {{fig-rules-proxy-server}}, with RuleID 1.

~~~~
+----------+
| RuleID 1 |
+----------+
~~~~

| Field              | FL           | FP | DI | TV                       | MO                  | CDA                | Sent <br> \[bits\] |
|--------------------|--------------|----|----|--------------------------|---------------------|--------------------|--------------------|
| CoAP <br> Version  | 2            | 1  | Bi | 1                        | equal               | not-sent           |                    |
| CoAP <br> Type     | 2            | 1  | Up | 0                        | equal               | not-sent           |                    |
| CoAP <br> Type     | 2            | 1  | Dw | \[0, 2\]                 | match- <br> mapping | mapping- <br> sent | T                  |
| CoAP <br> TKL      | 4            | 1  | Bi | 1                        | equal               | not-sent           |                    |
| CoAP <br> Code     | 8            | 1  | Up | \[1, 2, <br> 3, 4\]      | match- <br> mapping | mapping- <br> sent | CC                 |
| CoAP <br> Code     | 8            | 1  | Dw | \[65, 68, <br> 69, 132\] | match- <br> mapping | mapping- <br> sent | CC                 |
| CoAP <br> MID      | 16           | 1  | Bi | 0x00                     | MSB(12)             | LSB                | MMMM               |
| CoAP <br> Token    | tkl          | 1  | Bi | 0x70                     | MSB(5)              | LSB                | TTT                |
| CoAP <br> Uri-Host | var <br> (B) | 1  | Up |                          | ignore              | value- <br> sent   |                    |
| CoAP <br> Uri-Path | var          | 1  | Up | "temperature"            | equal               | not-sent           |                    |
{: #fig-rules-proxy-server title="SCHC Rule between the Proxy and the Application Server" align="center"}

First, the Device applies the Rule in {{fig-rules-device-proxy}} shared with the proxy to the CoAP request in {{fig-example-req}}. The result is the compressed CoAP request in {{fig-example-req-to-proxy}}, which the Device sends to the proxy.

~~~~~~~~~~~

Compressed message:
=================
0x00055b2bc30b6b836329731b7b68 (14 bytes)
0x00 RuleID
    055b2bc30b6b836329731b7b68 compression residue
                                and padded payload

Compression Residue (101 bits -> 13 bytes with padding)
0b   00 0001 010      1011  |  0x6578616d706c652e636f6d
   code  mid tkn  Uri-Host (residue size and residue)

Compressed message length: 14 bytes

~~~~~~~~~~~
{: #fig-example-req-to-proxy title="CoAP GET Request Compressed for the Proxy" artwork-align="left"}

Upon receiving the message in {{fig-example-req-to-proxy}}, the proxy decompresses it with the Rule in {{fig-rules-device-proxy}} shared with the Device, and obtains the same CoAP request in {{fig-example-req}}.

After that, the proxy removes the Proxy-Scheme Option from the decompressed CoAP request. Also, the proxy replaces the values of the CoAP Message ID and of the CoAP Token to 0x0004 and 0x75, respectively. The result is the CoAP request shown in {{fig-example-req-from-proxy}}.

~~~~~~~~~~~

Message to forward:
=================
0x41010004753b6578616d706c652e636f6d
  8b74656d7065726174757265

Header:
0x4101
01   Ver
  00   CON
    0001   TKL
        00000001   Request Code 1 "GET"

0x0004 = mid
0x75 = token

Options:

0x3b6578616d706c652e636f6d
Option 3: Uri-Host
Value = example.com

0x8b74656d7065726174757265
Option 11: Uri-Path
Value = temperature

Original message length: 29 bytes

~~~~~~~~~~~
{: #fig-example-req-from-proxy title="CoAP GET Request to be Compressed by the Proxy" artwork-align="left"}

Then, the proxy applies the Rule in {{fig-rules-proxy-server}} shared with the Application Server to the CoAP request in {{fig-example-req-from-proxy}}.

The result is the compressed CoAP request in {{fig-example-req-from-proxy-compressed}}, which the proxy forwards to the Application Server.

~~~~~~~~~~~

Compressed message to forward:
=================
0x0112db2bc30b6b836329731b7b68 (14 bytes)
0x01 RuleID
    12db2bc30b6b836329731b7b68 compression residue
                                and padded payload


Compression Residue (101 bits -> 13 bytes with padding)
0b   00 0100 101      1011  |  0x6578616d706c652e636f6d
   code  mid tkn  Uri-Host (residue size and residue)

Compressed message length: 14 bytes

~~~~~~~~~~~
{: #fig-example-req-from-proxy-compressed title="CoAP GET Request Forwarded by the Proxy" artwork-align="left"}

Upon receiving the message in {{fig-example-req-from-proxy-compressed}}, the Application Server decompresses it using the Rule in {{fig-rules-proxy-server}} shared with the proxy. The result is the same CoAP request in {{fig-example-req-from-proxy}}, which the Application Server delivers to the application.

After that, the Application Server produces the CoAP response in {{fig-example-resp}}, and compresses it using the Rule in {{fig-rules-proxy-server}} shared with the proxy. The result is the compressed CoAP response shown in {{fig-example-resp-to-proxy}}, which the Application Server sends to the proxy.

~~~~~~~~~~~

Compressed message:
=================
0x01c94c8cc810c0 (7 bytes)
0x01 RuleID
    c94c8cc810c0 compression residue
                 and padded payload


Compression Residue (10 bits -> 2 bytes with padding)
0b    1   10 0100 101
   type code  mid tkn

Payload
0x32332043 (4 bytes)

Compressed message length: 7 bytes

~~~~~~~~~~~
{: #fig-example-resp-to-proxy title="CoAP Content Response Compressed for the Proxy" artwork-align="left"}

Upon receiving the message in {{fig-example-resp-to-proxy}}, the proxy decompresses it using the Rule in {{fig-rules-proxy-server}} shared with the Application Server. The result is the same CoAP response in {{fig-example-resp}}.

Then, the proxy replaces the values of the CoAP Message ID and of the CoAP Token to 0x0001 and 0x82, respectively. The result is the CoAP response shown in {{fig-example-resp-from-proxy}}.

~~~~~~~~~~~

Message to forward:
=================
0x6145000182ff32332043

Header:
0x6145
01   Ver
  10   ACK
    0001   TKL
        01000101 Successful Response Code 69 "2.05 Content"

0x0001 = mid
0x82 = token


0xFF Payload marker

Payload:
0x32332043

Original message length: 10 bytes

~~~~~~~~~~~
{: #fig-example-resp-from-proxy title="CoAP Content Response to be Compressed by the Proxy" artwork-align="left"}

Then, the proxy compresses the CoAP response in {{fig-example-resp-from-proxy}} with the Rule in {{fig-rules-device-proxy}} shared with the Device. The result is the compressed CoAP response shown in {{fig-example-resp-from-proxy-compressed}}, which the proxy forwards to the Device.

~~~~~~~~~~~

Compressed message:
=================
0x00c28c8cc810c0 (7 bytes)
0x00 RuleID
    c28c8cc810c0 compression residue
                 and padded payload


Compression Residue (10 bits -> 2 bytes with padding)
0b    1   10 0001 010
   type code  mid tkn

Payload
0x32332043 (4 bytes)

Compressed message length: 7 bytes

~~~~~~~~~~~
{: #fig-example-resp-from-proxy-compressed title="CoAP Content Response Forwarded by the Proxy" artwork-align="left"}

Upon receiving the message in {{fig-example-resp-from-proxy-compressed}}, the Device decompresses it using the Rule in {{fig-rules-device-proxy}} shared with the proxy. The result is the same CoAP request in {{fig-example-resp-from-proxy}}, which the Device delivers to the application.

## With End-to-End Security ## {#examples-with-oscore}

In case OSCORE is used end-to-end between the Device and the Application Server, the following SCHC Rules are shared between the different entities. Based on those Rules, the SCHC Compression/Decompression is performed as per {{compression-with-proxies-with-oscore}}.

The Device and the Application Server share the SCHC Rule shown in {{fig-rules-oscore-device-server}}, with RuleID 2. The Device and the Application Server use this Rule to perform the Inner SCHC Compression/Decompression end-to-end.

~~~~
+----------+
| RuleID 2 |
+----------+
~~~~

| Field              | FL  | FP | DI | TV                       | MO                  | CDA                | Sent <br> \[bits\] |
|--------------------|-----|----|----|--------------------------|---------------------|--------------------|--------------------|
| CoAP <br> Code     | 8   | 1  | Up | \[1, 2, <br> 3, 4\]      | match- <br> mapping | mapping- <br> sent | CC                 |
| CoAP <br> Code     | 8   | 1  | Dw | \[65, 68, <br> 69, 132\] | match- <br> mapping | mapping- <br> sent | CC                 |
| CoAP <br> Uri-Path | var | 1  | Up | "temperature"            | equal               | not-sent           |                    |
{: #fig-rules-oscore-device-server title="Inner SCHC Rule between the Device and the Application Server" align="center"}

The Device and the proxy share the SCHC Rule shown in {{fig-rules-oscore-device-proxy}}, with RuleID 3. The Device and the proxy use this Rule to perform the Outer SCHC Compression/Decompression hop-by-hop on their communication leg.

~~~~
+----------+
| RuleID 3 |
+----------+
~~~~

| Field                     | FL             | FP | DI | TV       | MO                  | CDA                | Sent <br> \[bits\] |
|---------------------------|----------------|----|----|----------|---------------------|--------------------|--------------------|
| CoAP <br> Version         | 2              | 1  | Bi | 1        | equal               | not-sent           |                    |
| CoAP <br> Type            | 2              | 1  | Up | 0        | equal               | not-sent           |                    |
| CoAP <br> Type            | 2              | 1  | Dw | \[0, 2\] | match- <br> mapping | mapping- <br> sent | T                  |
| CoAP <br> TKL             | 4              | 1  | Bi | 1        | equal               | not-sent           |                    |
| CoAP <br> Code            | 8              | 1  | Up | 2        | equal               | not-sent           |                    |
| CoAP <br> Code            | 8              | 1  | Dw | 68       | equal               | not-sent           |                    |
| CoAP <br> MID             | 16             | 1  | Bi | 0x00     | MSB(12)             | LSB                | MMMM               |
| CoAP <br> Token           | tkl            | 1  | Bi | 0x80     | MSB(5)              | LSB                | TTT                |
| CoAP <br> Uri-Host        | var <br> (B)   | 1  | Up |          | ignore              | value- <br> sent   |                    |
| CoAP <br> OSCORE_flags    | var            | 1  | Up | 0x09     | equal               | not-sent           |                    |
| CoAP <br> OSCORE_piv      | var <br> (bit) | 1  | Up | 0x00     | MSB(4)              | LSB                | PPPP               |
| CoAP <br> OSCORE_kid      | var <br> (bit) | 1  | Up | 0x0000   | MSB(12)             | LSB                | KKKK               |
| CoAP <br> OSCORE_kidctx   | var            | 1  | Bi | b''      | equal               | not-sent           |                    |
| CoAP <br> OSCORE_x        | 8              | 1  | Bi | b''      | equal               | not-sent           |                    |
| CoAP <br> OSCORE_nonce    | osc.x.m        | 1  | Bi | b''      | equal               | not-sent           |                    |
| CoAP <br> OSCORE_y        | 8              | 1  | Bi | b''      | equal               | not-sent           |                    |
| CoAP <br> OSCORE_oldnonce | osc.y.w        | 1  | Bi | b''      | equal               | not-sent           |                    |
| CoAP <br> OSCORE_flags    | var            | 1  | Dw | b''      | equal               | not-sent           |                    |
| CoAP <br> OSCORE_piv      | var            | 1  | Dw | b''      | equal               | not-sent           |                    |
| CoAP <br> OSCORE_kid      | var            | 1  | Dw | b''      | equal               | not-sent           |                    |
| CoAP <br> Proxy-Scheme    | var            | 1  | Up | "coap"   | equal               | not-sent           |                    |
{: #fig-rules-oscore-device-proxy title="Outer SCHC Rule between the Device and the Proxy" align="center"}

The proxy and the Application Server share the SCHC Rule shown in {{fig-rules-oscore-proxy-server}}, with RuleID 4. The proxy and the Application Server use this Rule to perform the Outer SCHC Compression/Decompression hop-by-hop on their communication leg.

~~~~
 +----------+
 | RuleID 4 |
 +----------+
~~~~

| Field                     | FL             | FP | DI | TV       | MO                  | CDA                | Sent <br> \[bits\] |
|---------------------------|----------------|----|----|----------|---------------------|--------------------|--------------------|
| CoAP <br> Version         | 2              | 1  | Bi | 1        | equal               | not-sent           |                    |
| CoAP <br> Type            | 2              | 1  | Up | 0        | equal               | not-sent           |                    |
| CoAP <br> Type            | 2              | 1  | Dw | \[0, 2\] | match- <br> mapping | mapping- <br> sent | T                  |
| CoAP <br> TKL             | 4              | 1  | Bi | 1        | equal               | not-sent           |                    |
| CoAP <br> Code            | 8              | 1  | Up | 2        | equal               | not-sent           |                    |
| CoAP <br> Code            | 8              | 1  | Dw | 68       | equal               | not-sent           |                    |
| CoAP <br> MID             | 16             | 1  | Bi | 0x00     | MSB(12)             | LSB                | MMMM               |
| CoAP <br> Token           | tkl            | 1  | Bi | 0x70     | MSB(5)              | LSB                | TTT                |
| CoAP <br> Uri-Host        | var <br> (B)   | 1  | Up |          | ignore              | value- <br> sent   |                    |
| CoAP <br> OSCORE_flags    | var            | 1  | Up | 0x09     | equal               | not-sent           |                    |
| CoAP <br> OSCORE_piv      | var <br> (bit) | 1  | Up | 0x00     | MSB(4)              | LSB                | PPPP               |
| CoAP <br> OSCORE_kid      | var <br> (bit) | 1  | Up | 0x0000   | MSB(12)             | LSB                | KKKK               |
| CoAP <br> OSCORE_kidctx   | var            | 1  | Bi | b''      | equal               | not-sent           |                    |
| CoAP <br> OSCORE_x        | 8              | 1  | Bi | b''      | equal               | not-sent           |                    |
| CoAP <br> OSCORE_nonce    | osc.x.m        | 1  | Bi | b''      | equal               | not-sent           |                    |
| CoAP <br> OSCORE_y        | 8              | 1  | Bi | b''      | equal               | not-sent           |                    |
| CoAP <br> OSCORE_oldnonce | osc.y.w        | 1  | Bi | b''      | equal               | not-sent           |                    |
| CoAP <br> OSCORE_flags    | var            | 1  | Dw | b''      | equal               | not-sent           |                    |
| CoAP <br> OSCORE_piv      | var            | 1  | Dw | b''      | equal               | not-sent           |                    |
| CoAP <br> OSCORE_kid      | var            | 1  | Dw | b''      | equal               | not-sent           |                    |
{: #fig-rules-oscore-proxy-server title="Outer SCHC Rule between the Proxy and the Application Server" align="center"}

When the Device applies the Rule in {{fig-rules-oscore-device-server}} shared with the Application Server to the CoAP request in {{fig-example-req}}, this results in the Compressed Plaintext shown in {{fig-plaintext-req}}.

As per {{ssec-examples-oscore}}, the message follows the process of SCHC Inner Compression and encryption until the payload (if any). The ciphertext resulting from the overall Inner process is used as payload of the Outer OSCORE message.

~~~~~~~~~~~ aasvg

+-----------------------------------------------------+
|                                                     |
| OSCORE Plaintext                                    |
|                                                     |
| 0x01bb74656d7065726174757265  (13 bytes)            |
|                                                     |
| 0x01 Request Code GET                               |
|                                                     |
|      0xbb74656d7065726174757265 Option 11: Uri-Path |
|                                 Value = temperature |
|                                                     |
+-----------------------------------------------------+
                        |
                        | Inner SCHC Compression
                        |
                        v
+-------------------------------------------------+
|                                                 |
| Compressed Plaintext                            |
|                                                 |
| 0x0200 (2 bytes)                                |
|                                                 |
|                                                 |
| RuleID = 0x02 (1 byte)                          |
|                                                 |
|                                                 |
| Compression residue                             |
| and padded payload = 0x00 (1 byte)              |
|                                                 |
| 0b00 (2 bits match-mapping Compression Residue) |
| 0b000000 (6 bit padding)                        |
|                                                 |
+-------------------------------------------------+
                        |
                        | AEAD Encryption
                        |  (piv = 0x04)
                        |
                        v
+------------------------------------------------+
|                                                |
| encrypted_plaintext = 0xa2cf (2 bytes)         |
| tag = 0xc54fe1b434297b62 (8 bytes)             |
|                                                |
| ciphertext = 0xa2cfc54fe1b434297b62 (10 bytes) |
|                                                |
+------------------------------------------------+

~~~~~~~~~~~
{: #fig-plaintext-req title="Plaintext Compression and Encryption for the GET Request" artwork-align="center"}

When the Application Server applies the Rule in {{fig-rules-oscore-device-server}} shared with the Device to the CoAP response in {{fig-example-resp}}, this results in the Compressed Plaintext shown in {{fig-plaintext-resp}}.

As per {{ssec-examples-oscore}}, the message follows the process of SCHC Inner Compression and encryption until the payload (if any). The ciphertext resulting from the overall Inner process is used as payload of the Outer OSCORE message.

~~~~~~~~~~~ aasvg

+-------------------------------------------------+
|                                                 |
| OSCORE Plaintext                                |
|                                                 |
| 0x45ff32332043  (6 bytes)                       |
|                                                 |
| 0x45 Successful Response Code 69 "2.05 Content" |
|                                                 |
|     0xff Payload marker                         |
|                                                 |
|         0x32332043 Payload                      |
|                                                 |
+-------------------------------------------------+
                    |
                    | Inner SCHC Compression
                    |
                    v
+-------------------------------------------------+
|                                                 |
| Compressed Plaintext                            |
|                                                 |
| 0x028c8cc810c0 (6 bytes)                        |
|                                                 |
|                                                 |
| RuleID = 0x02                                   |
|                                                 |
|                                                 |
| Compression residue                             |
| and padded payload = 0x8c8cc810c0 (5 bytes)     |
|                                                 |
| 0b10 (2 bits match-mapping Compression Residue) |
|       0x32332043 >> 2 (shifted payload)         |
|                        0b000000 Padding         |
|                                                 |
+-------------------------------------------------+
                    |
                    | AEAD Encryption
                    |  (piv = 0x04)
                    |
                    v
+---------------------------------------------------------+
|                                                         |
|  encrypted_plaintext = 0x10c6d7c26cc1 (6 bytes)         |
|  tag = 0xe9aef3f2461e0c29 (8 bytes)                     |
|                                                         |
|  ciphertext = 0x10c6d7c26cc1e9aef3f2461e0c29 (14 bytes) |
|                                                         |
+---------------------------------------------------------+

~~~~~~~~~~~
{: #fig-plaintext-resp title="Plaintext Compression and Encryption for the Content Response" artwork-align="center"}

After having performed the SCHC Inner Compression of the CoAP request in {{fig-example-req}}, the Device protects it with OSCORE by considering the Compressed Plaintext in {{fig-plaintext-req}}. The result is the OSCORE-protected CoAP request shown in {{fig-example-oscore-req}}.

~~~~~~~~~~~

Protected message:
==================
0x41020001823b6578616d706c652e636f6d
  6409040005d411636f6170ffa2cfc54fe1b434297b62
(39 bytes)

Header:
0x4102
01   Ver
  00   CON
    0001   TKL
        00000010   Request Code 2 "POST"

0x0001 = mid
0x82 = token

Options:

0x3b6578616d706c652e636f6d
Option 3: Uri-Host
Value = example.com

0x6409040005
Option 9: OSCORE
Value = 0x09040005
          09 = 000 0 1 001 flag byte
                   h k  n
            04 piv
              0005 kid

0xd411636f6170
Option 39: Proxy-Scheme
Value = coap


0xFF Payload marker

Payload:
0xa2cfc54fe1b434297b62 (10 bytes)

~~~~~~~~~~~
{: #fig-example-oscore-req title="Protected and Inner SCHC Compressed CoAP GET Request" artwork-align="left"}


Then, the Device applies the Rule in {{fig-rules-oscore-device-proxy}} shared with the proxy to the OSCORE-protected CoAP request in {{fig-example-oscore-req}}, thus performing the SCHC Outer Compression of such request. The result is the OSCORE-protected and Outer Compressed CoAP request shown in {{fig-example-oscore-req-to-proxy}}, which the Device sends to the proxy.

~~~~~~~~~~~

Compressed message:
=================
0x03156caf0c2dae0d8ca5cc6deda888b459f8a9fc3686852f6c40 (26 bytes)
0x03 RuleID
    156caf0c2dae0d8ca5cc6deda888b459f8a9fc3686852f6c40 compression
                                                       residue and
                                                       padded payload


Compression Residue
0b 0001 010      1011  |  0x6578616d706c652e636f6d  |
    mid tkn  Uri-Host (residue size and residue)

0b 0100 0100
         piv (residue size and residue)

   0100 0101
         kid (residue size and residue)

   (115 bits -> 15 bytes with padding)

Payload
0xa2cfc54fe1b434297b62 (10 bytes)

Compressed message length: 26 bytes

~~~~~~~~~~~
{: #fig-example-oscore-req-to-proxy title="SCHC-OSCORE CoAP GET Request Compressed for the Proxy" artwork-align="left"}

Upon receiving the message in {{fig-example-oscore-req-to-proxy}}, the proxy decompresses it with the Rule in {{fig-rules-oscore-device-proxy}} shared with the Device, thus performing the SCHC Outer Decompression. The result is the same OSCORE-protected CoAP request in {{fig-example-oscore-req}}.

After that, the proxy removes the Proxy-Scheme Option from the decompressed OSCORE-protected CoAP request. Also, the proxy replaces the values of the CoAP Message ID and of the CoAP Token to 0x0004 and 0x75, respectively. The result is the OSCORE-protected CoAP request shown in {{fig-example-oscore-req-from-proxy}}.

~~~~~~~~~~~

Protected message:
==================
0x41020004753b6578616d706c652e636f6d
  6409040005ffa2cfc54fe1b434297b62
(33 bytes)

Header:
0x4102
01   Ver
  00   CON
    0001   TKL
        00000010   Request Code 2 "POST"

0x0004 = mid
0x75 = token

Options:

0x3b6578616d706c652e636f6d
Option 3: Uri-Host
Value = example.com

0x6409040005
Option 9: OSCORE
Value = 0x09040005
          09 = 000 0 1 001 flag byte
                   h k  n
            04 piv
              0005 kid


0xFF Payload marker

Payload:
0xa2cfc54fe1b434297b62 (10 bytes)

~~~~~~~~~~~
{: #fig-example-oscore-req-from-proxy title="SCHC-OSCORE CoAP GET Request to be Compressed by the Proxy" artwork-align="left"}

Then, the proxy applies the Rule in {{fig-rules-oscore-proxy-server}} shared with the Application Server to the OSCORE-protected CoAP request in {{fig-example-oscore-req-from-proxy}}, thus performing the SCHC Outer Compression of such request. The result is the OSCORE-protected and Outer Compressed CoAP request shown in {{fig-example-oscore-req-from-proxy-compressed}}, which the proxy forwards to the Application Server.

~~~~~~~~~~~

Compressed message:
=================
0x044b6caf0c2dae0d8ca5cc6deda888b459f8a9fc3686852f6c40 (26 bytes)
0x04 RuleID
    4b6caf0c2dae0d8ca5cc6deda888b459f8a9fc3686852f6c40 compression
                                                       residue and
                                                       padded payload


Compression Residue
0b 0100 101      1011  |  0x6578616d706c652e636f6d
    mid tkn  Uri-Host (residue size and residue)

0b 0100 0100
         piv (residue size and residue)

0b 0100 0101
         kid (residue size and residue)

   (115 bits -> 15 bytes with padding)


Payload
0xa2cfc54fe1b434297b62 (10 bytes)

Compressed message length: 26 bytes

~~~~~~~~~~~
{: #fig-example-oscore-req-from-proxy-compressed title="SCHC-OSCORE CoAP GET Request Forwarded by the Proxy" artwork-align="left"}

Upon receiving the message in {{fig-example-oscore-req-from-proxy-compressed}}, the Application Server decompresses it using the Rule in {{fig-rules-oscore-proxy-server}} shared with the proxy, thus performing the SCHC Outer Decompression. The result is the same OSCORE-protected CoAP request in {{fig-example-oscore-req-from-proxy}}.

The Application Server decrypts and verifies such a request, which results in the same Compressed Plaintext in {{fig-plaintext-req}}. Then, the Application Server applies the Rule in {{fig-rules-oscore-device-server}} shared with the Device to such a Compressed Plaintext, thus performing the SCHC Inner Decompression. The result is used to rebuild the same CoAP request in {{fig-example-req}}, which the Application Server delivers to the application.

After having performed the SCHC Inner Compression of the CoAP response in {{fig-example-resp}}, the Application Server protects it with OSCORE by considering the Compressed Plaintext in {{fig-plaintext-resp}}. The result is the OSCORE-protected CoAP response shown in {{fig-example-oscore-resp}}.

~~~~~~~~~~~

Protected message:
==================
0x614400047590ff10c6d7c26cc1e9aef3f2461e0c29
(21 bytes)

Header:
0x6144
01   Ver
  10   ACK
    0001   TKL
        01000100   Successful Response Code 68 "2.04 Changed"

0x0004 = mid
0x75 = token

Options:

0x90
Option 9: OSCORE
Value = b''


0xFF Payload marker

Payload:
0x10c6d7c26cc1e9aef3f2461e0c29 (14 bytes)

~~~~~~~~~~~
{: #fig-example-oscore-resp title="Protected and Inner SCHC Compressed CoAP Content Response" artwork-align="left"}


Then, the Application Server applies the Rule in {{fig-rules-oscore-proxy-server}} shared with the proxy to the OSCORE-protected CoAP response in {{fig-example-oscore-resp}}, thus performing the SCHC Outer Compression of such response. The result is the OSCORE-protected and Outer Compressed CoAP response shown in {{fig-example-oscore-resp-to-proxy}}, which the Application Server sends to the proxy.

~~~~~~~~~~~

Compressed message:
=================
0x04a510c6d7c26cc1e9aef3f2461e0c29  (16 bytes)
0x04 RuleID
    a510c6d7c26cc1e9aef3f2461e0c29 compression residue
                                   and padded payload


Compression Residue (8 bits -> 1 byte with padding)
0b    1 0100 101
   type  mid tkn

Payload
0x10c6d7c26cc1e9aef3f2461e0c29 (14 bytes)

Compressed message length: 16 bytes

~~~~~~~~~~~
{: #fig-example-oscore-resp-to-proxy title="SCHC-OSCORE CoAP Content Response Compressed for the Proxy" artwork-align="left"}

Upon receiving the message in {{fig-example-oscore-resp-to-proxy}}, the proxy decompresses it with the Rule in {{fig-rules-oscore-proxy-server}} shared with the Application Server, thus performing the SCHC Outer Decompression. The result is the same OSCORE-protected CoAP response in {{fig-example-oscore-resp}}.

After that, the proxy replaces the values of the CoAP Message ID and of the CoAP Token to 0x0001 and 0x82, respectively. The result is the OSCORE-protected CoAP response shown in {{fig-example-oscore-resp-from-proxy}}.

~~~~~~~~~~~

Protected message:
==================
0x614400018290ff10c6d7c26cc1e9aef3f2461e0c29
(21 bytes)

Header:
0x6144
01   Ver
  10   ACK
    0001   TKL
        01000100   Successful Response Code 68 "2.04 Changed"

0x0001 = mid
0x82 = token

Options:

0x90
Option 9: OSCORE
Value = b''


0xFF Payload marker

Payload:
0x10c6d7c26cc1e9aef3f2461e0c29 (14 bytes)

~~~~~~~~~~~
{: #fig-example-oscore-resp-from-proxy title="SCHC-OSCORE CoAP Content Response to be Compressed by the Proxy" artwork-align="left"}

Then, the proxy applies the Rule in {{fig-rules-oscore-device-proxy}} shared with the Device to the OSCORE-protected CoAP response in {{fig-example-oscore-resp-from-proxy}}, thus performing the SCHC Outer Compression of such response. The result is the OSCORE-protected and Outer Compressed CoAP response shown in {{fig-example-oscore-resp-from-proxy-compressed}}, which the proxy forwards to the Device.

~~~~~~~~~~~

Compressed message:
=================
0x038a10c6d7c26cc1e9aef3f2461e0c29 (16 bytes)
0x03 RuleID
    8a10c6d7c26cc1e9aef3f2461e0c29 compression residue
                                   and padded payload


Compression Residue (8 bits -> 1 byte with padding)
0b    1 0001 010
   type  mid tkn

Payload
0x10c6d7c26cc1e9aef3f2461e0c29 (14 bytes)

Compressed message length: 16 bytes

~~~~~~~~~~~
{: #fig-example-oscore-resp-from-proxy-compressed title="SCHC-OSCORE CoAP Content Response Forwarded by the Proxy" artwork-align="left"}

Upon receiving the message in {{fig-example-oscore-resp-from-proxy-compressed}}, the Device decompresses it using the Rule in {{fig-rules-oscore-device-proxy}} shared with the proxy, thus performing the SCHC Outer Decompression. The result is the same OSCORE-protected CoAP response in {{fig-example-oscore-resp-from-proxy}}.

The Device decrypts and verifies such a response, which results in the same Compressed Plaintext in {{fig-plaintext-resp}}. Then, the Device applies the Rule in {{fig-rules-oscore-device-server}} shared with the Application Server to such a Compressed Plaintext, thus performing the SCHC Inner Decompression. The result is used to rebuild the same CoAP response in {{fig-example-resp}}, which the Device delivers to the application.

# CoAP Fields # {#sec-coap-fields}

{{table-coap-fields}} lists the CoAP fields and subfields for which SCHC Compression has been defined or revised in this document.

| Field                    | Description                                                                        |
|--------------------------|------------------------------------------------------------------------------------|
| CoAP Version             | CoAP header field Version {{RFC7252}}                                              |
| CoAP Type                | CoAP header field Type {{RFC7252}}                                                 |
| CoAP Token Length (TKL)  | CoAP header field Token Length (TKL) {{RFC7252}}{{RFC8974}}                        |
| CoAP Code                | CoAP header field Code {{RFC7252}}                                                 |
| CoAP Code Class          | CoAP header field Code (subfield Class) {{RFC7252}}                                |
| CoAP Code Detail         | CoAP header field Code (subfield Detail) {{RFC7252}}                               |
| CoAP MID                 | CoAP header field Message ID {{RFC7252}}                                           |
| CoAP Token               | CoAP field Token {{RFC7252}}{{RFC8974}}                                            |
| CoAP If-Match            | CoAP option If-Match {{RFC7252}}                                                   |
| CoAP Uri-Host            | CoAP option Uri-Host {{RFC7252}}                                                   |
| CoAP ETag                | CoAP option ETag {{RFC7252}}                                                       |
| CoAP If-None-Match       | CoAP option If-None-Match {{RFC7252}}                                              |
| CoAP Observe             | CoAP option Observe {{RFC7641}}                                                    |
| CoAP Uri-Port            | CoAP option Uri-Port {{RFC7252}}                                                   |
| CoAP Location-Path       | CoAP option Location-Path {{RFC7252}}                                              |
| CoAP OSCORE              | CoAP option OSCORE {{RFC8613}}{{I-D.ietf-core-oscore-key-update}}                  |
| CoAP OSCORE Flags        | CoAP option OSCORE (subfield Flags) {{RFC8613}}{{I-D.ietf-core-oscore-key-update}} |
| CoAP OSCORE PIV          | CoAP option OSCORE (subfield PIV) {{RFC8613}}                                      |
| CoAP OSCORE kid          | CoAP option OSCORE (subfield kid) {{RFC8613}}                                      |
| CoAP OSCORE kidctx       | CoAP option OSCORE (subfield kid context) {{RFC8613}}                              |
| CoAP OSCORE x            | CoAP option OSCORE (subfield x) {{I-D.ietf-core-oscore-key-update}}                |
| CoAP OSCORE nonce        | CoAP option OSCORE (subfield nonce) {{I-D.ietf-core-oscore-key-update}}            |
| CoAP OSCORE y            | CoAP option OSCORE (subfield y) {{I-D.ietf-core-oscore-key-update}}                |
| CoAP OSCORE old_nonce    | CoAP option OSCORE (subfield old_nonce) {{I-D.ietf-core-oscore-key-update}}        |
| CoAP Uri-Path            | CoAP option Uri-Path {{RFC7252}}                                                   |
| CoAP Content-Format      | CoAP option Content-Format {{RFC7252}}                                             |
| CoAP Max-Age             | CoAP option Max-Age {{RFC7252}}                                                    |
| CoAP Uri-Query           | CoAP option Uri-Query {{RFC7252}}                                                  |
| CoAP Accept              | CoAP option Accept {{RFC7252}}                                                     |
| CoAP Location-Query      | CoAP option Location-Query {{RFC7252}}                                             |
| CoAP Block2              | CoAP option Block2 {{RFC7959}}{{RFC8323}}                                          |
| CoAP Block1              | CoAP option Block1 {{RFC7959}}{{RFC8323}}                                          |
| CoAP Size2               | CoAP option Size2 {{RFC7959}}                                                      |
| CoAP Proxy-Uri           | CoAP option Proxy-Uri {{RFC7252}}                                                  |
| CoAP Proxy-Scheme        | CoAP option Proxy-Scheme {{RFC7252}}                                               |
| CoAP Size1               | CoAP option Size1 {{RFC7252}}                                                      |
| CoAP Proxy-Cri           | CoAP option Proxy-Cri {{I-D.ietf-core-href}}                                       |
| CoAP Proxy-Scheme-Number | CoAP option Proxy-Scheme-Number {{I-D.ietf-core-href}}                             |
| CoAP No-Response         | CoAP option No-Response {{RFC7967}}                                                |
| CoAP Hop-Limit           | CoAP option Hop-Limit {{RFC8768}}                                                  |
| CoAP Echo                | CoAP option Echo {{RFC9175}}                                                       |
| CoAP Request-Tag         | CoAP option Request-Tag {{RFC9175}}                                                |
| CoAP Q-Block1            | CoAP option Q-Block1 {{RFC9177}}                                                   |
| CoAP Q-Block2            | CoAP option Q-Block2 {{RFC9177}}                                                   |
| CoAP EDHOC               | CoAP option EDHOC {{RFC9668}}                                   |
{: #table-coap-fields title="CoAP Fields" align="center"}

# Security Considerations

The use of SCHC header compression for CoAP header fields only affects the representation of the header information. SCHC header compression itself does not increase or decrease the overall level of security of the communication. When the connection does not use a security protocol (OSCORE, DTLS, etc.), it is necessary to use a Layer 2 security mechanism to protect the SCHC messages.

If an LPWAN is the Layer 2 technology being used, the SCHC security considerations discussed in {{RFC8724}} continue to apply. When using another Layer 2 protocol, the use of a cryptographic integrity-protection mechanism to protect the SCHC headers is REQUIRED. Such cryptographic integrity protection is necessary in order to continue to provide the properties that {{RFC8724}} relies upon.

When SCHC is used with OSCORE, the security considerations discussed in {{RFC8613}} continue to apply. When SCHC is used with Group OSCORE, the security considerations discussed in {{I-D.ietf-core-oscore-groupcomm}} apply. When SCHC is used in the presence of CoAP proxies, the security considerations discussed in {{Section 11.2 of RFC7252}} continue to apply.

When SCHC is used with the OSCORE Outer headers, the Initialization Vector (IV) size in the Compression Residue must be carefully selected. There is a trade-off between compression efficiency (with a longer MSB MO prefix) and the frequency at which the Device must renew its key material (in order to prevent the IV from expanding to an incompressible value). The key-renewal operation itself may require several message exchanges and result in energy-intensive computation, but the optimal trade-off will depend on the specifics of the Device and expected usage patterns. In order to renew its key material with another OSCORE endpoint, the Device can rely on the lightweight key update protocol KUDOS defined in {{I-D.ietf-core-oscore-key-update}}.

If an attacker can introduce a corrupted SCHC-compressed packet onto a link, DoS attacks can be mounted by causing excessive resource consumption at the decompressor. However, an attacker able to inject packets at the link layer is also capable of other, potentially more damaging, attacks.

SCHC compression emits variable-length Compression Residues for some CoAP fields. In the representation of the compressed header, the length field that is sent is not the length of the original header field but rather the length of the Compression Residue that is being transmitted. If a corrupted packet arrives at the decompressor with a longer or shorter length than that of the original compressed representation, the SCHC decompression procedures will detect an error and drop the packet.

SCHC header compression Rules MUST remain tightly coupled between the compressor and the decompressor. If the compression Rules get out of sync, a Compression Residue might be decompressed differently at the receiver, thus yielding a result different than the initial message submitted to compression procedures. Accordingly, any time the context Rules are updated on an OSCORE endpoint, that endpoint MUST trigger OSCORE key re-establishment, e.g., by running the lightweight key update protocol KUDOS {{I-D.ietf-core-oscore-key-update}}. Similar procedures may be appropriate to signal Rule updates when other message-protection mechanisms are in use.

## YANG Module {#sec-security-considerations-yang-module}

The YANG data model defined in {{sec-yang-module}} extends the ietf-schc module defined in {{RFC9363}}.

Therefore, all the security considerations compiled in {{Section 8 of RFC9363}} apply to the resulting, extended YANG data model as well.

# IANA Considerations

This document has the following actions for IANA.

Note to RFC Editor: Please replace all occurrences of "{{&SELF}}" with the RFC number of this specification and delete this paragraph.

## IETF XML

IANA is asked to register the following entry in the "IETF XML" registry {{RFC3688}}.

* URI: urn:ietf:params:xml:ns:yang:ietf-schc-coap
* Registrant Contact: The IESG.
* XML: N/A; the requested URI is an XML namespace.

## YANG Module Names

IANA is asked to register the following entry in the "YANG Module Names" registry {{RFC6020}}{{RFC8407}} within the "YANG Parameters" registry group.

* Name: ietf-schc-coap
* Namespace: urn:ietf:params:xml:ns:yang:ietf-schc-coap
* Prefix: schc-coap
* Reference: {{&SELF}}

## SCHC Compression of CoAP Fields # {#sec-iana-coap-fields}

IANA is asked to establish the "SCHC Compression of CoAP Fields" IANA registry.

As registration policy, the registry uses "Specification Required" per {{Section 4.6 of RFC8126}}. Expert Review guidelines are provided in {{sec-iana-expert-review}}.

### Intended Use

The objective of this registry is to collect a list of CoAP fields and subfields, for which it has been defined how to perform SCHC compression.

Such a definition does not necessarily have to be in the same documentation that defines the CoAP fields and subfields in question. While that can be the case, it is also possible to provide that definition in a separate specification.

Each entry of the registry is intended to include references to the documentation that defines the associated CoAP field or subfield, as well as references to the specifications that define the SCHC compression of that CoAP field or subfield.

When a specification defines how to perform SCHC compression of a CoAP field, the following applies.

* If a registry entry for the CoAP field does not already exist, it is strongly encouraged to register an associated new entry.

* If a registry entry for the CoAP field already exists, it is strongly encouraged to update its list of references. The update is intended to add references to the specification that provides the new or updated SCHC compression of the CoAP field, as well as to any documentation that updates the definition of the CoAP field itself.

If the defined SCHC compression considers the CoAP field as composed of subfields, it is strongly encouraged that the same as above is also performed for each subfield and the associated registry entry.

### Structure of Entries

The columns of this registry are:

* Field: a unique identifier of the CoAP field or subfield associated with this entry. This identifier can be used as value of the "Field" column in a SCHC compression Rule. This identifier must have a corresponding item or set of items in the YANG data model for the CoAP field or subfield associated with this entry, as specified in {{Section 6 of RFC9363}} or in {{sec-yang-module}} of {{&SELF}}.

* Description: a short description of the CoAP field or subfield associated with this entry, together with public references to the resources that define it.

* Reference: public references to the resources that define how a SCHC compression Rule works for the CoAP field or subfield associated with this entry.

This registry has been initially populated with the values in {{table-coap-fields}}. The "Reference" column for all of these entries refers to this document.

## Expert Review Instructions {#sec-iana-expert-review}

The IANA registry established in this document is defined as "Specification Required". This section gives some general guidelines for what the experts should be looking for, but they are being designated as experts for a reason so they should be given substantial latitude.

Expert reviewers should take into consideration the following points:

* Point squatting should be discouraged. Reviewers are encouraged to get sufficient information for registration requests to ensure that the usage is not going to duplicate one that is already registered and that the point is likely to be used in deployments.

  Specifically, for every CoAP field, only one corresponding registry entry is allowed. Also, for every CoAP subfield, only one corresponding registry entry is allowed.

* Consistent with the "Specification Required" registration policy, specifications should exist, but early assignment before a specification is available is considered to be permissible. When specifications are not provided, the description provided needs to have sufficient information to identify what the point is being used for.

If the expert becomes aware of a definition for SCHC compression of CoAP fields and subfields that is deployed and in use, the expert may also initiate a registration or update an existing one on their own, if they deem important that the definition in question gains visibility through the registry entry.

--- back

# YANG Data Model # {#sec-yang-module}

This appendix defines the ietf-schc-coap module, which extends the ietf-schc module defined in {{RFC9363}} to include the new CoAP options as defined in the present document.

~~~~~~~~~~~

<CODE BEGINS> file "ietf-schc-coap@2024-10-21.yang"

module ietf-schc-coap {
  yang-version 1.1;
  namespace "urn:ietf:params:xml:ns:yang:ietf-schc-coap";
  prefix schc-coap;

  import ietf-schc {
      prefix schc;
  }

  organization
    "IETF Static Context Header Compression (schc) Working Group";
  contact
    "WG Web:   <https://datatracker.ietf.org/wg/schc/about/>
     WG List:  <mailto:schc@ietf.org>
     Editor:   Marco Tiloca
       <mailto:marco.tiloca@ri.se>";
  description
    "Copyright (c) 2021 IETF Trust and the persons identified as
     authors of the code.  All rights reserved.
     Redistribution and use in source and binary forms, with or
     without modification, is permitted pursuant to, and subject to
     the license terms contained in, the Simplified BSD License set
     forth in Section 4.c of the IETF Trust's Legal Provisions
     Relating to IETF Documents
     (https://trustee.ietf.org/license-info).
     This version of this YANG module is part of RFC XXXX
     (https://www.rfc-editor.org/info/rfcXXXX); see the RFC itself
     for full legal notices.
     The key words 'MUST', 'MUST NOT', 'REQUIRED', 'SHALL', 'SHALL
     NOT', 'SHOULD', 'SHOULD NOT', 'RECOMMENDED', 'NOT RECOMMENDED',
     'MAY', and 'OPTIONAL' in this document are to be interpreted as
     described in BCP 14 (RFC 2119) (RFC 8174) when, and only when,
     they appear in all capitals, as shown here.
     ****************************************************************

     This module extends the ietf-schc module defined in RFC 9363 to
     include the new CoAP options as defined in RFC YYYY.";

  revision 2025-03-03 {
    description
      "New CoAP extensions and extended OSCORE fields.";
    reference
      "RFC YYYY Static Context Header Compression (SCHC) for the
                Constrained Application Protocol (CoAP) (see
                Sections 5 and 6)";
  }

  // Field ID

  identity fid-coap-option-proxy-cri {
    base "schc:fid-coap-option";
    description
      "Proxy-Cri option.";
    reference
      "RFC XXXX Constrained Resource Identifiers";
  }

  identity fid-coap-option-proxy-scheme-number {
    base "schc:fid-coap-option";
    description
      "Proxy-Scheme-Number option.";
    reference
      "RFC XXXX Constrained Resource Identifiers";
  }

  identity fid-coap-option-hop-limit {
    base "schc:fid-coap-option";
    description
      "Hop Limit option to avoid infinite forwarding loops.";
    reference
      "RFC 8768 Constrained Application Protocol (CoAP)
                Hop-Limit Option";
  }

  identity fid-coap-option-echo {
    base "schc:fid-coap-option";
    description
      "Echo option.";
    reference
      "RFC 9175 Constrained Application Protocol (CoAP):
                Echo, Request-Tag, and Token Processing";
  }

  identity fid-coap-option-request-tag {
    base "schc:fid-coap-option";
    description
      "Request-Tag option.";
    reference
      "RFC 9175 Constrained Application Protocol (CoAP):
                Echo, Request-Tag, and Token Processing";
  }

  identity fid-coap-option-q-block1 {
    base "schc:fid-coap-option";
    description
      "Q-Block1 option.";
    reference
      "RFC 9177 Constrained Application Protocol (CoAP)
                Block-Wise Transfer Options Supporting
                Robust Transmission";
  }

  identity fid-coap-option-q-block2 {
    base "schc:fid-coap-option";
    description
      "Q-Block2 option.";
    reference
      "RFC 9177 Constrained Application Protocol (CoAP)
                Block-Wise Transfer Options Supporting
                Robust Transmission";
  }

  identity fid-coap-option-edhoc {
    base "schc:fid-coap-option";
    description
      "EDHOC option.";
    reference
      "RFC 9668 Using Ephemeral Diffie-Hellman Over COSE (EDHOC)
                with the Constrained Application Protocol (CoAP)
                and Object Security for Constrained RESTful
                Environments (OSCORE)";
  }

  identity fid-coap-option-oscore-x {
       base "schc:fid-coap-option";
       description
         "CoAP option OSCORE x field.";
       reference
         "RFC YYYY Static Context Header Compression (SCHC) for the
                   Constrained Application Protocol (CoAP) (see
                   Section 6.4)
          RFC XXXX Key Update for OSCORE (KUDOS)";
  }

  identity fid-coap-option-oscore-nonce {
       base "schc:fid-coap-option";
       description
         "CoAP option OSCORE nonce field.";
       reference
         "RFC YYYY Static Context Header Compression (SCHC) for the
                   Constrained Application Protocol (CoAP) (see
                   Section 6.4)
          RFC XXXX Key Update for OSCORE (KUDOS)";
  }

  identity fid-coap-option-oscore-y {
       base "schc:fid-coap-option";
       description
         "CoAP option OSCORE y field.";
       reference
         "RFC YYYY Static Context Header Compression (SCHC) for the
                   Constrained Application Protocol (CoAP) (see
                   Section 6.4)
          RFC XXXX Key Update for OSCORE (KUDOS)";
  }

  identity fid-coap-option-oscore-oldnonce {
       base "schc:fid-coap-option";
       description
         "CoAP option OSCORE old_nonce field.";
       reference
         "RFC YYYY Static Context Header Compression (SCHC) for the
                   Constrained Application Protocol (CoAP) (see
                   Section 6.4)
          RFC XXXX Key Update for OSCORE (KUDOS)";
  }

  // Function Length

  identity fl-oscore-oscore-nonce-length {
       base "schc:fl-base-type";
       description
         "Size in bytes of the OSCORE nonce corresponding to m+1.";
       reference
         "RFC YYYY Static Context Header Compression (SCHC) for the
                   Constrained Application Protocol (CoAP) (see
                   Section 6.4)
          RFC XXXX Key Update for OSCORE (KUDOS)";
  }

  identity fl-oscore-oscore-oldnonce-length {
       base "schc:fl-base-type";
       description
         "Size in bytes of the OSCORE old_nonce corresponding to w+1.
         ";
       reference
         "RFC YYYY Static Context Header Compression (SCHC) for the
                   Constrained Application Protocol (CoAP) (see
                   Section 6.4)
          RFC XXXX Key Update for OSCORE (KUDOS)";
  }
}

<CODE ENDS>

~~~~~~~~~~~
{: #fig-yang-data-model title="SCHC CoAP Extension YANG Data Model" artwork-align="left"}

# Document Updates # {#sec-document-updates}
{:removeinrfc}

## Version -03 to -04 ## {#sec-03-04}

* Use "bit" instead of "b" as symbol for bit (per ISO/IEC 80000-13).

* Updated references.

* Fixes and editorial improvements.

## Version -02 to -03 ## {#sec-02-03}

* Consistent representation of "CoAP Version" 1 in example Rules.

* Split the compression of Token Length and Token into two sections.

* Disambiguated example of Rule on eliding a Uri-Path option.

* Fixed compression examples with OSCORE.

* Inherited security considerations on the YANG module from RFC 9363.

* Fixes and editorial improvements.

## Version -01 to -02 ## {#sec-01-02}

* Added compression for the CoAP options Proxy-Cri and Proxy-Scheme-Number.

* Defined new IANA registry "SCHC Compression of CoAP Fields".

* Updated the YANG data model.

* Fixes and editorial improvements.

## Version -00 to -01 ## {#sec-00-01}

* Fixed an example, as per the erratum with Errata ID 7623.

* Clarified building of Field Descriptor for CoAP options.

* Clarified what SCHC compression considers for CoAP options.

* Revised SCHC Compression of the ETag and If-Match CoAP option.

* Revised SCHC Compression of the If-None-Match CoAP option.

* Added YANG data model for the YANG module.

* Added IANA considerations.

* Fixes and editorial improvements.

# Acknowledgments # {#acknowledgments}
{:numbered="false"}

The authors sincerely thank {{{Christian Amsüss}}}, {{{Quentin Lampin}}}, {{{John Preuß Mattsson}}}, {{{Carles Gomez Montenegro}}}, {{{Göran Selander}}}, {{{Pascal Thubert}}}, and {{{Éric Vyncke}}} for their comments and feedback.

This work was supported by the Sweden's Innovation Agency VINNOVA within the EUREKA CELTIC-NEXT project CYPRESS; and by the H2020 projects SIFIS-Home (Grant agreement 952652) and ARCADIAN-IoT (Grant agreement 101020259).
