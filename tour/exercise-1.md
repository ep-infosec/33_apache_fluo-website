---
title: Word counts for unique documents exercise
---

This exercise gives you an opportunity to use everything you have learned so
far to attempt writing a simple Fluo application.  A bare minimum of code,
along with a conceptual sketch of a solution, is provided to get you started.
After completing this exercise, consider tweeting a link to your solution to
[#apachefluotour][aft].  If you have time, try to complete the exercise before
looking at the solutions.

The application should compute word counts for unique documents. This
application should do the following.

 * Deduplicate content based on hash
 * Count how many URIs reference content
 * For the unique words in content, update global word counts.
 * When new content is added increment the global counts.
 * When content is no longer referenced by any URIs, decrement the global word counts and delete
   that content.
 * Partition different types of data using row prefixes.  Use *u:* for URIs, use *d:* for document
   content, and use *w:* for word counts.

## Part 1 : Loading data.

The class below is a simple POJO for documents.

```java
package ft;

import com.google.common.hash.Hashing;

public class Document {
  public final String uri;
  public final String content;

  public Document(String uri, String content) {
    this.uri = uri;
    this.content = content;
  }

  public String hash() {
    //use short prefix of hash for example
    return Hashing.sha1().hashString(content).toString().substring(0, 7);
  }
}
```

The following code loads documents into Fluo.  It should do the following :

 * Keep track of the current hash associated with a URI.
 * Deduplicate content based on hash
 * Reference count how many URIs point to content.  Track this information in a column named
   *doc:refc* with a row based on the hash.
 * Track the status of whether content is referenced or unreferenced in a column named *doc:refs*.
   Note *refs* is short for reference status.   When the reference count for content is 0 this
   columns value should be *unreferenced*.  When the reference count is greater than 0, the
   *doc:refs* columns value should be *referenced*.  In a later example, an Observer will watch this
   column.
 * Track the content associated with a hash using the *doc:content* column.

Some of this is implemented below, but not all. The parts that are not done have TODOs.

```java
package ft;

import org.apache.fluo.api.client.Loader;
import org.apache.fluo.api.client.TransactionBase;
import org.apache.fluo.api.data.Column;

public class DocLoader implements Loader {

  private final Document doc;

  public static final Column HASH_COL = new Column("uri", "hash");
  public static final Column REF_COUNT_COL = new Column("doc", "refc");
  public static final Column REF_STATUS_COL = new Column("doc", "refs");
  public static final Column CONTENT_COL = new Column("doc", "content");

  public DocLoader(Document doc) {
    this.doc = doc;
  }

  @Override
  public void load(TransactionBase tx, Context context) throws Exception {
    String newHash = doc.hash();
    String oldHash = tx.gets("u:" + doc.uri, HASH_COL);

    // TODO check if uri already has the same content hash.  If so, then nothing to do.

    // TODO set the new hash associated with the URI

    if (oldHash != null) {
      // TODO decrement the reference count at row "d:"+oldHash
      // TODO set REF_STATUS_COL to "unreferenced" when the reference count goes from 1 to 0. Do
      // this for row "d:"+oldHash
    }

    // TODO increment the reference count for the newHash content.
    // TODO add the new content when the reference count does not exists
    // TODO set REF_STATUS_COL to "referenced" when the reference count for the new content goes
    // from 0 to 1.  Do this for row "d:"+newHash
  }
}
```

Add the following to the ft.Main class.

```java
  // some test data
  private static Document[] docs1 = new Document[] {
      new Document("http://news.com/a23",
          "Jebediah orbits Mun for 35 days.  No power, forgot solar panels."),
      new Document("http://news.com/a24",
          "Bill plans to rescue Jebediah after taking tourist to Minimus.")};

  private static Document[] docs2 = new Document[] {new Document("http://oldnews.com/a23",
      "Jebediah orbits Mun for 35 days.  No power, forgot solar panels.")};

  private static Document[] docs3 = new Document[] {
      new Document("http://news.com/a23",
          "Jebediah orbits Mun for 38 days.  No power, forgot solar panels."),
      new Document("http://news.com/a24",
          "Crisis at KSC.  Tourist stuck at Minimus.  Bill forgot solar panels.")};

  /**
   * Utility method for loading documents and printing out Fluo table after load completes.
   */
  private static void loadAndPrint(MiniFluo mini, FluoClient client, Document[] docs) {

    try (LoaderExecutor loaderExecutor = client.newLoaderExecutor()) {
      for (Document document : docs) {
        loaderExecutor.execute(new DocLoader(document));
      }
    } // this will close loaderExecutor and wait for all load transactions to complete

    //This line is not needed in this step of the exercise.  However the next step will need this
    //line.
    mini.waitForObservers();

    FluoITHelper.printFluoTable(client);
    System.out.println();
  }

  private static void exercise(MiniFluo mini, FluoClient client) {
    loadAndPrint(mini, client, docs1);
    loadAndPrint(mini, client, docs2);
    loadAndPrint(mini, client, docs3);
  }
```

Once the TODOs in the DocLoader class are implemented, running Main should print out the following.

```
== fluo start ==
d:a6c4d1f doc content	Jebediah orbits Mun for 35 days.  No power, forgot solar panels.
d:a6c4d1f doc refc	1
d:a6c4d1f doc refs	referenced
d:cf8ddc0 doc content	Bill plans to rescue Jebediah after taking tourist to Minimus.
d:cf8ddc0 doc refc	1
d:cf8ddc0 doc refs	referenced
u:http://news.com/a23 uri hash	a6c4d1f
u:http://news.com/a24 uri hash	cf8ddc0
=== fluo end ===

== fluo start ==
d:a6c4d1f doc content	Jebediah orbits Mun for 35 days.  No power, forgot solar panels.
d:a6c4d1f doc refc	2
d:a6c4d1f doc refs	referenced
d:cf8ddc0 doc content	Bill plans to rescue Jebediah after taking tourist to Minimus.
d:cf8ddc0 doc refc	1
d:cf8ddc0 doc refs	referenced
u:http://news.com/a23 uri hash	a6c4d1f
u:http://news.com/a24 uri hash	cf8ddc0
u:http://oldnews.com/a23 uri hash	a6c4d1f
=== fluo end ===

== fluo start ==
d:2732ebc doc content	Crisis at KSC.  Tourist stuck at Minimus.  Bill forgot solar panels.
d:2732ebc doc refc	1
d:2732ebc doc refs	referenced
d:6658252 doc content	Jebediah orbits Mun for 38 days.  No power, forgot solar panels.
d:6658252 doc refc	1
d:6658252 doc refs	referenced
d:a6c4d1f doc content	Jebediah orbits Mun for 35 days.  No power, forgot solar panels.
d:a6c4d1f doc refc	1
d:a6c4d1f doc refs	referenced
d:cf8ddc0 doc content	Bill plans to rescue Jebediah after taking tourist to Minimus.
d:cf8ddc0 doc refc	0
d:cf8ddc0 doc refs	unreferenced
u:http://news.com/a23 uri hash	6658252
u:http://news.com/a24 uri hash	2732ebc
u:http://oldnews.com/a23 uri hash	a6c4d1f
=== fluo end ===
```

## Part 2 : Computing word counts.

Now that you have data loading, create an observer that watches the reference
status column (*doc:refs*).  This observer should take the following actions 
based on the values of the columns *doc:refs* and *doc:processed*.  

| doc:refs     | doc:processed       | Action
| ------------ | ------------------- | ----------
| referenced   | false               | Increment word counts and set *doc:processed* to *true*
| referenced   | true                | None
| unreferenced | false               | Delete content
| unreferenced | true                | Delete content and decrement word counts

Make sure you handle the following scenario correctly.

 * content A becomes referenced
 * content A becomes unreferenced
 * an observer runs on content A

In this situation, word counts were never incremented for content A so there is no need to decrement
the word counts.  One way to handle this is to have a column that tracks if word counts were
incremented.

Below is a skeleton for an observer to compute word counts.

```java
package ft;

import java.util.*;

import org.apache.fluo.api.client.TransactionBase;
import org.apache.fluo.api.data.*;
import org.apache.fluo.api.observer.StringObserver;

public class ContentObserver implements StringObserver {

  public static final Column PROCESSED_COL = new Column("doc", "processed");
  public static final Column WORD_COUNT = new Column("word","docCount");

  /**
   * Utility method to tokenize the content of a document into unique words.
   */
  private List<String> tokenize(String content) {
    return Arrays.asList(content.split("[\\W]+"));
  }

  /**
   *  Adds the passed to delta to the values for each word.
   */
  private void adjustCounts(TransactionBase tx, int delta, List<String> words) {
     Set<String> uniqueWords = new HashSet<String>(words);
    // TODO make a single call to get all of the current word counts.  Could use
    //tx.gets(Collection<RowColumn>)

    // TODO for each word, add delta to the current value and set the new value
  }


  @Override
  public void process(TransactionBase tx, String row, Column col) throws Exception {

    Map<Column, String> colVals =
        tx.gets(row, DocLoader.CONTENT_COL, DocLoader.REF_STATUS_COL, PROCESSED_COL);

    String content = colVals.get(DocLoader.CONTENT_COL);
    String status = colVals.get(DocLoader.REF_STATUS_COL);
    String processed = colVals.getOrDefault(PROCESSED_COL, "false");

    // TODO if status is referenced and not already processed then adjustCounts by +1 and set
    // PROCESSED_COL to true

    // TODO if status is unreferenced then delete all columns for content
    // TODO if status is unreferenced and document was processed, then adjust counts by -1
  }
}
```

Something to think about: why observe the reference status column instead of the reference count
column?

When you are ready to run the observer, modify the `preInit()` method in `ft.Main` to configure the
observer as follows.

```java
  public static class TourObserverProvider implements ObserverProvider {
    @Override
    public void provide(Registry obsRegistry, Context ctx) {
      obsRegistry.forColumn(DocLoader.REF_STATUS_COL, NotificationType.STRONG)
          .useObserver(new ContentObserver());;
    }
  }

  private static void preInit(FluoConfiguration fluoConfig) {
    fluoConfig.setObserverProvider(TourObserverProvider.class);
  }
```

After implementing the Observer, the output of the program should look like the following.

```
== fluo start ==
d:a6c4d1f doc content	Jebediah orbits Mun for 35 days.  No power, forgot solar panels.
d:a6c4d1f doc processed	true
d:a6c4d1f doc refc	1
d:a6c4d1f doc refs	referenced
d:cf8ddc0 doc content	Bill plans to rescue Jebediah after taking tourist to Minimus.
d:cf8ddc0 doc processed	true
d:cf8ddc0 doc refc	1
d:cf8ddc0 doc refs	referenced
u:http://news.com/a23 uri hash	a6c4d1f
u:http://news.com/a24 uri hash	cf8ddc0
w:35 word docCount	1
w:Bill word docCount	1
w:Jebediah word docCount	2
w:Minimus word docCount	1
w:Mun word docCount	1
w:No word docCount	1
w:after word docCount	1
w:days word docCount	1
w:for word docCount	1
w:forgot word docCount	1
w:orbits word docCount	1
w:panels word docCount	1
w:plans word docCount	1
w:power word docCount	1
w:rescue word docCount	1
w:solar word docCount	1
w:taking word docCount	1
w:to word docCount	1
w:tourist word docCount	1
=== fluo end ===

== fluo start ==
d:a6c4d1f doc content	Jebediah orbits Mun for 35 days.  No power, forgot solar panels.
d:a6c4d1f doc processed	true
d:a6c4d1f doc refc	2
d:a6c4d1f doc refs	referenced
d:cf8ddc0 doc content	Bill plans to rescue Jebediah after taking tourist to Minimus.
d:cf8ddc0 doc processed	true
d:cf8ddc0 doc refc	1
d:cf8ddc0 doc refs	referenced
u:http://news.com/a23 uri hash	a6c4d1f
u:http://news.com/a24 uri hash	cf8ddc0
u:http://oldnews.com/a23 uri hash	a6c4d1f
w:35 word docCount	1
w:Bill word docCount	1
w:Jebediah word docCount	2
w:Minimus word docCount	1
w:Mun word docCount	1
w:No word docCount	1
w:after word docCount	1
w:days word docCount	1
w:for word docCount	1
w:forgot word docCount	1
w:orbits word docCount	1
w:panels word docCount	1
w:plans word docCount	1
w:power word docCount	1
w:rescue word docCount	1
w:solar word docCount	1
w:taking word docCount	1
w:to word docCount	1
w:tourist word docCount	1
=== fluo end ===

== fluo start ==
d:2732ebc doc content	Crisis at KSC.  Tourist stuck at Minimus.  Bill forgot solar panels.
d:2732ebc doc processed	true
d:2732ebc doc refc	1
d:2732ebc doc refs	referenced
d:6658252 doc content	Jebediah orbits Mun for 38 days.  No power, forgot solar panels.
d:6658252 doc processed	true
d:6658252 doc refc	1
d:6658252 doc refs	referenced
d:a6c4d1f doc content	Jebediah orbits Mun for 35 days.  No power, forgot solar panels.
d:a6c4d1f doc processed	true
d:a6c4d1f doc refc	1
d:a6c4d1f doc refs	referenced
u:http://news.com/a23 uri hash	6658252
u:http://news.com/a24 uri hash	2732ebc
u:http://oldnews.com/a23 uri hash	a6c4d1f
w:35 word docCount	1
w:38 word docCount	1
w:Bill word docCount	1
w:Crisis word docCount	1
w:Jebediah word docCount	2
w:KSC word docCount	1
w:Minimus word docCount	1
w:Mun word docCount	2
w:No word docCount	2
w:Tourist word docCount	1
w:at word docCount	1
w:days word docCount	2
w:for word docCount	2
w:forgot word docCount	3
w:orbits word docCount	2
w:panels word docCount	3
w:power word docCount	2
w:solar word docCount	3
w:stuck word docCount	1
=== fluo end ===
```

## Part 3 : Using Fluo Recipes

The suggested method of computing word counts above is prone to transaction collisions. One way to avoid
collisions is to use a CombineQueue provided by Fluo Recipes.  This will queue
updates for words and notify another observer to process the queued updates.  The updates are queued
in a way that will not cause collisions.  The CombineQueue has its own Observer which will call two functions
you provide.  One function combines updates for a key and the other consumes changes to values.

To try using a CombineQueue, first add the following class.  This class handles changes to word
counts by updating an inverted index of word counts.

```java
package ft;

import org.apache.fluo.api.client.TransactionBase;
import org.apache.fluo.recipes.core.combine.ChangeObserver;

import static ft.ContentObserver.WORD_COUNT;

public class WordCountChangeObserver implements ChangeObserver<String, Long> {

  @Override
  public void process(TransactionBase tx, Iterable<Change<String, Long>> changes) {
    // This print shows per bucket processing.
    System.out.println("== begin processing word count changes ==");

    for (Change<String, Long> change : changes) {

      long oldCount = change.getOldValue().orElse(0l);  //previous count for a word
      long newCount = change.getNewValue().orElse(0l);  //new count for a word

      if (change.getOldValue().isPresent()) {
        // delete old count for word from inverted index
        tx.delete(String.format("ic:%06d:%s", oldCount, change.getKey()), WORD_COUNT);
      }

      if (change.getNewValue().isPresent()) {
        // insert new count for word into inverted index
        tx.set(String.format("ic:%06d:%s", newCount, change.getKey()), WORD_COUNT, "");
      }

      // change.getKey() is a word
      System.out.printf("  update %s %d -> %d\n", change.getKey(), oldCount, newCount);
    }

    System.out.println("== end processing word count changes ==");
  }
}
```

Then modify `preInit()` in `Main` to the following.

```java
  public static class TourObserverProvider implements ObserverProvider {
    @Override
    public void provide(Registry obsRegistry, Context ctx) {
      CombineQueue<String, Long> wordCQ = CombineQueue.getInstance("wc", ctx.getAppConfiguration());

      // Pass the word count combine queue to the ContentObserver so it can queue updates 
      // when a documents word counts change.
      obsRegistry.forColumn(DocLoader.REF_STATUS_COL, NotificationType.STRONG)
          .useObserver(new ContentObserver(wordCQ));

      // Register observer to process queued updates, the observer will call the two functions.
      // SummingCombiner is provided by Fluo Recipes.  It sums all updates queued for a key.
      wordCQ.registerObserver(obsRegistry, new SummingCombiner<>(), new WordCountChangeObserver());
    }
  }

  private static void preInit(FluoConfiguration fluoConfig) {
    fluoConfig.setObserverProvider(TourObserverProvider.class);
    CombineQueue.configure("wc").keyType(String.class).valueType(Long.class).buckets(3)
        .save(fluoConfig);
  }
```

Add a constructor to `ContentObserver` and modify `adjustCounts()` to the
following.

```java

  private CombineQueue<String, Long> wordCQ;

  public ContentObserver(CombineQueue<String, Long> wordCQ) {
    this.wordCQ = wordCQ;
  }

  private void adjustCounts(TransactionBase tx, int delta, List<String> words) {
    HashMap<String, Long> wcUpdates = new HashMap<>();
    words.forEach(word -> wcUpdates.put(word, (long) delta));
    wordCQ.addAll(tx, wcUpdates);
  }
```

A CombineQueue  groups key values into buckets for efficiency and processes entire buckets in
a single transaction.  When you run this code, that is why `== begin processing word count changes ==` is seen 
multiple times for each group of documents loaded.

These changes produce two new prefixes in the output of the table scan.  First
the `wc:` prefix is where the CombineQueue stores its data.  By default the CombineQueue uses Kryo for serialization
and therefore the key values with this prefix contain non-ASCII characters.  The utility function
`FluoITHelper.printFluoTable()` escapes non-ASCII characters with `\x<HEX>`.   Second the `ic:`
prefix contains the inverted index of word counts.  This was created simply to show an example of a
follow on action when word counts change.  Ideally this follow on action would have a low chance of
collisions.  Creating the inverted index will not cause collisions because each word is in a single
CombineQueue bucket and each bucket is processed independently.

## Part 4 : Running this example on a real instance.

Everything in the tour so far has used MiniFluo to run code.  The following
instructions show how to run the code in this exercise on a real Fluo
instance.  [Uno] can be used to quickly setup Fluo on a single node.

The following two helper classes will be needed to run on a real instance.

```java
package ft;

import org.apache.fluo.api.config.FluoConfiguration;

/**
 * Generates application config.
 */
public class GenConfig {
  public static void main(String[] args) {
    FluoConfiguration conf = new FluoConfiguration();
    Main.preInit(conf);
    conf.save(System.out);
  }
}
```


```java
package ft;

import java.nio.charset.StandardCharsets;
import java.nio.file.*;
import javax.inject.Inject;
import org.apache.fluo.api.client.*;
import org.apache.fluo.api.config.FluoConfiguration;

/**
 * Loads one or more document passed in on the command line.
 */
public class Load {
  // when run with fluo exec command, the applications configuration will be injected
  @Inject
  private static FluoConfiguration fluoConfig;

  public static void main(String[] args) throws Exception {
    try (FluoClient client = FluoFactory.newClient(fluoConfig);
        LoaderExecutor loaderExecutor = client.newLoaderExecutor()) 
    {
      for (String filename : args) {
        Path path = Paths.get(filename);
        byte[] encoded = Files.readAllBytes(path);
        String docContent = new String(encoded, StandardCharsets.UTF_8);
        String uri = path.toAbsolutePath().normalize().toUri().toString();
        Document doc = new Document(uri, docContent);
        loaderExecutor.execute(new DocLoader(doc));
      }
    }
  }
}
```

The following command will run this application on a Fluo instance, assuming
`$FLUO_HOME` is set and `fluo` is on the path.

```bash
cd <your fluo tour dir>

#create a new Fluo application named wordCount
fluo new wordCount

#populate applications lib directory
mvn clean package
cp target/fluo-tour-0.0.1-SNAPSHOT.jar $FLUO_HOME/apps/wordCount/lib
mvn dependency:copy-dependencies -DincludeArtifactIds="fluo-recipes-core,fluo-recipes-accumulo,fluo-recipes-kryo,kryo,minlog,reflectasm,objenesis" -DoutputDirectory=$FLUO_HOME/apps/wordCount/lib

#add app specific config to properties file that Fluo init will use
fluo exec wordCount ft.GenConfig >> $FLUO_HOME/apps/wordCount/conf/fluo.properties

#initialize and start Fluo application
fluo init wordCount
fluo start wordCount
fluo info wordCount

#load some text files
fluo exec wordCount ft.Load <some filename> <some filename> ...

#wait for all notifications to process
fluo wait wordCount

#scan data in Fluo
fluo scan wordCount
fluo scan wordCount -p ic:

#Could try changing a file and reloading it

#stop Fluo application
fluo stop wordCount
```

[aft]: https://twitter.com/hashtag/apachefluotour
[Uno]: https://github.com/apache/fluo-uno
