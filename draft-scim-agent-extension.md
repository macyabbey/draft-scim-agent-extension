---
title: "SCIM Agents and Agentic Applications Extension"
abbrev: "SCIM Agents and Agentic Applications Extension"
category: std
docname: draft-scim-agent-extension-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
# area: AREA
workgroup: SCIM
keyword:
  - SCIM
  - Agents
  - Agentic applications
  - Workloads
stand_alone: yes
smart_quotes: no
pi: [toc, sortrefs, symrefs]
venue:
#  group: WG
#  type: Working Group
#  mail: WG@example.com
#  arch: https://example.com/WG
  github: "macyabbey/draft-scim-agent-extension"
  latest: "https://macyabbey.github.io/draft-scim-agent-extension/draft-scim-agent-extension.html"

author:
  - fullname: "Macy Abbey"
    organization: Okta
    email: "macy.abbey@okta.com"

# Normative/information references documented better here
#  https://github.com/cabo/kramdown-rfc
#
# WELL KNOWN
#  Lookup here: https://bib.ietf.org/
#  normative:
#    TCP: RFC0793
#  informative:
#    SST: DOI.10.1145/1282427.1282421
#
# FILL OUT YOURSELF
#   REST:
#     target: http://www.ics.uci.edu/~fielding/pubs/dissertation/fielding_dissertation.pdf
#     title: Architectural Styles and the Design of Network-based Software Architectures
#     author:
#       ins: R. Fielding
#       name: Roy Thomas Fielding
#       org: University of California, Irvine
#     date: 2000
#     seriesinfo:
#       "Ph.D.": "Dissertation, University of California, Irvine"
#     format:
#       PDF: http://www.ics.uci.edu/~fielding/pubs/dissertation/fielding_dissertation.pdf
normative:
  RFC7643:
informative:
  RFC7642:
  RFC7644:
  ENTITLEMENTS:
   target: https://github.com/ietf-scim-wg/draft-ietf-scim-roles-entitlements/blob/main/draft-ietf-scim-roles-entitlements.md
   title: SCIM Roles and Entitlements Extension
   author:
      - name: Danny Zollner
        org: Microsoft
        email: zollnerd@microsoft.com
      - name: Unmesh Vartak
        org: Okta
        email: unmesh.vartak@okta.com
#   draft-grizzle-scim-pam-ext-01:
#     target: https://datatracker.ietf.org/doc/id/draft-grizzle-scim-pam-ext-01.txt
#     format:
#       TXT: https://datatracker.ietf.org/doc/id/draft-grizzle-scim-pam-ext-01.txt

--- abstract

The System for Cross-domain Identity Management (SCIM) specification
{{!RFC7643}} provides schemas that represent common identity information
about users and groups, as well as a protocol for communicating that information
between systems.

The systems that tend to implement SCIM clients and servers are identity providers,
and service providers. These are the same systems that are now need to manage agents
and agentic applications across domains.

This document describes a SCIM 2.0 extension for agents and agentic applications,
which includes extensions to the core User and Group
objects, and new resource types and schemas for agentic constructs.

This extension is intended to provide greater interoperability between Identity
providers, agentic applications, agents and their clients while reducing the
responsibilities assumed by the every growing list of new protocols for agents.

--- middle

# Introduction

The SCIM protocol was originally developed to address an **abundance** of
complex standards for describing and exchanging user information.

As stated in the introduction of
[RFC7643#Section-1.1](https://datatracker.ietf.org/doc/html/rfc7643#section-1.1)

> While there are existing standards for describing and exchanging user
information, many of these standards can be difficult to implement and/or use...

> This increases both the cost and complexity associated with organizations adopting
products and services from multiple cloud providers, as they must perform
redundant integration development...SCIM seeks to simplify this problem through
an easily implemented specification suite...

With the rise of AI, agents, and agentic applications, we see another abundance
of protocols emerging, with varying levels of industry adoption,
as well as implementation complexity as many brilliant and enthusiastic
early adopters rush to define new standards for identity interopability.

This includes but is not limited to:

- [ACP](https://agentcommunicationprotocol.dev/core-concepts/agent-discovery),
- [A2A](https://a2a-protocol.org/latest/topics/agent-discovery/),
- [ANS](https://genai.owasp.org/resource/agent-name-service-ans-for-secure-al-agent-discovery-v1-0/),
- [AGNTCY](https://docs.agntcy.org/dir/overview/)

The intent of this SCIM extension is to offer a viable path for the industry
to re-leverage the well known core SCIM specifications, as well as existing
implementations of SCIM clients and SCIM servers, to solve for agent cross domain
management.

In doing so, we can free the emerging standards in the agentic AI space
to focus on truly novel concerns, instead of addressing the problems already
solved by SCIM for user and groups.

For example, in the A2A protocol, instead of [describing a very high level
concept of Curated registries](https://a2a-protocol.org/latest/topics/agent-discovery/#2-curated-registries-catalog-based-discovery)
we could offer more concrete guidance by stating Agent Cards may be discovered
by a SCIM client accessing any SCIM server that implements this extension.

# Conventions

{::boilerplate bcp14-tagged}

# Definitions

Agent: A workload with its own identifier, metadata and privileges which are
independent of a particular runtime environment or containing application.
An agent is distinct from a traditional software workloads (lambdas, services, etc...)
due to varying degrees of unpredictable behavior caused by delegation of
control flow to artificial intelligence models.

Agentic application: An application exposing one or more agents to its users.
An agentic application is similar to a traditional native or web application,
in that there are pre-defined ways authenticate and interact with the application;
however, as soon as the application exposes agents, there are additional
considerations for managing access to that application.

# Core Schema Extensions

## ServiceProviderConfig

SCIM endpoints that support Agent extensions MUST advertise this support
in the ServiceProviderConfig endpoint as defined:


      agentExtension
         A complex type that specifies Agent Extension configuration options.

         supported Boolean value specifying whether any aspect of the extension is supported.

         agentsSupported Boolean value specifying whether the agent resource type
                         is supported

         agenticApplicationsSupported Boolean value specifying whether the agent
                                      resource type is supported

This is required so that:

1) Clients may know if the server supports the concept of Agents.
2) Servers discourage clients from confusing users and agents.

If the server does not support the concept of agents, a SCIM client MAY choose
to create a User representation in the server for an Agent. All the reasons it
may choose to do so are beyond the scope of this document. If the client does so,
the client SHOULD indicate the user is linked to an agent using a LinkedObject
from [draft-grizzle-scim-pam-ext-01](https://datatracker.ietf.org/doc/id/draft-grizzle-scim-pam-ext-01.txt) This would allow a SCIM server that
supports that SCIM extension to add support for this extension and determine
what users in the server should be mapped to agents when support is added.

# Additional ResourceTypes and Schemas

This SCIM Agent extension defines additional
ResourceTypes and Schemas that MAY be implemented by the service
provider.  If implemented, these ResourceTypes SHOULD support all
SCIM operations {{RFC7644}}.  All attributes defined in the schemas are
optional unless explicitly marked as REQUIRED.

## Agent

This extension adds a new resource type of "Agent".

Pursuant to {{RFC7643}}
[Section 3.2 Defining New Resource Types](https://datatracker.ietf.org/doc/html/rfc7643#section-3.2)
this document define the ResourceType, Schema and Extensions for Agent.

### Agent Resource Type

The Agent Resource Type schema is:

```json
{
   "schemas": ["urn:ietf:params:scim:schemas:core:2.0:ResourceType"],
   "id": "Agent",
   "name": "Agent",
   "endpoint": "/Agents",
   "description": "Agent identities",
   "schema": "urn:ietf:params:scim:schemas:core:2.0:agent",
}
```

### Agent filtering

Clients MAY have a reference to the Agent name or externalId but not the ID.
For this reason, it is RECOMMENDED that service providers implement
filtering that allows equality matching on the "name" and "externalId" attributes.

Example (note that escaping has been removed for readability):

      GET /scim/v2/Agents?filter=name eq 'Helpdesk bot'

      GET /scim/v2/Agents?filter=externalId eq '8ccc535b-716d-4d32-b3e9-57c8be449c82'

### Agent Common Attributes

The agent resource type contains the common SCIM resource type attributes
defined in {{RFC7643}} [Section 3.1 Common Attributes](https://datatracker.ietf.org/doc/html/rfc7643#section-3.1)

They are listed here for completeness:

   + id
   + externalId
   + meta

### Agent Core Schema

The core agent schema provides the minimal representation of a resource "Agent".

It contains only those attributes that any agent may need, and only one
attribute is required.  It is identified using the schema URI:

"urn:ietf:params:scim:schemas:core:2.0:Agent"

The following attributes are defined in the core agent schema.

      name  The name of the Agent.  REQUIRED

      displayName
         The display name of the Agent.  If displayName is unassigned,
         the name MAY be used as the display name.

      description
         The description of the Agent.

      type
         The type of agent. There are no canonical values defined
         for type, but service providers MAY choose to define the valid
         types.

      active
         A Boolean value indicating the agent's administrative status.  The
         definitive meaning of this attribute is determined by the service
         provider.  As a typical example, a value of true implies that the
         agent is running, while a value of false implies that the
         agent has been suspended.

      entitlements
         An optional complex object that indicates entitlements the agent has.
         Its form is precisely the same as that defined in Section 4.1.2 of
         {{RFC7643}}.

      roles:
         An optional complex object that indicates roles the agent assumes.
         Its form is precisely the same as that defined in Section 4.1.2 of
         {{RFC7643}}.

      groups:
         An optional read-only complex object that indicates group
         membership.  Its form is precisely the same as that defined in
         Section 4.1.2 of {{RFC7643}}.

      applications
         A complex multi-valued attribute referencing applications this agent
         shares a trust boundary with. See "Agentic Application" section of
         this document.

      <!-- TODO subject be in an extension instead of core? -->
      subject An optional attribute that clients may specify when
              provisioning an agent so that
              service providers implementing inbound token federation
              may correlate the agent with the `sub` claim in
              an inbound token from an OpenID connect provider.

      <!-- TODO should protocols be in an extension instead of core? -->
      protocols
         A complex attribute that informs service providers of the
         various communication protocols an agent may support.
         This information can help service providers automatically
         support agent to agent or human to agent communication scenarios.
         An agent that supports no protocols is understood to the service provider
         to not be directly accessible. For example, when an agent can only
         be accessed via its containing agentic application.

         The following sub-attributes are defined.

            type The type of the protocol. A number of canonical values
                 are provided based on known agent protocols. They are:
                 A2A, OpenAPI, MCP-Client, MCP-Server

            <!-- TODO  example values per spec type -->
            specificationUrl The URL the service provider may retrieve the
                             specification document describing the agent's specific
                             information for that protocol.


      <!-- TODO should owners be in an extension instead of core? -->
      parent
         A complex attribute that defines the parent Agent of this
         Agent if the service provider supports hierarchies of
         agents.  The following sub-attributes are defined.

         value  The ID of the agent that is the parent of this
            Agent in the hierarchy.

         $ref  A URI reference to the Agent that is the parent of this
            Agent in the hierarchy.

         display  The display name of the Agent that is the parent of
            this Agent in the hierarchy.

      <!-- TODO should owners be in an extension instead of core? -->
      owners
         A complex multi-valued attribute that defines the User or Group objects
         that are owners of this Agent.  OPTIONAL.  The following sub-attributes are
         defined for each value object.

         value  The ID of the User that owns this Agent.

         $ref  A URI reference to the User that owns this Agent.

         display  The display name of the user that owns this Agent.



#### Agent JSON Example

<!-- TODO -->
```json

```

#### Agent Schema Json

<!-- TODO -->
```json
```

### Agent extensions

<!-- See TODOs above, should some of those attributes come out and be extensions? -->
<!-- Do we need an "EnterpriseAgent" similar to "EnterpriseUser" extension? -->

## Agentic application

### Resource Type
### Filtering
### Schema
### Example

# Schema JSON Representations



# Security Considerations

> fill out

# IANA Considerations

This document has no IANA actions.

# Change Log

-00

+ Initial draft extension.

--- back

# Acknowledgments

We would like to thanks the authors of the
[SCIM Extension for Privileged Access Management](https://datatracker.ietf.org/doc/id/draft-grizzle-scim-pam-ext-01.txt) and [Device Schema Extensions to the SCIM model](https://datatracker.ietf.org/doc/draft-ietf-scim-device-model/)
which served as excellent guidance on how to document proposed extension to
the SCIM protocol.

Additionaly, we would like to thank all the contributors the emerging agent
standards which inspired this extension, including:

- [Agent communication protocol](https://agentcommunicationprotocol.dev/core-concepts/agent-discovery)
- [Agent 2 Agent](https://a2a-protocol.org/latest/topics/agent-discovery/)
- [Agent name service](https://genai.owasp.org/resource/agent-name-service-ans-for-secure-al-agent-discovery-v1-0/)
- [AGNTCY directory](https://docs.agntcy.org/dir/overview/)
