# WorldApp

Some loose thoughts about dealing with the Distributed Application Platform part of the Fixing The Web problem.

To make a distributed application system as powerful as the Web, you need: A client-side scripting system, an RPC system, and a
display system.  Currently these are Javascript, disorganized JSON/REST/HTTP mishmash, and HTML+CSS.

Next, they SHALL be: wasm, some nicer and more sensible RPC mechanism, and...  ???

If I knew the details it'd already exist, I suppose.  You also need an addressing and transport system to tie it all together; HTTP(2) is p. okay at this.

A general-purpose mid-level display format seems to be display lists in general, which are basically geometry, plus some useful special-case things such as raster images and text that get fed fairly easily into a GPU.  Servo, the experimental Rust web browser renderer, is an example of a display-list based renderer.  However, I really don't have the domain knowledge to make a good one of these.