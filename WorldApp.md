# WorldApp

Some loose thoughts about dealing with the Distributed Application Platform part of the Fixing The Web problem.

To make a distributed application system as powerful as the Web, you need: A client-side scripting system, an RPC system, and a
display system.  Currently these are Javascript, disorganized JSON/REST/HTTP mishmash, and HTML+CSS.

Next, they SHALL be: wasm, some nicer and more sensible RPC mechanism, and...  ???

If I knew the details it'd already exist, I suppose.  You also need an addressing and transport system to tie it all together;
HTTP(2) is p. okay at this.
