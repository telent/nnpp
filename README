Introducing the Neighbour-Net Proxy Protocol

# Goals

* To implement tools for users to construct and maintain social
  networks a la Facebook and Twitter, in a decentralised distributed
  privacy-aware fashion with builtin redundancy if nodes go off-line
  or are compromised.

  - status updates and "public walls"
  - "blog" posting for the publication of news, articles, and essays
  - public and semi-public discussion forums
  - private/direct messaging a la email

* To allow people to keep their content in their own house instead of
  at the mercy of third parties "in the cloud".  See "Eben Moglen's
  Freedom Cloud talk":http://www.softwarefreedom.org/events/2010/isoc-ny/FreedomInTheCloud-transcript.html

* By means of caching, be reasonably performant for people who want to
  read that content, without requiring everyone to have "5 9s" uptimes
  for their own hardware in their own living rooms.

* Make some attempt at being robust against DNS shutdowns or poisoning.

* Use existing standards where possible.


# Non-goals

The following are not goals.  No judgement on whether or not they are
desirable goals is intended by their omission, but they are different
problems from the problems which this proposal sets out to solve.

- We are not in the business of providing an API for "Facebook
App"-style third party we-page-in-a-frame applications.

- We are not catering for use cases where anonymity is important.
Your identity in this system may or may not correspond to your local
government's record of your identity, but it is still your identity.

- We are largely assuming you are operating in a place where the rule
of law is mostly upheld.


# Architecture

- an Atompub server with pervasive xml-enc/xml-dsig (based on PGP)
- which includes an http caching proxy open to our friends
- and participates in a distributed hash table mapping node addresses 
  (actually PGP key fingerprints) to IP addresses
- the DHT also to be interrogable via the DNS protocol

# Node

Each NNPP node is an HTTP origin server and a proxy which will cache
resources from neighbours.  "Neighbours" are typically resources
linked to from local content, or available from ATOM feeds to which the
node owner has subscribed.

An NNPP-aware browser identifies an NNPP-aware server by the presence
of a 'Cache-Control: public, nnpp-cache-links' header on its
responses.  It may then submit proxy requests to the same server for
further content that it thinks the server is likely to hold.  It must
use an 'Cache-Control: only-if-cached' header on these requests [or
something like it: might not want to usurp that token if there are
legitimate reasons for using it].  The server may respond from cache
(subject to normal HTTP cacheability rules) or by forwarding a request
from upstream, or may reject the request ("504 Gateway timeout") if it
doesn't want to serve that document.

# Content

The local content available from a node is defined by an Atompub
service document.  Various collections may exist to group resources
for different audiences (e.g. Whiteboard, Public, Work, Family, Friends,
Clubmembers, Inbox).  Whiteboard, Inbox and Public have special semantics

* Public - others can see this, the owner can post
* Whiteboard - others can see this, others can post (like a facebook wall)
* Inbox - owner can see this, others can post (a "private message" feed)

"Others" in all three of these cases is user-defined according to
preferred privacy settings.

[ to make this a credible email alternative there will need to be
provision either for tagging inbox messages or for rule-based routing
them ]

Other collections may be created ad-hoc for other purposes: users
might wish for example to create special-interest collections to avoid
spamming "normal" friends with "geek" interests, or vice versa

Remotely created content which is cached here may also be described in
a similar way: we keep a Collection in which each entry is one of our
friends service documents, and clients may expect that an HTTP proxy
request to this node for any document contained in those feeds will be
answered.

# Security

http://www.atomenabled.org/developers/syndication/atom-format-spec.php#rfc.section.5
http://bitworking.org/projects/atom/rfc5023.html#rfc.section.15.5

We use xml-enc,xml-dsig to PGP-(encrypt,sign) each entry when it is
destined for a limited-access audience.  The corresponding private key
is given to each member of the group being posted to [ how? ] and
changed if a group member is to be evicted [ again, how? ]

Note that each collection defines an http-addressable Feed document,
by atompub decree.  The information that a new document has been
posted to a classified feed may itself be classified, so these feed
resources must be regenerated and re-encrypted whenever they change.
We can limit the work involved here by making them partial feeds with
only a few entries, and by making them link to the entries instead of
including them (but I think APP dictates this latter choice anyway)

[ need to sort out all the mechanism for how to get the key and
signing it and all that jazz.  ]

Public keys should be stored by the node when first encountered.  They
can be cached by intermediaries just like any other document, but if
there is no trust relationship between the nodes then the client UA
should caution the user that the document is not verified.  To
reiterate that last point: the pubkey can be cached according to the
same rules as everything else, because it's the trust web that
matters.

This proposal currently assumes that PGP keys correspond to nodes and
not directly to users.  Thus, if you also have a mobile device
(e.g. smartphone) that you want to participate in the system, it
becomes an independent node and can be independently revoked if it is
lost or taken from you.  There are several annoying niggly bits here I
haven't fully explored: what happens if someone sends you a private
message encrypted to your home node and you are not *at* your home
node?  I think the answer will be that your default identity is in
fact a group, so your friend should encrypt for all of your multiple
keys unless he specifically wants to send you something (perhaps a
particularly sensitive document) that you can read only at home.


# Bootstrapping/DNS

If a node is publishing controversial material, an offended party may
attempt to make it inaccessible by putting pressure on the DNS
provider.  DNS in any case is a faff for people whose primary nodes
are connected with dynamically assigned IP (DSL, dialup, mobiles).

We identify nodes by their PGP key.  We make the key fingerprint the
basic unit of addressing, and the PGP key name becomes the friendly
node name.  Then we can give all nodes addresses of the form

1297D4B5.node.nnrp.net (shorthand for convenience)
13D702AD866527E08F715326776FD7C0.1297D4B5.node.nnrp.net (unambiguous)

and implement a DHT mapping node addresses to IP addresses, which
nodes will use internally in preference to ordinary DNS.  A node may
also be a public DNS server for this zone, and the published zone file
can contain as many relatively stable nodes as we want.  For the most
part we only need conventional DNS when fetching from new nodes that
we have no previous contact with, so there shouldn't be major outages
if that blows up.

To avoid dns cache poisoning or similar attacks, after fetching a
resource from a node, the client must verify that the fingerprint
matches.

