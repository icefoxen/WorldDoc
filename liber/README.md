Random old notes from https://github.com/liber-project/rfcs

Discussion on RFC 00000:

 matthiasbeyer commented on Dec 1, 2017 •

This is the first RFC (WIP) for the general outline.

Rendered.
matthiasbeyer added some commits on Nov 29, 2017
@matthiasbeyer
Initial import: RFC #00000
5fbd552
@matthiasbeyer
fixup! Initial import: RFC #00000
7f5794a
@matthiasbeyer
Add some points
d572ab1
@binary132
binary132 commented on Dec 1, 2017

Hi. Just wanted to say I saw what you were doing and I like it! The design on your blog is
very reminiscent of the implementation of Git (graph, links, object hashes, etc.) It would
be cool if you integrate the idea that, instead of IPFS as a single source of truth, each
client has its own "view" of the network (much like Git clients have their local repo.) Then
in order to update the IPFS backend you would need some kind of graph merge, or
reconciliation. Maybe something a bit like a Merge Request?

Feel free to ignore this, just sharing a few things that came to mind! grin
@matthiasbeyer
Member
matthiasbeyer commented on Dec 2, 2017

@binary132 Hi, thanks for your kind words.

Indeed I was thinking of a git-like layout of the data. But I don't think merging is a good
idea, as it creates unnecessary network traffic and there's no benefit from it. Each node
would maintain its own "profile" (or multiple) and other nodes are simply able to cache the
content, but not "merge" them into some global "chain of truth" or something (which would be
a blockchain, and I'm trying to avoid that).
@icefoxen
Member
icefoxen commented on Dec 2, 2017

I think the design looks like git because they're both essentially a Merkle DAG; a set of
objects linked by content identifiers. I think this is a rather fundamental structure and we
are only starting to plumb the depths of what can be done with them...

I like the idea of not having IPFS as a single source of truth, but I don't think that each
client needs its own "view" of the network. I think that permitting different transports or
content-addressed namespaces is perfectly reasonable; one idea I had was being able to
"hyperlink" to a book by referring to its ISBN, or to a scientific paper by its DOI index
number. These uniquely identify a particular resource; how to retrieve the resource is more
or less up to the client.
@icefoxen
Member
icefoxen commented on Dec 2, 2017 •

I have a few practical problems with this structure. Not that it's bad, just insufficient.
This also reflects my own bias.

IMO IPNS is not terribly useful for quickly-updating dynamic data:

    IPNS changes are slow. IPNS, like DNS, uses a timeout to propagate information, which is
by default 24 hours.
    IPNS hashes are not very human-readable. Though this can be easily solved by having a
display name in the profile info, I guess.
    IPNS has what I call The Private Key Problem: Handling private keys, backing them up,
restoring them, re-creating them when they are lost in hardware failures, etc. is all
generally horrible.

PubSub is not sufficient, maybe?

    Basically, as I understand it (which may be wrong), PubSub publishes a transient message
to everyone interested in it. But there's no way, in my understanding, for anyone to say
"Get me all the messages I've missed in the last 5 hours". If you are specifically following
users you can ping their profiles and retrieve what THEY have said in the last 5 hours, but
you can't inspect what unknown people have said in the last 5 hours, a la a Reddit thread or
Mastodon topic. So you WILL need, one way or another, a way to ask either a server or other
peers on the network to replay portions of the pubsub history upon request.
    IPFS pubsub is currently a research topic, not something usable at large scale. There
may be other technologies that are more robust. Scuttlebutt, maybe? Haven't looked at
zeronet, I'll check it out. How does Mastodon do it? I need to learn how OStatus and
ActivityPub work.

Misc stuff:

    Metadata depends on a user's node. If/when that node vanishes, their profile vanishes,
and there's no way to actively query the network and say "find me anything that such-and-so
user posted". Not sure if this is a problem or not?
    I think we can do better than JSON. JSON loses a lot of its human-readability and
simplicity when you try to do markup in it, and if we embed HTML or such into the body of
the post then suddenly we have to deal with two formats instead of one. I lean towards
having "post", "document", "profile", etc all basically be subsets of the same sort of
thing. ...but, as I said, I'm interested in a more general framework than just social media.
I think you can take this same basic idea of "hash-linked messages" and have it able to do
everything: email, chat/IM, mailing lists, forums, usenet, microblogging like Mastodon, etc.

@matthiasbeyer
Member
matthiasbeyer commented on Dec 3, 2017

Sorry for taking so long to type out an answer, I had to think for a bit.

    I think the design looks like git because they're both essentially a Merkle DAG

Yes, that's where I got inspired, of course.

    I like the idea of not having IPFS as a single source of truth, but I don't think that
each client needs its own "view" of the network. I think that permitting different
transports or content-addressed namespaces is perfectly reasonable; one idea I had was being
able to "hyperlink" to a book by referring to its ISBN, or to a scientific paper by its DOI
index number. These uniquely identify a particular resource; how to retrieve the resource is
more or less up to the client.

I'm not quite sure whether I understand that paragraph. My idea would be to keep everything
in IPFS, so there wouldn't be need to refer to something by a ISBN for example. Not only
gets lookup impossible with (for example) ISBN but outright impossible. With having IPFS
hashes, one simply has to fetch the content addressed by the hash.

In detail:

    I like the idea of not having IPFS as a single source of truth, but I don't think that
each client needs its own "view" of the network.

Not sure what you refer to.

    I think that permitting different transports

Also not sure what you mean,... but IPFS works over different transports as far as I know.
IP is only the only one implemented right now, AFAIU.

    one idea I had was being able to "hyperlink" to a book by referring to its ISBN, or to a
scientific paper by its DOI index number. These uniquely identify a particular resource; how
to retrieve the resource is more or less up to the client.

This is almost impossible, I'd say. Or at least it is in the complexity space of "Hey, find
4b9c8f57f87e54f9e1e799d029c6747aee2f74ec - but I won't tell you what it is or where to
search".

    IMO IPNS is not terribly useful for quickly-updating dynamic data

This is right.

    IPNS hashes are not very human-readable. Though this can be easily solved by having a
display name in the profile info, I guess.

Right, though as you say, not a problem because users would never actually see them.

    IPNS has what I call The Private Key Problem: Handling private keys, backing them up,
restoring them, re-creating them when they are lost in hardware failures, etc. is all
generally horrible.

That is, as far as I can see, only a problem implementation wise on the IPNS side. Time to
request features upstream if we run into problems with that.

    PubSub

Not a solution because there's no playback of old messages and only connected peers get
informed about something. Can be used for short-term notification as in "hey, here's new
content"-style messages

    But there's no way, in my understanding, for anyone to say "Get me all the messages I've
missed in the last 5 hours".

Yep, that's what I see, too.

    If you are specifically following users you can ping their profiles and retrieve what
THEY have said in the last 5 hours, but you can't inspect what unknown people have said in
the last 5 hours, a la a Reddit thread or Mastodon topic. So you WILL need, one way or
another, a way to ask either a server or other peers on the network to replay portions of
the pubsub history upon request.

Yes, that's what I thought of. A client would simply store a list of profiles it knows and
connected peers can ask the instance for a list of known "profiles" (literally IPNS hashes
in my idea-space). A client can then fetch each of these "profiles" and continue to ask them
about profiles they know... thus ultimatively "traverse" the whole network (so to speak).

That whole process could be implemented using PubSub.

    There may be other technologies that are more robust. Scuttlebutt, maybe? Haven't looked
at zeronet, I'll check it out.

I am not sure how Scuttlebutt and zeronet work under the hood, but from what I saw so far:
They implement a content-addressable "thing" themselves (which is also why I want to use
IPFS - don't reinvent the wheel). How they do propagation of "new content here"-messages,...
don't know. Feel free to share your findings.

    How does Mastodon do it? I need to learn how OStatus and ActivityPub work.

Mastodon uses ActivityPub but (as far as I know from wikipedia) not since the beginning. I
assume they invented their own wheel. I would use known standards as much as possible,
though.

ActivityPub seems to be a good idea at first glance, though I guess it is not because it is
designed for federated use... which is what I try to avoid! Every node in the network should
be equal, no need for special federated instances!

On the misc stuff:

    Metadata depends on a user's node. If/when that node vanishes, their profile vanishes,
and there's no way to actively query the network and say "find me anything that such-and-so
user posted". Not sure if this is a problem or not?

I'd consider that a feature.

If you know hashes of content (like "there was this article some time ago, I want to read
that again, so I fetch hash 2345tz") that'd simply work because IPFS (as long as nodes are
online which have the content).

As metadata is also stored in IPFS, so others can cache it, that is not a problem. If nobody
caches anymore, content is gone forever. It's a simple "care or forget" process...

    I think we can do better than JSON.

I think we don't have to, here's why:

    JSON loses a lot of its human-readability and simplicity when you try to do markup in
it, and if we embed HTML or such into the body

But when would you? The JSON would only be there to host metadata and refering to actual
content. HTML, Markdown, .epub, .pdf, .mov, .mp3, .avi ... everything would be in IPFS and
only be refered to from a JSON DOM by its IPFS hash.

The JSON would only contain semantics and meta-information about how everything ties
together.
The example from the diff (see "Content JSON format") is (in my eyes) a complete one.

    I'm interested in a more general framework than just social media.

And I see that and think that with a more general framework, we could build a (lets say for
the sake of simplicity) content-publishing platform which could also be seen/used as a
social network.

    I think you can take this same basic idea of "hash-linked messages" and have it able to
do everything: email, chat/IM, mailing lists, forums, usenet, microblogging like Mastodon,
etc.

... video publishing, audio publishing, (scientific) paper publishing ... well I guess
"content-publishing platform" is not that wrong, right?

I am seeing in which direction this should go for you and I think it is great! +1 I think we
have to get onto the same page as in how this should technically work. But for that, we
should get our requirements straight, because I fear that this might end up in bikeshedding
if we don't have a list of points to discuss in the first place...

I'm not sure where to go from here, though.

BTW: If you know people which might be interested, don't hesitate to ping them and bring
them into the discussion. And if they are from the scuttlebut, zeronet, patchwork or any
other project with similar goals, even better!
matthiasbeyer added some commits on Dec 3, 2017
@matthiasbeyer
Add section on why a profile may contain multiple old-profile-version…
ae7e077
@matthiasbeyer
Add notes on multi-device support
a107a77
@icefoxen
Member
icefoxen commented on Dec 3, 2017

Thanks for such a complete response! Sorry if I came off as overly critical, I was trying to
get all my thoughts down in one place, precisely to try to find the places where we have
different views on the desired approach.

It'll probably take me a little while to make worthy replies, I'll go through and update
this with responses as I think of them.
@icefoxen
Member
icefoxen commented on Dec 4, 2017 •

    I'm not quite sure whether I understand that paragraph. My idea would be to keep
everything in IPFS, so there wouldn't be need to refer to something by a ISBN for example.
Not only gets lookup impossible with (for example) ISBN but outright impossible. With having
IPFS hashes, one simply has to fetch the content addressed by the hash.

Sorry for not being clear... I'm kind of mixing up two things here. The first is the
question of transport: It should be more or less possible to use any sort of
content-addressed storage system to move the actual data around, same way you can load web
pages over FTP or something if you really want to.

The second part is the thought that if you can have multiple different transports, you have
multiple different namespaces, and you might have documents that link to things in different
namespaces. And then I thought, hey, what other content-addressed namespaces exist already?
Turns out there's a couple, like ISBN (okay, not really content-addressed, but still a
unique immutable identifier for a particular document). What if, instead of referring to a
particular book by linking to Amazon, I could just link to an identifier that could be
looked up in multiple ways?

Naturally, it is up to the client to be able to interpret links into those namespaces. (ie,
be able to find and look up entries in an ISBN database.)

I confess, none of this is immediately relevant or useful. But content-addressed systems are
awesome and let you do lots of things that location-addressed systems like the Web don't.

So it's mostly just me getting distracted thinking "wouldn't it be cool if..."

    Right, though as you say, not a problem because users would never actually see them.

True enough, I suppose! There's ways around non-human-readable names, it can just get
awkward to search for them. "icefox@alopex.li" as a unique identifier tells you a lot more
than "QmNgzC6ESkawND9uqMjrDyMwH4UpPsN9wg3HwmPGc2ry4g", and is a lot easier to synthesize and
play with. For instance, in theory you can email "abuse@example.com" or
"hostmaster@example.com" and have it go somewhere meaningful relating to "example.com" no
matter who is actually on the receiving end. It's hard to get properties like that with
purely hash-based identifiers. Still! This is just something to play with.

This sort of demonstrates the difference between content-based ID's (IPFS hashes) and
location-based ID's (url's, email addresses, etc). There's advantages and disadvantages to
both.

    That is, as far as I can see, only a problem implementation wise on the IPNS side. Time
to request features upstream if we run into problems with that.

True enough.

        But there's no way, in my understanding, for anyone to say "Get me all the messages
I've missed in the last 5 hours".

    Yep, that's what I see, too.

This is the sort of functionality that, as far as I can tell, you need a proper database of
some kind or another to have. Whether it's a distributed database like orbitdb, or a web
server running Postgres like a Mastodon instance, or whatever, you need to be able to say
"get me the list of things obeying this predicate". Lots of things come down to this sort of
query.

    Yes, that's what I thought of. A client would simply store a list of profiles it knows
and connected peers can ask the instance for a list of known "profiles" (literally IPNS
hashes in my idea-space). A client can then fetch each of these "profiles" and continue to
ask them about profiles they know... thus ultimatively "traverse" the whole network (so to
speak).

Doing this well, efficiently and consistently seems to be on the road to re-inventing a
distributed database anyway... Both involve traversing and optionally filtering all the data
available on the network. Traversing the reachable part of the network is not super hard,
it's the "efficiently" and "consistently" parts that seem to make life hard. But I am far
from an expert on such things! It would be fun to experiment with though.

    ActivityPub seems to be a good idea at first glance, though I guess it is not because it
is designed for federated use... which is what I try to avoid! Every node in the network
should be equal, no need for special federated instances!

Ah, here we have a fundamental question! Purely peer-to-peer, or client-server? There's
advantages and disadvantages to both. I lean towards a federated type system, because it's
way easier to implement and reason about... I don't actually know very much about
peer-to-peer networks! It also has some logistical benefits for users: it's easier to log
into an existing system and use it than to host your own software, it's usually nicer to
serve stuff from a server a datacenter than from a laptop on a home internet connection,
etc.

That said, if we can make something work in a purely peer-to-peer fashion, that would be
pretty darn cool. Ideally these would converge; for example anyone can host their own
Mastodon node, it's just a pain in the butt to do so. If it was really easy to do so,
federation was mostly automatic, and things were designed to disconnect and reconnect
smoothly and restore state when that happened, then it would look much less client-server
and more peer-to-peer.

So far in my own work I've ended up assuming storage is distributed using IPFS, but there
are "index" systems that store databases containing metadata and such and letting users
search and relate things. (For a social network type thing, these are the systems you would
ask "What has such-and-so person said in the last 5 hours?", and they would be notified of
new posts via pubsub.) Like web or email servers anyone can host their own index, and like
web or email servers they can offer different sorts of functionality; something I want is a
language/API for users to query what functionality they offer and search them in a
consistent fashion that works with any index server. But if we can have a distributed
database that provides the same sort of functionality, that would be better! It may also be
possible to start with a federated system or such and then make it obsolete as development
continues. If we end up working in terms of an API or such for making queries, it may be
possible to translate from client-server to peer-to-peer databases and use the same language
to talk to both.

    But when would you? The JSON would only be there to host metadata and refering to actual
content. HTML, Markdown, .epub, .pdf, .mov, .mp3, .avi ... everything would be in IPFS and
only be refered to from a JSON DOM by its IPFS hash.

This is actually a very good point that I hadn't considered. I've been thinking in terms of
HTML/XML where metadata is basically part of the content... Internal vs. external metadata
is something I will have to think about more. On the one hand it's really nice to have some
metadata (author, publication date, etc) basically always attached to content (a text post).
On the other hand, if you have a video file it can be useful to attach more metadata to it
in an easily readable form, say if you had a system that tagged it with a list of actors.
Both are useful.

"Content-publishing platform" is probably a good way to think of it!

    I think we have to get onto the same page as in how this should technically work. But
for that, we should get our requirements straight, because I fear that this might end up in
bikeshedding if we don't have a list of points to discuss in the first place...

Totally. I think there's a lot of stuff we can easily agree on and a few things where our
vision differs. Having a lot of shared technology will be good for both of us, but we should
also experiment and try things out! I'll be turning a few parts of my WorldDoc notes into
proper RFC's, hopefully soon.
@icefoxen
Member
icefoxen commented on Dec 4, 2017 •

Oh! I just thought of a possible way to handle the transient nature of pubsub notifications.
IF each post from a person has a link to a previous post, then by only getting a single
notification you get an entry point to the user's entire history, if you care to retrieve
it.

A user's profile might be a nice-ish place to keep a reference to the latest post they've
made. You have the problem of IPNS updates being slow, but it's better than nothing.
@matthiasbeyer
Member
matthiasbeyer commented on Dec 4, 2017 •

Let me split up my answer
Transport and Adressing

    The second part is the thought that if you can have multiple different transports, you
have multiple different namespaces, and you might have documents that link to things in
different namespaces. And then I thought, hey, what other content-addressed namespaces exist
already? Turns out there's a couple, like ISBN (okay, not really content-addressed, but
still a unique immutable identifier for a particular document). What if, instead of
referring to a particular book by linking to Amazon, I could just link to an identifier that
could be looked up in multiple ways?

So basically instead of refering via an IPFS hash, one would refer by ipfs://<ipfs hash> or
isbn://<ISBN> (or git, ssb, zeronet hash, or whatever) and the client is now able to resolve
or not. Sounds like a neat idea, at least for adressing the content.
Identification of "Publishers"/"Profiles"

    True enough, I suppose! There's ways around non-human-readable names, it can just get
awkward to search for them. "icefox@alopex.li" as a unique identifier tells you a lot more
than "QmNgzC6ESkawND9uqMjrDyMwH4UpPsN9wg3HwmPGc2ry4g", and is a lot easier to synthesize and
play with. For instance, in theory you can email "abuse@example.com" or
"hostmaster@example.com" and have it go somewhere meaningful relating to "example.com" no
matter who is actually on the receiving end. It's hard to get properties like that with
purely hash-based identifiers. Still! This is just something to play with.

I agree. The problem is that for (lets call them) "natural language names", you need a
"claiming system" and thus a central component how to resolve a hash to a natural language
name... or something like that. It gets complicated pretty fast.

One idea would be that a client may do the resolve. For example: If a profile is known, its
hash is known. The metadata of the profile also lists a name (or multiple) for the profile.
The user may now use the name to refer to the profile (like "Hi, @alex I like your kittens
pictures") and the client automatically resolves this @alex to the hash before the post gets
published (just stick with my thought-process here).
Now, if there are two profiles named alex, the client must decide which to use. For example,
it could tell the user to identify the profile via a click on it (or whatever).

Ultimatively, Hashes must be used for identification. But a client can cover them up for
convenient use...
Querying content

            But there's no way, in my understanding, for anyone to say "Get me all the
messages I've missed in the last 5 hours".

        Yep, that's what I see, too.

    This is the sort of functionality that, as far as I can tell, you need a proper database
of some kind or another to have. Whether it's a distributed database like orbitdb, or a web
server running Postgres like a Mastodon instance, or whatever, you need to be able to say
"get me the list of things obeying this predicate". Lots of things come down to this sort of
query.

I don't think you need a database-like thing. You only need a mechanism to query connected
nodes for public data. For example, a node may provide data on which other nodes it knows
and maybe what profiles it knows. If one could now tell the node "give me all the hashes of
the profiles you know" ... that's pretty much a great way to do querying... and because the
profiles can then be fetched and filtered, we basically have a way to "get all profiles
which have a name 'alex'" or something like this.

One could even implement query-selectors ... like "get all profiles where name=='alex'"...
so less data is transmitted, but more computation is triggered on the remote host. Maybe
this should be optionally, after all.

    Doing this well, efficiently and consistently seems to be on the road to re-inventing a
distributed database anyway... Both involve traversing and optionally filtering all the data
available on the network. Traversing the reachable part of the network is not super hard,
it's the "efficiently" and "consistently" parts that seem to make life hard. But I am far
from an expert on such things! It would be fun to experiment with though.

True. Basically underlines what I've written above. After re-reading my last paragraph
you're right: This looks pretty much like a distributed database.

The "consistently" part seems not to complicated for me ... or I'm misunderstanding
something. The "efficiently" part does, though. Not sure how fast things like orbitdb work.
Federated vs. completely Distributed

    Ah, here we have a fundamental question! Purely peer-to-peer, or client-server? There's
advantages and disadvantages to both.

I'd go for purely p2p. Not because federation is bad, but because distribution is better.
Every node in the network serving content is so much more powerful. Censorship is absolutely
impossible if every node is a full node. Think mastodon: We have about 1k nodes in Mastodon
right now. Bringing such a network down is harder than bringing down a centralized network,
sure. But bringing it down is not impossible: Most of the nodes are ran by private providers
like you and me, using a VPS for $5/mo on $somehoster. Distributing the whole system over
each user would yield 1 million nodes (with mastodon as an example). Nobody can bring down
1m nodes.

Also, one does not need internet connectivity if every node is a full node on the network. I
can use the system while beeing on a roadtrip without internet connection, while beeing on a
ship in the ocean, while climbing mount everest... or simply while the internet connection
at home is broken. Simply because my instance of the software provides everything I need to
use it. And if my neighbor has some way to connect to my node, we can even discuss why the
f*ing internet is broken again because some shitty provider does stupid things...

... I got a bit into a rant here. Sorry for that. I just wanted to point out that pure
distribution has very nice upsides, despite beeing harder to implement right.

That said, I would still provide the possibility to host "federated caching nodes". These
nodes would run the exact same software as the client - only headless and with a slightly
other configuration: For example, profile hashes and maybe profile history would be cached
much longer. The admin of the instance may decide that profile history (as in: metadata of
the profile) is automatically pinned (because that's only a few KB per profile). An admin
may even configure the cache server to cache text posts, images, videos and audio files...
if there's enough storage and if she's willing to provide. All these things can be done by
end-user-clients as well, of course. The whole point with a caching node is the caching and
that it is running headless, though.
Index servers

    So far in my own work I've ended up assuming storage is distributed using IPFS, but
there are "index" systems that store databases containing metadata and such and letting
users search and relate things. (For a social network type thing, these are the systems you
would ask "What has such-and-so person said in the last 5 hours?", and they would be
notified of new posts via pubsub.)

Can you elaborate? I'm not sure what you mean here.

With the "What has such-and-so person said in the last 5 hours?"-question, the node would go
out into the network, fetch the profile history of "such-and-so" and read the last 5 hours
off of it, showing it to the user. The "out into the network"-part would be:

    asking all connected nodes for a list of profiles
    getting the "such-and-so" person from that list (if not in there, ask all connected
nodes for list of connected nodes and start restart process at 1.)
    fetching the profile history of the profile of "such-and-so" (if not found we can abort
the operation here)
    filter the profile history, for each relevant "commit" in the history fetch the data
which was published with the "commit" (here one might ignore video posts or posts with the
tag "dogs") and show things.

    It may also be possible to start with a federated system or such and then make it
obsolete as development continues.

Please note that in my list above I did not distinguish whether we ask a "index server" or a
"user node"! There wouldn't be a difference between the two - at least not for the node
issuing the query! Thus, Index servers would only be there for caching and be literally
"always-online-user-nodes".
The rest...

    Totally. I think there's a lot of stuff we can easily agree on and a few things where
our vision differs. Having a lot of shared technology will be good for both of us, but we
should also experiment and try things out! I'll be turning a few parts of my WorldDoc notes
into proper RFC's, hopefully soon.

Agreed. I am looking forward to your proposals! Also, if you have suggestions on how to
continue on this PR, feel free to share them (or even push commits to this PR for me to
cherry-pick/merge).

    IF each post from a person has a link to a previous post, then by only getting a single
notification you get an entry point to the user's entire history, if you care to retrieve
it.
    A user's profile might be a nice-ish place to keep a reference to the latest post
they've made. You have the problem of IPNS updates being slow, but it's better than nothing.

Yes. I would rather have a link from each post to the profile IPNS name it was published
with and each profile version has a hash pointing to the prior version, but I guess that's
an implementation detail.

profile1 <--- profile2 <--- profile3 <--- profile-IPNS-name
|             |             |               ^ ^ ^
v             v             v               | | |
post1         post2         post3           | | |
|             |             |               | | |
|             |             |               | | |
|             |             +---------------+ | |
|             +-------------------------------+ |
+-----------------------------------------------+

I'd also love to get some IPFS people into the discussion. I guess I will now randomly ping
some of the people from the IPFS project I know by name:

@whyrusleeping
@jbenet

Hope you two don't mind. fearful
@matthiasbeyer
Member
matthiasbeyer commented on Dec 6, 2017 •

Just a quick self-reminder: we should also have a look at the matrix protocol. Maybe this
could be a protocol to use for building the application, if it is not a federation-protocol.

Update: seems like matrix is a federation protocol. So not an option for me.
@icefoxen
Member
icefoxen commented on Dec 6, 2017 •
Identification of "Publishers"/"Profiles"

    The problem is that for (lets call them) "natural language names", you need a "claiming
system" and thus a central component how to resolve a hash to a natural language name... or
something like that. It gets complicated pretty fast.

This seems to be another special case of the "distributed database" problem: everyone must
agree on a namespace and be able to look up a natural language name and turn it into a
content ID or public key or something. The "claiming system" is a question of access
control.

    Ultimatively, Hashes must be used for identification. But a client can cover them up for
convenient use...

This seems to be where RFC 00001 has gone, yes. I am currently happy with the assumption
that an identity is an IPFS object containing a public key and metadata, and links to
supporting data (previous identities, web-of-trust signatures, etc). Human-meaningful and
human-constructable names can be built atop this without changing the underlying guts,
whether it's via some federation system or a fully distributed database.
Querying content

    True. Basically underlines what I've written above. After re-reading my last paragraph
you're right: This looks pretty much like a distributed database.

This is the hard problem. Lots of other problems come down to either querying or updating a
distributed database.

My preference is for saying "use federated databases to get something working, then make it
work with a distributed database", as you've probably noticed by now. ;-) But it occurs to
me that if we have a universal format for communicating these sorts of queries and responses
(ActivityStreams perhaps???), then we might be able to use that same format (or most of it)
regardless of how the backing database works.

    The "consistently" part seems not to complicated for me ... or I'm misunderstanding
something. The "efficiently" part does, though. Not sure how fast things like orbitdb work.

As I understand the problem, "consistently" basically means "everyone on the network sees
the same data without contradictions". A simple example might be status notifications from
multiple devices. My client on Device A goes from "online" to "away" automatically after
timing out. I set my client on Device B from "online" to "do not disturb". Both these
operations are valid, but what order they appear in depends on where you are in the network
and how quickly messages propagate. Which state is the "correct" one?

I can think of a couple ways of resolving this particular conflict, but as far as I
understand this kind of problem is not easy to solve in the general case.
Federated vs. completely Distributed

    Not because federation is bad, but because distribution is better.

I don't agree; I think they're different. Both make different things easy and hard.
Federation makes administration, convenience and communication easy. Distribution makes
resiliance, persistence and replication easy.

    That said, I would still provide the possibility to host "federated caching nodes".

I do like that idea. Even if the network is purely p2p, no reason you can't have persistent
nodes that just sit around and helpfully assist in keeping the network running.
Index servers

    With the "What has such-and-so person said in the last 5 hours?"-question, the node
would go out into the network, fetch the profile history of "such-and-so" and read the last
5 hours off of it, showing it to the user.

Aha, here I'm stumbling over the "content publishing" vs. "social network" conceptual
divide. I hadn't considered the possibility of having messages link to all the previous
messages for a particular user, and having a profile offer an entry point to it.

Whether having everything that you say so easily findable is desirable, well... Easy enough
to get around that problem by having multiple profiles and encrypting sensitive information.
Protocols

    Just a quick self-reminder: we should also have a look at the matrix protocol.

It looks like Matrix assumes federation, yes. API reference here. It also appears to very
much be an "instant messaging protocol" specifically, not a "general communication
protocol".

I've also looked at ActivityPub some and it looks like it doesn't quite assume federation,
but is very social-network-specific in terms of the things it describes, and so maybe not
entirely suitable for my own desires. Building something different off of the underlying
ActivityStreams protocol may or may not be nicer. It might also make interoperating with
Mastodon, FediBook, etc. easier, which could be an advantage. Haven't looked at it in detail
though.

Like many networking problems, whatever we choose it will basically come down to defining an
RPC system. JSON-RPC might be a nice basis for that as well.

Secure Scuttlebutt is something that gets mentioned a lot, but I have no idea how it works
or how well it works... But the overview is incredibly interesting to read, whether it's
what you want or not. Start with https://www.scuttlebutt.nz/principles.html and just go
through...
@matthiasbeyer matthiasbeyer referenced this pull request on Dec 6, 2017
Open
IPFS as storage? #454




Discussion on RFC 00001:

 RFC 00001: A simple decentralized public key exchange system #3
Open
icefoxen wants to merge 7 commits into liber-project:master from icefoxen:rfc-00001
+230 −0
Conversation 23 Commits 7 Checks 0 Files changed 1
Conversation
Reviewers

@matthiasbeyer matthiasbeyer
Assignees
No one assigned
Labels
None yet
Milestone
No milestone
Notifications

You’re receiving notifications because you authored the thread.
2 participants
@icefoxen
@matthiasbeyer
Allow edits from maintainers.
@icefoxen
Member
icefoxen commented on Dec 4, 2017 •

Still WIP, but I'd like feedback on the basic idea. It is a very simple sketch of an idea
that does not try to do fancy things... yet.

Rendered
icefoxen added some commits on Dec 4, 2017
@icefoxen
Merge pull request #1 from liber-project/master
60421b5
@icefoxen
Proposal for a simple decentralized public key exchange system
7b86d14
@icefoxen
As soon as I rough this out I think of an entirely new and different …
d79f7c0
@icefoxen
Breakthrough!
29af7d4
@matthiasbeyer
matthiasbeyer requested changes on Dec 4, 2017

The basic fundamental problem I see (and I discussed this with @jonashaag just two days ago)
is that initial trust.

If I create a profile right now, create a pub/priv keypair with it and publish the public
key to the world, nobody has to trust this key (and shouldn't).

So, it would be beneficial if I could get others to sign my key. A web of trust is needed.

So, others must be alllowed to publish signatures for my public key and there must be a way
for me to attach the signatures to my key (for re-announcing my newly signed key to the
network).

That shouldn't be to complicated either, but it is something to think about.
rfcs/00001-decentralized-identity.md
rfcs/00001-decentralized-identity.md
rfcs/00001-decentralized-identity.md
rfcs/00001-decentralized-identity.md
rfcs/00001-decentralized-identity.md
rfcs/00001-decentralized-identity.md
rfcs/00001-decentralized-identity.md
rfcs/00001-decentralized-identity.md
rfcs/00001-decentralized-identity.md
rfcs/00001-decentralized-identity.md
icefoxen added some commits on Dec 4, 2017
@icefoxen
Updated to fully IPFS-based storage
5c98f36
@icefoxen
Oh, web of trust would be even better in this model
0269371
@icefoxen
Member
icefoxen commented on Dec 4, 2017

I've rethought this idea in the middle of publishing it and made significant changes!
@matthiasbeyer
Member
matthiasbeyer commented on Dec 4, 2017

Ah, ... I wondered why all my annotations are gone. Please re-check all annotations I made,
just in case. Then report back and I'll re-read!
@icefoxen
Member
icefoxen commented on Dec 4, 2017

Yeah, sorry about that! I sort of had a realization while thinking about what you had said
about using IPNS for user profiles and realized you could cut whole inconvenient chunks out
of this and still have it work well, and operate as the basis for more powerful systems.
@icefoxen
Minor editing, typo fixes, and corrections.
956f5b2
@icefoxen
Member
icefoxen commented on Dec 4, 2017 •

    If I create a profile right now, create a pub/priv keypair with it and publish the
public key to the world, nobody has to trust this key (and shouldn't).
    So, it would be beneficial if I could get others to sign my key. A web of trust is
needed.

...yes and no, sort of. Or rather, a web of trust is needed for some things but not others.

As I understand it, and you plainly know more about GPG-y things than I do, the web of trust
is a way of assuring "X is actually who they say they are, because I've confirmed it". The
more interconnected it is, the better the assurance is, and it becomes harder to someone to
pretend to be someone they're not(?). However, for this, I'm only concerned that a message
provably comes from a particular identity. It doesn't matter what that identity is or how it
relates to anything except itself. As this stands now, this is not interested in proving
that joe#QmFoo is Joe Example, junior technical salescritter at Widgets, Inc. It just proves
that, basically, all messages signed with a particular identity actually come from the same
identity and none of them are forged or spoofed.

I suppose the web of trust is a mechanism for resilience, so that WHEN a key does get lost,
you can claim that joe#QmBar is the same as joe#QmFoo even without the key to prove it? I
fear I really don't understand enough about it.

Anyway, sorry for changing all this out from under you! I THINK I am done making drastic
changes for now, and I like how it feels a lot better than before. It's simpler, entirely
distributed, more convenient, and if desired one can easily build the more-centralized ID
system atop it as the original draft described... and still have it entirely interoperable
with the distributed system.
@matthiasbeyer matthiasbeyer changed the title from RFC-00001: A simple decentralized public
key exchange system to RFC 00001: A simple decentralized public key exchange system on Dec
6, 2017
@icefoxen
Member
icefoxen commented on Jan 1

https://w3c-ccg.github.io/did-spec/ may be quite interesting!
Merge state

Add more commits by pushing to the rfc-00001 branch on icefoxen/rfcs.

