---
layout: fluo-doc
title:  Contributing to Fluo
version:  1.1.0-incubating
---
## Building Fluo

If you have [Git], [Maven], and [Java][java] (version 8+) installed, run these commands to build
Fluo:

    git clone https://github.com/apache/incubator-fluo.git
    cd fluo
    mvn package

## Testing Fluo

Fluo has a test suite that consists of the following:

*  Units tests which are run by `mvn package`
*  Integration tests which are run using `mvn verify`. These tests start a local Fluo instance
   (called MiniFluo) and run against it.

## See Also

* [How to Contribute][contribute] on Apache Fluo project website

[Git]: http://git-scm.com/
[java]: http://openjdk.java.net/
[Maven]: http://maven.apache.org/
[contribute]: https://fluo.apache.org/how-to-contribute/
