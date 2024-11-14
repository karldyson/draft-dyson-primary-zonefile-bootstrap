---
title: "Zonefile Bootstrap on DNS Primary Server"
category: std
updates: 9432

docname: draft-dyson-primary-zonefile-bootstrap-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
stand_alone: true
number:
date:
consensus: true
v: 3
# area: Operations and Management
# workgroup: Internet Engineering Task Force
keyword:
 - catalog zone
 - zonefile bootstrap
 - primary server
#venue:
#  github: karldyson/draft-dyson-primary-zonefile-bootstrap

author:
 -
    fullname: Karl Dyson
    organization: Nominet UK
    street:
        - Minerva House
        - Edmund Halley Road
        - Oxford Science Park
    city: Oxford
    code: OX4 4DQ
    country: UK
    email: karl.dyson@nominet.uk
    uri: https://nominet.uk

normative:
  RFC9432:
  RFC2136:
  RFC2119:

informative:

--- abstract

This document specifies an update to {{RFC9432}} such that the primary DNS server for a zone can bootstrap the underlying zonefile using information contained within a catalog zone.


--- middle

# Introduction

Once a DNS zone file exists, there are standards compliant ways to distribute that zone to secondary servers in {{RFC9432}} DNS Catalog Zones, as well as standards compliant ways to alter the contents of the zone in {{RFC2136}} Dynamic Updates.

However there is no standards compliant method of bootstrapping the presence of a new empty zone file ready for those two processes to work with.

There are vendor specific methods, such as RNDC for ISC BIND, which can be used remotely, but which still requires you to have created the underlying zone file on the primary server filesystem.

PowerDNS has a proprietary API that can be used, and other products likewise have mechanisms.

However, there's no standards compliant vendor independent mechanism of signalling to the primary server that a new zone file is to be created.

Operators of large scale DNS systems may want to be able to signal the creation of a new zone without wanting to be tied to a particular vendor's proprietary software, and without the need or overhead of engineering a bespoke solution with the ongoing need to support and maintain that.

It is anticipated that the reason for desiring the ability to dynamically provision a zone on the primary server is because the operator will then manage resource records in the zone via Dynamic Updates {{RFC2136}}, and will want to distribute the zones to their secondary servers via DNS Catalog Zones {{RFC9432}}.

The scope of this document is therefore confined to the initial bootstrap provisioning of the zone file, and MAY include signalling of initial DNSSEC policy or configuration (see {{dnssecConsideration}}).

Broader provisioning of the base nameserver configuration is beyond the scope of this discussion and document.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

This document doesn't alter the conventions and definitions as defined in {{RFC9432}} DNS Catalog Zones.

# Catalog Properties

## Zonefile Bootstrap (boot property)

When suitable configuration is activated in the implementation, and a new member zone entry is added to the catalog being served by the primary, the primary should create the underlying zonefile with the properties and parameters outlined in the boot property.

The boot property is the parent label to the other labels that facilitate the adding of the various properties and parameters.

The implementation MAY permit the following on a global, or per catalog basis, by way of suitable configuration parameters:

  * The zone file is ONLY created if the zonefile does not already exist
  * The zone file is NEVER created (effectively, the bootstrap capability is disabled for this catalog or primary server)
  * The zone file is ALWAYS created, overwriting any existing zone file

## Start Of Authority (soa property)

The soa property is used to specify the SOA that will be applied to the created zonefile for the member zone.

Clearly an actual SOA record type cannot be used here, and so the parameters to be applied shall be constructed in a TXT record type as follows:

~~~~
soa.boot.$CATZ 0 IN TXT ( "mname=<mname>; rname=<rname>; "
      "refresh=<refresh>; retry=<retry>; expire=<expire>; "
      "minimum=<minimum>" )
~~~~

There MUST NOT be more than a single soa property record with the exception that a member zone record can be specified to override the default (see {{memberZoneSection}} below).

Multiple soa property records constitues a broken catalog zone, which MUST NOT be processed (see {{RFC9432}} section 5.1).


## Nameservers (ns property)

Actual NS records cannot be used, as we do not want to actually delegate outside of this catalog zone.

The nameservers will be specified in a TXT record as follows, along with the associated IP addresses, if appropriate or required.

Specifying the nameserver IP addresses is OPTIONAL, with the exception that if the zone that the nameservers reside in is to be created within the catalog, then they MUST be specified in order that the relevant records can be created in the zone at zone bootstrap time.

If the nameservers are in-bailiwick and address records are therefore required, suitable address records MUST be created in the member zone file from the entries specified.

If the nameservers are in-bailiwick of a zone in the catalog, and an address is not specified, this denotes a broken catalog zone, which MUST NOT be processed.

The ns property can be specified multiple times, with one nameserver specified per entry.

~~~~
ns.boot.$CATZ 0 IN TXT ( "ns=some.name.server.; "
      "ipv4=192.0.2.1; ipv6=2001:db8::1" )
~~~~


# Member Zone Properties {#memberZoneSection}

The default properties outlined above can be overridden on a per member zone basis. Where per member zone entries are specified in the catalog, they MUST be used in preference to the default properties specified at the catalog level.

A subset MAY be specified here; for example, the SOA could be omitted here and just the NS records or DNSSEC parameters specified, with the omitted properties taken from the catalog level values.

~~~~
<unique-N>.zones.$CATZ 0 IN PTR example.com.
soa.boot.<unique-N>.zones.$CATZ 0 IN TXT ( "mname=<mname>; "
      "rname=<rname>; refresh=<refresh>; retry=<retry>; "
      "expire:<expire>; minimum:<minimum>" )
ns.boot.<unique-N>.zones.$CATZ 0 IN TXT ( "ns=some.name.server.; "
      "ipv4=192.0.2.1; ipv6=2001:db8::1" )
~~~~


# Name Server Behaviour

## General Behaviour

The parameters specified in the boot property will contain hostnames, for example in the NS records and in the SOA; these will be replicated verbatim into the zone upon creation, and so it should be noted that if they would be fully qualified in a manually created zone file, they MUST be fully quallified in the parameter specification in the property.

# Security Considerations

This document does not alter the security considerations outlined in {{RFC9432}}


# IANA Considerations {#IANA}

IANA is requested to add the following entries to the registry:

Registry Name: DNS Catalog Zones Properties

Reference: this document

| Property Prefix | Description        | Status          | Reference     |
| boot            | Bootstrap          | Standards Track | this document |
| soa             | Start Of Authority | Standards Track | this document |
| ns              | Name Server        | Standards Track | this document |
{:title="DNS Catalog Zones Properies Registry"}

Field meanings are unchanged from {{RFC9432}}


--- back

# Catalog Zone Example

The following is an example showing the additional properties and parameters as outlined in this document.

There are defaults specified for the SOA and NS records, which would be used by the example.com. zone.

The example.net. zone would utilise the default SOA record, but would utilise the more specific NS records.

The default nameservers are in-bailiwick of example.com, which is in the catalog, and so the address record details are supplied in order to facilitate the addition of the address records.

~~~~
catz.invalid. 0 SOA invalid. invalid. 1 3600 600 2419200 3600
catz.invalid. 0 NS invalid.
soa.boot.catz.invalid. 0 TXT ( "mname=ns.example.com.; "
      "rname=hostmaster.example.com.; refresh=14400; "
      "retry=900; expire=2419200; minimum=3600" )
ns.boot.catz.invalid. 0 TXT ( "ns=ns1.example.com. ipv4=192.0.2.1 "
      "ipv6=2001:db8::1" )
ns.boot.catz.invalid. 0 TXT ( "ns=ns2.example.com. ipv4=192.0.2.2 "
      "ipv6=2001:db8::2" )
kahdkh6f.zones.catz.invalid. 0 PTR example.com.
hajhsjha.zones.catz.invalid. 0 PTR example.net.
ns.hajhsjha.zones.catz.invalid. 0 TXT "ns=ns.example.org"
ns.hajhsjha.zones.catz.invalid. 0 TXT ( "ns=ns.example.net"
      "ipv4=192.0.2.250; ipv6=2001:db8:ff::149" )
~~~~

# Author Notes/Thoughts

NB: To be removed by the RFC Editor prior to publication.

## Is catalog zones the right place for this?

Much consideration has been given as to whether the primary server should be consuming the/a catalog zone, rather than simply serving it to secondary servers for consumption.

It does feel a little bit like it muddies the waters between zone distribution and zone "provisioning" but:

1. In a catalog zone scenario, the catalog equally feels like the place for zone related parameters
1. It feels less like Dynamic Updates would be the right place for it
1. An API for *just* zone bootstrapping feels like a big thing that would likely not get implemented, and would likely be a part of a wider implementation's general nameserver configuration and operations API, which is waaaaay beyond the scope of this document/standardisation

It may be considered that this is "nameserver configuration", however, it has strong parallels in this regard to the "configuration" on secondary servers, including such considerations as to which entities are allowed to notify and/or transfer the zone, as are conveyed to those secondary servers in {{RFC9432}} DNS Catalog Zones. Indeed, much of the same configuration may be needed by or shared with the primary server for those same zones.

TODO - add more detail explaining the above, reasoning, etc...?

## DNSSEC {#dnssecConsideration}

It seems that it'd be useful to signal initial policy/settings for DNSSEC in a standardised way also, but the primary might, or might not be the signer; the signer may be downstream. But it might be very nice to be able to signal to the signer that it should create some keys, sign the zone...

## Properties

### General

Should the properties be listed in the registry as "soa.boot" and "ns.boot", given "boot" itself is a placeholder label, and doesn't (currently?) take any parameters or records of its own?

### coo Property

Are there any considerations around change of ownership that need mentioning or documenting here...?

### soa Property

Consideration was given as to whether things like SOA parameters should be individual records, but it seemed unnecessary to break them out and create the additional records.

Should we permit the property to be made up of multiple TXT records so long as a given parameter is not repeated?

Should we specify an soa serial format? or an initial soa serial value...?

Given that it's pretty much expected that the operator is going to start making changes to the zone via dynamic updates, it'd be reasonable to expect them to be able to set those parameters. Which does beg the question, do we need to specify soa and nameserver values at all, or just specify that the zonefile is or is not to be created, and fill some template default values with the expectation that the operator would immediatly overwrite them with "correct" values...?

### ns Property

Is there a circular dependency or race condition issue here...?

# Change Log

NB: To be removed by the RFC Editor prior to publication.

## 00 - Initial draft

# Acknowledgments {#Acknowledgements}
{:numbered="false"}

TODO acknowledge.
