---
layout: recipes-doc
title: Transient data
version: 1.0.0-beta-1
---

## Background

Some recipes store transient data in a portion of the Fluo table.  Transient
data is data that's continually being added and deleted.  Also these transient
data ranges contain no long term data.  The way Fluo works, when data is
deleted a delete marker is inserted but the data is actually still there.  Over
time these transient ranges of the table will have a lot more delete markers
than actual data if nothing is done.  If nothing is done, then processing
transient data will get increasingly slower over time.

These deleted markers can be cleaned up by forcing Accumulo to compact the
Fluo table, which will run Fluo's garbage collection iterator. However,
compacting the entire table to clean up these ranges within a table is
overkill. Alternatively,  Accumulo supports compacting ranges of a table.   So
a good solution to the delete marker problem is to periodically compact just
the transient ranges. 

Fluo Recipes provides helper code to deal with transient data ranges in a
standard way.

## Registering Transient Ranges

Recipes like [Export Queue](../export-queue/) will automatically register
transient ranges when configured.  If you would like to register your own
transient ranges, use [TransientRegistry][1].  Below is a simple example of
using this.

```java
FluoConfiguration fluoConfig = ...;
TransientRegistry transientRegistry = new TransientRegistry(fluoConfig.getAppConfiguration());
transientRegistry.addTransientRange(new RowRange(startRow, endRow));

//Initialize Fluo using fluoConfig. This will store the registered ranges in
//zookeeper making them available on any node later.
```

## Compacting Transient Ranges

Although you may never need to register transient ranges directly, you will
need to periodically compact transient ranges if using a recipe that registers
them.  Using [TableOperations][2] this can be done with one line of Java code
like the following.

```java
FluoConfiguration fluoConfig = ...;
TableOperations.compactTransient(fluoConfig);
```

Fluo recipes provides and easy way to call `compactTransient()` from the
command line using the `fluo exec` command as follows:

```
fluo exec <app name> io.fluo.recipes.accumulo.cmds.CompactTransient [<interval> [<count>]]
```

If no arguments are specified the command will call `compactTransient()` once.
If only `<interval>` is specified, the command will loop forever calling
`compactTransient()` sleeping `<interval>` seconds between calls.  If `<count>`
is additionally specified then the command will only loop `<count>` times.

[1]: {{ site.old_api_static }}/fluo-recipes-core/1.0.0-beta-1/io/fluo/recipes/common/TransientRegistry.html
[2]: {{ site.old_api_static }}/fluo-recipes-accumulo/1.0.0-beta-1/io/fluo/recipes/accumulo/ops/TableOperations.html
