---
layout: fluo-doc
title:  Contributing to Fluo
version: 1.0.0-beta-2
---

Building Fluo
-------------

If you have [Git], [Maven], and [Java][java] (version 7+) installed, run these commands
to build Fluo:

    git clone https://github.com/fluo-io/fluo.git
    cd fluo
    mvn package

Testing Fluo
------------

Fluo has a test suite that consists of the following:

* Units tests which are run by `mvn package`
* Integration tests which are run using `mvn verify`.  These tests start
a local Fluo instance (called MiniFluo) and run against it.
* A [Stress test][Stress] application designed to run on a cluster.

[Git]: http://git-scm.com/
[java]: https://www.oracle.com/java/index.html
[Maven]: http://maven.apache.org/
[Stress]: https://github.com/fluo-io/fluo-stress
