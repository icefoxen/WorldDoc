
# Text

So let's think a little bit about this semantic web thing, now that I know a thing or two about data and networks.

So first let's lay out some axioms, so we don't have to wonder about them:

 1. **All data will be content-addressed.**  It will thus also be immutable; changes will create a new datum with a new address.  Assume there will be some sort of human-readable representation that can be updated to point at new data objects.
 1. **Data is separate from style.**  We'll leave it there for now.
 1. **Data has some sort of encoding.**  We'll use XML for now until we figure out something better.
 1. **For now we're dealing with documents.**  Other sorts of text and data, structured and unstructured, are out of scope for the moment.
 1. **There will be indices.**  Lots of them.  Don't sweat the details yet.
 1. **It will be transferred over HTTP(2) and should be able to be rendered as a webpage.**  Gotta make it remotely possible to work with current tech.  (Though IPFS or something would be cool.)

So we're not going to consider too much right how now data is STORED or LISTED or DISPLAYED, just how it is FORMED.  We'll investigate the rest later.

I'm bad at theory, so let's start out with a practical application or four: Blogs, forums, Twitter, public mailing lists.

These are ALL sort of variations on a theme: Someone is publishing text for many people to read and potentially reply to.

FIRST off, let us consider the differences: lengths, topic-ness, broadcast vs. subscription vs. you-have-to-go-look-for-it.  You can also consider fora direct messages or tweets or such that have a *particular target*, and can be public or private... now suddenly we can encompass email as well.

Similarities: They all are marked-up text.  They all have a lot of similar traits: Author, creation date, sometimes title/topic, sometimes metadata about topic and such.  They can all have crossreferences inside them or attached to them.

Okay, so let's consider what a POST of this sort might look like:

```xml
<post>
<title>Semantic Web Thoughts</title>
<author>Simon Heath</author>
<date>2017-19-09 13:56 EST</date>
<lang>en_US</lang>
<categories>
    <category>computer</category>
</categories>
<text>
    So let's think a little bit about this semantic web thing, now that I know a thing or two about data and networks...
</text>
</post>
```

There, that's pretty simple.  Easy, right?  (We will IGNORE the details of categories right now, see Axiom 5.)  Great.  Okay...  Well, what if we want to edit this page?  Well we can't change it because everything's content-addressed and thus immutable, so we have to publish a new copy.  We want the new copy to have SOME relation to the old copy, so we need some sort of reference.  We'll call it "previous-draft" for now but there's probably a better name for it.

```xml
<post>
<title>Semantic Web Thoughts</title>
<author>Simon Heath</author>
<date>2017-19-09 14:05 EST</date>
<lang>en_US</lang>
<previous-draft>some address goes here</previous-draft>
<categories>
    <category>computer</category>
</categories>
<text>
    Okay, honestly I don't know anything.
</text>
</post>
```

Okay, first off, boo hoo we're storing lots of redundant data.  Don't worry about that right now; there might be a diff format or something this compresses down to, see Axiom 3.  We have a backref to the old version but we CAN NOT have a forward-ref from the older to newer one, so we NEED Axiom 5.  These things might also contain references to a suggested index which is in charge of tracking this shit, but we're going to need to leave that to Axiom 3 for now.

All right, so this is a blog post, so you're going to have a comment section.  Each comment is really just... another document:

```xml
<post>
<author>Carebarestare</author>
<date>2017-19-09 14:25 EST</date>
<lang>en_US</lang>
<response-to>some address of Semantic Web Thoughts here</response-to>
<text>
    Aww, don't be so hard on yourself
</text>
</post>
```

So we can have threads of comments why not:

```xml
<post>
<author>Trolly McTrollyface</author>
<date>2017-19-09 14:30 EST</date>
<lang>????</lang>
<response-to>some address of previous comment</response-to>
<text>
    lol no u suk
</text>
</post>
```

Great, easy, we have blogs now.  With comments.  Threaded, branching comments.

I'm really not sure that forums would be any different.  You might want a functionality to quote text, and you could do this by copying it inline...  Or, since things are immutable, you could use transclusions to refer to specific bits of specific posts and make it easy to zip back and forth.  This would also make it trivial to see edits, addenda, that sort of thing.  Mailing lists have basically the exact same content, so.

We do a lot of turning one-way references into two-way references in this system, because one way references is all we can represent but two-way ones are often quite interesting to users.  Immutable content-addressed data means that only one-way references are possible to store in the data, at least until we invent a way to travel back in time.  So the client will be doing a lot of collecting and indexing of STUFF.  Or a server might do a lot of it for it, and give the client a selection of indices or such to choose from.

A thought: Perhaps we could represent a human-readable address as a URL plus a version number?  Or have a URL that includes both the human-readable and content-hash addresses, so you can know "oh this WAS pointing at this very specific thing but that has changed since".  idk.

Okay, let's extend our hypothetical forum-y thing a bit.  What if we wanted to include upvotes and downvotes?  Well, the SIMPLE way would be to add messages referring to the original post with, say, "+1" or "ME TOO" or whatever as the text.  That's what we used to do, and it was wasteful and noisy.  Now it's becoming more common to just have the server have an integer attached to a post.  ...buuuut we can't do that, we have to... basically, count in unary.  But we need to do that ANYWAY on the server side if we ensure that users can't upvote or downvote something more than once, so really we're not losing anything.  The server MAY wish to advertise such information as the count of up/down vote posts as some sort of metadata field.  The client can choose what to do with it, or not, without having to ask explicitly.

So this brings us to one interesting insight about metadata: Metadata can change even if the data itself does not.  So that's something to keep in mind for user interactivity.

Seems wasteful to have an entire message with text field, date and such for just an upvote.  It might be possible to make a more compact type of message?  So we'll end up with three competing non-standards for different ways of representing upvotes... BUT that's not really any worse than now, when they're all silo'ed anyway.  But it also means that any ol' guy can make some sort of message type and it's entirely up to the client to interpret it properly.  That's the same as we have now, one way or another, we just have Javascript on the client doing the interpreting.

Hmmmmm.

Anyway.  Let's consider our continuation of discussion forms.  Twitter: Exactly the same.  For @'s we have reply-to's, for hashtags we have category metadata, likes and retweets are basically identical to Reddit upvotes/downvotes.

So yeah, that wasn't hard.  Pretty much all of human online discourse: Who said it, what they said, when they said it, with optional what they said it about and who they said it too.  All else is basically a database lookup based on those properties.

Is there any reason IM things can't use basically the same format?  Not that I can think of.

Next week, we eat a bus.

# Structured data

The current trend towards JSON API's actually does this halfway decently apart from a few well-known caveats (no date types, relatively non-dense, etc).  Besides that, there's a few *actual* problems with it as it stands:

 * The schema and verification technology sucks and nobody uses it
 * Interfaces are totally undiscoverable (there's no way you can point a browser at some sort of machine-generated index and see what's available)
 * Security is a shambles (CSRF, XSS, whatever kind of crap)
 * Most of the time you're really just querying a database but everyone implements their own stupid way of doing it
 * Most of the rest of the time you're not actually just grabbing data but making remote procedure calls to do some computation on the server and everyone implements their own stupid way of doing it
 * It's hard to prevent abuse (rate-limiting, pagination) and everyone implements their own stupid way of doing it

These aren't really problems with the structured data format but all the crap *around* it though.  Hmmmm.

# Binary/dense data (media)

These are actually generally quite simple 'cause they tend to be terminal data types, that don't point to or rely on other objects.  You might need to represent metadata ABOUT them but you rarely need to think about how they relate to each other...  Hmm.

Relations between these CAN be described by an external document of some kind that expresses the semantic connections between them.  For instance, Youtube description sections are often used for this purpose, to say "these are other videos that I want to call your attention to in relation to this one".

This may or may not end up being a medium-specific sub-format, for instance to describe song metadata (genre, artist, etc).  Whether it does or not doesn't really matter right now, just keep it in mind as a possibility.

# Take two: WorldDoc

So I started thinking more about how a document system based on this sort of thing would work and it kinda spiralled, and then I started thinking about the distributed application platform side of it and realized that the document system is really useful even without the application platform bit.  So I want to think in a bit more detail about how the document part of it would actually work.

So first, you have to start with a name shortcut system.  DNS more or less works fine for this, though there will probably be more atop it.  This changes human-readable names into content addresses, and also serves as an entry point that is easily updated to whatever content the author considers current.

This leads to the concept of a site, which is a collection of related documents.  It defines some sort of structure or interrelationship between documents, gives you some idea of what documents there are, and probably gives you both human-readable names, and shortcuts to content addresses (you will only need one of them but both would be ideal.)

There's other ways of getting references to documents; rather useful ones could be a pull-notification system (similar to RSS), a push-notification system (similar to email or IM).

A document might also provide a shortcut to an authoritative source to retrieve it from, a host that *should* always have it.  That means you don't have to go through a DHT lookup every damn time you want something.  It's trivial to make sure you get the right document by its hash.  However, to make sure the document you got is actually made by the person it says it is by, they can include a public key signature.  That means that you can check that it is produced by a given author, and it's impossible for someone for someone to tamper with that.

Now we need a key signing system; that's an open problem for now.  Part of the problem is that crypto is very absolute; either you have access or you don't, and that works poorly in messy human systems (you need some way to recover a lost/forgotten key).  So this will probably, practically, be outsourced to some authority whose job it is to associate ID's with keys, and let you update the key (versioned keys/id's, by date perhaps?).  As long as everyone speaks the same language for key exchange it can actually operate however you want; compare email, OAuth(???).

Anyway, this suddenly leads to identity and trust, so we have a sensible way to allow people to send push notifications to a particular person.  And so you can avoid spam from untrusted/untrustworthy sources via conventional methods (ip blocks, content classification, etc).  These will all be public, but confidential information can be automatically encrypted with the recipient's public key.  (This does mean that a key breach WILL make everything encrypted with that key public forever... but that's basically the case now anyway.  (Ooh, could we just make the system regenerate keys every month/year/whatever automatically, and everyone just uses the latest available???))

Note that all this crypto/signing/stuff is entirely separate from any transport-layer encryption.  For example if this is all going over HTTP2 (which seems less likely as time goes but still) then everything will be SSL encrypted anyway.

Another service you can provide is basically a forward-link-service...  Some way to provide a notice that "this document has a newer revision, go here to see it".  This would, for instance, let authors update/revise their documents and let anyone viewing the old revision see, oh wait, there's a newer one available.  (The place to look for this information could be coded in the document itself, actually?)  This would also effectively solve broken links (for instance when Microsoft fucking updates their knowledge base system and everything now has different URL's).  (or would it solve that?  Think about this more.)  It does mean that you basically need to keep this information around *forever* or you will end up with broken links anyway, but a) the information is small so no big deal and b) no worse than the current system, again.

And this circles back around to, say, a forum application again; people submit comments by producing new documents and then asking the index system to include.  The index system actually can *not* change the contents of the documents, but has a way of letting people provide extra structure and shortcuts.  Thus the index system has to trust the client a bit because the client is telling it to update its state, and so you need a login or something (or just, you know, check that the document's signature matches a trusted user's public key).

So what we seem to end up with is a very resilient and flexible system for *humans to communicate with each other*, that basically subsumes most of the actual-communicating-with-people functionality of the Web.  Like I said, this *really* doesn't actually solve the problem of "distributed application platform", but as far as I can tell provides some very nice primitives upon which to build applications that can work together (the crypto subsystem is an application, the site/indexing stuff is an application, the forward-link service is an application... in the end you will ONE way or another communicate with these systems through RPC calls, whether it's JSON+HTTP or something else.)

Sidebar thoughts on document format: needs to be extensible, alas.  (Example: Spoiler tags are something that are often useful but I would NEVER think of this starting with a clean-slate document design.)  Needs to be easy to write (regardless of how it is actually encoded/transmitted by machines).  XML and HTML honestly fit the bill but are also terrible.  org-mode fits the bill for most purposes but is kinda wobbly and hard to pin down.  Docbook?  Nobody bloody uses docbook.  Perhaps something that is basically org-mode with machine-readable annotations/metadata/tags might be nice...  Something that allows basic formatting, and also embedding of (fairly arbitrary) richer things.  This could also be nice for applications that wish to be paranoid; it would provide a basic subset of formatting etc. that *everything* can do, and anything beyond that is trivial to strip out.  Think of what it takes to properly sanitize HTML in, say, a forum system, and the evolution of bbcode and markdown and crap like that to work around it; makes the idea of a minimal subset kinda appealing.

Another sidebar thought on distributed application system: The GREAT AND FUCKING AMAZING thing about the web is that it bundles up an application's client-side processing code, its RPC code, and its UI code allllll in one package.  The user gets it all together, all built on the same technology.  Hmmmm.

Also this all has NOTHING AT ALL to do with an actual semantic knowledge system.  Like I said, this is both very primitive and seems like it would be extremely useful on its own.

## A few random thoughts now that it's no longer 4 am

It may well be useful to separate the *content* of a post (title, author, date, categories, text) with *metadata* about a post (authoritative location, PGP signature, etc)?  I'm not so sure about that now because separating the PGP signature sort of breaks some trust, and now we need two PGP signatures; one for the post itself, one in the metadata, both contained in the metadata.  

A prototype of this should be implemented as a web site running off a (potentially local) server.  The go-ipfs program has a REST API.

Apart from the document storage itself, most of this is very fundamentally built on a collection of RPC services.  So we're going to need a solid common RPC model.

The semi-distributed authentication and signing is so amazing that there must be something wrong with the idea.

## Case study

Okay let's try to dig in more depth how exactly this works.  The moving parts are:

 * Document store.  For now let's assume it's IPFS.  Takes a content address, returns a document.
 * Identity server.  This will take a request with a user ID and return that user's public key (we'll assume GPG).  Updating/requesting a user's private key is out of scope of this for now, and doesn't need a standardized interface anyway.
 * Name server.  This will take a human-readable name and returns a content address.  For now let's assume DNS?  IPNS???  It can also take a request to update a name, and will update that name to a new content address optionally based on the message sender's signature.  The update request requires a monotonically-increasing date or some server-provided salt to be included, to prevent replay attacks.

With this alone we can make a basic chat system.  A "chat room" is a particular human-readable name.  Users send messages to it replying to the current message.  If the user is authenticated, the name server will update to provide that message as the "current" one, and then clients can follow the linked list of messages backwards as far as it cares to.  We don't even need the identity server, if we want to make something aggressively anonymous.

There's a problem here in that if there are two replies to the same message then the name server really just has to pick one and ditch the other; there's no defined aggregation or indexing as of the moment, and there's no reliable ordering.  (IPFS does provide a directory-like aggregation mechanism, as well as a way to just have a single chunk contain lists of other chunks, but I don't know enough about how those work.)  But as the most trivial possible setup it works.

Okay, say we have an index server now.  It will ONLY take a request for some sort of arbitrary identifier (we'll call it the same sort of human-readable name as the name server), and return a LIST of content addresses.  This list might just be another content address containing a document, really.  Not necessarily the same sort of document as messages or whatever.  But the point is it will contain references to ALL the messages the server knows about, so the client can arrange dangling stuff however it wants without conflict or ambiguity.

One issue is, what if the index is very long?  The index server needs pagination, and it would be convenient for it to be able to do searches for the client based on date, reply-to, and other parameters.  Rate-limiting, too.  And it might elect to be friendly and just include the documents inline, if the client wants.  It would be very convenient for all this stuff to be standardized between index servers.

Okay, so now we have a mailing list or blog-type thing, a potentially-branching conversation chain ordered by time.  We can expand it to a forum just by allowing multiple threads of conversation, each with their own location, and a way to get a list of them.  Generalizing to the functionality of a wiki or real website requires more control over how things are organized, laid out and presented to the user.

# Prototype sketch

 * Assume the client is running their own IPFS system so we don't have to worry about that.
 * Identity server: This can be just an HTTP server that returns a user's GPG public key as a document.  As a testbed this doesn't need anything to be dynamic.
 * Name server: This will also be an HTTP server with a little REST interface.  You either GET a name to get the content address associated with the name, or POST to an existing name to update it with a new content address.  Your POST request will contain: a user id, a datetime, an IPFS address to update to, and a signature of the other elements composed.  Errors will occur if: the user is not authorized, the signature does not match.  Errors should occur (but won't for the prototype) if the IPFS address does not exist or points to some invalid sort of data.
 * Client: Minimum possible thing, probably just a command-line thing like curl.  Will take a server name and either fetch the latest message or post a new message.  Once that works it will be configurable to fetch the last N messages.

A LOT of this is just a key-value database that provides optional mutation via a signed message.  hmm.

Madly, this actually bloody works.  VERY AD HOC version on on `/ipfs/QmZiYbNT1wkozLi5ZmJFzrnoQywGNGZQUPXeHY251LSm9h` as of Oct 2 2017, written in Rust.

# Actual specification

## Public key server

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

## Private key server

This is entirely optional and there's plenty of other ways of doing it, including having the users manage their own private keys somehow.  The PURPOSE is to, one way or another, get a private key into the browser/viewer/client application, and allow the user to easily get the corresponding public key onto a public key server..

The way I envision this, for practical purposes, is as a centralized keystore that can be either hosted by a user on their own or by some company for their users or whatever.  It goes through some implementation-defined authentication (for example, logging in with a shared secret), and then gives the user operations to manipulate their private keys, probably with corresponding effects on their public keys.  Operations should probably include:

 * Create new keypair
 * Delete private key (for security/ergonomics reasons?  Probably not strictly necessary)
 * Replace keypair (creates a new keypair and public key, publishes a new pubkey version)

You COULD imagine the "publish new public key" operation being the domain of a public key server; any request to publish a new public key, signed with the old private key, would be accepted.  I think this is actually a bad idea in the general case, because if someone subverts an identity and publishes a new public key for it you basically lose control of that identity forever, unless a human steps in and rolls it back.  Doing it here basically provides an extra layer of authentication for the process.

Actually just having pubkey servers take a signed message is maybe not too bad; either way a human has to step in and take corrective action to roll back a compromised identity, which has to happen anyway.  And having pubkey servers handle it is really part of their job anyway, and makes legitimate updates super duper easy.  Also allows identity providers (pubkey servers) be entirely separate from privkey stores.

Note that both public and private key storage should be completely log-based, append only and letting you step back in time to view all changes.

## Indexer

Evolution of the name server concept, which will be obsolete once this is fleshed out.

Essentially, there are three document operations:

 * Retrieve a document by name (rather than content address)
 * Submit a new document to be indexed (does this subsume the "update name to point to new document" operation?)
 * Ask for a list of documents that have a particular relation to an existing document

There is one server operation:

 * Ask for the types of relations it knows about

Most responses should be stored as IPFS nodes.  Responses that return non-fixed-sized data (queries) should be chunked/paginated, rate limited, etc.  (They can also, optionally, return an IPFS content address to the whole content for the client to retrieve at their leisure.)

## Name server

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

## Pubsub

aieeeeee okay here things get very vague.

There's two types of notifications: Pull (RSS) and push (email).  Every push notification eventually becomes a pull notification (user opens up their email client to see what's up), the difference is that pull comes from a known source, push comes from an unknown source.

Pure pull is pretty easy, that's much like what the static name server provides.

It can't be purely pubsub for an email-like personal communication system because you don't want to allow anyone to push stuff into your mail box without your consent.  So you need to give them authorization (blacklist by default and add whitelist).  Then you can have an id or key that whitelists by default so it's POSSIBLE for unknown people to contact you but you have to explicitly check for them...

eh, whitelist/blacklist policy is kind of an implementation detail.  Important parts: Messages to a specific person are encrypted with recipient's public key by default.  Messages are signed with sender's private key by default.  Anonymous messages are possible just by using a throwaway private key, but you still have an ID server somewhere willing to take responsibility for the thing.

This ends up looking a lot like email but encrypted and stored on IPFS.  That's... sort of the purpose.

What about messages which are private but sent to multiple recipients?  Easy way: encrypt and send multiple copies of the message.  Less easy way: Produce a keypair for that specific mailing list, distribute it separately to the people subscribed to that list.


# Notes

DNS CAN work for low-latency updates but basically just by setting the TTL to 1 second.  Having programs that make that easy will basically involve writing new server and client software anyway.  DNS is for things that are almost always read rather than written, while HTTP for example has more sophisticated caching rules that allow clients, servers and proxies to all know when something has changed or not.  So probably better to use a different name resolution policy in the first place.

An author also needs to be able to specify the content ID of their public key so in the event that an ID server goes kerpoof then their key might still be findable.

Good versioning of keys is a sort of open problem right now.  Ideally each new key would be signed with the old one, now that it occurs to me.  This won't always be possible, but should be nice.

I'm kind of imagining the central program of this being an Indexer, in the end.  A Notifier can be the pubsub portion which is potentially but not necessarily separate.

We ARE going to need a canonical form of some kind, so that everything formats the same content the same way before hashing it.  This can also omit meta-data that the server might want to offer.

Identity domain-to-actual-server resolution can take place through a SRV record.  And should!

Remember, hash algorithm always has to be specified!  Even if it doesn't use a pubkey, is it still useful?

The encryption and metadata is kind of an envelope format, kinda like HTTP headers maybe.  The actual contents can be basically arbitrary.  The indexer needs to be able to parse it to search for links, but beyond that its contents are really irrelevant...

The format of the index page is also sorta irrelevant.  It can be HTML for all we care?  The interesting stuff is the format of the documents and how they connect together.  (HTML might not be useful because the index page may well want to include/transclude documents into it.)

Retrieve document by id.  Retrieve document by metadata; can be specific or in a range.  Return document that has a relationship to another document.  Is... that really it?
Server will automatically paginate responses, rate-limit, etc.

Semantic-web type stuff is relationships between arbitrary bits of random data.  What we're focusing on at the moment is relationships between documents that explicitly refer to each other.

Need to have a place to aggregate all the chaff of thoughts on protocol, data format and RPC where they won't get in the way. 

You CAN go without a public key server altogether, and just include an IPFS address instead, or the actual public key inline.  That's perfectly valid, it's just you also lose the convenience of a known authority backing that identity.

Database query responses being IPFS objects can be very useful, because that makes it very easy for both an indexer to cache and offload state, and a user to version and cache state.

So we need to be able to ask the server for what sorts of relations it knows about, and then ask it more or less arbitrary queries involving those relations.  Aha!  Finally we're back in semantic territory!  Maybe we can shove that sort of information into HTML after all?

## RPC

JSON-RPC over HTTP is probably the easy way.  I like capn proto as well.  Everything should use essentially the same RPC mechanism.  CBOR is an IETF proposal and also looks reasonable, so that's a point in its favor.

Whatever the data format, going over HTTP has some very big advantages: It's simple, well-understood and has LOTS of network effects.

 * [JSON-RPC](http://www.jsonrpc.org/specification)
 * [Cap'n Proto RPC](https://capnproto.org/rpc.html)? (that actually looks potentially very good.)  
 * Protobuf maybe, though cap'n proto is designed specifically to be better than it
 * [Thrift](https://thrift.apache.org/)  WAMP, nanomsg, OMG-DDS, [HTTP-RPC](https://github.com/gk-brown/HTTP-RPC) you can go down the rabbit hole all day and I probably will.

## Data format

There's a number of pretty decent binary data formats.

 * CBOR
 * Cap'n Proto
 * Protobuf
 * Thrift
 * XDR

I don't really have any terribly strong opinions about any of them.

There's VERY FEW good text data formats.  Odd.

 * JSON (kinda meh)
 * JSON5 (better)
 * XML (kinda meh)
 * HTML (not great and we'd have to add/restrict a LOT of structure, but the network effects might be worth it)
 * YAML (pretty bad)
 * TOML (probably not appropriate)

Okay, the REAL solution here is this: We MUST have a canonical format anyway, so we can turn an object into a KNOWN content hash.  For that it must be unambiguous: ie, cosmetic changes to indentation or spacing or such of the layout won't affect the generated data.  HTML, XML, JSON, YAML, Markdown, etc. all allow syntactically-insignificant whitespace and such, and so if we use something like this we MUST also have a canonical text layout and a pretty-printer.

SO, instead we can have a canonical binary format, and can take SEVERAL sort of text formats and translate them into it.  So, HTML, Markdown, LaTeX and so on should all be able to be converted into WorldDoc, and a WorldDoc document within the target language's capabilities should be able to be losslessly (hopefully) translated back.  Pandoc may be a good tool for doing this, or there may be others.

I'm leaning towards a CBOR encoding as the default canonical format.  It's flexible, it's fairly simple, it's fairly comprehensive, it's a IETF standard, it seems in general Not Terrible.

Now things to think about: How to provide a schema, and do we care about a meta-schema system?

### Document structure

None of this is exhaustive, very much a WIP!  Sources to look at: HTML, Docbook, LaTeX, RST, Pandoc

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
 * chapter?  Section?  Are these the same

Contents:

 * strong
 * emphasized
 * particular font face/style (for example to represent smallcaps if you're writing for Death in a Terry Pratchett book)
 * Image/figure
 * Table
 * Math (LaTeX or MathML may both be worth investigating in terms of what primitives you want to be able to provide)
 * Footnote
 * xref: Cross reference to another document/object.  This should probably be content-addressed, but there's no specific reason it exactly HAS to be.  Also, it does not necessarily need to be a WorldDoc document, or in the IPFS namespace.  Other content-addressed namespaces that might be interesting, off the top of my head, are: DOI, ISBN, ...
 * List
 * Quote
 * Transclusion?
 * font/style span?
 * Subscript, superscript
 * Definition?  Abbreviation?
 * insertion, deletion (strikethrough)
 * Fixed-width text?  Preformatted text is a better way of putting it.
 * Semantic tags, such as addresses?
 * Comments?
 * Internal links/anchors
 * Arbitrary semantic tags?  I kindasorta feel like being able to label things `<spoiler>` or `<shitpost>` would be...  I'm not sure I can say "virtuous" or "useful", but would get a lot of usage.

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

## Pubsub mechanisms

https://github.com/ssbc/secure-scuttlebutt looks like it might work for pubsub-y stuff maybe ???

Look into the sorts of things that Mastodon uses.

## Visualization

One of the problems of HTML is it super duper intertwingles data with layout crap.  And metadata crap.  So I am thinking about something a little like static site generators at the moment, 'cause I'm messing with a static site generator...  You have a Layout, which may describe what goes where on the screen.  You have a Style, which describes *specifically* what things look like (font, color, etc).  And then there's ways to slot the Actual Data from one (or more?) document into this Layout; I'm imagining something a little like XPath queries or something.

Layout and Style probably associate quite closely with each other, or maybe are basically two parts of the same thing.  I don't know yet.

The GOAL is that the Layout and Style are basically irrelevant to actually fetching, displaying and associating data.  Or metadata, for that matter; a fair amount of space in (even good) webpages is devoted to navigation.