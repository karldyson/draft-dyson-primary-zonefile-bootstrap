---
###
# Internet-Draft Markdown Template
#
# Rename this file from draft-todo-yourname-protocol.md to get started.
# Draft name format is "draft-<yourname>-<workgroup>-<name>.md".
#
# For initial setup, you only need to edit the first block of fields.
# Only "title" needs to be changed; delete "abbrev" if your title is short.
# Any other content can be edited, but be careful not to introduce errors.
# Some fields will be set automatically during setup if they are unchanged.
#
# Don't include "-00" or "-latest" in the filename.
# Labels in the form draft-<yourname>-<workgroup>-<name>-latest are used by
# the tools to refer to the current version; see "docname" for example.
#
# This template uses kramdown-rfc: https://github.com/cabo/kramdown-rfc
# You can replace the entire file if you prefer a different format.
# Change the file extension to match the format (.xml for XML, etc...)
#
###
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
area: Operations and Management
workgroup: Internet Engineering Task Force
keyword:
 - catalog zone
 - zonefile bootstrap
 - primary server

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

This document specifies an update to {{RFC9432}} such that the primary server can bootstrap the zonefile using information contained within a catalog zone.


--- middle

# Introduction

Once a DNS zone file exists, there are standards compliant ways to distribute that zone to secondary servers in {{RFC9432}} DNS Catalog Zones, as well as standards compliant ways to alter the contents of the zone in {{RFC2136}} Dynamic Updates.

However there is no standards compliant method of bootstrapping the presence of a new empty zone file ready for those two processes to work with.

There are vendor specific methods, such as RNDC for ISC BIND, which can be used remotely, but which still requires you to have created the underlying zone file on the primary server filesystem.

PowerDNS has a proprietary API that can be used, and other products likewise have mechanisms.

However, there's no standards compliant vendor independent mechanism of signalling to the primary server that a new zone file is to be created.

Operators of large scale DNS systems will want to be able to signal the creation of a new zone without wanting to be tied to a particular vendor's software, and without the need or overhead of engineering a bespoke solution with the ongoing need to support and maintain that.

This document sets out to instigate discussion of possible solutions to this problem with an initial strawman.

It may be considered that this is "nameserver configuration", however, it has strong parallels in this regard to the "configuration" on secondary servers, including such considerations as to which entities are allowed to notify and/or transfer the zone, as are conveyed to those secondary servers in {{RFC9432}} DNS Catalog Zones. Indeed, much of the same configuration may be needed by or shared with the primary server for those same zones.

The scope may of this document is confined to the initial bootstrap provisioning of the zone file, and broader provisioning of the base nameserver configuration is beyond the scope of this discussion and document.


# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Member Zone Properties

# Security Considerations

This document does not alter the security considerations already outlined in {{RFC9432}}


# IANA Considerations

This document has no IANA actions.


--- back

# Catalog Zone Example

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
