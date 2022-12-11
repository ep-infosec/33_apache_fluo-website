---
layout: fluo-doc
title:  Fluo 1.1.0-incubating Documentation
version:  1.1.0-incubating
---
**Apache Fluo lets users make incremental updates to large data sets stored in Apache Accumulo.**

[Apache Fluo][fluo] is an open source implementation of [Percolator][percolator] (which populates
Google's search index) for [Apache Accumulo][accumulo]. Fluo makes it possible to update the results
of a large-scale computation, index, or analytic as new data is discovered. Check out the Fluo
[project website][fluo] for news and general information.

## Getting Started

* Take the [Fluo Tour][tour] if you are completely new to Fluo.
* Read the [install instructions][install] to install Fluo and start a Fluo application in YARN on a
  cluster where Accumulo, Hadoop & Zookeeper are running. If you need help setting up these
  dependencies, see the [related projects page][related] for external projects that may help.

## Applications

Below are helpful resources for Fluo application developers:

*  [Instructions][apps] for creating Fluo applications
*  [Fluo API][api] javadocs
*  [Fluo Recipes][recipes] is a project that provides common code for Fluo application developers
   implemented using the Fluo API.

## Implementation

*  [Architecture] - Overview of Fluo's architecture
*  [Contributing] - Documentation for developers who want to contribute to Fluo
*  [Metrics] - Fluo metrics are visible via JMX by default but can be configured to send to Graphite
   or Ganglia

**Find documentation for all Fluo releases in the [archive](/docs/)**.

[fluo]: https://fluo.apache.org/
[related]: https://fluo.apache.org/related-projects/
[tour]: https://fluo.apache.org/tour/
[accumulo]: https://accumulo.apache.org
[percolator]: https://research.google.com/pubs/pub36726.html
[install]: /docs/fluo/1.1.0-incubating/install/
[apps]: /docs/fluo/1.1.0-incubating/applications/
[api]: https://javadoc.io/doc/org.apache.fluo/fluo-api/1.1.0-incubating
[recipes]: https://github.com/apache/incubator-fluo-recipes
[Metrics]: /docs/fluo/1.1.0-incubating/metrics/
[Contributing]: /docs/fluo/1.1.0-incubating/contributing/
[Architecture]: /docs/fluo/1.1.0-incubating/architecture/
[li]: http://img.shields.io/badge/license-ASL-blue.svg
[ll]: https://github.com/apache/incubator-fluo/blob/main/LICENSE
[mi]: https://maven-badges.herokuapp.com/maven-central/org.apache.fluo/fluo-api/badge.svg
[ml]: https://maven-badges.herokuapp.com/maven-central/org.apache.fluo/fluo-api/
[ji]: https://javadoc-emblem.rhcloud.com/doc/org.apache.fluo/fluo-api/badge.svg
[jl]: http://www.javadoc.io/doc/org.apache.fluo/fluo-api
[logo]: contrib/fluo-logo.png
