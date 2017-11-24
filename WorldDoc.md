This document is for the actual specification for the current state of the art.  This is likely to evolve in all SORTS of ways, so we're going to call this Revision 0.

Original version: <https://wiki.alopex.li/SemanticWebThoughts>.  Moved to a git repository on github so thing's a little easier to organize and structure.

[Notes, thoughts and inspiration](WorldDocNotes.md)

This is loosely structured on IETF RFC formats.

# Abstract

# Introduction

# Terms and definitions

We're going to define all structures as Rust code structures because they're unambiguous and that's how I'm going to be coding them.  These will define the semantics of the structures, nothing about the layout.

# Documents


## Data format

Okay, the REAL solution here is this: We MUST have a canonical format anyway, so we can turn an object into a KNOWN content hash.  For that it must be unambiguous: ie, cosmetic changes to indentation or spacing or such of the layout won't affect the generated data.  HTML, XML, JSON, YAML, Markdown, etc. all allow syntactically-insignificant whitespace and such, and so if we use something like this we MUST also have a canonical text layout and a pretty-printer.

SO, instead we can have a canonical binary format, and can take MANY sort of text formats and translate them into it.  So, HTML, Markdown, LaTeX and so on should all be able to be converted into WorldDoc, and a WorldDoc document within the target language's capabilities should be able to be losslessly (hopefully) translated back.  Pandoc may be a good tool for doing this, or there may be others.

I'm leaning towards a CBOR encoding as the default canonical format.  It's flexible, it's fairly simple, it's fairly comprehensive, it's a IETF standard, it seems in general Not Terrible.  Cap'n Proto and a couple others are decent contenders.  Then we can use XML as a simple source format to translate into this canonical format.

Now things to think about: How to provide a schema, and do we care about a meta-schema system?

## Document structure

Remember, this is Version 0.  It is intended to provide the BASIC structure and content.  We're going to leave OUT complicated questions of sublanguages, extensibility, and general-purposes semantic content.

Navigational things like sidebars, title bars, etc. should be part of the viewer application generated from metadata the index server provides.  Same for layout.  How this works is yet to be specified.  The focus for documents is *structure*.

Metadata:

 * Title (multiple subtitles or title-versions or such might be valid since something can be known by more than one title)
 * Date
 * Author(s)
 * Revisions/previous revisions
 * Subject
 * Categories (tags) (can you specify a label for tags as well, since tags might have semantic content as well?  Consider, say, Library of Congress index numbers...  So, you can specify categories, or categories-from-a-vocabulary, or such.)
 * in-response-to
 * reply-to
 * Language
 * Format/version
 * Character encoding
 * Document type?  Message, article, book, etc...  mmmmmmmmmaybe.  Might be useful, should not be necessary.  Might be a useful venue for allowing extension sublanguages to be defined.
 * Documents should be able to be nested, for instance for anthologies.

Structure:

 * paragraph
 * chapter
 * section, subsection, sub-subsection, etc.

Contents:

 * strong
 * emphasized
 * particular font face/style (for example to represent smallcaps if you're writing for Death in a Terry Pratchett book)
 * Image/figure
 * Table
 * Footnote
 * xref: Cross reference to another document/object.  This should probably be content-addressed, but there's no specific reason it exactly HAS to be.  Also, it does not necessarily need to be a WorldDoc document, or in the IPFS namespace.  Other content-addressed namespaces that might be interesting, off the top of my head, are: DOI, ISBN, ...
 * List
 * Quote (this can be inline, or a transclusion from another document, or both?)
 * Transclusion?
 * Subscript, superscript
 * Definition?  Abbreviation?
 * insertion, deletion (strikethrough)
 * Fixed-width text?  Preformatted text is a better way of putting it.  Inline vs. its own block?  Huh.
 * Comments?
 * Internal links/anchors

```rust
struct Document {
    contents: Vec<Part>
}

enum Part {
    Body(Vec<Segment>),
    Section {
        level: u32, 
        contents: Vec<Segment>
    },
}

enum Segment {
    Para(Vec<Element>),
    Table {
        header: ???
        body: ???
        footer: ???
    },
    Figure(???),
    List {
        type: ListType,
        elements: Vec<Segment>,
    },
    Quote(Segment),
}

enum ListType {
    Bulleted,
    Numbered,
}

enum Element {
    Text(String),
    Strong(Element),
    Emphasized(Element),
    Footnote(Element),
    Xref(Element, other stuff),
    Subscript(Element),
    Superscript(Element),
    Insertion(Element),
    Deletion(Element),
    Preformatted(Element),
    Comment(String),
    Anchor(String, Element???),

}
```

 ### Things to do later

 * Semantic tags, such as addresses?
 * Arbitrary semantic tags?  I kindasorta feel like being able to label things `<spoiler>` or `<shitpost>` would be...  I'm not sure I can say "virtuous" or "useful", but would get a lot of usage.
 * Math (LaTeX or MathML may both be worth investigating in terms of what primitives you want to be able to provide)
 * font/style span?


Some of these things, such as tables, math, perhaps images (SVG?!) are more or less their own sublanguages...

Separate section for external markup?  Crossreferences?  Aux data such as images?  Annotations?  MAYBE.  Something to think about more.

Okay, there's a few main things.  There's things that relate one text to another (reply to, etc).  There's things that tell you about the MEANING of bits of the text (emphasized, quote, list, etc).  There's things that tell you about the meaning of parts of the CONTENTS of the text; semantic tags such as addresses, marking a term as foreign, etc.  Then there's stuff tell you stuff ABOUT a text RELATING to another text (X is a translation of Y, X is commentary on Y, etc).

These are things that mark the TYPE of things in the text, then there's things that describe the STRUCTURE of text.  Hmwrm.

It's a bit of a bind because this is both an authoring format and a information-linking format and the needs for both are a little disjoint.  And being really good at both is hard.

Commentary from Azure:

```
@icefox @dasyatidprime Being able to specify a set of titles is useful since texts are often known under more than one (e.g. 'Song of Solomon', 'Song of Songs', 'Canticles') Even if there's only one display title, it's worth being able to look up the document under multiple titles.

Subject and categories are interesting. I think what you actually want is something like:

<subject vocabulary="Foo" entity="Bar" />

(Compare Library of Congress to Dewey Decimal.)

Categories are similar. In addition to the folksonomy, you might want tag from multiple controlled vocabularies. (Or even have multiple folksonomies. If I were to search for tags as applied by Youtube Commenters hat might be different from tags applied by SF Publishers.)

As to reply/response to that should probably be handled with a generalized link element and some vocabulary of relations.

Structure seems a bit impoverished. One would want to have multiple divisions. Perhaps have a general section container that can do for chapters/articles/etc. and maybe annotations.

There should be some notion for embedding one self-contained WORK within another, like anthologies.

LaTeX is not an acceptable way to present math. Presentation MathML is not an acceptable way to present math. Content MathML is bad but acceptable.

Specific font isn't ideal. Maybe some notion of span but with the idea of attaching some meaning that may be defined later and as a /fallback/ a few formatting /suggestions/.

Xref is interesting as is transclusion. It's probably worth being able to mark text as being /commentary/, amplification, or translation of some other text.

We'd probably also want some way to synchronize multiple versions of a text, so that someone look at ten of them can compare.

It's worth looking at Plantinga's ThML <https://www.ccel.org/ThML/ccel-URI.htm> since both that and various classicist markups do a lot to deal with texts that relate to and are variants of each other.

I don't think you want strikethrough, you want INS and DEL and maybe other editorial marks. Perhaps the ability to say who demanded/made this change when and why.

Bold and italic are overloaded. Maybe not bold so much.

But there should be separate tags for emphasis, indirect quotation, internal monologue, marking a term as foreign, marking a title.

Various 'lexical mentions' of a word including the introductory definition where we say what something is.

Words translators added in the bible… none of these should be marked 'italics'. They should have their own semantic markup. new semantic markup should be able to be added. Ideally it should try to subtype from something known.

Woops, meant to point you to https://www.ccel.org/ThML/ThML1.04.htm

I don't think 'fixed width' is something the author has any business specifying, but they DO have reason to specify things like computer input and output, source listings, etc.

Preformatted is interesting. I would tend to think of it as more 'literal'. (i.e. I am inserting a LITERAL BLOCK OF SOURCE CODE.)

Semantic tags are important. I'm less sure about superscript and subscript. I'd like to include as many things that they're used to represent as possible.
```

## Distributed storage

IPFS works.  Though it's a little pinky-and-the-brain-y.  I really wish they would just concentrate on making a DHT that is as good as possible for static storage instead of making it do everything.

Also look into https://datproject.org/

## Distributed databases

 * https://github.com/orbitdb/orbit-db
 * https://github.com/attic-labs/noms

Making distributed update notifications is hard: it seems you kinda either have to use pubsub, which is transient, or you have to use some sort of distributed database, which is hard and not necessarily low-latency.  I was thinking about a thing where users can basically broadcast queries, like "are there any newer versions of this particular document?" or such, and upon contemplation you could ask for *any* kind of query that way.  Replies could be signed to prevent forgery, though DoS could be hard to stop; you could verify that the stuff they give you is actually from the original author, but you would have a harder time preventing you from being flooded with junk and having to do something about it.

Like, on one level really is just "ask your gossip-y neighbors on the network to tell you what they have", but it really comes down actual database stuff.  The goal is to basically make centralized index servers not necessary.  One of the advantages of centralized index servers though, is it's easy to decide whether nor not to trust them to.  (No it isn't.  But it's easiER.)

No reason this protocol can't evolve though.  If you can query a single server for relations and such, then you can probably make the same query to a distributed database and get more or less the same info back.  The main thing you can't do is ask "tell me what you think I should have for this document".  You also can't easily ask "tell me EVERYTHING you know about this document" because that will just feed you the entire internet.


# Identity protocol

## Public keys

A HTTP server that takes an email-style username and just returns the public key for the user associated with that url.  Suggested translation: `icefox@alopex.li` becomes `https://alopex.li/pubkey/icefox` or maybe `https://pubkey.alopex.li/icefox`.  You could argue for ditching the email-style user ID, which is fine, but I would argue that a) it contains exactly what is needed and nothing more, and b) it is what people expect to see.

How the public key gets there and how it is updated is undefined.

The response message must contain:

 * username
 * Encryption algorithm (which ones are valid is currently undefined but should include ed25519 at least)
 * public key as base64-encoded text
 * Last update date/time
 * OPTIONAL: cache TTL?  Essentially "when this key is expected to be replaced".

To do: You're going to want to be able to fetch old public keys so you can validate old messages.  A datestamp field and an easy way to get the first public key published before that date seems worthy.  A monotonic increasing serial number might also be useful but really is entirely supplanted by an ISO datetime which is also monotonic increasing.

Security holes: This depends on no MITM, no domain spoofing.  The source of authority basically becomes whoever controls `alopex.li`.  The goal is to make it decentralized (anyone can run their own thing and anyone's thing can talk to each other), not distributed (no clients or servers).

## Private keys


This is entirely optional and there's plenty of other ways of doing it, including having the users manage their own private keys somehow.  The PURPOSE is to, one way or another, get a private key into the browser/viewer/client application, and allow the user to easily get the corresponding public key onto a public key server..

The way I envision this, for practical purposes, is as a centralized keystore that can be either hosted by a user on their own or by some company for their users or whatever.  It goes through some implementation-defined authentication (for example, logging in with a shared secret), and then gives the user operations to manipulate their private keys, probably with corresponding effects on their public keys.  Operations should probably include:

 * Create new keypair
 * Delete private key (for security/ergonomics reasons?  Probably not strictly necessary)
 * Replace keypair (creates a new keypair and public key, publishes a new pubkey version)

You COULD imagine the "publish new public key" operation being the domain of a public key server; any request to publish a new public key, signed with the old private key, would be accepted.  I think this is actually a bad idea in the general case, because if someone subverts an identity and publishes a new public key for it you basically lose control of that identity forever, unless a human steps in and rolls it back.  Doing it here basically provides an extra layer of authentication for the process.

Actually just having pubkey servers take a signed message is maybe not too bad; either way a human has to step in and take corrective action to roll back a compromised identity, which has to happen anyway.  And having pubkey servers handle it is really part of their job anyway, and makes legitimate updates super duper easy.  Also allows identity providers (pubkey servers) be entirely separate from privkey stores.

Note that both public and private key storage should be completely log-based, append only and letting you step back in time to view all changes.



Evolution of the name server concept, which will be obsolete once this is fleshed out.

Essentially, there are three document operations:

 * Retrieve a document by name (rather than content address)
 * Submit a new document to be indexed (does this subsume the "update name to point to new document" operation?)
 * Ask for a list of documents that have a particular relation to an existing document

There is one server operation:

 * Ask for the types of relations it knows about

Most responses should be stored as IPFS nodes.  Responses that return non-fixed-sized data (queries) should be chunked/paginated, rate limited, etc.  (They can also, optionally, return an IPFS content address to the whole content for the client to retrieve at their leisure.)

# Name servers

First off: Why not just use DNS?  DNS already gets used for various bits of this.  The answer is: DNS is not really designed for rapid-updating content.  The Web provides essentially instant feedback and we need that too.  DNS is not designed to contain URL's.  DNS is also not designed to handle query strings, post requests, or other such things.

Second off: Why is it client-server instead of peer-to-peer?  The answer is: client-server is easy, both for users and the person actually running the damn thing.  It's a solved problem.  I like solved problems.

Okay, now that's out of the way, the fundamental mechanism is to turn a URL into an (IPFS) content address.  This lets you trivially look up mutable IPFS objects using human-readable names, just with an HTTP GET request to a URL.  Then you can take a POST request to the same URL that lets a user update the content address with a message that contains:

 * User ID
 * Datetime (to prevent replay attacks)
 * Target server (to prevent attackers from shuffling a recorded request somewhere it shouldn't go)
 * New address
 * Crypto signature (containing all the above)

But it alone is not really useful though, because really you want to get a document, and some set of associated documents.  You very quickly end up doing relational database queries, it just returns content addresses that the client fetches from IPFS.

So we very quickly come to the "document" part of "WorldDoc", with questions like, how do you send various sorts of database queries?  A LOT of this, and a lot of other web app type things, are really just RPC database lookups.  How does the server reply with aggregates of content ID's, expressing the relationships between them?

Fundamental operations we want to do:

 * Get document and responses to it
 * Post response to document
 * Get previous version of document by datetime
 * List documents???  If so, need to specify a datetime range.
 * Search documents???  By author, by datetime, by number of responses, etc, etc...  Categories, newer revisions, other basically-arbitrary metadata.  So we need to either have a way to describe this to the client, or a way to describe it to the user so they can tell the client what to do.
 * Tell the server that you're linking to one of the things it cares about.  This probably would work by just submitting a document to the server and letting it parse it and figure out how to connect it to everything else.

So this rapidly becomes less a name server and more a searchable document database.  Really that's a distinction between "mostly-static web server" and "conversation engine".  Really the first part can be implemented basically entirely with a static web server, apart from posting updates.  That's fine for very static webpages...  But we want something beyond that.  Comments, conversations, messaging, all rely on users being able to search for relations between documents, not just pull fixed URL's.

In a website, this relationship is described by the user; they explicitly make hyperlinks to link documents together.  In this system, I want to have as much of that as possible ABLE to happen automatically, described to you by the server; so the server can say "for this document, here's all the things I know about it" and let you search in those terms?  "here's all the documents referring to it", "here's all the documents replying to it", "here's all the documents listing it as an earlier version"... so we have different TYPES of relations to express, ahhhh, NOW we are getting into semantic-web territory at last.

So the thing is... this is still a particular application, of which there can be many different types with different capabilities.

Client implementation: Probably a firefox plugin or such, do some research; it'll have to talk to IPFS somehow.  Running it as a website pulling from a known IPFS source (local or remote) is also desirable.  It will DEFINITELY be nice to be able to render this into an HTML page.

# Pubsub

aieeeeee okay here things get very vague.

There's two types of notifications: Pull (RSS) and push (email).  Every push notification eventually becomes a pull notification (user opens up their email client to see what's up), the difference is that pull comes from a known source, push comes from an unknown source.

Pure pull is pretty easy, that's much like what the static name server provides.

It can't be purely pubsub for an email-like personal communication system because you don't want to allow anyone to push stuff into your mail box without your consent.  So you need to give them authorization (blacklist by default and add whitelist).  Then you can have an id or key that whitelists by default so it's POSSIBLE for unknown people to contact you but you have to explicitly check for them...

eh, whitelist/blacklist policy is kind of an implementation detail.  Important parts: Messages to a specific person are encrypted with recipient's public key by default.  Messages are signed with sender's private key by default.  Anonymous messages are possible just by using a throwaway private key, but you still have an ID server somewhere willing to take responsibility for the thing.

This ends up looking a lot like email but encrypted and stored on IPFS.  That's... sort of the purpose.

What about messages which are private but sent to multiple recipients?  Easy way: encrypt and send multiple copies of the message.  Less easy way: Produce a keypair for that specific mailing list, distribute it separately to the people subscribed to that list.


# RPC???

FOR NOW we just use JSON-RPC

# Security considerations

# References

# Appendices
