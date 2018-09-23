# WorldDoc

Heyyyy, let's replace the Web with something cooler!

This repository is for various documentation and ideas and stuff.  This is a big rambling
pile of ideas and thoughts so is not complete.  The basic idea is that we can build
something better than the World Wide Web atop a decentralized block store, and then use that
to transfer documents, applications and hold identity tokens as well as probably lots of
other stuff.

Originally the idea was to use IPFS, but IPFS is kind of a pain to interoperate with so now
the thoughts seem to be wandering elsewhere.

Inspirations:  IPFS, DAT, BitTorrent, Secure Scuttlebutt

# Overview

 * WorldDoc.md: More-or-less solid specification
 * WorldDocNotes.md: Notes and thoughts.
 * WorldApp.md: Random thoughts about application systems and such
 * Original inspiration: <https://wiki.alopex.li/SemanticWebThoughts>
 * liber-project/: Notes from an older project along the same line.

Theses: 

 * WorldDat: A content-addressed distributed block store is a great way to distribute stuff.  
 * WorldDoc: The current web technologies (HTML+JS) are shitty partially 'cause they're
   being torn in two directions: distribute and store documents, and distribute and store
   sandboxed remote applications.  Separating the two might make life a lot easier.
   Honestly just using DocBook for the document section may be nice.  I'm leaning towards an
   easily-machine-readable format that can be compiled from a variety of source formats,
   including HTML and MarkDown.
 * WorldId: Content-addressed distributed block store makes PKI identity distribution
   trivial.

# Subprojects

 * <https://github.com/icefoxen/WorldDat>: Experimental replacement for IPFS
 * <https://github.com/icefoxen/WorldDocCode>: Playing with WorldDoc ideas.
