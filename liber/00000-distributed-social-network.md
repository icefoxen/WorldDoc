---
title: Distributed Social Network
status: draft
---

# Summary
[summary]: #summary

Implement a distributed social network _without_ a blockchain, need to host a
server or any other technical skills required to use it.

This is the main RFC for the whole project and the initial layout how things
should be done.

# Motivation
[motivation]: #motivation

All social networks suck. This one sucks less.

# Detailed design
[design]: #detailed-design

* Each client is a full node in the network
    * Producing content
        * Each "profile" creates an append-only log (think git log) chain of
          profile changes (only metadata, no content)
        * A profile is announced via a IPNS hash
        * A change is announced to all conntected nodes via Pubsub

    * Processing content
        * Each notification about new content is remembered, optionally:
            * The changes (only metadata) are fetched + cached
            * The changes (metadata+data) are fetched + cached

    * Forwarding content
        * Every time content gets announced to the node, it can (optionally)
          notify all contected nodes that it now knows a profile (IPNS hash)
          or an article (IPFS hash).

    * Providing meta-information (optionally)
        * Which profiles are known (IPNS hashes)
        * What content is known (IPFS hashes)
        * What profiles are cached (IPNS hashes + IPFS hashes)
        * What content is cached (IPFS hashes)
        * This is required for using servers as federated caching nodes

* A "profile change" (could be named commit) is a full JSON blob of the
  profiles metadata, containing:
    * A hash of the previous profile state
    * A list of published content
    * Profile metainformation, e.g. Names, other structured data
    * Is (optionally?) signed via GPG/PubPriv-Keypair

* Publishing an article
    1. Publishes the content
    1. Publishes the hash of the content with meta information:
        * Size of the published content
        * (MIME?)-Format of the content
        * Other structured meta information (tags, category)
        * (IPNS-)Hash of the profile published the content (optionally?)

## Profile JSON format
[profile-json]: #profile-json


```json
{
  "previous": [ "<ipfs hash>" ],
  "to be": "defined",
}
```

The idea of having an _array_ in the "previous" key is because merges might
happen.

Consider a profile at hash `A`. If now a user (Alice) updates her profile on two
devices at the same time (thus creating version `B` and `C` of her profile),
there are suddenly two versions of a profile which are, indeed, current.

As soon as the clients Alice uses can reach eachother, they are able to merge
`B` and `C` and agree upon _one_ current state of Alices profile: `D`. `D`,
which is itself just another entry in the storage, would then simply look like
this:

```json
{ "previous": [ "B", "C" ] }
```

whereas both `B` and `C` are

```json
{ "previous": [ "A" ] }
```

Thus, the following DAG is automatically created:

```
A <-- B <-- D
^           |
 \         /
  --- C <--
```

## Content JSON format

```json
{
  "previous": [ "<ipfs hash>" ],
  "post": {
    "type": "<post type>",
    "nodes": ["<ipfs hash>"],
    "metadata": {
      "date": "2017-12-12T12:00:00+0200",
      "tags": [],
      "category": "kittens",
      "custom": {}
    }
  }
}
```

## Multi-device publishing

In [profile-json] was described how profile versions would be linked and
create a DAG. Hence, the problem of concurrent updating of a profile can be
considered solved at this point.

What is not yet described is how the content is _announced_.

### Pub/Priv Key layout

To explain how content announcing from multiple devices/clients would work,
the layout for pub/private key usage must be explained first.

Each "profile" would be identified by one public key.
Each device would identified by one public key as well, thus each device would
hold two private key: The one to publish an IPNS name for the current version
of a profile _by device_ and one to publish for the current version of the
profile overall.

By resolving the IPNS name of the current profile and all IPNS names of other
devices, a client is able to find out whether versions have diverged and is
now able to merge versions and publish a new version itself.

## Multi-profile functionality

Multi-profile support would be as simple as using another set of keys to
publish content.

## Required technology
[tech]: #tech

* IPFS
    * as storage framework
    * pubsub
* IPNS
    * "mutable content"
* IPLD
    * linking, traversal of links
* JSON
    * metadata format
    * Fault-Tolerant parsing required
* Frontend Framework

# Alternatives
[alternatives]: #alternatives

Port scuttlebutt or zeronet to IPFS. Implications unclear.

# Unresolved questions
[unresolved]: #unresolved-questions

# Future work
[future]: #future-work

