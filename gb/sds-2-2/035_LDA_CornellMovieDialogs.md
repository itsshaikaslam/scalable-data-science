[SDS-2.2, Scalable Data Science](https://lamastex.github.io/scalable-data-science/sds/2/2/)
===========================================================================================

Archived YouTube video of this live unedited lab-lecture:

[![Archived YouTube video of this live unedited lab-lecture](http://img.youtube.com/vi/GF-VFR39dIU/0.jpg)](https://www.youtube.com/embed/GF-VFR39dIU?start=0&end=410&autoplay=1) [![Archived YouTube video of this live unedited lab-lecture](http://img.youtube.com/vi/atwM-8fXNQY/0.jpg)](https://www.youtube.com/embed/atwM-8fXNQY?start=0&end=2372&autoplay=1) [![Archived YouTube video of this live unedited lab-lecture](http://img.youtube.com/vi/Uh5J9hqk12I/0.jpg)](https://www.youtube.com/embed/Uh5J9hqk12I?start=0&end=2858&autoplay=1) [![Archived YouTube video of this live unedited lab-lecture](http://img.youtube.com/vi/Se_E00wrdqM/0.jpg)](https://www.youtube.com/embed/Se_E00wrdqM?start=0&end=2242&autoplay=1)

Topic Modeling of Movie Dialogs with Latent Dirichlet Allocation
================================================================

### Let us cluster the conversations from different movies!

This notebook will provide a brief algorithm summary, links for further reading, and an example of how to use LDA for Topic Modeling.

**not tested in Spark 2.2 yet (see 034 notebook for syntactic issues, if any)**

Algorithm Summary
-----------------

-   **Task**: Identify topics from a collection of text documents
-   **Input**: Vectors of word counts
-   **Optimizers**:
    -   EMLDAOptimizer using [Expectation Maximization](https://en.wikipedia.org/wiki/Expectation%E2%80%93maximization_algorithm)
    -   OnlineLDAOptimizer using Iterative Mini-Batch Sampling for [Online Variational Bayes](https://www.cs.princeton.edu/~blei/papers/HoffmanBleiBach2010b.pdf)

Links
-----

-   Spark API docs
    -   Scala: [LDA](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.mllib.clustering.LDA)
    -   Python: [LDA](https://spark.apache.org/docs/latest/api/python/pyspark.mllib.html#pyspark.mllib.clustering.LDA)
-   [MLlib Programming Guide](http://spark.apache.org/docs/latest/mllib-clustering.html#latent-dirichlet-allocation-lda)
-   [ML Feature Extractors & Transformers](http://spark.apache.org/docs/latest/ml-features.html)
-   [Wikipedia: Latent Dirichlet Allocation](https://en.wikipedia.org/wiki/Latent_Dirichlet_allocation)

Readings for LDA
----------------

-   A high-level introduction to the topic from Communications of the ACM
    -   <https://www.cs.princeton.edu/~blei/papers/Blei2012.pdf>
-   A very good high-level humanities introduction to the topic (recommended by Chris Thomson in English Department at UC, Ilam):
    -   <http://journalofdigitalhumanities.org/2-1/topic-modeling-and-digital-humanities-by-david-m-blei/>

Also read the methodological and more formal papers cited in the above links if you want to know more.

Let's get a bird's eye view of LDA from <https://www.cs.princeton.edu/~blei/papers/Blei2012.pdf> next.

-   See pictures (hopefully you read the paper last night!)
-   Algorithm of the generative model (this is unsupervised clustering)
-   For a careful introduction to the topic see Section 27.3 and 27.4 (pages 950-970) pf Murphy's *Machine Learning: A Probabilistic Perspective, MIT Press, 2012*.
-   We will be quite application focussed or applied here!

<p class="htmlSandbox"><iframe 
 src="https://en.wikipedia.org/wiki/Latent_Dirichlet_allocation#Topics_in_LDA"
 width="95%" height="250"
 sandbox>
  <p>
    <a href="http://spark.apache.org/docs/latest/ml-features.html">
      Fallback link for browsers that, unlikely, don't support frames
    </a>
  </p>
</iframe></p>

<p class="htmlSandbox"><iframe 
 src="https://en.wikipedia.org/wiki/Latent_Dirichlet_allocation#Model"
 width="95%" height="600"
 sandbox>
  <p>
    <a href="http://spark.apache.org/docs/latest/ml-features.html">
      Fallback link for browsers that, unlikely, don't support frames
    </a>
  </p>
</iframe></p>

<p class="htmlSandbox"><iframe 
 src="https://en.wikipedia.org/wiki/Latent_Dirichlet_allocation#Mathematical_definition"
 width="95%" height="910"
 sandbox>
  <p>
    <a href="http://spark.apache.org/docs/latest/ml-features.html">
      Fallback link for browsers that, unlikely, don't support frames
    </a>
  </p>
</iframe></p>

Probabilistic Topic Modeling Example
------------------------------------

This is an outline of our Topic Modeling workflow. Feel free to jump to any subtopic to find out more.

-   Step 0. Dataset Review
-   Step 1. Downloading and Loading Data into DBFS
    -   (Step 1. only needs to be done once per shard - see details at the end of the notebook for Step 1.)
-   Step 2. Loading the Data and Data Cleaning
-   Step 3. Text Tokenization
-   Step 4. Remove Stopwords
-   Step 5. Vector of Token Counts
-   Step 6. Create LDA model with Online Variational Bayes
-   Step 7. Review Topics
-   Step 8. Model Tuning - Refilter Stopwords
-   Step 9. Create LDA model with Expectation Maximization
-   Step 10. Visualize Results

Step 0. Dataset Review
----------------------

In this example, we will use the [Cornell Movie Dialogs Corpus](https://people.mpi-sws.org/~cristian/Cornell_Movie-Dialogs_Corpus.html).

Here is the `README.txt`:

------------------------------------------------------------------------

------------------------------------------------------------------------

Cornell Movie-Dialogs Corpus

Distributed together with:

"Chameleons in imagined conversations: A new approach to understanding coordination of linguistic style in dialogs"
Cristian Danescu-Niculescu-Mizil and Lillian Lee
Proceedings of the Workshop on Cognitive Modeling and Computational Linguistics, ACL 2011.

(this paper is included in this zip file)

NOTE: If you have results to report on these corpora, please send email to cristian@cs.cornell.edu or llee@cs.cornell.edu so we can add you to our list of people using this data. Thanks!

Contents of this README:

        A) Brief description
        B) Files description
        C) Details on the collection procedure
        D) Contact

A) Brief description:

This corpus contains a metadata-rich collection of fictional conversations extracted from raw movie scripts:

-   220,579 conversational exchanges between 10,292 pairs of movie characters
-   involves 9,035 characters from 617 movies
-   in total 304,713 utterances
-   movie metadata included:
    - genres
    - release year
    - IMDB rating
    - number of IMDB votes
    - IMDB rating
-   character metadata included:
    - gender (for 3,774 characters)
    - position on movie credits (3,321 characters)

B) Files description:

In all files the field separator is " +++\$+++ "

-   movie*titles*metadata.txt
    - contains information about each movie title
    - fields:
    - movieID,
    - movie title,
    - movie year,
    - IMDB rating,
    - no. IMDB votes,
    - genres in the format \['genre1','genre2',...,'genreN'\]

-   movie*characters*metadata.txt
    - contains information about each movie character
    - fields:
    - characterID
    - character name
    - movieID
    - movie title
    - gender ("?" for unlabeled cases)
    - position in credits ("?" for unlabeled cases)

-   movie\_lines.txt
    - contains the actual text of each utterance
    - fields:
    - lineID
    - characterID (who uttered this phrase)
    - movieID
    - character name
    - text of the utterance

-   movie*conversations.txt
    - the structure of the conversations
    - fields
    - characterID of the first character involved in the conversation
    - characterID of the second character involved in the conversation
    - movieID of the movie in which the conversation occurred
    - list of the utterances that make the conversation, in chronological
    order: \['lineID1','lineID2',...,'lineIDN'\]
    has to be matched with movie*lines.txt to reconstruct the actual content

-   raw*script*urls.txt
    - the urls from which the raw sources were retrieved

C) Details on the collection procedure:

We started from raw publicly available movie scripts (sources acknowledged in
raw*script*urls.txt). In order to collect the metadata necessary for this study
and to distinguish between two script versions of the same movie, we automatically
matched each script with an entry in movie database provided by IMDB (The Internet
Movie Database; data interfaces available at http://www.imdb.com/interfaces). Some
amount of manual correction was also involved. When more than one movie with the same
title was found in IMBD, the match was made with the most popular title
(the one that received most IMDB votes)

After discarding all movies that could not be matched or that had less than 5 IMDB
votes, we were left with 617 unique titles with metadata including genre, release
year, IMDB rating and no. of IMDB votes and cast distribution. We then identified
the pairs of characters that interact and separated their conversations automatically
using simple data processing heuristics. After discarding all pairs that exchanged
less than 5 conversational exchanges there were 10,292 left, exchanging 220,579
conversational exchanges (304,713 utterances). After automatically matching the names
of the 9,035 involved characters to the list of cast distribution, we used the
gender of each interpreting actor to infer the fictional gender of a subset of
3,321 movie characters (we raised the number of gendered 3,774 characters through
manual annotation). Similarly, we collected the end credit position of a subset
of 3,321 characters as a proxy for their status.

D) Contact:

Please email any questions to: cristian@cs.cornell.edu (Cristian Danescu-Niculescu-Mizil)

------------------------------------------------------------------------

------------------------------------------------------------------------

Step 2. Loading the Data and Data Cleaning
------------------------------------------

We have already used the wget command to download the file, and put it in our distributed file system (this process takes about 1 minute). To repeat these steps or to download data from another source follow the steps at the bottom of this worksheet on **Step 1. Downloading and Loading Data into DBFS**.

Let's make sure these files are in dbfs now:

``` scala
// this is where the data resides in dbfs (see below to download it first, if you go to a new shard!)
display(dbutils.fs.ls("dbfs:/datasets/sds/nlp/cornell_movie_dialogs_corpus/")) 
```

| path                                                                                   | name                            | size        |
|----------------------------------------------------------------------------------------|---------------------------------|-------------|
| dbfs:/datasets/sds/nlp/cornell\_movie\_dialogs\_corpus/README.txt                      | README.txt                      | 4181.0      |
| dbfs:/datasets/sds/nlp/cornell\_movie\_dialogs\_corpus/movie\_characters\_metadata.txt | movie\_characters\_metadata.txt | 705695.0    |
| dbfs:/datasets/sds/nlp/cornell\_movie\_dialogs\_corpus/movie\_conversations.txt        | movie\_conversations.txt        | 6760930.0   |
| dbfs:/datasets/sds/nlp/cornell\_movie\_dialogs\_corpus/movie\_lines.txt                | movie\_lines.txt                | 3.4641919e7 |
| dbfs:/datasets/sds/nlp/cornell\_movie\_dialogs\_corpus/movie\_titles\_metadata.txt     | movie\_titles\_metadata.txt     | 67289.0     |
| dbfs:/datasets/sds/nlp/cornell\_movie\_dialogs\_corpus/raw\_script\_urls.txt           | raw\_script\_urls.txt           | 56177.0     |

Conversations Data
------------------

``` scala
// Load text file, leave out file paths, convert all strings to lowercase
val conversationsRaw = sc.textFile("dbfs:/datasets/sds/nlp/cornell_movie_dialogs_corpus/movie_conversations.txt").zipWithIndex()
```

>     conversationsRaw: org.apache.spark.rdd.RDD[(String, Long)] = ZippedWithIndexRDD[50] at zipWithIndex at <console>:33

Review first 5 lines to get a sense for the data format.

``` scala
conversationsRaw.top(5).foreach(println) // the first five Strings in the RDD
```

>     (u999 +++$+++ u1006 +++$+++ m65 +++$+++ ['L227588', 'L227589', 'L227590', 'L227591', 'L227592', 'L227593', 'L227594', 'L227595', 'L227596'],8954)
>     (u998 +++$+++ u1005 +++$+++ m65 +++$+++ ['L228159', 'L228160'],8952)
>     (u998 +++$+++ u1005 +++$+++ m65 +++$+++ ['L228157', 'L228158'],8951)
>     (u998 +++$+++ u1005 +++$+++ m65 +++$+++ ['L228130', 'L228131'],8950)
>     (u998 +++$+++ u1005 +++$+++ m65 +++$+++ ['L228127', 'L228128', 'L228129'],8949)

``` scala
conversationsRaw.count // there are over 83,000 conversations in total
```

>     res0: Long = 83097

``` scala
import scala.util.{Failure, Success}

val regexConversation = """\s*(\w+)\s+(\+{3}\$\+{3})\s*(\w+)\s+(\2)\s*(\w+)\s+(\2)\s*(\[.*\]\s*$)""".r

case class conversationLine(a: String, b: String, c: String, d: String)

val conversationsRaw = sc.textFile("dbfs:/datasets/sds/nlp/cornell_movie_dialogs_corpus/movie_conversations.txt")
 .zipWithIndex()
  .map(x => 
          {
            val id:Long = x._2
            val line = x._1
            val pLine = regexConversation.findFirstMatchIn(line)
                               .map(m => conversationLine(m.group(1), m.group(3), m.group(5), m.group(7))) 
                                  match {
                                    case Some(l) => Success(l)
                                    case None => Failure(new Exception(s"Non matching input: $line"))
                                  }
              (id,pLine)
           }
  )
```

>     import scala.util.{Failure, Success}
>     regexConversation: scala.util.matching.Regex = \s*(\w+)\s+(\+{3}\$\+{3})\s*(\w+)\s+(\2)\s*(\w+)\s+(\2)\s*(\[.*\]\s*$)
>     defined class conversationLine
>     conversationsRaw: org.apache.spark.rdd.RDD[(Long, Product with Serializable with scala.util.Try[conversationLine])] = MapPartitionsRDD[57] at map at <console>:40

``` scala
conversationsRaw.filter(x => x._2.isSuccess).count()
```

>     res1: Long = 83097

``` scala
conversationsRaw.filter(x => x._2.isFailure).count()
```

>     res66: Long = 0

The conversation number and line numbers of each conversation are in one line in `conversationsRaw`.

``` scala
conversationsRaw.filter(x => x._2.isSuccess).take(5).foreach(println)
```

>     (0,Success(conversationLine(u0,u2,m0,['L194', 'L195', 'L196', 'L197'])))
>     (1,Success(conversationLine(u0,u2,m0,['L198', 'L199'])))
>     (2,Success(conversationLine(u0,u2,m0,['L200', 'L201', 'L202', 'L203'])))
>     (3,Success(conversationLine(u0,u2,m0,['L204', 'L205', 'L206'])))
>     (4,Success(conversationLine(u0,u2,m0,['L207', 'L208'])))

Let's create `conversations` that have just the coversation id and line-number with order information.

``` scala
val conversations 
    = conversationsRaw
      .filter(x => x._2.isSuccess)
      .flatMap { 
        case (id,Success(l))  
                  => { val conv = l.d.replace("[","").replace("]","").replace("'","").replace(" ","")
                       val convLinesIndexed = conv.split(",").zipWithIndex
                       convLinesIndexed.map( cLI => (id, cLI._2, cLI._1))
                      }
       }.toDF("conversationID","intraConversationID","lineID")
```

>     conversations: org.apache.spark.sql.DataFrame = [conversationID: bigint, intraConversationID: int, lineID: string]

``` scala
conversations.show(15)
```

>     +--------------+-------------------+------+
>     |conversationID|intraConversationID|lineID|
>     +--------------+-------------------+------+
>     |             0|                  0|  L194|
>     |             0|                  1|  L195|
>     |             0|                  2|  L196|
>     |             0|                  3|  L197|
>     |             1|                  0|  L198|
>     |             1|                  1|  L199|
>     |             2|                  0|  L200|
>     |             2|                  1|  L201|
>     |             2|                  2|  L202|
>     |             2|                  3|  L203|
>     |             3|                  0|  L204|
>     |             3|                  1|  L205|
>     |             3|                  2|  L206|
>     |             4|                  0|  L207|
>     |             4|                  1|  L208|
>     +--------------+-------------------+------+
>     only showing top 15 rows

Movie Titles
------------

``` scala
val moviesMetaDataRaw = sc.textFile("dbfs:/datasets/sds/nlp/cornell_movie_dialogs_corpus/movie_titles_metadata.txt")
moviesMetaDataRaw.top(5).foreach(println)
```

>     m99 +++$+++ indiana jones and the temple of doom +++$+++ 1984 +++$+++ 7.50 +++$+++ 112054 +++$+++ ['action', 'adventure']
>     m98 +++$+++ indiana jones and the last crusade +++$+++ 1989 +++$+++ 8.30 +++$+++ 174947 +++$+++ ['action', 'adventure', 'thriller', 'action', 'adventure', 'fantasy']
>     m97 +++$+++ independence day +++$+++ 1996 +++$+++ 6.60 +++$+++ 151698 +++$+++ ['action', 'adventure', 'sci-fi', 'thriller']
>     m96 +++$+++ invaders from mars +++$+++ 1953 +++$+++ 6.40 +++$+++ 2115 +++$+++ ['horror', 'sci-fi']
>     m95 +++$+++ i am legend +++$+++ 2007 +++$+++ 7.10 +++$+++ 156084 +++$+++ ['drama', 'sci-fi', 'thriller']
>     moviesMetaDataRaw: org.apache.spark.rdd.RDD[String] = dbfs:/datasets/sds/nlp/cornell_movie_dialogs_corpus/movie_titles_metadata.txt MapPartitionsRDD[73] at textFile at <console>:33

``` scala
moviesMetaDataRaw.count() // number of movies
```

>     res4: Long = 617

``` scala
import scala.util.{Failure, Success}

/*  - contains information about each movie title
  - fields:
          - movieID,
          - movie title,
          - movie year,
          - IMDB rating,
          - no. IMDB votes,
          - genres in the format ['genre1','genre2',...,'genreN']
          */
val regexMovieMetaData = """\s*(\w+)\s+(\+{3}\$\+{3})\s*(.+)\s+(\2)\s+(.+)\s+(\2)\s+(.+)\s+(\2)\s+(.+)\s+(\2)\s+(\[.*\]\s*$)""".r

case class lineInMovieMetaData(movieID: String, movieTitle: String, movieYear: String, IMDBRating: String, NumIMDBVotes: String, genres: String)

val moviesMetaDataRaw = sc.textFile("dbfs:/datasets/sds/nlp/cornell_movie_dialogs_corpus/movie_titles_metadata.txt")
  .map(line => 
          {
            val pLine = regexMovieMetaData.findFirstMatchIn(line)
                               .map(m => lineInMovieMetaData(m.group(1), m.group(3), m.group(5), m.group(7), m.group(9), m.group(11))) 
                                  match {
                                    case Some(l) => Success(l)
                                    case None => Failure(new Exception(s"Non matching input: $line"))
                                  }
              pLine
           }
  )
```

>     import scala.util.{Failure, Success}
>     regexMovieMetaData: scala.util.matching.Regex = \s*(\w+)\s+(\+{3}\$\+{3})\s*(.+)\s+(\2)\s+(.+)\s+(\2)\s+(.+)\s+(\2)\s+(.+)\s+(\2)\s+(\[.*\]\s*$)
>     defined class lineInMovieMetaData
>     moviesMetaDataRaw: org.apache.spark.rdd.RDD[Product with Serializable with scala.util.Try[lineInMovieMetaData]] = MapPartitionsRDD[79] at map at <console>:49

``` scala
moviesMetaDataRaw.count
```

>     res5: Long = 617

``` scala
moviesMetaDataRaw.filter(x => x.isSuccess).count()
```

>     res6: Long = 617

``` scala
moviesMetaDataRaw.filter(x => x.isSuccess).take(10).foreach(println)
```

>     Success(lineInMovieMetaData(m0,10 things i hate about you,1999,6.90,62847,['comedy', 'romance']))
>     Success(lineInMovieMetaData(m1,1492: conquest of paradise,1992,6.20,10421,['adventure', 'biography', 'drama', 'history']))
>     Success(lineInMovieMetaData(m2,15 minutes,2001,6.10,25854,['action', 'crime', 'drama', 'thriller']))
>     Success(lineInMovieMetaData(m3,2001: a space odyssey,1968,8.40,163227,['adventure', 'mystery', 'sci-fi']))
>     Success(lineInMovieMetaData(m4,48 hrs.,1982,6.90,22289,['action', 'comedy', 'crime', 'drama', 'thriller']))
>     Success(lineInMovieMetaData(m5,the fifth element,1997,7.50,133756,['action', 'adventure', 'romance', 'sci-fi', 'thriller']))
>     Success(lineInMovieMetaData(m6,8mm,1999,6.30,48212,['crime', 'mystery', 'thriller']))
>     Success(lineInMovieMetaData(m7,a nightmare on elm street 4: the dream master,1988,5.20,13590,['fantasy', 'horror', 'thriller']))
>     Success(lineInMovieMetaData(m8,a nightmare on elm street: the dream child,1989,4.70,11092,['fantasy', 'horror', 'thriller']))
>     Success(lineInMovieMetaData(m9,the atomic submarine,1959,4.90,513,['sci-fi', 'thriller']))

``` scala
//moviesMetaDataRaw.filter(x => x.isFailure).take(10).foreach(println) // to regex refine for casting
```

``` scala
val moviesMetaData 
    = moviesMetaDataRaw
      .filter(x => x.isSuccess)
      .map { case Success(l) => l }
      .toDF().select("movieID","movieTitle","movieYear")
```

>     moviesMetaData: org.apache.spark.sql.DataFrame = [movieID: string, movieTitle: string, movieYear: string]

``` scala
moviesMetaData.show(10,false)
```

>     +-------+---------------------------------------------+---------+
>     |movieID|movieTitle                                   |movieYear|
>     +-------+---------------------------------------------+---------+
>     |m0     |10 things i hate about you                   |1999     |
>     |m1     |1492: conquest of paradise                   |1992     |
>     |m2     |15 minutes                                   |2001     |
>     |m3     |2001: a space odyssey                        |1968     |
>     |m4     |48 hrs.                                      |1982     |
>     |m5     |the fifth element                            |1997     |
>     |m6     |8mm                                          |1999     |
>     |m7     |a nightmare on elm street 4: the dream master|1988     |
>     |m8     |a nightmare on elm street: the dream child   |1989     |
>     |m9     |the atomic submarine                         |1959     |
>     +-------+---------------------------------------------+---------+
>     only showing top 10 rows

Lines Data
----------

``` scala
val linesRaw = sc.textFile("dbfs:/datasets/sds/nlp/cornell_movie_dialogs_corpus/movie_lines.txt")
```

>     linesRaw: org.apache.spark.rdd.RDD[String] = dbfs:/datasets/sds/nlp/cornell_movie_dialogs_corpus/movie_lines.txt MapPartitionsRDD[94] at textFile at <console>:35

``` scala
linesRaw.count() // number of lines making up the conversations
```

>     res9: Long = 304713

Review first 5 lines to get a sense for the data format.

``` scala
linesRaw.top(5).foreach(println)
```

>     L99999 +++$+++ u4166 +++$+++ m278 +++$+++ DULANEY +++$+++ You didn't know about it before that?
>     L99998 +++$+++ u4168 +++$+++ m278 +++$+++ JOANNE +++$+++ To show you this.  It's a letter from that lawyer, Koehler.  He wrote it to me the day after I saw him.  He's the one who told me I could get the money if Miss Lawson went to jail.
>     L99997 +++$+++ u4166 +++$+++ m278 +++$+++ DULANEY +++$+++ Why'd you come here?
>     L99996 +++$+++ u4168 +++$+++ m278 +++$+++ JOANNE +++$+++ I'm gonna go to jail.  I know they're gonna make it look like I did it. They gotta put it on someone.
>     L99995 +++$+++ u4168 +++$+++ m278 +++$+++ JOANNE +++$+++ What do you think I've got?  A gun? Maybe I'm gonna kill you too.  Maybe I'll blow your head off right now.

To see 5 random lines in the `lines.txt` evaluate the following cell.

``` scala
linesRaw.takeSample(false, 5).foreach(println)
```

>     L144853 +++$+++ u4635 +++$+++ m306 +++$+++ QUINN +++$+++ M.J., I'm going to have to borrow Ruben.  The alien-smuggling thing in Chinatown is going down tomorrow night and Jack's kid got hit by a car.  I gotta give Ruben to Nikko.
>     L597838 +++$+++ u8315 +++$+++ m565 +++$+++ AULON +++$+++ Yes, my lord.
>     L107915 +++$+++ u613 +++$+++ m39 +++$+++ BOURNE +++$+++ -- it's always bad and it's never anything but bits and pieces anyway!  You ever think that maybe it's just making it worse? You don't wonder that?
>     L65662 +++$+++ u3864 +++$+++ m256 +++$+++ AUDREY +++$+++ ...Who is this?
>     L124159 +++$+++ u4395 +++$+++ m291 +++$+++ MALE VOICE +++$+++ She's yours.  What are we waiting on?

``` scala
import scala.util.{Failure, Success}

/*  field in line.txt are:
          - lineID
          - characterID (who uttered this phrase)
          - movieID
          - character name
          - text of the utterance
          */
val regexLine = """\s*(\w+)\s+(\+{3}\$\+{3})\s*(\w+)\s+(\2)\s*(\w+)\s+(\2)\s*(.+)\s+(\2)\s*(.*$)""".r

case class lineInMovie(lineID: String, characterID: String, movieID: String, characterName: String, text: String)

val linesRaw = sc.textFile("dbfs:/datasets/sds/nlp/cornell_movie_dialogs_corpus/movie_lines.txt")
  .map(line => 
          {
            val pLine = regexLine.findFirstMatchIn(line)
                               .map(m => lineInMovie(m.group(1), m.group(3), m.group(5), m.group(7), m.group(9))) 
                                  match {
                                    case Some(l) => Success(l)
                                    case None => Failure(new Exception(s"Non matching input: $line"))
                                  }
              pLine
           }
  )
```

>     import scala.util.{Failure, Success}
>     regexLine: scala.util.matching.Regex = \s*(\w+)\s+(\+{3}\$\+{3})\s*(\w+)\s+(\2)\s*(\w+)\s+(\2)\s*(.+)\s+(\2)\s*(.*$)
>     defined class lineInMovie
>     linesRaw: org.apache.spark.rdd.RDD[Product with Serializable with scala.util.Try[lineInMovie]] = MapPartitionsRDD[101] at map at <console>:49

``` scala
linesRaw.filter(x => x.isSuccess).count()
```

>     res11: Long = 304713

``` scala
linesRaw.filter(x => x.isFailure).count()
```

>     res12: Long = 0

``` scala
linesRaw.filter(x => x.isSuccess).take(5).foreach(println)
```

>     Success(lineInMovie(L1045,u0,m0,BIANCA,They do not!))
>     Success(lineInMovie(L1044,u2,m0,CAMERON,They do to!))
>     Success(lineInMovie(L985,u0,m0,BIANCA,I hope so.))
>     Success(lineInMovie(L984,u2,m0,CAMERON,She okay?))
>     Success(lineInMovie(L925,u0,m0,BIANCA,Let's go.))

Let's make a DataFrame out of the successfully parsed line.

``` scala
val lines 
    = linesRaw
      .filter(x => x.isSuccess)
      .map { case Success(l) => l }
      .toDF()
      .join(moviesMetaData, "movieID") // and join it to get movie meta data
```

>     lines: org.apache.spark.sql.DataFrame = [movieID: string, lineID: string, characterID: string, characterName: string, text: string, movieTitle: string, movieYear: string]

``` scala
lines.show(5)
```

>     +-------+-------+-----------+-------------------+--------------------+------------+---------+
>     |movieID| lineID|characterID|      characterName|                text|  movieTitle|movieYear|
>     +-------+-------+-----------+-------------------+--------------------+------------+---------+
>     |   m124|L357776|      u1889|ASSISTANT SECRETARY|Let me have it.  ...|lost horizon|     1937|
>     |   m124|L357775|      u1892|              CLERK|Conway's gone aga...|lost horizon|     1937|
>     |   m124|L357774|      u1889|ASSISTANT SECRETARY|I'll dispatch a c...|lost horizon|     1937|
>     |   m124|L357773|      u1892|              CLERK|           Yes, sir.|lost horizon|     1937|
>     |   m124|L357772|      u1889|ASSISTANT SECRETARY|Yes. Might as wel...|lost horizon|     1937|
>     +-------+-------+-----------+-------------------+--------------------+------------+---------+
>     only showing top 5 rows

Dialogs with Lines
------------------

Let's join ght two DataFrames on `lineID` next.

``` scala
val convLines = conversations.join(lines, "lineID").sort($"conversationID", $"intraConversationID")
```

>     convLines: org.apache.spark.sql.DataFrame = [lineID: string, conversationID: bigint, intraConversationID: int, movieID: string, characterID: string, characterName: string, text: string, movieTitle: string, movieYear: string]

``` scala
convLines.count
```

>     res15: Long = 304713

``` scala
conversations.count
```

>     res16: Long = 304713

``` scala
display(convLines)
```

| lineID | conversationID | intraConversationID | movieID | characterID | characterName | text                                                                                                                                                                                                                       | movieTitle                 | movieYear |
|--------|----------------|---------------------|---------|-------------|---------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------|-----------|
| L194   | 0.0            | 0.0                 | m0      | u0          | BIANCA        | Can we make this quick?  Roxanne Korrine and Andrew Barrett are having an incredibly horrendous public break- up on the quad.  Again.                                                                                      | 10 things i hate about you | 1999      |
| L195   | 0.0            | 1.0                 | m0      | u2          | CAMERON       | Well, I thought we'd start with pronunciation, if that's okay with you.                                                                                                                                                    | 10 things i hate about you | 1999      |
| L196   | 0.0            | 2.0                 | m0      | u0          | BIANCA        | Not the hacking and gagging and spitting part.  Please.                                                                                                                                                                    | 10 things i hate about you | 1999      |
| L197   | 0.0            | 3.0                 | m0      | u2          | CAMERON       | Okay... then how 'bout we try out some French cuisine.  Saturday?  Night?                                                                                                                                                  | 10 things i hate about you | 1999      |
| L198   | 1.0            | 0.0                 | m0      | u0          | BIANCA        | You're asking me out.  That's so cute. What's your name again?                                                                                                                                                             | 10 things i hate about you | 1999      |
| L199   | 1.0            | 1.0                 | m0      | u2          | CAMERON       | Forget it.                                                                                                                                                                                                                 | 10 things i hate about you | 1999      |
| L200   | 2.0            | 0.0                 | m0      | u0          | BIANCA        | No, no, it's my fault -- we didn't have a proper introduction ---                                                                                                                                                          | 10 things i hate about you | 1999      |
| L201   | 2.0            | 1.0                 | m0      | u2          | CAMERON       | Cameron.                                                                                                                                                                                                                   | 10 things i hate about you | 1999      |
| L202   | 2.0            | 2.0                 | m0      | u0          | BIANCA        | The thing is, Cameron -- I'm at the mercy of a particularly hideous breed of loser.  My sister.  I can't date until she does.                                                                                              | 10 things i hate about you | 1999      |
| L203   | 2.0            | 3.0                 | m0      | u2          | CAMERON       | Seems like she could get a date easy enough...                                                                                                                                                                             | 10 things i hate about you | 1999      |
| L204   | 3.0            | 0.0                 | m0      | u2          | CAMERON       | Why?                                                                                                                                                                                                                       | 10 things i hate about you | 1999      |
| L205   | 3.0            | 1.0                 | m0      | u0          | BIANCA        | Unsolved mystery.  She used to be really popular when she started high school, then it was just like she got sick of it or something.                                                                                      | 10 things i hate about you | 1999      |
| L206   | 3.0            | 2.0                 | m0      | u2          | CAMERON       | That's a shame.                                                                                                                                                                                                            | 10 things i hate about you | 1999      |
| L207   | 4.0            | 0.0                 | m0      | u0          | BIANCA        | Gosh, if only we could find Kat a boyfriend...                                                                                                                                                                             | 10 things i hate about you | 1999      |
| L208   | 4.0            | 1.0                 | m0      | u2          | CAMERON       | Let me see what I can do.                                                                                                                                                                                                  | 10 things i hate about you | 1999      |
| L271   | 5.0            | 0.0                 | m0      | u0          | BIANCA        | C'esc ma tete. This is my head                                                                                                                                                                                             | 10 things i hate about you | 1999      |
| L272   | 5.0            | 1.0                 | m0      | u2          | CAMERON       | Right.  See?  You're ready for the quiz.                                                                                                                                                                                   | 10 things i hate about you | 1999      |
| L273   | 5.0            | 2.0                 | m0      | u0          | BIANCA        | I don't want to know how to say that though.  I want to know useful things. Like where the good stores are.  How much does champagne cost?  Stuff like Chat.  I have never in my life had to point out my head to someone. | 10 things i hate about you | 1999      |
| L274   | 5.0            | 3.0                 | m0      | u2          | CAMERON       | That's because it's such a nice one.                                                                                                                                                                                       | 10 things i hate about you | 1999      |
| L275   | 5.0            | 4.0                 | m0      | u0          | BIANCA        | Forget French.                                                                                                                                                                                                             | 10 things i hate about you | 1999      |
| L276   | 6.0            | 0.0                 | m0      | u0          | BIANCA        | How is our little Find the Wench A Date plan progressing?                                                                                                                                                                  | 10 things i hate about you | 1999      |
| L277   | 6.0            | 1.0                 | m0      | u2          | CAMERON       | Well, there's someone I think might be --                                                                                                                                                                                  | 10 things i hate about you | 1999      |
| L280   | 7.0            | 0.0                 | m0      | u2          | CAMERON       | There.                                                                                                                                                                                                                     | 10 things i hate about you | 1999      |
| L281   | 7.0            | 1.0                 | m0      | u0          | BIANCA        | Where?                                                                                                                                                                                                                     | 10 things i hate about you | 1999      |
| L363   | 8.0            | 0.0                 | m0      | u2          | CAMERON       | You got something on your mind?                                                                                                                                                                                            | 10 things i hate about you | 1999      |
| L364   | 8.0            | 1.0                 | m0      | u0          | BIANCA        | I counted on you to help my cause. You and that thug are obviously failing. Aren't we ever going on our date?                                                                                                              | 10 things i hate about you | 1999      |
| L365   | 9.0            | 0.0                 | m0      | u2          | CAMERON       | You have my word.  As a gentleman                                                                                                                                                                                          | 10 things i hate about you | 1999      |
| L366   | 9.0            | 1.0                 | m0      | u0          | BIANCA        | You're sweet.                                                                                                                                                                                                              | 10 things i hate about you | 1999      |
| L367   | 10.0           | 0.0                 | m0      | u2          | CAMERON       | How do you get your hair to look like that?                                                                                                                                                                                | 10 things i hate about you | 1999      |
| L368   | 10.0           | 1.0                 | m0      | u0          | BIANCA        | Eber's Deep Conditioner every two days. And I never, ever use a blowdryer without the diffuser attachment.                                                                                                                 | 10 things i hate about you | 1999      |

Truncated to 30 rows

Let's amalgamate the texts utered in the same conversations together.

By doing this we loose all the information in the order of utterance.

But this is fine as we are going to do LDA with just the *first-order information of words uttered in each conversation* by anyone involved in the dialogue.

``` scala
import org.apache.spark.sql.functions.{collect_list, udf, lit, concat_ws}

val corpusDF = convLines.groupBy($"conversationID",$"movieID")
  .agg(concat_ws(" :-()-: ",collect_list($"text")).alias("corpus"))
  .join(moviesMetaData, "movieID") // and join it to get movie meta data
  .select($"conversationID".as("id"),$"corpus",$"movieTitle",$"movieYear")
  .cache()
```

>     import org.apache.spark.sql.functions.{collect_list, udf, lit, concat_ws}
>     corpusDF: org.apache.spark.sql.DataFrame = [id: bigint, corpus: string, movieTitle: string, movieYear: string]

``` scala
corpusDF.count()
```

>     res18: Long = 83097

``` scala
corpusDF.take(5).foreach(println)
```

>     [17668,This would be funny - if it wasn't so pathetic. Why, she isn't a day over twenty! :-()-: You're wrong, George. :-()-: I'm not wrong. She told me so. Besides, she wouldn't have to tell me. I'd know anyway.  I found out a lot of things last night.  I'm not ashamed of it either. It's probably one of the few decent things that's ever happened in this hellish place.,lost horizon,1937]
>     [17598,Cave, eh? Where? :-()-: Over by that hill.,lost horizon,1937]
>     [17663,Something grand and beautiful, George. Something I've been searching for all my life. The answer to the confusion and bewilderment of a lifetime. I've found it, George, and I can't leave it. You mustn't either. :-()-: I don't know what you're talking about. You're carrying around a secret that seems to be eating you up. If you'll only tell me about it. :-()-: I will, George. I want to tell you. I'll burst with it if I don't. It's weird and fantastical and sometimes unbelievable, but so beautiful!  Well, as you know, we were kidnapped and brought here . . .,lost horizon,1937]
>     [17593,You see? You get the idea? From this reservoir here I can pipe in the whole works. Oh, I'm going to get a great kick out of this. Of course it's just to keep my hand in, but with the equipment we have here, I can put a plumbing system in for the whole village down there. Can rig it up in no time.  Do you realize those poor people are still going to the well for water? :-()-: It's unbelievable. :-()-: Think of it! In times like these. :-()-: Say, what about that gold deal? :-()-: Huh? :-()-: Gold. You were going to� :-()-: Oh - that! That can wait. Nobody's going to run off with it.  Say, I've got to get busy. I want to show this whole layout to Chang.  So long. Don't you take any wooden nickels. :-()-: All right.,lost horizon,1937]
>     [17658,Let me up! Let me up! :-()-: All right. Sorry, George.,lost horizon,1937]

``` scala
display(corpusDF)
```

| id      | corpus                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          | movieTitle   | movieYear |
|---------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------|-----------|
| 17668.0 | This would be funny - if it wasn't so pathetic. Why, she isn't a day over twenty! :-()-: You're wrong, George. :-()-: I'm not wrong. She told me so. Besides, she wouldn't have to tell me. I'd know anyway.  I found out a lot of things last night.  I'm not ashamed of it either. It's probably one of the few decent things that's ever happened in this hellish place.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     | lost horizon | 1937      |
| 17598.0 | Cave, eh? Where? :-()-: Over by that hill.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      | lost horizon | 1937      |
| 17663.0 | Something grand and beautiful, George. Something I've been searching for all my life. The answer to the confusion and bewilderment of a lifetime. I've found it, George, and I can't leave it. You mustn't either. :-()-: I don't know what you're talking about. You're carrying around a secret that seems to be eating you up. If you'll only tell me about it. :-()-: I will, George. I want to tell you. I'll burst with it if I don't. It's weird and fantastical and sometimes unbelievable, but so beautiful!  Well, as you know, we were kidnapped and brought here . . .                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | lost horizon | 1937      |
| 17593.0 | You see? You get the idea? From this reservoir here I can pipe in the whole works. Oh, I'm going to get a great kick out of this. Of course it's just to keep my hand in, but with the equipment we have here, I can put a plumbing system in for the whole village down there. Can rig it up in no time.  Do you realize those poor people are still going to the well for water? :-()-: It's unbelievable. :-()-: Think of it! In times like these. :-()-: Say, what about that gold deal? :-()-: Huh? :-()-: Gold. You were going to� :-()-: Oh - that! That can wait. Nobody's going to run off with it.  Say, I've got to get busy. I want to show this whole layout to Chang.  So long. Don't you take any wooden nickels. :-()-: All right.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | lost horizon | 1937      |
| 17658.0 | Let me up! Let me up! :-()-: All right. Sorry, George.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          | lost horizon | 1937      |
| 17610.0 | That would suit me perfectly. I'm always broke. How did you pay for them? :-()-: Our Valley is very rich in a metal called gold, which fortunately for us is valued very highly in the outside world. So we merely . . . :-()-: �buy and sell? :-()-: Buy and - sell? No, no, pardon me, exchange . :-()-: I see. Gold for ideas. You know Mr. Chang, there's something so simple and naive about all of this that I suspect there has been a shrewd, guiding intelligence somewhere. Whose idea was it? How did it all start? :-()-: That, my dear Conway, is the story of a remarkable man. :-()-: Who? :-()-: A Belgian priest by the name of Father Perrault, the first European to find this place, and a very great man indeed. He is responsible for everything you see here. He built Shangri-La, taught our natives, and began our collection of art. In fact, Shangri-La is Father Perrault. :-()-: When was all this? :-()-: Oh, let me see - way back in 1713, I think it was, that Father Perrault stumbled into the Valley, half frozen to death. It was typical of the man that, one leg being frozen, and of course there being no doctors here, he amputated the leg himself. :-()-: He amputated his own leg? :-()-: Yes. Oddly enough, later, when he had learned to understand their language, the natives told him he could have saved his leg. It would have healed without amputation. :-()-: Well, they didn't actually mean that. :-()-: Yes, yes. They were very sincere about it too. You see, a perfect body in perfect health is the rule here. They've never known anything different. So what was true for them they thought would naturally be true for anyone else living here. :-()-: Well, is it? :-()-: Rather astonishingly so, yes. And particularly so in the case of Father Perrault himself. Do you know when he and the natives were finished building Shangri-La, he was 108 years old and still very active, in spite of only having one leg? :-()-: 108 and still active? :-()-: You're startled? :-()-: Oh, no. Just a little bowled over, that's all. :-()-: Forgive me. I should have told you it is quite common here to live to a very ripe old age. Climate, diet, mountain water, you might say. But we like to believe it is the absence of struggle in the way we live. In your countries, on the other hand, how often do you hear the expression, "He worried himself to death?" or, "This thing or that killed him?" :-()-: Very often. :-()-: And very true. Your lives are therefore, as a rule, shorter, not so much by natural death as by indirect suicide. :-()-: That's all very fine if it works out. A little amazing, of course. :-()-: Why, Mr. Conway, you surprise me! :-()-: I surprise you? Now that's news. :-()-: I mean, your amazement. I could have understood it in any of your companions, but you - who have dreamed and written so much about better worlds. Or is it that you fail to recognize one of your own dreams when you see it? :-()-: Mr. Chang, if you don't mind, I think I'll go on being amazed - in moderation, of course. :-()-: Then everything is quite all right, isn't it? | lost horizon | 1937      |
| 17648.0 | What are these people? :-()-: I don't know. I can't get the dialect.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            | lost horizon | 1937      |
| 17562.0 | I didn't care for 'sister' last night, and I don't like 'Lovey' this morning. My name is Lovett - Alexander, P. :-()-: I see. :-()-: I see. :-()-: Well, it's a good morning, anyway. :-()-: I'm never conversational before I coffee.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          | lost horizon | 1937      |
| 17681.0 | Huh? I give it up. But this not knowing where you're going is exciting anyway. :-()-: Well, Mr. Conway, for a man who is supposed to be a leader, your do- nothing attitude is very disappointing.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | lost horizon | 1937      |
| 17573.0 | How about you Lovey? Come on. Let's you and I play a game of honeymoon bridge. :-()-: I'm thinking. :-()-: Thinking? What about some double solitaire? :-()-: As a matter of fact, I'm very good at double solitaire. :-()-: No kidding? :-()-: Yes. :-()-: Then I'm your man.  Come on, Toots.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | lost horizon | 1937      |
| 17638.0 | The power house - they've blown it up! The planes can't land without lights. :-()-: Come on! We'll burn the hangar. That will make light for them!                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | lost horizon | 1937      |
| 17622.0 | In that event, we better make arrangements to get some porters immediately. Some means to get us back to civilization. :-()-: Are you so certain you are away from it? :-()-: As far away as I ever want to be. :-()-: Oh, dear.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | lost horizon | 1937      |
| 17660.0 | For heaven's sake, Bob, what's the matter with you? You went out there for the purpose of� :-()-: George. George - do you mind? I'm sorry, but I can't talk about it tonight.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   | lost horizon | 1937      |
| 17633.0 | That Conway seemed to belong here. In fact, it was suggested that someone be sent to bring him here. :-()-: That I be brought here? Who had that brilliant idea? :-()-: Sondra Bizet. :-()-: Oh, the girl at the piano? :-()-: Yes. She has read your books and has a profound admiration for you, as have we all. :-()-: Of course I have suspected that our being here is no accident. Furthermore, I have a feeling that we're never supposed to leave. But that, for the moment, doesn't concern me greatly. I'll meet that when it comes. What particularly interests me at present is, why was I brought here? What possible use can I be to an already thriving community? :-()-: We need men like you here, to be sure that our community will continue to thrive. In return for which, Shangri-La has much to give you. You are still, by the world's standards, a youngish man. Yet in the normal course of existence, you can expect twenty or thirty years of gradually diminishing activity. Here, however, in Shangri- La, by our standards your life has just begun, and may go on and on. :-()-: But to be candid, Father, a prolonged future doesn't excite me. It would have to have a point. I've sometimes doubted whether life itself has any. And if that is so, then long life must be even more pointless. No, I'd need a much more definite reason for going on and on. :-()-: We have reason. It is the entire meaning and purpose of Shangri-La. It came to me in a vision, long, long, ago. I saw all the nations strengthening, not in wisdom, but in the vulgar passions and the will to destroy. I saw their machine power multiply until a single weaponed man might match a whole army. I foresaw a time when man, exulting in the technique of murder, would rage so hotly over the world that every book, every treasure, would be doomed to destruction. This vision was so vivid and so moving that I determined to gather together all the things of beauty and culture that I could and preserve them here against the doom toward which the world is rushing.  Look at the world today! Is there anything more pitiful?  What madness there is, what blindness, what unintelligent leadership! A scurrying mass of bewildered humanity crashing headlong against each other, propelled by an orgy of greed and brutality. The time must come, my friend, when this orgy will spend itself, when brutality and the lust for power must perish by its own sword. Against that time is why I avoided death and am here, and why you were brought here. For when that day comes, the world must begin to look for a new life. And it is our hope that they may find it here. For here we shall be with their books and their music and a way of life based on one simple rule: Be Kind.  When that day comes, it is our hope that the brotherly love of Shangri-La will spread throughout the world.  Yes, my son, when the strong have devoured each other, the Christian ethic may at last be fulfilled, and the meek shall inherit the earth.                                                                                            | lost horizon | 1937      |
| 17628.0 | Are you taking me? :-()-: Yes, of course. Certainly. Come on!                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   | lost horizon | 1937      |
| 17601.0 | And mine's Conway. :-()-: How do you do? :-()-: You've no idea, sir, how unexpected and very welcome you are. My friends and I - and the lady in the plane - left Baskul night before last for Shanghai, but we suddenly found ourselves traveling in the opposite direction�                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   | lost horizon | 1937      |
| 17650.0 | What is it? Has he fainted? :-()-: It looks like it.  Smell those fumes?                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        | lost horizon | 1937      |
| 17634.0 | Yes, of course, your brother is a problem. It was to be expected. :-()-: I knew you'd understand. That's why I came to you for help. :-()-: You must not look to me for help. Your brother is no longer my problem. He is now your problem, Conway. :-()-: Mine? :-()-: Because, my son, I am placing in your hands the future and destiny of Shangri-La.  For I am going to die.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               | lost horizon | 1937      |
| 17580.0 | Hey Lovey, come here! Lovey, I asked for a glass of wine and look what I got. Come on, sit down. :-()-: So that's where you are. I might of known it. No wonder you couldn't hear me. :-()-: You were asked to have a glass of wine. Sit down! :-()-: And be poisoned out here in the open? :-()-: Certainly not!                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               | lost horizon | 1937      |
| 17699.0 | Why, he's speaking English. :-()-: English!                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     | lost horizon | 1937      |
| 17645.0 | Don't worry, George. Nothing's going to happen. I'll fall right into line. I'll be the good little boy that everybody wants me to be. I'll be the best little Foreign Secretary we ever had, just because I haven't the nerve to be anything else. :-()-: Do try to sleep, Bob. :-()-: Huh? Oh, sure, Freshie. Good thing, sleep.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               | lost horizon | 1937      |
| 17564.0 | He might have lost his way. :-()-: Of course. That's what I told them last night. You can't expect a man to sail around in the dark.\[5\] During this George has been looking around - he rises.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | lost horizon | 1937      |
| 17570.0 | Yeah? If this be execution, lead me to it. :-()-: That's what they do with cattle just before the slaughter. Fatten them. :-()-: Uh-huh. You're a scream, Lovey. :-()-: Please don't call me Lovey.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             | lost horizon | 1937      |
| 17646.0 | Oh, stop it! :-()-: The bloke up there looks a Chinese, or a Mongolian, or something.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           | lost horizon | 1937      |
| 17587.0 | It's better than freezing to death down below, isn't it? :-()-: I'll say.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       | lost horizon | 1937      |
| 17652.0 | What is it? :-()-: See that spot? :-()-: Yes. :-()-: That's where we were this morning. He had it marked. Right on the border of Tibet. Here's where civilization ends. We must be a thousand miles beyond it - just a blank on the map. :-()-: What's it mean? :-()-: It means we're in unexplored country - country nobody ever reached.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      | lost horizon | 1937      |
| 17582.0 | �then the bears came right into the bedroom and the little baby bear said, "Oh, somebody's been sleeping in my bed." And then the mama bear said, "Oh dear, somebody's been sleeping in my bed!" And then the big papa bear, he roared, "And somebody's been sleeping in my bed!" Well, you have to admit the poor little bears were in a quandary! :-()-: I'm going to sleep in my bed. Come on, Lovey! :-()-: They were in a quandary, and� :-()-: Come on, Lovey. :-()-: Why? Why 'come on' all the time? What's the matter?  Are you going to be a fuss budget all your life? Here, drink it up! Aren't you having any fun? Where was I? :-()-: In a quandary.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | lost horizon | 1937      |
| 17647.0 | George, what are you going to do? :-()-: I'm going to drag him out and force him to tell us what his game is.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   | lost horizon | 1937      |
| 17577.0 | Yes. :-()-: Sounds like a stall to me.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          | lost horizon | 1937      |
| 17642.0 | Just what I needed too. :-()-: You? :-()-: Just this once, Bob. I feel like celebrating. Just think of it, Bob - a cruiser sent to Shanghai just to take you back to England. You know what it means.  Here you are. Don't bother about those cables now. I want you to drink with me.  Gentlemen, I give you Robert Conway - England's new Foreign Secretary.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  | lost horizon | 1937      |

Truncated to 30 rows

Feature extraction and transformation APIs
------------------------------------------

We will use the convenient [Feature extraction and transformation APIs](http://spark.apache.org/docs/latest/ml-features.html).

Step 3. Text Tokenization
-------------------------

We will use the RegexTokenizer to split each document into tokens. We can setMinTokenLength() here to indicate a minimum token length, and filter away all tokens that fall below the minimum.
See:

-   <http://spark.apache.org/docs/latest/ml-features.html#tokenizer>.

``` scala
import org.apache.spark.ml.feature.RegexTokenizer

// Set params for RegexTokenizer
val tokenizer = new RegexTokenizer()
.setPattern("[\\W_]+") // break by white space character(s)
.setMinTokenLength(4) // Filter away tokens with length < 4
.setInputCol("corpus") // name of the input column
.setOutputCol("tokens") // name of the output column

// Tokenize document
val tokenized_df = tokenizer.transform(corpusDF)
```

>     import org.apache.spark.ml.feature.RegexTokenizer
>     tokenizer: org.apache.spark.ml.feature.RegexTokenizer = regexTok_610667693de3
>     tokenized_df: org.apache.spark.sql.DataFrame = [id: bigint, corpus: string, movieTitle: string, movieYear: string, tokens: array<string>]

``` scala
display(tokenized_df.sample(false,0.001,1234L)) 
```

``` scala
display(tokenized_df.sample(false,0.001,1234L).select("tokens"))
```

Step 4. Remove Stopwords
------------------------

We can easily remove stopwords using the StopWordsRemover(). See:

-   <http://spark.apache.org/docs/latest/ml-features.html#stopwordsremover>.

If a list of stopwords is not provided, the StopWordsRemover() will use [this list of stopwords](http://ir.dcs.gla.ac.uk/resources/linguistic_utils/stop_words), also shown below, by default.

``` a,about,above,across,after,afterwards,again,against,all,almost,alone,along,already,also,although,always,am,among,amongst,amoungst,amount,an,and,another,any,anyhow,anyone,anything,anyway,anywhere,
are,around,as,at,back,be,became,because,become,becomes,becoming,been,before,beforehand,behind,being,below,beside,besides,between,beyond,bill,both,bottom,but,by,call,can,cannot,cant,co,computer,con,could,
couldnt,cry,de,describe,detail,do,done,down,due,during,each,eg,eight,either,eleven,else,elsewhere,empty,enough,etc,even,ever,every,everyone,everything,everywhere,except,few,fifteen,fify,fill,find,fire,first,
five,for,former,formerly,forty,found,four,from,front,full,further,get,give,go,had,has,hasnt,have,he,hence,her,here,hereafter,hereby,herein,hereupon,hers,herself,him,himself,his,how,however,hundred,i,ie,if,
in,inc,indeed,interest,into,is,it,its,itself,keep,last,latter,latterly,least,less,ltd,made,many,may,me,meanwhile,might,mill,mine,more,moreover,most,mostly,move,much,must,my,myself,name,namely,neither,never,
nevertheless,next,nine,no,nobody,none,noone,nor,not,nothing,now,nowhere,of,off,often,on,once,one,only,onto,or,other,others,otherwise,our,ours,ourselves,out,over,own,part,per,perhaps,please,put,rather,re,same,
see,seem,seemed,seeming,seems,serious,several,she,should,show,side,since,sincere,six,sixty,so,some,somehow,someone,something,sometime,sometimes,somewhere,still,such,system,take,ten,than,that,the,their,them,
themselves,then,thence,there,thereafter,thereby,therefore,therein,thereupon,these,they,thick,thin,third,this,those,though,three,through,throughout,thru,thus,to,together,too,top,toward,towards,twelve,twenty,two,
un,under,until,up,upon,us,very,via,was,we,well,were,what,whatever,when,whence,whenever,where,whereafter,whereas,whereby,wherein,whereupon,wherever,whether,which,while,whither,who,whoever,whole,whom,whose,why,will,
with,within,without,would,yet,you,your,yours,yourself,yourselves
```

You can use `getStopWords()` to see the list of stopwords that will be used.

In this example, we will specify a list of stopwords for the StopWordsRemover() to use. We do this so that we can add on to the list later on.

``` scala
display(dbutils.fs.ls("dbfs:/tmp/stopwords")) // check if the file already exists from earlier wget and dbfs-load
```

| path                | name      | size   |
|---------------------|-----------|--------|
| dbfs:/tmp/stopwords | stopwords | 2237.0 |

If the file `dbfs:/tmp/stopwords` already exists then skip the next two cells, otherwise download and load it into DBFS by uncommenting and evaluating the next two cells.

``` scala
//%sh wget http://ir.dcs.gla.ac.uk/resources/linguistic_utils/stop_words -O /tmp/stopwords # uncomment '//' at the beginning and repeat only if needed again
```

>     --2016-04-07 00:21:20--  http://ir.dcs.gla.ac.uk/resources/linguistic_utils/stop_words
>     Resolving ir.dcs.gla.ac.uk (ir.dcs.gla.ac.uk)... 130.209.240.253
>     Connecting to ir.dcs.gla.ac.uk (ir.dcs.gla.ac.uk)|130.209.240.253|:80... connected.
>     HTTP request sent, awaiting response... 200 OK
>     Length: 2237 (2.2K) [text/plain]
>     Saving to: '/tmp/stopwords'
>
>          0K ..                                                    100%  310M=0s
>
>     2016-04-07 00:21:21 (310 MB/s) - '/tmp/stopwords' saved [2237/2237]

``` scala
//%fs cp file:/tmp/stopwords dbfs:/tmp/stopwords # uncomment '//' at the beginning and repeat only if needed again
```

>     res31: Boolean = true

``` scala
// List of stopwords
val stopwords = sc.textFile("/tmp/stopwords").collect()
```

>     stopwords: Array[String] = Array(a, about, above, across, after, afterwards, again, against, all, almost, alone, along, already, also, although, always, am, among, amongst, amoungst, amount, an, and, another, any, anyhow, anyone, anything, anyway, anywhere, are, around, as, at, back, be, became, because, become, becomes, becoming, been, before, beforehand, behind, being, below, beside, besides, between, beyond, bill, both, bottom, but, by, call, can, cannot, cant, co, computer, con, could, couldnt, cry, de, describe, detail, do, done, down, due, during, each, eg, eight, either, eleven, else, elsewhere, empty, enough, etc, even, ever, every, everyone, everything, everywhere, except, few, fifteen, fify, fill, find, fire, first, five, for, former, formerly, forty, found, four, from, front, full, further, get, give, go, had, has, hasnt, have, he, hence, her, here, hereafter, hereby, herein, hereupon, hers, herself, him, himself, his, how, however, hundred, i, ie, if, in, inc, indeed, interest, into, is, it, its, itself, keep, last, latter, latterly, least, less, ltd, made, many, may, me, meanwhile, might, mill, mine, more, moreover, most, mostly, move, much, must, my, myself, name, namely, neither, never, nevertheless, next, nine, no, nobody, none, noone, nor, not, nothing, now, nowhere, of, off, often, on, once, one, only, onto, or, other, others, otherwise, our, ours, ourselves, out, over, own, part, per, perhaps, please, put, rather, re, same, see, seem, seemed, seeming, seems, serious, several, she, should, show, side, since, sincere, six, sixty, so, some, somehow, someone, something, sometime, sometimes, somewhere, still, such, system, take, ten, than, that, the, their, them, themselves, then, thence, there, thereafter, thereby, therefore, therein, thereupon, these, they, thick, thin, third, this, those, though, three, through, throughout, thru, thus, to, together, too, top, toward, towards, twelve, twenty, two, un, under, until, up, upon, us, very, via, was, we, well, were, what, whatever, when, whence, whenever, where, whereafter, whereas, whereby, wherein, whereupon, wherever, whether, which, while, whither, who, whoever, whole, whom, whose, why, will, with, within, without, would, yet, you, your, yours, yourself, yourselves)

``` scala
stopwords.length // find the number of stopwords in the scala Array[String]
```

>     res23: Int = 319

Finally, we can just remove the stopwords using the `StopWordsRemover` as follows:

``` scala
import org.apache.spark.ml.feature.StopWordsRemover

// Set params for StopWordsRemover
val remover = new StopWordsRemover()
.setStopWords(stopwords) // This parameter is optional
.setInputCol("tokens")
.setOutputCol("filtered")

// Create new DF with Stopwords removed
val filtered_df = remover.transform(tokenized_df)
```

>     import org.apache.spark.ml.feature.StopWordsRemover
>     remover: org.apache.spark.ml.feature.StopWordsRemover = stopWords_bcdc59bf2ac8
>     filtered_df: org.apache.spark.sql.DataFrame = [id: bigint, corpus: string, movieTitle: string, movieYear: string, tokens: array<string>, filtered: array<string>]

Step 5. Vector of Token Counts
------------------------------

LDA takes in a vector of token counts as input. We can use the `CountVectorizer()` to easily convert our text documents into vectors of token counts.

The `CountVectorizer` will return `(VocabSize, Array(Indexed Tokens), Array(Token Frequency))`.

Two handy parameters to note:

-   `setMinDF`: Specifies the minimum number of different documents a term must appear in to be included in the vocabulary.
-   `setMinTF`: Specifies the minimum number of times a term has to appear in a document to be included in the vocabulary.

See:

-   <http://spark.apache.org/docs/latest/ml-features.html#countvectorizer>.

``` scala
import org.apache.spark.ml.feature.CountVectorizer

// Set params for CountVectorizer
val vectorizer = new CountVectorizer()
.setInputCol("filtered")
.setOutputCol("features")
.setVocabSize(10000) 
.setMinDF(5) // the minimum number of different documents a term must appear in to be included in the vocabulary.
.fit(filtered_df)
```

>     import org.apache.spark.ml.feature.CountVectorizer
>     vectorizer: org.apache.spark.ml.feature.CountVectorizerModel = cntVec_daa8106c510f

``` scala
// Create vector of token counts
val countVectors = vectorizer.transform(filtered_df).select("id", "features")
```

>     countVectors: org.apache.spark.sql.DataFrame = [id: bigint, features: vector]

``` scala
// see the first countVectors
countVectors.take(1)
```

>     res24: Array[org.apache.spark.sql.Row] = Array([17668,(10000,[0,9,32,33,37,45,56,63,71,79,124,235,293,1562,1660,1899],[1.0,1.0,1.0,2.0,1.0,1.0,1.0,2.0,1.0,1.0,1.0,1.0,1.0,1.0,1.0,1.0])])

To use the LDA algorithm in the MLlib library, we have to convert the DataFrame back into an RDD.

``` scala
// Convert DF to RDD
import org.apache.spark.mllib.linalg.Vector

val lda_countVector = countVectors.map { case Row(id: Long, countVector: Vector) => (id, countVector) }
```

>     import org.apache.spark.mllib.linalg.Vector
>     lda_countVector: org.apache.spark.rdd.RDD[(Long, org.apache.spark.mllib.linalg.Vector)] = MapPartitionsRDD[305] at map at <console>:86

``` scala
// format: Array(id, (VocabSize, Array(indexedTokens), Array(Token Frequency)))
lda_countVector.take(1)
```

>     res25: Array[(Long, org.apache.spark.mllib.linalg.Vector)] = Array((17668,(10000,[0,9,32,33,37,45,56,63,71,79,124,235,293,1562,1660,1899],[1.0,1.0,1.0,2.0,1.0,1.0,1.0,2.0,1.0,1.0,1.0,1.0,1.0,1.0,1.0,1.0])))

Let's get an overview of LDA in Spark's MLLIB
---------------------------------------------

See:

-   <http://spark.apache.org/docs/latest/mllib-clustering.html#latent-dirichlet-allocation-lda>.

Create LDA model with Online Variational Bayes
----------------------------------------------

We will now set the parameters for LDA. We will use the OnlineLDAOptimizer() here, which implements Online Variational Bayes.

Choosing the number of topics for your LDA model requires a bit of domain knowledge. As we do not know the number of "topics", we will set numTopics to be 20.

``` scala
val numTopics = 20
```

>     numTopics: Int = 20

We will set the parameters needed to build our LDA model. We can also setMiniBatchFraction for the OnlineLDAOptimizer, which sets the fraction of corpus sampled and used at each iteration. In this example, we will set this to 0.8.

``` scala
import org.apache.spark.mllib.clustering.{LDA, OnlineLDAOptimizer}

// Set LDA params
val lda = new LDA()
.setOptimizer(new OnlineLDAOptimizer().setMiniBatchFraction(0.8))
.setK(numTopics)
.setMaxIterations(3)
.setDocConcentration(-1) // use default values
.setTopicConcentration(-1) // use default values
```

>     import org.apache.spark.mllib.clustering.{LDA, OnlineLDAOptimizer}
>     lda: org.apache.spark.mllib.clustering.LDA = org.apache.spark.mllib.clustering.LDA@1d493912

Create the LDA model with Online Variational Bayes.

``` scala
val ldaModel = lda.run(lda_countVector)
```

>     ldaModel: org.apache.spark.mllib.clustering.LDAModel = org.apache.spark.mllib.clustering.LocalLDAModel@3945c264

Watch **Online Learning for Latent Dirichlet Allocation** in NIPS2010 by Matt Hoffman (right click and open in new tab)

[!\[Matt Hoffman's NIPS 2010 Talk Online LDA\]](http://videolectures.net/nips2010_hoffman_oll/thumb.jpg)\](http://videolectures.net/nips2010*hoffman*oll/)

Also see the paper on *Online varioational Bayes* by Matt linked for more details (from the above URL): [http://videolectures.net/site/normal*dl/tag=83534/nips2010*1291.pdf](http://videolectures.net/site/normal_dl/tag=83534/nips2010_1291.pdf)

Note that using the OnlineLDAOptimizer returns us a [LocalLDAModel](http://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.mllib.clustering.LocalLDAModel), which stores the inferred topics of your corpus.

Review Topics
-------------

We can now review the results of our LDA model. We will print out all 20 topics with their corresponding term probabilities.

Note that you will get slightly different results every time you run an LDA model since LDA includes some randomization.

Let us review results of LDA model with Online Variational Bayes, step by step.

``` scala
val topicIndices = ldaModel.describeTopics(maxTermsPerTopic = 5)
```

>     topicIndices: Array[(Array[Int], Array[Double])] = Array((Array(4, 1, 9, 2, 8),Array(0.0015151627605402241, 0.001486317713687992, 0.0011681552786581887, 0.0010054261952954218, 8.615063157995299E-4)), (Array(1, 5, 15, 13, 14),Array(0.0019868644391922855, 0.0012407287861833166, 0.001222938318274118, 8.919429867442377E-4, 8.877996244770699E-4)), (Array(5, 3, 28, 9, 30),Array(0.0021699398068533824, 0.0011749233194760728, 9.013899165935946E-4, 8.504816848342215E-4, 8.252271581728695E-4)), (Array(5, 16, 1, 6, 2),Array(0.0052403657467525845, 0.0042033140402619375, 0.003763558294118634, 0.003396547452689663, 0.003021645305713142)), (Array(3, 4, 0, 1, 9),Array(0.0033139517864565287, 0.0021697727562965366, 0.0021377421844192557, 0.0018117462306041533, 0.0016275001664372737)), (Array(0, 1, 2, 22, 4),Array(0.0023854706148061567, 0.0019261905458742943, 0.0016576667581656617, 0.0011917968671869056, 0.0011248880469085086)), (Array(0, 4, 2, 1, 11),Array(0.004655511405730442, 0.002458768397723486, 0.0021078794201011583, 0.0017387793236628348, 0.0014236215935311085)), (Array(0, 1, 3, 6, 8),Array(0.0018921794022548181, 0.0018483719649855822, 0.0014386272597831232, 0.0014182374956709625, 0.0013764615715091382)), (Array(2, 10, 7, 12, 61),Array(0.0051946490753460636, 0.00495793071050275, 0.003580668739640197, 0.003184743897654473, 0.002981524063809674)), (Array(0, 2, 9, 4, 26),Array(0.0021335471974973886, 9.625379885382163E-4, 9.199660101772105E-4, 9.033959582006918E-4, 8.747265482123576E-4)), (Array(42, 121, 1, 4, 0),Array(0.0025288929149109313, 0.002359902253314637, 0.002261894922516035, 0.0018626814528567444, 0.0017576647100866413)), (Array(4, 1, 6, 0, 7),Array(0.007977613731363398, 0.0074256547675291915, 0.006054432245366723, 0.0047629498600014275, 0.004741787642644756)), (Array(0, 11, 1, 2, 7),Array(0.003854363383063756, 0.0024260532260151663, 0.0022162035474150243, 0.0018817267913487192, 0.0015001333324918243)), (Array(6, 1, 2, 70, 13),Array(0.0015776417631711249, 0.0014414685828715156, 0.0014279021959578673, 0.0012813104112466842, 8.294352348310147E-4)), (Array(1, 2, 0, 5, 23),Array(0.0036920461187951023, 0.00359354972907098, 0.002507896270867163, 0.0021850418950099, 0.0016447369107874818)), (Array(6, 0, 3, 35, 2),Array(0.002464771356929917, 0.0021840298145414266, 0.0013576445527878143, 0.0011473959400518262, 0.0011075469842802296)), (Array(4, 1, 10, 0, 5),Array(0.0019877776653926984, 0.0019088812832900985, 0.0014118885885063602, 0.0011336973397614043, 9.384609226085285E-4)), (Array(0, 2, 5, 4, 3),Array(0.003453703972841875, 0.00327608562414095, 0.0025174800837702693, 0.002218982477404188, 0.0017805786184620824)), (Array(0, 2, 1, 3, 6),Array(0.018645078429696073, 0.01154979812954807, 0.0093996276023053, 0.008649180109527853, 0.008565200297515204)), (Array(0, 1, 2, 5, 8),Array(0.019137707865734686, 0.014515895598928779, 0.012282554660302827, 0.010060286236470243, 0.009978216413893133)))

``` scala
val vocabList = vectorizer.vocabulary
```

>     vocabList: Array[String] = Array(know, just, like, want, think, right, going, good, yeah, tell, come, time, look, didn, mean, make, okay, really, little, sure, gonna, thing, people, said, maybe, need, sorry, love, talk, thought, doing, life, night, things, work, money, better, told, long, help, believe, years, shit, does, away, place, hell, doesn, great, home, feel, fuck, kind, remember, dead, course, wouldn, wait, kill, guess, understand, thank, girl, wrong, leave, listen, talking, real, hear, stop, nice, happened, fine, wanted, father, gotta, mind, fucking, house, wasn, getting, world, stay, mother, left, came, care, thanks, knew, room, trying, guys, went, looking, coming, heard, friend, haven, seen, best, tonight, live, used, matter, killed, pretty, business, idea, couldn, head, miss, says, wife, called, woman, morning, tomorrow, start, stuff, saying, play, hello, baby, hard, probably, minute, days, took, somebody, today, school, meet, gone, crazy, wants, damn, forget, cause, problem, deal, case, friends, point, hope, jesus, afraid, looks, knows, year, worry, exactly, aren, half, thinking, shut, hold, wanna, face, minutes, bring, doctor, word, read, everybody, supposed, makes, story, turn, true, watch, thousand, family, brother, kids, week, happen, fuckin, working, open, happy, lost, john, hurt, town, ready, alright, late, actually, gave, married, beautiful, soon, jack, times, sleep, door, having, hand, drink, easy, gets, chance, young, trouble, different, anybody, rest, shot, hate, death, second, later, asked, phone, wish, check, quite, walk, change, police, couple, question, close, taking, heart, hours, making, comes, anymore, truth, trust, dollars, important, captain, telling, funny, person, honey, goes, eyes, reason, inside, stand, break, means, number, tried, high, white, water, suppose, body, sick, game, excuse, party, women, country, answer, waiting, christ, office, send, pick, alive, sort, blood, black, daddy, line, husband, goddamn, book, fifty, thirty, fact, million, died, hands, power, started, stupid, shouldn, months, boys, city, dinner, sense, running, hour, shoot, fight, drive, george, speak, figure, living, ship, dear, street, ahead, lady, seven, free, feeling, scared, frank, able, children, safe, moment, outside, news, president, brought, write, happens, sent, bullshit, lose, light, glad, girls, child, sister, sounds, promise, till, lives, sound, weren, save, cool, poor, shall, asking, plan, bitch, king, daughter, beat, weeks, york, cold, worth, taken, harry, needs, piece, movie, fast, possible, small, goin, straight, human, hair, company, tired, food, lucky, pull, wonderful, touch, looked, state, thinks, picture, words, leaving, control, clear, known, special, buddy, luck, order, follow, expect, mary, mouth, catch, worked, mister, learn, playing, perfect, dream, calling, questions, hospital, coffee, ride, takes, parents, miles, works, secret, hotel, explain, worse, kidding, past, outta, general, unless, felt, drop, throw, interested, hang, certainly, absolutely, earth, loved, wonder, dark, accident, seeing, clock, doin, simple, turned, date, sweet, meeting, clean, sign, feet, handle, music, report, giving, army, fucked, cops, charlie, information, smart, yesterday, fall, fault, class, bank, month, blow, major, caught, swear, paul, road, choice, talked, plane, boss, david, paid, wear, american, worried, lord, goodbye, clothes, paper, ones, terrible, strange, mistake, given, kept, hurry, blue, murder, finish, apartment, sell, middle, nothin, careful, hasn, meant, walter, moving, changed, imagine, fair, difference, quiet, happening, near, quit, marry, personal, figured, future, rose, agent, building, michael, kinda, mama, early, private, watching, trip, certain, record, busy, jimmy, broke, store, sake, longer, finally, stick, boat, born, sitting, evening, bucks, ought, lying, chief, history, honor, lunch, kiss, darling, respect, uncle, favor, fool, rich, killing, land, liked, peter, tough, brain, interesting, nick, completely, welcome, problems, radio, wake, dick, honest, cash, dance, dude, james, bout, floor, weird, court, calls, jail, window, involved, drunk, johnny, needed, officer, asshole, books, spend, situation, relax, pain, service, dangerous, grand, security, letter, stopped, realize, offer, table, bastard, message, killer, instead, jake, deep, nervous, pass, somethin, evil, english, bought, short, step, ring, machine, likes, picked, eddie, voice, carry, upset, lived, forgot, afternoon, fear, quick, finished, count, forgive, wrote, decided, totally, named, space, team, lawyer, pleasure, doubt, station, suit, gotten, bother, return, prove, pictures, slow, strong, bunch, list, wearing, driving, join, christmas, tape, force, church, attack, appreciate, hungry, standing, college, present, dying, prison, missing, charge, board, truck, public, gold, staying, calm, ball, hardly, hadn, missed, lead, government, island, cover, horse, reach, joke, french, fish, star, surprise, moved, mike, america, soul, self, seconds, movies, dress, club, putting, price, saved, listening, lots, cost, smell, mark, peace, gives, crime, entire, dreams, single, usually, department, holy, beer, west, stuck, wall, protect, nose, teach, ways, forever, awful, grow, train, type, billy, rock, detective, walking, dumb, planet, papers, beginning, folks, attention, park, card, hide, birthday, test, share, reading, starting, master, lieutenant, partner, field, enjoy, twice, mess, film, blame, bomb, dollar, loves, round, south, girlfriend, gentlemen, records, using, plenty, especially, evidence, silly, experience, admit, normal, fired, talkin, notice, mission, fighting, memory, lock, louis, wedding, crap, promised, guns, idiot, marriage, glass, orders, ground, impossible, heaven, knock, hole, spent, green, animal, neck, wondering, press, drugs, nuts, position, broken, names, jerry, asleep, acting, visit, feels, plans, boyfriend, wind, tells, paris, smoke, cross, gimme, holding, sheriff, walked, mention, brothers, double, judge, writing, code, pardon, keeps, fellow, closed, lovely, angry, fell, surprised, percent, cute, charles, bathroom, correct, agree, address, summer, ridiculous, andy, tommy, rules, account, note, group, learned, proud, laugh, sing, pulled, sleeping, colonel, upstairs, area, difficult, built, jump, river, betty, breakfast, bobby, bridge, dirty, locked, amazing, north, feelings, alex, plus, definitely, worst, accept, kick, file, gettin, wild, seriously, grace, steal, stories, advice, relationship, nature, places, contact, waste, spot, favorite, beach, stole, apart, knowing, faith, song, risk, level, loose, foot, played, eating, patient, action, washington, witness, turns, build, obviously, begin, split, crew, command, games, nurse, decide, keeping, tight, copy, runs, form, bird, complete, insane, arrest, scene, taste, jeffrey, consider, shoes, teeth, sooner, career, henry, devil, monster, weekend, heavy, gift, innocent, hall, showed, study, destroy, greatest, keys, track, comin, raise, danger, bruce, suddenly, hanging, carl, california, apologize, concerned, program, blind, seventy, chicken, sweetheart, medical, forward, drinking, willing, legs, suspect, shop, professor, admiral, guard, data, ticket, camp, tree, losing, goodnight, dunno, paying, murdered, burn, television, trick, possibly, senator, credit, extra, dropped, meaning, starts, warm, hiding, sold, stone, cheap, taught, marty, lately, simply, lookin, science, queen, following, majesty, jeff, corner, harold, duty, cars, training, heads, seat, discuss, noticed, helped, enemy, bear, common, screw, responsible)

``` scala
val topics = topicIndices.map { case (terms, termWeights) =>
  terms.map(vocabList(_)).zip(termWeights)
}
```

>     topics: Array[Array[(String, Double)]] = Array(Array((think,0.0015151627605402241), (just,0.001486317713687992), (tell,0.0011681552786581887), (like,0.0010054261952954218), (yeah,8.615063157995299E-4)), Array((just,0.0019868644391922855), (right,0.0012407287861833166), (make,0.001222938318274118), (didn,8.919429867442377E-4), (mean,8.877996244770699E-4)), Array((right,0.0021699398068533824), (want,0.0011749233194760728), (talk,9.013899165935946E-4), (tell,8.504816848342215E-4), (doing,8.252271581728695E-4)), Array((right,0.0052403657467525845), (okay,0.0042033140402619375), (just,0.003763558294118634), (going,0.003396547452689663), (like,0.003021645305713142)), Array((want,0.0033139517864565287), (think,0.0021697727562965366), (know,0.0021377421844192557), (just,0.0018117462306041533), (tell,0.0016275001664372737)), Array((know,0.0023854706148061567), (just,0.0019261905458742943), (like,0.0016576667581656617), (people,0.0011917968671869056), (think,0.0011248880469085086)), Array((know,0.004655511405730442), (think,0.002458768397723486), (like,0.0021078794201011583), (just,0.0017387793236628348), (time,0.0014236215935311085)), Array((know,0.0018921794022548181), (just,0.0018483719649855822), (want,0.0014386272597831232), (going,0.0014182374956709625), (yeah,0.0013764615715091382)), Array((like,0.0051946490753460636), (come,0.00495793071050275), (good,0.003580668739640197), (look,0.003184743897654473), (thank,0.002981524063809674)), Array((know,0.0021335471974973886), (like,9.625379885382163E-4), (tell,9.199660101772105E-4), (think,9.033959582006918E-4), (sorry,8.747265482123576E-4)), Array((shit,0.0025288929149109313), (hello,0.002359902253314637), (just,0.002261894922516035), (think,0.0018626814528567444), (know,0.0017576647100866413)), Array((think,0.007977613731363398), (just,0.0074256547675291915), (going,0.006054432245366723), (know,0.0047629498600014275), (good,0.004741787642644756)), Array((know,0.003854363383063756), (time,0.0024260532260151663), (just,0.0022162035474150243), (like,0.0018817267913487192), (good,0.0015001333324918243)), Array((going,0.0015776417631711249), (just,0.0014414685828715156), (like,0.0014279021959578673), (nice,0.0012813104112466842), (didn,8.294352348310147E-4)), Array((just,0.0036920461187951023), (like,0.00359354972907098), (know,0.002507896270867163), (right,0.0021850418950099), (said,0.0016447369107874818)), Array((going,0.002464771356929917), (know,0.0021840298145414266), (want,0.0013576445527878143), (money,0.0011473959400518262), (like,0.0011075469842802296)), Array((think,0.0019877776653926984), (just,0.0019088812832900985), (come,0.0014118885885063602), (know,0.0011336973397614043), (right,9.384609226085285E-4)), Array((know,0.003453703972841875), (like,0.00327608562414095), (right,0.0025174800837702693), (think,0.002218982477404188), (want,0.0017805786184620824)), Array((know,0.018645078429696073), (like,0.01154979812954807), (just,0.0093996276023053), (want,0.008649180109527853), (going,0.008565200297515204)), Array((know,0.019137707865734686), (just,0.014515895598928779), (like,0.012282554660302827), (right,0.010060286236470243), (yeah,0.009978216413893133)))

Feel free to take things apart to understand!

``` scala
topicIndices(0)
```

>     res26: (Array[Int], Array[Double]) = (Array(4, 1, 9, 2, 8),Array(0.0015151627605402241, 0.001486317713687992, 0.0011681552786581887, 0.0010054261952954218, 8.615063157995299E-4))

``` scala
topicIndices(0)._1
```

>     res27: Array[Int] = Array(4, 1, 9, 2, 8)

``` scala
topicIndices(0)._1(0)
```

>     res28: Int = 4

``` scala
vocabList(topicIndices(0)._1(0))
```

>     res29: String = think

Review Results of LDA model with Online Variational Bayes - Doing all four steps earlier at once.

``` scala
val topicIndices = ldaModel.describeTopics(maxTermsPerTopic = 5)
val vocabList = vectorizer.vocabulary
val topics = topicIndices.map { case (terms, termWeights) =>
  terms.map(vocabList(_)).zip(termWeights)
}
println(s"$numTopics topics:")
topics.zipWithIndex.foreach { case (topic, i) =>
  println(s"TOPIC $i")
  topic.foreach { case (term, weight) => println(s"$term\t$weight") }
  println(s"==========")
}
```

>     20 topics:
>     TOPIC 0
>     think	0.0015151627605402241
>     just	0.001486317713687992
>     tell	0.0011681552786581887
>     like	0.0010054261952954218
>     yeah	8.615063157995299E-4
>     ==========
>     TOPIC 1
>     just	0.0019868644391922855
>     right	0.0012407287861833166
>     make	0.001222938318274118
>     didn	8.919429867442377E-4
>     mean	8.877996244770699E-4
>     ==========
>     TOPIC 2
>     right	0.0021699398068533824
>     want	0.0011749233194760728
>     talk	9.013899165935946E-4
>     tell	8.504816848342215E-4
>     doing	8.252271581728695E-4
>     ==========
>     TOPIC 3
>     right	0.0052403657467525845
>     okay	0.0042033140402619375
>     just	0.003763558294118634
>     going	0.003396547452689663
>     like	0.003021645305713142
>     ==========
>     TOPIC 4
>     want	0.0033139517864565287
>     think	0.0021697727562965366
>     know	0.0021377421844192557
>     just	0.0018117462306041533
>     tell	0.0016275001664372737
>     ==========
>     TOPIC 5
>     know	0.0023854706148061567
>     just	0.0019261905458742943
>     like	0.0016576667581656617
>     people	0.0011917968671869056
>     think	0.0011248880469085086
>     ==========
>     TOPIC 6
>     know	0.004655511405730442
>     think	0.002458768397723486
>     like	0.0021078794201011583
>     just	0.0017387793236628348
>     time	0.0014236215935311085
>     ==========
>     TOPIC 7
>     know	0.0018921794022548181
>     just	0.0018483719649855822
>     want	0.0014386272597831232
>     going	0.0014182374956709625
>     yeah	0.0013764615715091382
>     ==========
>     TOPIC 8
>     like	0.0051946490753460636
>     come	0.00495793071050275
>     good	0.003580668739640197
>     look	0.003184743897654473
>     thank	0.002981524063809674
>     ==========
>     TOPIC 9
>     know	0.0021335471974973886
>     like	9.625379885382163E-4
>     tell	9.199660101772105E-4
>     think	9.033959582006918E-4
>     sorry	8.747265482123576E-4
>     ==========
>     TOPIC 10
>     shit	0.0025288929149109313
>     hello	0.002359902253314637
>     just	0.002261894922516035
>     think	0.0018626814528567444
>     know	0.0017576647100866413
>     ==========
>     TOPIC 11
>     think	0.007977613731363398
>     just	0.0074256547675291915
>     going	0.006054432245366723
>     know	0.0047629498600014275
>     good	0.004741787642644756
>     ==========
>     TOPIC 12
>     know	0.003854363383063756
>     time	0.0024260532260151663
>     just	0.0022162035474150243
>     like	0.0018817267913487192
>     good	0.0015001333324918243
>     ==========
>     TOPIC 13
>     going	0.0015776417631711249
>     just	0.0014414685828715156
>     like	0.0014279021959578673
>     nice	0.0012813104112466842
>     didn	8.294352348310147E-4
>     ==========
>     TOPIC 14
>     just	0.0036920461187951023
>     like	0.00359354972907098
>     know	0.002507896270867163
>     right	0.0021850418950099
>     said	0.0016447369107874818
>     ==========
>     TOPIC 15
>     going	0.002464771356929917
>     know	0.0021840298145414266
>     want	0.0013576445527878143
>     money	0.0011473959400518262
>     like	0.0011075469842802296
>     ==========
>     TOPIC 16
>     think	0.0019877776653926984
>     just	0.0019088812832900985
>     come	0.0014118885885063602
>     know	0.0011336973397614043
>     right	9.384609226085285E-4
>     ==========
>     TOPIC 17
>     know	0.003453703972841875
>     like	0.00327608562414095
>     right	0.0025174800837702693
>     think	0.002218982477404188
>     want	0.0017805786184620824
>     ==========
>     TOPIC 18
>     know	0.018645078429696073
>     like	0.01154979812954807
>     just	0.0093996276023053
>     want	0.008649180109527853
>     going	0.008565200297515204
>     ==========
>     TOPIC 19
>     know	0.019137707865734686
>     just	0.014515895598928779
>     like	0.012282554660302827
>     right	0.010060286236470243
>     yeah	0.009978216413893133
>     ==========
>     topicIndices: Array[(Array[Int], Array[Double])] = Array((Array(4, 1, 9, 2, 8),Array(0.0015151627605402241, 0.001486317713687992, 0.0011681552786581887, 0.0010054261952954218, 8.615063157995299E-4)), (Array(1, 5, 15, 13, 14),Array(0.0019868644391922855, 0.0012407287861833166, 0.001222938318274118, 8.919429867442377E-4, 8.877996244770699E-4)), (Array(5, 3, 28, 9, 30),Array(0.0021699398068533824, 0.0011749233194760728, 9.013899165935946E-4, 8.504816848342215E-4, 8.252271581728695E-4)), (Array(5, 16, 1, 6, 2),Array(0.0052403657467525845, 0.0042033140402619375, 0.003763558294118634, 0.003396547452689663, 0.003021645305713142)), (Array(3, 4, 0, 1, 9),Array(0.0033139517864565287, 0.0021697727562965366, 0.0021377421844192557, 0.0018117462306041533, 0.0016275001664372737)), (Array(0, 1, 2, 22, 4),Array(0.0023854706148061567, 0.0019261905458742943, 0.0016576667581656617, 0.0011917968671869056, 0.0011248880469085086)), (Array(0, 4, 2, 1, 11),Array(0.004655511405730442, 0.002458768397723486, 0.0021078794201011583, 0.0017387793236628348, 0.0014236215935311085)), (Array(0, 1, 3, 6, 8),Array(0.0018921794022548181, 0.0018483719649855822, 0.0014386272597831232, 0.0014182374956709625, 0.0013764615715091382)), (Array(2, 10, 7, 12, 61),Array(0.0051946490753460636, 0.00495793071050275, 0.003580668739640197, 0.003184743897654473, 0.002981524063809674)), (Array(0, 2, 9, 4, 26),Array(0.0021335471974973886, 9.625379885382163E-4, 9.199660101772105E-4, 9.033959582006918E-4, 8.747265482123576E-4)), (Array(42, 121, 1, 4, 0),Array(0.0025288929149109313, 0.002359902253314637, 0.002261894922516035, 0.0018626814528567444, 0.0017576647100866413)), (Array(4, 1, 6, 0, 7),Array(0.007977613731363398, 0.0074256547675291915, 0.006054432245366723, 0.0047629498600014275, 0.004741787642644756)), (Array(0, 11, 1, 2, 7),Array(0.003854363383063756, 0.0024260532260151663, 0.0022162035474150243, 0.0018817267913487192, 0.0015001333324918243)), (Array(6, 1, 2, 70, 13),Array(0.0015776417631711249, 0.0014414685828715156, 0.0014279021959578673, 0.0012813104112466842, 8.294352348310147E-4)), (Array(1, 2, 0, 5, 23),Array(0.0036920461187951023, 0.00359354972907098, 0.002507896270867163, 0.0021850418950099, 0.0016447369107874818)), (Array(6, 0, 3, 35, 2),Array(0.002464771356929917, 0.0021840298145414266, 0.0013576445527878143, 0.0011473959400518262, 0.0011075469842802296)), (Array(4, 1, 10, 0, 5),Array(0.0019877776653926984, 0.0019088812832900985, 0.0014118885885063602, 0.0011336973397614043, 9.384609226085285E-4)), (Array(0, 2, 5, 4, 3),Array(0.003453703972841875, 0.00327608562414095, 0.0025174800837702693, 0.002218982477404188, 0.0017805786184620824)), (Array(0, 2, 1, 3, 6),Array(0.018645078429696073, 0.01154979812954807, 0.0093996276023053, 0.008649180109527853, 0.008565200297515204)), (Array(0, 1, 2, 5, 8),Array(0.019137707865734686, 0.014515895598928779, 0.012282554660302827, 0.010060286236470243, 0.009978216413893133)))
>     vocabList: Array[String] = Array(know, just, like, want, think, right, going, good, yeah, tell, come, time, look, didn, mean, make, okay, really, little, sure, gonna, thing, people, said, maybe, need, sorry, love, talk, thought, doing, life, night, things, work, money, better, told, long, help, believe, years, shit, does, away, place, hell, doesn, great, home, feel, fuck, kind, remember, dead, course, wouldn, wait, kill, guess, understand, thank, girl, wrong, leave, listen, talking, real, hear, stop, nice, happened, fine, wanted, father, gotta, mind, fucking, house, wasn, getting, world, stay, mother, left, came, care, thanks, knew, room, trying, guys, went, looking, coming, heard, friend, haven, seen, best, tonight, live, used, matter, killed, pretty, business, idea, couldn, head, miss, says, wife, called, woman, morning, tomorrow, start, stuff, saying, play, hello, baby, hard, probably, minute, days, took, somebody, today, school, meet, gone, crazy, wants, damn, forget, cause, problem, deal, case, friends, point, hope, jesus, afraid, looks, knows, year, worry, exactly, aren, half, thinking, shut, hold, wanna, face, minutes, bring, doctor, word, read, everybody, supposed, makes, story, turn, true, watch, thousand, family, brother, kids, week, happen, fuckin, working, open, happy, lost, john, hurt, town, ready, alright, late, actually, gave, married, beautiful, soon, jack, times, sleep, door, having, hand, drink, easy, gets, chance, young, trouble, different, anybody, rest, shot, hate, death, second, later, asked, phone, wish, check, quite, walk, change, police, couple, question, close, taking, heart, hours, making, comes, anymore, truth, trust, dollars, important, captain, telling, funny, person, honey, goes, eyes, reason, inside, stand, break, means, number, tried, high, white, water, suppose, body, sick, game, excuse, party, women, country, answer, waiting, christ, office, send, pick, alive, sort, blood, black, daddy, line, husband, goddamn, book, fifty, thirty, fact, million, died, hands, power, started, stupid, shouldn, months, boys, city, dinner, sense, running, hour, shoot, fight, drive, george, speak, figure, living, ship, dear, street, ahead, lady, seven, free, feeling, scared, frank, able, children, safe, moment, outside, news, president, brought, write, happens, sent, bullshit, lose, light, glad, girls, child, sister, sounds, promise, till, lives, sound, weren, save, cool, poor, shall, asking, plan, bitch, king, daughter, beat, weeks, york, cold, worth, taken, harry, needs, piece, movie, fast, possible, small, goin, straight, human, hair, company, tired, food, lucky, pull, wonderful, touch, looked, state, thinks, picture, words, leaving, control, clear, known, special, buddy, luck, order, follow, expect, mary, mouth, catch, worked, mister, learn, playing, perfect, dream, calling, questions, hospital, coffee, ride, takes, parents, miles, works, secret, hotel, explain, worse, kidding, past, outta, general, unless, felt, drop, throw, interested, hang, certainly, absolutely, earth, loved, wonder, dark, accident, seeing, clock, doin, simple, turned, date, sweet, meeting, clean, sign, feet, handle, music, report, giving, army, fucked, cops, charlie, information, smart, yesterday, fall, fault, class, bank, month, blow, major, caught, swear, paul, road, choice, talked, plane, boss, david, paid, wear, american, worried, lord, goodbye, clothes, paper, ones, terrible, strange, mistake, given, kept, hurry, blue, murder, finish, apartment, sell, middle, nothin, careful, hasn, meant, walter, moving, changed, imagine, fair, difference, quiet, happening, near, quit, marry, personal, figured, future, rose, agent, building, michael, kinda, mama, early, private, watching, trip, certain, record, busy, jimmy, broke, store, sake, longer, finally, stick, boat, born, sitting, evening, bucks, ought, lying, chief, history, honor, lunch, kiss, darling, respect, uncle, favor, fool, rich, killing, land, liked, peter, tough, brain, interesting, nick, completely, welcome, problems, radio, wake, dick, honest, cash, dance, dude, james, bout, floor, weird, court, calls, jail, window, involved, drunk, johnny, needed, officer, asshole, books, spend, situation, relax, pain, service, dangerous, grand, security, letter, stopped, realize, offer, table, bastard, message, killer, instead, jake, deep, nervous, pass, somethin, evil, english, bought, short, step, ring, machine, likes, picked, eddie, voice, carry, upset, lived, forgot, afternoon, fear, quick, finished, count, forgive, wrote, decided, totally, named, space, team, lawyer, pleasure, doubt, station, suit, gotten, bother, return, prove, pictures, slow, strong, bunch, list, wearing, driving, join, christmas, tape, force, church, attack, appreciate, hungry, standing, college, present, dying, prison, missing, charge, board, truck, public, gold, staying, calm, ball, hardly, hadn, missed, lead, government, island, cover, horse, reach, joke, french, fish, star, surprise, moved, mike, america, soul, self, seconds, movies, dress, club, putting, price, saved, listening, lots, cost, smell, mark, peace, gives, crime, entire, dreams, single, usually, department, holy, beer, west, stuck, wall, protect, nose, teach, ways, forever, awful, grow, train, type, billy, rock, detective, walking, dumb, planet, papers, beginning, folks, attention, park, card, hide, birthday, test, share, reading, starting, master, lieutenant, partner, field, enjoy, twice, mess, film, blame, bomb, dollar, loves, round, south, girlfriend, gentlemen, records, using, plenty, especially, evidence, silly, experience, admit, normal, fired, talkin, notice, mission, fighting, memory, lock, louis, wedding, crap, promised, guns, idiot, marriage, glass, orders, ground, impossible, heaven, knock, hole, spent, green, animal, neck, wondering, press, drugs, nuts, position, broken, names, jerry, asleep, acting, visit, feels, plans, boyfriend, wind, tells, paris, smoke, cross, gimme, holding, sheriff, walked, mention, brothers, double, judge, writing, code, pardon, keeps, fellow, closed, lovely, angry, fell, surprised, percent, cute, charles, bathroom, correct, agree, address, summer, ridiculous, andy, tommy, rules, account, note, group, learned, proud, laugh, sing, pulled, sleeping, colonel, upstairs, area, difficult, built, jump, river, betty, breakfast, bobby, bridge, dirty, locked, amazing, north, feelings, alex, plus, definitely, worst, accept, kick, file, gettin, wild, seriously, grace, steal, stories, advice, relationship, nature, places, contact, waste, spot, favorite, beach, stole, apart, knowing, faith, song, risk, level, loose, foot, played, eating, patient, action, washington, witness, turns, build, obviously, begin, split, crew, command, games, nurse, decide, keeping, tight, copy, runs, form, bird, complete, insane, arrest, scene, taste, jeffrey, consider, shoes, teeth, sooner, career, henry, devil, monster, weekend, heavy, gift, innocent, hall, showed, study, destroy, greatest, keys, track, comin, raise, danger, bruce, suddenly, hanging, carl, california, apologize, concerned, program, blind, seventy, chicken, sweetheart, medical, forward, drinking, willing, legs, suspect, shop, professor, admiral, guard, data, ticket, camp, tree, losing, goodnight, dunno, paying, murdered, burn, television, trick, possibly, senator, credit, extra, dropped, meaning, starts, warm, hiding, sold, stone, cheap, taught, marty, lately, simply, lookin, science, queen, following, majesty, jeff, corner, harold, duty, cars, training, heads, seat, discuss, noticed, helped, enemy, bear, common, screw, responsible)
>     topics: Array[Array[(String, Double)]] = Array(Array((think,0.0015151627605402241), (just,0.001486317713687992), (tell,0.0011681552786581887), (like,0.0010054261952954218), (yeah,8.615063157995299E-4)), Array((just,0.0019868644391922855), (right,0.0012407287861833166), (make,0.001222938318274118), (didn,8.919429867442377E-4), (mean,8.877996244770699E-4)), Array((right,0.0021699398068533824), (want,0.0011749233194760728), (talk,9.013899165935946E-4), (tell,8.504816848342215E-4), (doing,8.252271581728695E-4)), Array((right,0.0052403657467525845), (okay,0.0042033140402619375), (just,0.003763558294118634), (going,0.003396547452689663), (like,0.003021645305713142)), Array((want,0.0033139517864565287), (think,0.0021697727562965366), (know,0.0021377421844192557), (just,0.0018117462306041533), (tell,0.0016275001664372737)), Array((know,0.0023854706148061567), (just,0.0019261905458742943), (like,0.0016576667581656617), (people,0.0011917968671869056), (think,0.0011248880469085086)), Array((know,0.004655511405730442), (think,0.002458768397723486), (like,0.0021078794201011583), (just,0.0017387793236628348), (time,0.0014236215935311085)), Array((know,0.0018921794022548181), (just,0.0018483719649855822), (want,0.0014386272597831232), (going,0.0014182374956709625), (yeah,0.0013764615715091382)), Array((like,0.0051946490753460636), (come,0.00495793071050275), (good,0.003580668739640197), (look,0.003184743897654473), (thank,0.002981524063809674)), Array((know,0.0021335471974973886), (like,9.625379885382163E-4), (tell,9.199660101772105E-4), (think,9.033959582006918E-4), (sorry,8.747265482123576E-4)), Array((shit,0.0025288929149109313), (hello,0.002359902253314637), (just,0.002261894922516035), (think,0.0018626814528567444), (know,0.0017576647100866413)), Array((think,0.007977613731363398), (just,0.0074256547675291915), (going,0.006054432245366723), (know,0.0047629498600014275), (good,0.004741787642644756)), Array((know,0.003854363383063756), (time,0.0024260532260151663), (just,0.0022162035474150243), (like,0.0018817267913487192), (good,0.0015001333324918243)), Array((going,0.0015776417631711249), (just,0.0014414685828715156), (like,0.0014279021959578673), (nice,0.0012813104112466842), (didn,8.294352348310147E-4)), Array((just,0.0036920461187951023), (like,0.00359354972907098), (know,0.002507896270867163), (right,0.0021850418950099), (said,0.0016447369107874818)), Array((going,0.002464771356929917), (know,0.0021840298145414266), (want,0.0013576445527878143), (money,0.0011473959400518262), (like,0.0011075469842802296)), Array((think,0.0019877776653926984), (just,0.0019088812832900985), (come,0.0014118885885063602), (know,0.0011336973397614043), (right,9.384609226085285E-4)), Array((know,0.003453703972841875), (like,0.00327608562414095), (right,0.0025174800837702693), (think,0.002218982477404188), (want,0.0017805786184620824)), Array((know,0.018645078429696073), (like,0.01154979812954807), (just,0.0093996276023053), (want,0.008649180109527853), (going,0.008565200297515204)), Array((know,0.019137707865734686), (just,0.014515895598928779), (like,0.012282554660302827), (right,0.010060286236470243), (yeah,0.009978216413893133)))

Going through the results, you may notice that some of the topic words returned are actually stopwords that are specific to our dataset (for eg: "writes", "article"...). Let's try improving our model.

Step 8. Model Tuning - Refilter Stopwords
-----------------------------------------

We will try to improve the results of our model by identifying some stopwords that are specific to our dataset. We will filter these stopwords out and rerun our LDA model to see if we get better results.

``` scala
val add_stopwords = Array("whatever") // add  more stop-words like the name of your company!
```

>     add_stopwords: Array[String] = Array(whatever)

``` scala
// Combine newly identified stopwords to our exising list of stopwords
val new_stopwords = stopwords.union(add_stopwords)
```

>     new_stopwords: Array[String] = Array(a, about, above, across, after, afterwards, again, against, all, almost, alone, along, already, also, although, always, am, among, amongst, amoungst, amount, an, and, another, any, anyhow, anyone, anything, anyway, anywhere, are, around, as, at, back, be, became, because, become, becomes, becoming, been, before, beforehand, behind, being, below, beside, besides, between, beyond, bill, both, bottom, but, by, call, can, cannot, cant, co, computer, con, could, couldnt, cry, de, describe, detail, do, done, down, due, during, each, eg, eight, either, eleven, else, elsewhere, empty, enough, etc, even, ever, every, everyone, everything, everywhere, except, few, fifteen, fify, fill, find, fire, first, five, for, former, formerly, forty, found, four, from, front, full, further, get, give, go, had, has, hasnt, have, he, hence, her, here, hereafter, hereby, herein, hereupon, hers, herself, him, himself, his, how, however, hundred, i, ie, if, in, inc, indeed, interest, into, is, it, its, itself, keep, last, latter, latterly, least, less, ltd, made, many, may, me, meanwhile, might, mill, mine, more, moreover, most, mostly, move, much, must, my, myself, name, namely, neither, never, nevertheless, next, nine, no, nobody, none, noone, nor, not, nothing, now, nowhere, of, off, often, on, once, one, only, onto, or, other, others, otherwise, our, ours, ourselves, out, over, own, part, per, perhaps, please, put, rather, re, same, see, seem, seemed, seeming, seems, serious, several, she, should, show, side, since, sincere, six, sixty, so, some, somehow, someone, something, sometime, sometimes, somewhere, still, such, system, take, ten, than, that, the, their, them, themselves, then, thence, there, thereafter, thereby, therefore, therein, thereupon, these, they, thick, thin, third, this, those, though, three, through, throughout, thru, thus, to, together, too, top, toward, towards, twelve, twenty, two, un, under, until, up, upon, us, very, via, was, we, well, were, what, whatever, when, whence, whenever, where, whereafter, whereas, whereby, wherein, whereupon, wherever, whether, which, while, whither, who, whoever, whole, whom, whose, why, will, with, within, without, would, yet, you, your, yours, yourself, yourselves, whatever)

``` scala
import org.apache.spark.ml.feature.StopWordsRemover

// Set Params for StopWordsRemover with new_stopwords
val remover = new StopWordsRemover()
.setStopWords(new_stopwords)
.setInputCol("tokens")
.setOutputCol("filtered")

// Create new df with new list of stopwords removed
val new_filtered_df = remover.transform(tokenized_df)
```

>     import org.apache.spark.ml.feature.StopWordsRemover
>     remover: org.apache.spark.ml.feature.StopWordsRemover = stopWords_4029322e305c
>     new_filtered_df: org.apache.spark.sql.DataFrame = [id: bigint, corpus: string, movieTitle: string, movieYear: string, tokens: array<string>, filtered: array<string>]

``` scala
// Set Params for CountVectorizer
val vectorizer = new CountVectorizer()
.setInputCol("filtered")
.setOutputCol("features")
.setVocabSize(10000)
.setMinDF(5)
.fit(new_filtered_df)

// Create new df of countVectors
val new_countVectors = vectorizer.transform(new_filtered_df).select("id", "features")
```

>     vectorizer: org.apache.spark.ml.feature.CountVectorizerModel = cntVec_68b272a90616
>     new_countVectors: org.apache.spark.sql.DataFrame = [id: bigint, features: vector]

``` scala
// Convert DF to RDD
val new_lda_countVector = new_countVectors.map { case Row(id: Long, countVector: Vector) => (id, countVector) }
```

>     new_lda_countVector: org.apache.spark.rdd.RDD[(Long, org.apache.spark.mllib.linalg.Vector)] = MapPartitionsRDD[358] at map at <console>:92

We will also increase MaxIterations to 10 to see if we get better results.

``` scala
// Set LDA parameters
val new_lda = new LDA()
.setOptimizer(new OnlineLDAOptimizer().setMiniBatchFraction(0.8))
.setK(numTopics)
.setMaxIterations(10)
.setDocConcentration(-1) // use default values
.setTopicConcentration(-1) // use default values
```

>     new_lda: org.apache.spark.mllib.clustering.LDA = org.apache.spark.mllib.clustering.LDA@5c137f48

#### How to find what the default values are?

Dive into the source!!!

1.  Let's find the default value for `docConcentration` now.
2.  Got to Apache Spark package Root: <https://spark.apache.org/docs/latest/api/scala/#package>

-   search for 'ml' in the search box on the top left (ml is for ml library)
-   Then find the `LDA` by scrolling below on the left to mllib's `clustering` methods and click on `LDA`
-   Then click on the source code link which should take you here:
    -   <https://github.com/apache/spark/blob/v1.6.1/mllib/src/main/scala/org/apache/spark/ml/clustering/LDA.scala>
    -   Now, simply go to the right function and see the following comment block:

    <!-- -->

        /**
         * Concentration parameter (commonly named "alpha") for the prior placed on documents'
         * distributions over topics ("theta").
         *
         * This is the parameter to a Dirichlet distribution, where larger values mean more smoothing
         * (more regularization).
         *
         * If not set by the user, then docConcentration is set automatically. If set to
         * singleton vector [alpha], then alpha is replicated to a vector of length k in fitting.
         * Otherwise, the [[docConcentration]] vector must be length k.
         * (default = automatic)
         *
         * Optimizer-specific parameter settings:
         *  - EM
         *     - Currently only supports symmetric distributions, so all values in the vector should be
         *       the same.
         *     - Values should be > 1.0
         *     - default = uniformly (50 / k) + 1, where 50/k is common in LDA libraries and +1 follows
         *       from Asuncion et al. (2009), who recommend a +1 adjustment for EM.
         *  - Online
         *     - Values should be >= 0
         *     - default = uniformly (1.0 / k), following the implementation from
         *       [[https://github.com/Blei-Lab/onlineldavb]].
         * @group param
         */

**HOMEWORK:** Try to find the default value for `TopicConcentration`.

``` scala
// Create LDA model with stopwords refiltered
val new_ldaModel = new_lda.run(new_lda_countVector)
```

>     new_ldaModel: org.apache.spark.mllib.clustering.LDAModel = org.apache.spark.mllib.clustering.LocalLDAModel@643fbf3b

``` scala
val topicIndices = new_ldaModel.describeTopics(maxTermsPerTopic = 5)
val vocabList = vectorizer.vocabulary
val topics = topicIndices.map { case (terms, termWeights) =>
  terms.map(vocabList(_)).zip(termWeights)
}
println(s"$numTopics topics:")
topics.zipWithIndex.foreach { case (topic, i) =>
  println(s"TOPIC $i")
  topic.foreach { case (term, weight) => println(s"$term\t$weight") }
  println(s"==========")
}
```

>     20 topics:
>     TOPIC 0
>     know	0.003380285974360492
>     just	0.0032710843636220392
>     mean	0.0020403580269282785
>     going	0.0018412081716573069
>     yeah	0.0017292076410595036
>     ==========
>     TOPIC 1
>     know	0.002722716605262849
>     think	0.0023160709876635563
>     want	0.0015682033014027693
>     tell	0.0013765056029302197
>     really	0.0013190996836754377
>     ==========
>     TOPIC 2
>     know	0.005285852017130324
>     gonna	0.004450466114304245
>     like	0.003808477141912193
>     wait	0.00257579095912931
>     yeah	0.002507435691768419
>     ==========
>     TOPIC 3
>     think	0.006461906040549763
>     come	0.0038559336578359326
>     like	0.002329773059820553
>     just	0.002290146790129036
>     time	0.0022316260674415563
>     ==========
>     TOPIC 4
>     know	0.002839310966830335
>     just	0.0020525839506887987
>     like	0.0016688236712771678
>     good	0.0014274841432802734
>     yeah	0.0013850182617787298
>     ==========
>     TOPIC 5
>     look	0.006430414745047661
>     good	0.005126904726913699
>     know	0.0038267418634824993
>     like	0.0036370167197777273
>     wrong	0.002887042876288361
>     ==========
>     TOPIC 6
>     know	0.007168914271817762
>     like	0.0033303014922639114
>     maybe	0.001955212601872564
>     just	0.0019152703247870325
>     didn	0.001636430956103975
>     ==========
>     TOPIC 7
>     know	0.002439426756082817
>     like	0.0022689767941727434
>     okay	0.001495915190048933
>     think	0.0013473171307872285
>     right	0.0012347189576881645
>     ==========
>     TOPIC 8
>     just	0.001595528475061785
>     like	0.001569574938286783
>     right	0.0014295885338975265
>     good	0.0013466020121047748
>     said	0.0011352192707646653
>     ==========
>     TOPIC 9
>     know	0.016901222618068566
>     doing	0.004577649410574112
>     think	0.003971192161188001
>     like	0.00302620634958447
>     going	0.002669274623796597
>     ==========
>     TOPIC 10
>     know	0.0038140269563962073
>     going	0.0031781682895507344
>     just	0.0028816331972935914
>     like	0.002153017945640159
>     think	0.0020204879213200793
>     ==========
>     TOPIC 11
>     know	0.0015301133248231589
>     think	0.0014066243941540167
>     want	0.0013542853436208809
>     just	0.001304010526888571
>     come	0.0011507128067044772
>     ==========
>     TOPIC 12
>     believe	0.005486844829169185
>     love	0.005165142844000885
>     like	0.002752377771173137
>     yeah	0.002435701398970671
>     know	0.002172817525760321
>     ==========
>     TOPIC 13
>     good	0.0017598365263938752
>     watch	0.0014049463901361563
>     know	0.0013008619267103779
>     just	0.0010998243109867924
>     like	0.0010793139503736371
>     ==========
>     TOPIC 14
>     know	0.003878373108166717
>     want	0.0030755628561658353
>     right	0.0022294955855522
>     going	0.0021720024876757753
>     just	0.0018231807557625098
>     ==========
>     TOPIC 15
>     just	0.0028533537480122657
>     admiral	0.0026257588674351673
>     captain	0.002238753458417432
>     want	0.0018200655345899832
>     know	0.0014445516378355958
>     ==========
>     TOPIC 16
>     know	0.021238887749001945
>     just	0.015968186460355357
>     like	0.014999425078825599
>     want	0.01113115647988978
>     think	0.010478256508567313
>     ==========
>     TOPIC 17
>     yeah	0.00584394109274795
>     hello	0.005492554407760574
>     just	0.0029978901524305417
>     like	0.0018868921065350684
>     right	0.001718570248830763
>     ==========
>     TOPIC 18
>     sure	0.002274494414898221
>     right	0.0017297649569008693
>     want	0.0014831519281558064
>     know	0.0014498408868561115
>     like	0.0014485786883881421
>     ==========
>     TOPIC 19
>     just	0.0013129899588363335
>     like	0.0011555068372966766
>     going	0.0010399640593116153
>     know	8.095270505187395E-4
>     time	7.192560022056095E-4
>     ==========
>     topicIndices: Array[(Array[Int], Array[Double])] = Array((Array(0, 1, 14, 6, 8),Array(0.003380285974360492, 0.0032710843636220392, 0.0020403580269282785, 0.0018412081716573069, 0.0017292076410595036)), (Array(0, 4, 3, 9, 17),Array(0.002722716605262849, 0.0023160709876635563, 0.0015682033014027693, 0.0013765056029302197, 0.0013190996836754377)), (Array(0, 20, 2, 57, 8),Array(0.005285852017130324, 0.004450466114304245, 0.003808477141912193, 0.00257579095912931, 0.002507435691768419)), (Array(4, 10, 2, 1, 11),Array(0.006461906040549763, 0.0038559336578359326, 0.002329773059820553, 0.002290146790129036, 0.0022316260674415563)), (Array(0, 1, 2, 7, 8),Array(0.002839310966830335, 0.0020525839506887987, 0.0016688236712771678, 0.0014274841432802734, 0.0013850182617787298)), (Array(12, 7, 0, 2, 63),Array(0.006430414745047661, 0.005126904726913699, 0.0038267418634824993, 0.0036370167197777273, 0.002887042876288361)), (Array(0, 2, 24, 1, 13),Array(0.007168914271817762, 0.0033303014922639114, 0.001955212601872564, 0.0019152703247870325, 0.001636430956103975)), (Array(0, 2, 16, 4, 5),Array(0.002439426756082817, 0.0022689767941727434, 0.001495915190048933, 0.0013473171307872285, 0.0012347189576881645)), (Array(1, 2, 5, 7, 23),Array(0.001595528475061785, 0.001569574938286783, 0.0014295885338975265, 0.0013466020121047748, 0.0011352192707646653)), (Array(0, 30, 4, 2, 6),Array(0.016901222618068566, 0.004577649410574112, 0.003971192161188001, 0.00302620634958447, 0.002669274623796597)), (Array(0, 6, 1, 2, 4),Array(0.0038140269563962073, 0.0031781682895507344, 0.0028816331972935914, 0.002153017945640159, 0.0020204879213200793)), (Array(0, 4, 3, 1, 10),Array(0.0015301133248231589, 0.0014066243941540167, 0.0013542853436208809, 0.001304010526888571, 0.0011507128067044772)), (Array(40, 27, 2, 8, 0),Array(0.005486844829169185, 0.005165142844000885, 0.002752377771173137, 0.002435701398970671, 0.002172817525760321)), (Array(7, 169, 0, 1, 2),Array(0.0017598365263938752, 0.0014049463901361563, 0.0013008619267103779, 0.0010998243109867924, 0.0010793139503736371)), (Array(0, 3, 5, 6, 1),Array(0.003878373108166717, 0.0030755628561658353, 0.0022294955855522, 0.0021720024876757753, 0.0018231807557625098)), (Array(1, 945, 233, 3, 0),Array(0.0028533537480122657, 0.0026257588674351673, 0.002238753458417432, 0.0018200655345899832, 0.0014445516378355958)), (Array(0, 1, 2, 3, 4),Array(0.021238887749001945, 0.015968186460355357, 0.014999425078825599, 0.01113115647988978, 0.010478256508567313)), (Array(8, 121, 1, 2, 5),Array(0.00584394109274795, 0.005492554407760574, 0.0029978901524305417, 0.0018868921065350684, 0.001718570248830763)), (Array(19, 5, 3, 0, 2),Array(0.002274494414898221, 0.0017297649569008693, 0.0014831519281558064, 0.0014498408868561115, 0.0014485786883881421)), (Array(1, 2, 6, 0, 11),Array(0.0013129899588363335, 0.0011555068372966766, 0.0010399640593116153, 8.095270505187395E-4, 7.192560022056095E-4)))
>     vocabList: Array[String] = Array(know, just, like, want, think, right, going, good, yeah, tell, come, time, look, didn, mean, make, okay, really, little, sure, gonna, thing, people, said, maybe, need, sorry, love, talk, thought, doing, life, night, things, work, money, better, told, long, help, believe, years, shit, does, away, place, hell, doesn, great, home, feel, fuck, kind, remember, dead, course, wouldn, wait, kill, guess, understand, thank, girl, wrong, leave, listen, talking, real, hear, stop, nice, happened, fine, wanted, father, gotta, mind, fucking, house, wasn, getting, world, stay, mother, left, came, care, thanks, knew, room, trying, guys, went, looking, coming, heard, friend, haven, seen, best, tonight, live, used, matter, killed, pretty, business, idea, couldn, head, miss, says, wife, called, woman, morning, tomorrow, start, stuff, saying, play, hello, baby, hard, probably, minute, days, took, somebody, today, school, meet, gone, crazy, wants, damn, forget, problem, cause, deal, case, friends, point, hope, jesus, afraid, looks, knows, year, worry, exactly, aren, half, thinking, shut, hold, wanna, face, minutes, bring, word, read, doctor, everybody, supposed, makes, story, turn, true, watch, thousand, family, brother, kids, week, happen, fuckin, working, open, happy, lost, john, hurt, town, ready, alright, late, actually, married, gave, beautiful, soon, jack, times, sleep, door, having, drink, hand, easy, gets, chance, young, trouble, different, anybody, shot, rest, hate, death, second, later, asked, phone, wish, check, quite, walk, change, police, couple, question, close, taking, heart, hours, making, comes, anymore, truth, trust, dollars, important, captain, telling, funny, person, honey, goes, eyes, reason, inside, stand, break, means, number, tried, high, white, water, suppose, body, sick, game, excuse, party, women, country, answer, christ, waiting, office, send, pick, alive, sort, blood, black, daddy, line, husband, goddamn, book, fifty, thirty, fact, million, died, hands, power, stupid, started, shouldn, months, boys, city, sense, dinner, running, hour, shoot, fight, drive, speak, george, ship, living, figure, dear, street, ahead, lady, seven, scared, free, feeling, frank, able, children, safe, moment, outside, news, president, brought, write, happens, sent, bullshit, lose, light, glad, child, girls, sounds, sister, promise, lives, till, sound, weren, save, poor, cool, shall, asking, plan, king, bitch, daughter, weeks, beat, york, cold, worth, taken, harry, needs, piece, movie, fast, possible, small, goin, straight, human, hair, company, food, tired, lucky, pull, wonderful, touch, looked, thinks, state, picture, leaving, words, control, clear, known, special, buddy, luck, order, follow, expect, mary, catch, mouth, worked, mister, learn, playing, perfect, dream, calling, questions, hospital, takes, ride, coffee, miles, parents, works, secret, hotel, explain, kidding, worse, past, outta, general, felt, drop, unless, throw, interested, hang, certainly, absolutely, earth, loved, dark, wonder, accident, seeing, turned, clock, simple, doin, date, sweet, meeting, clean, sign, feet, handle, music, report, giving, army, fucked, cops, charlie, smart, yesterday, information, fall, fault, bank, class, month, blow, swear, caught, major, paul, road, talked, choice, plane, boss, david, paid, wear, american, worried, lord, paper, goodbye, clothes, ones, terrible, strange, given, mistake, finish, kept, blue, murder, hurry, apartment, sell, middle, nothin, careful, hasn, meant, walter, moving, changed, imagine, fair, difference, quiet, happening, near, quit, personal, marry, figured, future, rose, building, mama, michael, early, agent, kinda, watching, private, trip, record, certain, busy, jimmy, broke, sake, longer, store, boat, stick, finally, born, evening, sitting, bucks, ought, chief, lying, history, kiss, honor, darling, lunch, favor, fool, uncle, respect, rich, land, liked, killing, peter, tough, brain, interesting, completely, problems, welcome, nick, wake, honest, radio, dick, cash, dance, dude, james, bout, floor, weird, court, calls, jail, drunk, window, involved, johnny, officer, needed, asshole, situation, spend, books, relax, pain, service, grand, dangerous, letter, security, stopped, offer, realize, table, bastard, message, instead, killer, jake, deep, nervous, somethin, pass, evil, english, bought, short, step, ring, picked, likes, machine, eddie, voice, upset, forgot, carry, lived, afternoon, fear, quick, finished, count, forgive, wrote, named, decided, totally, space, team, pleasure, doubt, lawyer, station, gotten, suit, bother, prove, return, slow, pictures, bunch, strong, list, wearing, driving, join, tape, christmas, attack, appreciate, force, church, college, hungry, standing, present, dying, prison, missing, charge, board, truck, public, calm, gold, staying, ball, hardly, hadn, missed, lead, island, government, horse, cover, french, reach, joke, fish, star, mike, surprise, america, moved, soul, dress, seconds, club, self, putting, movies, lots, cost, listening, price, saved, smell, mark, peace, dreams, entire, crime, gives, usually, single, department, holy, beer, west, protect, stuck, wall, nose, ways, teach, forever, grow, train, type, awful, rock, detective, billy, walking, dumb, papers, beginning, planet, folks, park, attention, birthday, hide, card, master, share, reading, test, starting, lieutenant, field, partner, enjoy, twice, film, dollar, bomb, mess, blame, south, loves, girlfriend, round, records, using, plenty, especially, gentlemen, evidence, silly, experience, admit, fired, normal, talkin, mission, louis, memory, fighting, lock, notice, crap, wedding, promised, marriage, ground, guns, glass, idiot, orders, impossible, heaven, knock, hole, neck, animal, spent, green, wondering, nuts, press, drugs, broken, position, names, asleep, jerry, visit, boyfriend, acting, feels, plans, paris, smoke, tells, wind, cross, holding, sheriff, gimme, walked, mention, writing, double, brothers, code, judge, pardon, keeps, fellow, fell, closed, lovely, angry, cute, charles, surprised, percent, correct, bathroom, agree, address, andy, ridiculous, summer, tommy, rules, group, account, note, pulled, sleeping, sing, learned, proud, laugh, colonel, upstairs, river, difficult, built, jump, area, dirty, betty, bridge, breakfast, bobby, locked, amazing, north, feelings, alex, plus, definitely, worst, accept, kick, seriously, grace, steal, wild, stories, file, gettin, relationship, advice, nature, contact, spot, places, waste, knowing, beach, stole, apart, favorite, faith, level, loose, risk, song, eating, foot, played, patient, washington, turns, witness, action, build, obviously, begin, split, crew, command, games, tight, decide, nurse, keeping, runs, form, bird, copy, insane, complete, arrest, consider, taste, scene, jeffrey, teeth, shoes, career, henry, sooner, devil, monster, showed, weekend, gift, innocent, study, heavy, hall, comin, danger, greatest, track, keys, raise, destroy, concerned, program, carl, blind, apologize, suddenly, hanging, bruce, california, chicken, seventy, forward, drinking, sweetheart, medical, suspect, admiral, guard, shop, professor, legs, willing, camp, data, ticket, tree, goodnight, television, losing, senator, murdered, burn, dunno, paying, possibly, trick, dropped, credit, extra, starts, warm, hiding, meaning, sold, stone, taught, marty, lately, cheap, lookin, science, simply, jeff, corner, harold, following, majesty, queen, duty, cars, training, heads, seat, discuss, bear, enemy, helped, noticed, common, screw, dave)
>     topics: Array[Array[(String, Double)]] = Array(Array((know,0.003380285974360492), (just,0.0032710843636220392), (mean,0.0020403580269282785), (going,0.0018412081716573069), (yeah,0.0017292076410595036)), Array((know,0.002722716605262849), (think,0.0023160709876635563), (want,0.0015682033014027693), (tell,0.0013765056029302197), (really,0.0013190996836754377)), Array((know,0.005285852017130324), (gonna,0.004450466114304245), (like,0.003808477141912193), (wait,0.00257579095912931), (yeah,0.002507435691768419)), Array((think,0.006461906040549763), (come,0.0038559336578359326), (like,0.002329773059820553), (just,0.002290146790129036), (time,0.0022316260674415563)), Array((know,0.002839310966830335), (just,0.0020525839506887987), (like,0.0016688236712771678), (good,0.0014274841432802734), (yeah,0.0013850182617787298)), Array((look,0.006430414745047661), (good,0.005126904726913699), (know,0.0038267418634824993), (like,0.0036370167197777273), (wrong,0.002887042876288361)), Array((know,0.007168914271817762), (like,0.0033303014922639114), (maybe,0.001955212601872564), (just,0.0019152703247870325), (didn,0.001636430956103975)), Array((know,0.002439426756082817), (like,0.0022689767941727434), (okay,0.001495915190048933), (think,0.0013473171307872285), (right,0.0012347189576881645)), Array((just,0.001595528475061785), (like,0.001569574938286783), (right,0.0014295885338975265), (good,0.0013466020121047748), (said,0.0011352192707646653)), Array((know,0.016901222618068566), (doing,0.004577649410574112), (think,0.003971192161188001), (like,0.00302620634958447), (going,0.002669274623796597)), Array((know,0.0038140269563962073), (going,0.0031781682895507344), (just,0.0028816331972935914), (like,0.002153017945640159), (think,0.0020204879213200793)), Array((know,0.0015301133248231589), (think,0.0014066243941540167), (want,0.0013542853436208809), (just,0.001304010526888571), (come,0.0011507128067044772)), Array((believe,0.005486844829169185), (love,0.005165142844000885), (like,0.002752377771173137), (yeah,0.002435701398970671), (know,0.002172817525760321)), Array((good,0.0017598365263938752), (watch,0.0014049463901361563), (know,0.0013008619267103779), (just,0.0010998243109867924), (like,0.0010793139503736371)), Array((know,0.003878373108166717), (want,0.0030755628561658353), (right,0.0022294955855522), (going,0.0021720024876757753), (just,0.0018231807557625098)), Array((just,0.0028533537480122657), (admiral,0.0026257588674351673), (captain,0.002238753458417432), (want,0.0018200655345899832), (know,0.0014445516378355958)), Array((know,0.021238887749001945), (just,0.015968186460355357), (like,0.014999425078825599), (want,0.01113115647988978), (think,0.010478256508567313)), Array((yeah,0.00584394109274795), (hello,0.005492554407760574), (just,0.0029978901524305417), (like,0.0018868921065350684), (right,0.001718570248830763)), Array((sure,0.002274494414898221), (right,0.0017297649569008693), (want,0.0014831519281558064), (know,0.0014498408868561115), (like,0.0014485786883881421)), Array((just,0.0013129899588363335), (like,0.0011555068372966766), (going,0.0010399640593116153), (know,8.095270505187395E-4), (time,7.192560022056095E-4)))

Step 9. Create LDA model with Expectation Maximization
------------------------------------------------------

Let's try creating an LDA model with Expectation Maximization on the data that has been refiltered for additional stopwords. We will also increase MaxIterations here to 100 to see if that improves results. See:

-   <http://spark.apache.org/docs/latest/mllib-clustering.html#latent-dirichlet-allocation-lda>.

``` scala
import org.apache.spark.mllib.clustering.EMLDAOptimizer

// Set LDA parameters
val em_lda = new LDA()
.setOptimizer(new EMLDAOptimizer())
.setK(numTopics)
.setMaxIterations(100)
.setDocConcentration(-1) // use default values
.setTopicConcentration(-1) // use default values
```

>     import org.apache.spark.mllib.clustering.EMLDAOptimizer
>     em_lda: org.apache.spark.mllib.clustering.LDA = org.apache.spark.mllib.clustering.LDA@58eb41bb

``` scala
val em_ldaModel = em_lda.run(new_lda_countVector)
```

>     em_ldaModel: org.apache.spark.mllib.clustering.LDAModel = org.apache.spark.mllib.clustering.DistributedLDAModel@4c061b42

``` scala
import org.apache.spark.mllib.clustering.DistributedLDAModel;
val em_DldaModel = em_ldaModel.asInstanceOf[DistributedLDAModel]
```

>     import org.apache.spark.mllib.clustering.DistributedLDAModel
>     em_DldaModel: org.apache.spark.mllib.clustering.DistributedLDAModel = org.apache.spark.mllib.clustering.DistributedLDAModel@4c061b42

``` scala
val top10ConversationsPerTopic = em_DldaModel.topDocumentsPerTopic(10)
```

>     top10ConversationsPerTopic: Array[(Array[Long], Array[Double])] = Array((Array(45776, 25563, 51613, 9235, 68318, 5157, 11905, 24977, 40553, 67360),Array(0.9961264134254786, 0.9950270052543277, 0.9950270052543277, 0.9924001084156447, 0.9921097017756921, 0.9909719072281145, 0.9902587177355318, 0.9902587177355318, 0.9902587177355318, 0.9902587177355318)), (Array(4607, 13728, 20619, 49998, 57763, 68412, 62993, 78110, 77311, 67361),Array(0.9981351648010939, 0.9981351648010939, 0.9981351648010939, 0.9976040780523643, 0.9976040780523643, 0.9976040780523643, 0.9976040780523643, 0.9976040780523643, 0.9976040780523643, 0.9976040780523643)), (Array(22243, 39967, 18136, 18149, 59043, 61513, 34087, 75874, 66270, 68876),Array(0.9986758340945384, 0.99866200816902, 0.9982983538060165, 0.9982983538060165, 0.9982983538060165, 0.9982983538060165, 0.9982983538060165, 0.9982983538060165, 0.9982983538060165, 0.9982983538060165)), (Array(21148, 67869, 12688, 20739, 40083, 40129, 27806, 75641, 1663, 77901),Array(0.9817652189208659, 0.980700428974543, 0.9798397727732211, 0.9789931510969716, 0.9789931510969716, 0.9636375540751915, 0.9636375540751915, 0.9612359093010769, 0.957557550980721, 0.9569465559108808)), (Array(42942, 37341, 15565, 74482, 32456, 25089, 3141, 50336, 82618, 33828),Array(0.9939396829305085, 0.9880586512000573, 0.9871149370176592, 0.9871149370176592, 0.9840808312524657, 0.9835970502807867, 0.9814091796150347, 0.9780807097081116, 0.972269084170973, 0.9708875960232249)), (Array(67529, 868, 50163, 17285, 31256, 39173, 68250, 56529, 57385, 82132),Array(0.9990171162073754, 0.9987989768565186, 0.9987989768565186, 0.9987989768565186, 0.9987989768565186, 0.9987989768565186, 0.9987989768565186, 0.9987989768565186, 0.9984565293818629, 0.9984565293818629)), (Array(896, 79044, 226, 2542, 33809, 73304, 17479, 54249, 44108, 58480),Array(0.9951465128675782, 0.9941697875839299, 0.9937720178281022, 0.9937720178281022, 0.9937720178281022, 0.9937720178281022, 0.9937720178281022, 0.9934015414899598, 0.9934015414899598, 0.9925205038702167)), (Array(15133, 57890, 37120, 62101, 49733, 62224, 19643, 20696, 27094, 74712),Array(0.9961777807935454, 0.9961777807935454, 0.9961777807935454, 0.9961777807935454, 0.9961777807935454, 0.9961777807935454, 0.9909458180572681, 0.9883569450619376, 0.9883569450619376, 0.9871659577050831)), (Array(46112, 32447, 21254, 79186, 12440, 32559, 38008, 45936, 50053, 80392),Array(0.9947043941017288, 0.9947043941017288, 0.9914118273877935, 0.9908995276275648, 0.9896454314885597, 0.9896454314885597, 0.9896454314885597, 0.9896454314885597, 0.9889938386614177, 0.9889938386614177)), (Array(47218, 71056, 66874, 54438, 13157, 61253, 27098, 34759, 78067, 33129),Array(0.9918301515424207, 0.989527598260155, 0.988737685597458, 0.9880786022690486, 0.9880786022690486, 0.9855872102039211, 0.9855872102039211, 0.9855872102039211, 0.9846571642561236, 0.9761849108973557)), (Array(10909, 60039, 63593, 21882, 75809, 14975, 26777, 32670, 31570, 41302),Array(0.9973740341856386, 0.9961189372597797, 0.9961189372597797, 0.9958810463768091, 0.9958810463768091, 0.9949580019223903, 0.9937049015968403, 0.9915056466638248, 0.9911205228856524, 0.9894494048438851)), (Array(49871, 26018, 7226, 20897, 25022, 69680, 7264, 58074, 38442, 927),Array(0.9958001315343138, 0.9948320077327377, 0.993420744064234, 0.993420744064234, 0.993420744064234, 0.993420744064234, 0.993420744064234, 0.9933689215528814, 0.9844728578787995, 0.9836994292694718)), (Array(15479, 4870, 28685, 38717, 46052, 62635, 61273, 52996, 61532, 75144),Array(0.9972304302653124, 0.9964428638445055, 0.9964428638445055, 0.9964428638445055, 0.9964428638445055, 0.9964428638445055, 0.9964428638445055, 0.9942396378765068, 0.9942396378765068, 0.9942396378765068)), (Array(56678, 54226, 37967, 70482, 79904, 55867, 3951, 22481, 15637, 16284),Array(0.9884283619972736, 0.9884283619972736, 0.9884283619972736, 0.9847051914960039, 0.9834197321871639, 0.9806515661478516, 0.9780648613351506, 0.9665807649810713, 0.9635100690930093, 0.9592789291709471)), (Array(1330, 22023, 12743, 11957, 20784, 20836, 58277, 71065, 74242, 60163),Array(0.9949071821036793, 0.9949071821036793, 0.9949071821036793, 0.993464672490183, 0.993464672490183, 0.993464672490183, 0.993464672490183, 0.993464672490183, 0.993464672490183, 0.993464672490183)), (Array(7934, 19175, 61027, 52818, 60405, 74083, 60861, 6618, 82100, 27924),Array(0.9949268017027434, 0.991799421967856, 0.991799421967856, 0.991799421967856, 0.991799421967856, 0.9902468390459326, 0.9884682815008006, 0.9860910086076227, 0.9860910086076227, 0.9848277860275663)), (Array(67818, 7480, 63485, 60855, 63307, 23703, 82500, 82517, 48206, 22670),Array(0.9968162762527881, 0.995087903194635, 0.9929799052183865, 0.9860078579426556, 0.9860078579426556, 0.9851742336964862, 0.9669075510849617, 0.9669075510849617, 0.9579905278544458, 0.9433314243295607)), (Array(62320, 21240, 71931, 48811, 52547, 47548, 1942, 13216, 75832, 77558),Array(0.992895391400667, 0.9902996426102841, 0.9861346032361376, 0.9833673909943765, 0.9821199886922376, 0.9808558567908174, 0.9807256271833505, 0.9782411601710114, 0.9760380328906055, 0.9753126166244693)), (Array(73390, 16886, 30665, 69951, 46486, 45420, 45405, 69981, 56065, 2567),Array(0.9918560757530436, 0.9918560757530436, 0.9911613794647571, 0.9911613794647571, 0.9878914660339583, 0.983227570718236, 0.9829920874239972, 0.9809675308108912, 0.9556902685157634, 0.9553454993545188)), (Array(47079, 49027, 70161, 14247, 70245, 60008, 76300, 73926, 10269, 55704),Array(0.9937115752364689, 0.9917556559535897, 0.9917556559535897, 0.9917556559535897, 0.9917556559535897, 0.9912891905255827, 0.9912891905255827, 0.991219475657918, 0.9907866976083468, 0.9907866976083468)))

``` scala
top10ConversationsPerTopic.length // number of topics
```

>     res53: Int = 20

``` scala
//em_DldaModel.topicDistributions.take(10).foreach(println)
```

Note that the EMLDAOptimizer produces a DistributedLDAModel, which stores not only the inferred topics but also the full training corpus and topic distributions for each document in the training corpus.

``` scala
val topicIndices = em_ldaModel.describeTopics(maxTermsPerTopic = 5)
```

>     topicIndices: Array[(Array[Int], Array[Double])] = Array((Array(1, 2, 3, 135, 6),Array(0.030515134931284552, 0.02463563559747823, 0.022529385381465025, 0.02094828832824297, 0.0203407289886203)), (Array(8, 12, 0, 57, 32),Array(0.10787301090151602, 0.0756831002291994, 0.04815746564274915, 0.03897182014529944, 0.0341458394828345)), (Array(20, 35, 42, 51, 58),Array(0.08118584492034046, 0.051736711600637544, 0.04620430294274594, 0.0399843125556081, 0.03672740843080258)), (Array(22, 0, 34, 43, 4),Array(0.020091372023286612, 0.018613400462887356, 0.016775643603287843, 0.015522555458447744, 0.012161168331925723)), (Array(0, 1, 3, 9, 5),Array(0.031956573561538214, 0.030674598809934856, 0.027663491240851962, 0.025727217382788027, 0.02300853167338119)), (Array(27, 74, 31, 168, 202),Array(0.05932570200934131, 0.030080735900045442, 0.01769248067468245, 0.016281752071881345, 0.014927950883812253)), (Array(53, 92, 180, 113, 166),Array(0.03998401809663685, 0.01737965538107633, 0.016916065536574213, 0.016443441316683228, 0.014849882671062261)), (Array(78, 110, 5, 171, 232),Array(0.028911209424810257, 0.025669944694943093, 0.02091105252727788, 0.017862939987512365, 0.013959164390834044)), (Array(119, 0, 107, 106, 219),Array(0.022939827090645636, 0.021335083902970984, 0.017628999871937747, 0.017302568063786224, 0.012284217866942303)), (Array(0, 2, 24, 1, 3),Array(0.051876601466269136, 0.03828159069993671, 0.03754385940676905, 0.031938551661426284, 0.02876693222824349)), (Array(41, 6, 140, 162, 177),Array(0.032537676027398765, 0.030596831997667568, 0.02049555392502822, 0.018671171294737107, 0.017672067172167016)), (Array(118, 130, 181, 174, 170),Array(0.02236582778896705, 0.020057798194969816, 0.017134198006217606, 0.017075852415410653, 0.017013413435021035)), (Array(18, 62, 2, 114, 122),Array(0.08663446368316245, 0.035120377589734936, 0.02992080326340266, 0.0240813719635157, 0.022471517953608963)), (Array(0, 64, 11, 3, 1),Array(0.0283115823590395, 0.02744935904744228, 0.02050833156294194, 0.020124145131863225, 0.019466336438890477)), (Array(13, 2, 67, 59, 111),Array(0.08220031921979461, 0.05062323326717784, 0.03087838046777391, 0.02452989702353384, 0.022815035397008333)), (Array(158, 11, 233, 274, 295),Array(0.018541518543996716, 0.014737962244588431, 0.012594614743931537, 0.01193707771669708, 0.011260576815409516)), (Array(16, 1, 5, 0, 49),Array(0.08153575328080886, 0.050004142902999975, 0.03438984898476042, 0.02821327795933634, 0.023397063860326372)), (Array(257, 279, 313, 291, 351),Array(0.011270500385627474, 0.010428408353623762, 0.009392162067926028, 0.00799742811584178, 0.007597974486019279)), (Array(0, 4, 17, 14, 1),Array(0.09541058020800194, 0.0698707939786508, 0.06881812755565207, 0.02909700228968688, 0.028699687473471538)), (Array(54, 2, 198, 248, 266),Array(0.03833642117149438, 0.017873711992106994, 0.015280854355409379, 0.013718491413582671, 0.012699265888344448)))

``` scala
val vocabList = vectorizer.vocabulary
```

>     vocabList: Array[String] = Array(know, just, like, want, think, right, going, good, yeah, tell, come, time, look, didn, mean, make, okay, really, little, sure, gonna, thing, people, said, maybe, need, sorry, love, talk, thought, doing, life, night, things, work, money, better, told, long, help, believe, years, shit, does, away, place, hell, doesn, great, home, feel, fuck, kind, remember, dead, course, wouldn, wait, kill, guess, understand, thank, girl, wrong, leave, listen, talking, real, hear, stop, nice, happened, fine, wanted, father, gotta, mind, fucking, house, wasn, getting, world, stay, mother, left, came, care, thanks, knew, room, trying, guys, went, looking, coming, heard, friend, haven, seen, best, tonight, live, used, matter, killed, pretty, business, idea, couldn, head, miss, says, wife, called, woman, morning, tomorrow, start, stuff, saying, play, hello, baby, hard, probably, minute, days, took, somebody, today, school, meet, gone, crazy, wants, damn, forget, problem, cause, deal, case, friends, point, hope, jesus, afraid, looks, knows, year, worry, exactly, aren, half, thinking, shut, hold, wanna, face, minutes, bring, word, read, doctor, everybody, supposed, makes, story, turn, true, watch, thousand, family, brother, kids, week, happen, fuckin, working, open, happy, lost, john, hurt, town, ready, alright, late, actually, married, gave, beautiful, soon, jack, times, sleep, door, having, drink, hand, easy, gets, chance, young, trouble, different, anybody, shot, rest, hate, death, second, later, asked, phone, wish, check, quite, walk, change, police, couple, question, close, taking, heart, hours, making, comes, anymore, truth, trust, dollars, important, captain, telling, funny, person, honey, goes, eyes, reason, inside, stand, break, means, number, tried, high, white, water, suppose, body, sick, game, excuse, party, women, country, answer, christ, waiting, office, send, pick, alive, sort, blood, black, daddy, line, husband, goddamn, book, fifty, thirty, fact, million, died, hands, power, stupid, started, shouldn, months, boys, city, sense, dinner, running, hour, shoot, fight, drive, speak, george, ship, living, figure, dear, street, ahead, lady, seven, scared, free, feeling, frank, able, children, safe, moment, outside, news, president, brought, write, happens, sent, bullshit, lose, light, glad, child, girls, sounds, sister, promise, lives, till, sound, weren, save, poor, cool, shall, asking, plan, king, bitch, daughter, weeks, beat, york, cold, worth, taken, harry, needs, piece, movie, fast, possible, small, goin, straight, human, hair, company, food, tired, lucky, pull, wonderful, touch, looked, thinks, state, picture, leaving, words, control, clear, known, special, buddy, luck, order, follow, expect, mary, catch, mouth, worked, mister, learn, playing, perfect, dream, calling, questions, hospital, takes, ride, coffee, miles, parents, works, secret, hotel, explain, kidding, worse, past, outta, general, felt, drop, unless, throw, interested, hang, certainly, absolutely, earth, loved, dark, wonder, accident, seeing, turned, clock, simple, doin, date, sweet, meeting, clean, sign, feet, handle, music, report, giving, army, fucked, cops, charlie, smart, yesterday, information, fall, fault, bank, class, month, blow, swear, caught, major, paul, road, talked, choice, plane, boss, david, paid, wear, american, worried, lord, paper, goodbye, clothes, ones, terrible, strange, given, mistake, finish, kept, blue, murder, hurry, apartment, sell, middle, nothin, careful, hasn, meant, walter, moving, changed, imagine, fair, difference, quiet, happening, near, quit, personal, marry, figured, future, rose, building, mama, michael, early, agent, kinda, watching, private, trip, record, certain, busy, jimmy, broke, sake, longer, store, boat, stick, finally, born, evening, sitting, bucks, ought, chief, lying, history, kiss, honor, darling, lunch, favor, fool, uncle, respect, rich, land, liked, killing, peter, tough, brain, interesting, completely, problems, welcome, nick, wake, honest, radio, dick, cash, dance, dude, james, bout, floor, weird, court, calls, jail, drunk, window, involved, johnny, officer, needed, asshole, situation, spend, books, relax, pain, service, grand, dangerous, letter, security, stopped, offer, realize, table, bastard, message, instead, killer, jake, deep, nervous, somethin, pass, evil, english, bought, short, step, ring, picked, likes, machine, eddie, voice, upset, forgot, carry, lived, afternoon, fear, quick, finished, count, forgive, wrote, named, decided, totally, space, team, pleasure, doubt, lawyer, station, gotten, suit, bother, prove, return, slow, pictures, bunch, strong, list, wearing, driving, join, tape, christmas, attack, appreciate, force, church, college, hungry, standing, present, dying, prison, missing, charge, board, truck, public, calm, gold, staying, ball, hardly, hadn, missed, lead, island, government, horse, cover, french, reach, joke, fish, star, mike, surprise, america, moved, soul, dress, seconds, club, self, putting, movies, lots, cost, listening, price, saved, smell, mark, peace, dreams, entire, crime, gives, usually, single, department, holy, beer, west, protect, stuck, wall, nose, ways, teach, forever, grow, train, type, awful, rock, detective, billy, walking, dumb, papers, beginning, planet, folks, park, attention, birthday, hide, card, master, share, reading, test, starting, lieutenant, field, partner, enjoy, twice, film, dollar, bomb, mess, blame, south, loves, girlfriend, round, records, using, plenty, especially, gentlemen, evidence, silly, experience, admit, fired, normal, talkin, mission, louis, memory, fighting, lock, notice, crap, wedding, promised, marriage, ground, guns, glass, idiot, orders, impossible, heaven, knock, hole, neck, animal, spent, green, wondering, nuts, press, drugs, broken, position, names, asleep, jerry, visit, boyfriend, acting, feels, plans, paris, smoke, tells, wind, cross, holding, sheriff, gimme, walked, mention, writing, double, brothers, code, judge, pardon, keeps, fellow, fell, closed, lovely, angry, cute, charles, surprised, percent, correct, bathroom, agree, address, andy, ridiculous, summer, tommy, rules, group, account, note, pulled, sleeping, sing, learned, proud, laugh, colonel, upstairs, river, difficult, built, jump, area, dirty, betty, bridge, breakfast, bobby, locked, amazing, north, feelings, alex, plus, definitely, worst, accept, kick, seriously, grace, steal, wild, stories, file, gettin, relationship, advice, nature, contact, spot, places, waste, knowing, beach, stole, apart, favorite, faith, level, loose, risk, song, eating, foot, played, patient, washington, turns, witness, action, build, obviously, begin, split, crew, command, games, tight, decide, nurse, keeping, runs, form, bird, copy, insane, complete, arrest, consider, taste, scene, jeffrey, teeth, shoes, career, henry, sooner, devil, monster, showed, weekend, gift, innocent, study, heavy, hall, comin, danger, greatest, track, keys, raise, destroy, concerned, program, carl, blind, apologize, suddenly, hanging, bruce, california, chicken, seventy, forward, drinking, sweetheart, medical, suspect, admiral, guard, shop, professor, legs, willing, camp, data, ticket, tree, goodnight, television, losing, senator, murdered, burn, dunno, paying, possibly, trick, dropped, credit, extra, starts, warm, hiding, meaning, sold, stone, taught, marty, lately, cheap, lookin, science, simply, jeff, corner, harold, following, majesty, queen, duty, cars, training, heads, seat, discuss, bear, enemy, helped, noticed, common, screw, dave)

``` scala
vocabList.size
```

>     res32: Int = 10000

``` scala
val topics = topicIndices.map { case (terms, termWeights) =>
  terms.map(vocabList(_)).zip(termWeights)
}
```

>     topics: Array[Array[(String, Double)]] = Array(Array((just,0.030515134931284552), (like,0.02463563559747823), (want,0.022529385381465025), (damn,0.02094828832824297), (going,0.0203407289886203)), Array((yeah,0.10787301090151602), (look,0.0756831002291994), (know,0.04815746564274915), (wait,0.03897182014529944), (night,0.0341458394828345)), Array((gonna,0.08118584492034046), (money,0.051736711600637544), (shit,0.04620430294274594), (fuck,0.0399843125556081), (kill,0.03672740843080258)), Array((people,0.020091372023286612), (know,0.018613400462887356), (work,0.016775643603287843), (does,0.015522555458447744), (think,0.012161168331925723)), Array((know,0.031956573561538214), (just,0.030674598809934856), (want,0.027663491240851962), (tell,0.025727217382788027), (right,0.02300853167338119)), Array((love,0.05932570200934131), (father,0.030080735900045442), (life,0.01769248067468245), (true,0.016281752071881345), (young,0.014927950883812253)), Array((remember,0.03998401809663685), (went,0.01737965538107633), (lost,0.016916065536574213), (called,0.016443441316683228), (story,0.014849882671062261)), Array((house,0.028911209424810257), (miss,0.025669944694943093), (right,0.02091105252727788), (family,0.017862939987512365), (important,0.013959164390834044)), Array((saying,0.022939827090645636), (know,0.021335083902970984), (idea,0.017628999871937747), (business,0.017302568063786224), (police,0.012284217866942303)), Array((know,0.051876601466269136), (like,0.03828159069993671), (maybe,0.03754385940676905), (just,0.031938551661426284), (want,0.02876693222824349)), Array((years,0.032537676027398765), (going,0.030596831997667568), (case,0.02049555392502822), (doctor,0.018671171294737107), (working,0.017672067172167016)), Array((stuff,0.02236582778896705), (school,0.020057798194969816), (john,0.017134198006217606), (week,0.017075852415410653), (thousand,0.017013413435021035)), Array((little,0.08663446368316245), (girl,0.035120377589734936), (like,0.02992080326340266), (woman,0.0240813719635157), (baby,0.022471517953608963)), Array((know,0.0283115823590395), (leave,0.02744935904744228), (time,0.02050833156294194), (want,0.020124145131863225), (just,0.019466336438890477)), Array((didn,0.08220031921979461), (like,0.05062323326717784), (real,0.03087838046777391), (guess,0.02452989702353384), (says,0.022815035397008333)), Array((minutes,0.018541518543996716), (time,0.014737962244588431), (captain,0.012594614743931537), (thirty,0.01193707771669708), (ship,0.011260576815409516)), Array((okay,0.08153575328080886), (just,0.050004142902999975), (right,0.03438984898476042), (know,0.02821327795933634), (home,0.023397063860326372)), Array((country,0.011270500385627474), (power,0.010428408353623762), (president,0.009392162067926028), (fight,0.00799742811584178), (possible,0.007597974486019279)), Array((know,0.09541058020800194), (think,0.0698707939786508), (really,0.06881812755565207), (mean,0.02909700228968688), (just,0.028699687473471538)), Array((dead,0.03833642117149438), (like,0.017873711992106994), (hand,0.015280854355409379), (white,0.013718491413582671), (blood,0.012699265888344448)))

``` scala
vocabList(47) // 47 is the index of the term 'university' or the first term in topics - this may change due to randomness in algorithm
```

>     res33: String = doesn

This is just doing it all at once.

``` scala
val topicIndices = em_ldaModel.describeTopics(maxTermsPerTopic = 5)
val vocabList = vectorizer.vocabulary
val topics = topicIndices.map { case (terms, termWeights) =>
  terms.map(vocabList(_)).zip(termWeights)
}
println(s"$numTopics topics:")
topics.zipWithIndex.foreach { case (topic, i) =>
  println(s"TOPIC $i")
  topic.foreach { case (term, weight) => println(s"$term\t$weight") }
  println(s"==========")
}
```

>     20 topics:
>     TOPIC 0
>     just	0.030515134931284552
>     like	0.02463563559747823
>     want	0.022529385381465025
>     damn	0.02094828832824297
>     going	0.0203407289886203
>     ==========
>     TOPIC 1
>     yeah	0.10787301090151602
>     look	0.0756831002291994
>     know	0.04815746564274915
>     wait	0.03897182014529944
>     night	0.0341458394828345
>     ==========
>     TOPIC 2
>     gonna	0.08118584492034046
>     money	0.051736711600637544
>     shit	0.04620430294274594
>     fuck	0.0399843125556081
>     kill	0.03672740843080258
>     ==========
>     TOPIC 3
>     people	0.020091372023286612
>     know	0.018613400462887356
>     work	0.016775643603287843
>     does	0.015522555458447744
>     think	0.012161168331925723
>     ==========
>     TOPIC 4
>     know	0.031956573561538214
>     just	0.030674598809934856
>     want	0.027663491240851962
>     tell	0.025727217382788027
>     right	0.02300853167338119
>     ==========
>     TOPIC 5
>     love	0.05932570200934131
>     father	0.030080735900045442
>     life	0.01769248067468245
>     true	0.016281752071881345
>     young	0.014927950883812253
>     ==========
>     TOPIC 6
>     remember	0.03998401809663685
>     went	0.01737965538107633
>     lost	0.016916065536574213
>     called	0.016443441316683228
>     story	0.014849882671062261
>     ==========
>     TOPIC 7
>     house	0.028911209424810257
>     miss	0.025669944694943093
>     right	0.02091105252727788
>     family	0.017862939987512365
>     important	0.013959164390834044
>     ==========
>     TOPIC 8
>     saying	0.022939827090645636
>     know	0.021335083902970984
>     idea	0.017628999871937747
>     business	0.017302568063786224
>     police	0.012284217866942303
>     ==========
>     TOPIC 9
>     know	0.051876601466269136
>     like	0.03828159069993671
>     maybe	0.03754385940676905
>     just	0.031938551661426284
>     want	0.02876693222824349
>     ==========
>     TOPIC 10
>     years	0.032537676027398765
>     going	0.030596831997667568
>     case	0.02049555392502822
>     doctor	0.018671171294737107
>     working	0.017672067172167016
>     ==========
>     TOPIC 11
>     stuff	0.02236582778896705
>     school	0.020057798194969816
>     john	0.017134198006217606
>     week	0.017075852415410653
>     thousand	0.017013413435021035
>     ==========
>     TOPIC 12
>     little	0.08663446368316245
>     girl	0.035120377589734936
>     like	0.02992080326340266
>     woman	0.0240813719635157
>     baby	0.022471517953608963
>     ==========
>     TOPIC 13
>     know	0.0283115823590395
>     leave	0.02744935904744228
>     time	0.02050833156294194
>     want	0.020124145131863225
>     just	0.019466336438890477
>     ==========
>     TOPIC 14
>     didn	0.08220031921979461
>     like	0.05062323326717784
>     real	0.03087838046777391
>     guess	0.02452989702353384
>     says	0.022815035397008333
>     ==========
>     TOPIC 15
>     minutes	0.018541518543996716
>     time	0.014737962244588431
>     captain	0.012594614743931537
>     thirty	0.01193707771669708
>     ship	0.011260576815409516
>     ==========
>     TOPIC 16
>     okay	0.08153575328080886
>     just	0.050004142902999975
>     right	0.03438984898476042
>     know	0.02821327795933634
>     home	0.023397063860326372
>     ==========
>     TOPIC 17
>     country	0.011270500385627474
>     power	0.010428408353623762
>     president	0.009392162067926028
>     fight	0.00799742811584178
>     possible	0.007597974486019279
>     ==========
>     TOPIC 18
>     know	0.09541058020800194
>     think	0.0698707939786508
>     really	0.06881812755565207
>     mean	0.02909700228968688
>     just	0.028699687473471538
>     ==========
>     TOPIC 19
>     dead	0.03833642117149438
>     like	0.017873711992106994
>     hand	0.015280854355409379
>     white	0.013718491413582671
>     blood	0.012699265888344448
>     ==========
>     topicIndices: Array[(Array[Int], Array[Double])] = Array((Array(1, 2, 3, 135, 6),Array(0.030515134931284552, 0.02463563559747823, 0.022529385381465025, 0.02094828832824297, 0.0203407289886203)), (Array(8, 12, 0, 57, 32),Array(0.10787301090151602, 0.0756831002291994, 0.04815746564274915, 0.03897182014529944, 0.0341458394828345)), (Array(20, 35, 42, 51, 58),Array(0.08118584492034046, 0.051736711600637544, 0.04620430294274594, 0.0399843125556081, 0.03672740843080258)), (Array(22, 0, 34, 43, 4),Array(0.020091372023286612, 0.018613400462887356, 0.016775643603287843, 0.015522555458447744, 0.012161168331925723)), (Array(0, 1, 3, 9, 5),Array(0.031956573561538214, 0.030674598809934856, 0.027663491240851962, 0.025727217382788027, 0.02300853167338119)), (Array(27, 74, 31, 168, 202),Array(0.05932570200934131, 0.030080735900045442, 0.01769248067468245, 0.016281752071881345, 0.014927950883812253)), (Array(53, 92, 180, 113, 166),Array(0.03998401809663685, 0.01737965538107633, 0.016916065536574213, 0.016443441316683228, 0.014849882671062261)), (Array(78, 110, 5, 171, 232),Array(0.028911209424810257, 0.025669944694943093, 0.02091105252727788, 0.017862939987512365, 0.013959164390834044)), (Array(119, 0, 107, 106, 219),Array(0.022939827090645636, 0.021335083902970984, 0.017628999871937747, 0.017302568063786224, 0.012284217866942303)), (Array(0, 2, 24, 1, 3),Array(0.051876601466269136, 0.03828159069993671, 0.03754385940676905, 0.031938551661426284, 0.02876693222824349)), (Array(41, 6, 140, 162, 177),Array(0.032537676027398765, 0.030596831997667568, 0.02049555392502822, 0.018671171294737107, 0.017672067172167016)), (Array(118, 130, 181, 174, 170),Array(0.02236582778896705, 0.020057798194969816, 0.017134198006217606, 0.017075852415410653, 0.017013413435021035)), (Array(18, 62, 2, 114, 122),Array(0.08663446368316245, 0.035120377589734936, 0.02992080326340266, 0.0240813719635157, 0.022471517953608963)), (Array(0, 64, 11, 3, 1),Array(0.0283115823590395, 0.02744935904744228, 0.02050833156294194, 0.020124145131863225, 0.019466336438890477)), (Array(13, 2, 67, 59, 111),Array(0.08220031921979461, 0.05062323326717784, 0.03087838046777391, 0.02452989702353384, 0.022815035397008333)), (Array(158, 11, 233, 274, 295),Array(0.018541518543996716, 0.014737962244588431, 0.012594614743931537, 0.01193707771669708, 0.011260576815409516)), (Array(16, 1, 5, 0, 49),Array(0.08153575328080886, 0.050004142902999975, 0.03438984898476042, 0.02821327795933634, 0.023397063860326372)), (Array(257, 279, 313, 291, 351),Array(0.011270500385627474, 0.010428408353623762, 0.009392162067926028, 0.00799742811584178, 0.007597974486019279)), (Array(0, 4, 17, 14, 1),Array(0.09541058020800194, 0.0698707939786508, 0.06881812755565207, 0.02909700228968688, 0.028699687473471538)), (Array(54, 2, 198, 248, 266),Array(0.03833642117149438, 0.017873711992106994, 0.015280854355409379, 0.013718491413582671, 0.012699265888344448)))
>     vocabList: Array[String] = Array(know, just, like, want, think, right, going, good, yeah, tell, come, time, look, didn, mean, make, okay, really, little, sure, gonna, thing, people, said, maybe, need, sorry, love, talk, thought, doing, life, night, things, work, money, better, told, long, help, believe, years, shit, does, away, place, hell, doesn, great, home, feel, fuck, kind, remember, dead, course, wouldn, wait, kill, guess, understand, thank, girl, wrong, leave, listen, talking, real, hear, stop, nice, happened, fine, wanted, father, gotta, mind, fucking, house, wasn, getting, world, stay, mother, left, came, care, thanks, knew, room, trying, guys, went, looking, coming, heard, friend, haven, seen, best, tonight, live, used, matter, killed, pretty, business, idea, couldn, head, miss, says, wife, called, woman, morning, tomorrow, start, stuff, saying, play, hello, baby, hard, probably, minute, days, took, somebody, today, school, meet, gone, crazy, wants, damn, forget, problem, cause, deal, case, friends, point, hope, jesus, afraid, looks, knows, year, worry, exactly, aren, half, thinking, shut, hold, wanna, face, minutes, bring, word, read, doctor, everybody, supposed, makes, story, turn, true, watch, thousand, family, brother, kids, week, happen, fuckin, working, open, happy, lost, john, hurt, town, ready, alright, late, actually, married, gave, beautiful, soon, jack, times, sleep, door, having, drink, hand, easy, gets, chance, young, trouble, different, anybody, shot, rest, hate, death, second, later, asked, phone, wish, check, quite, walk, change, police, couple, question, close, taking, heart, hours, making, comes, anymore, truth, trust, dollars, important, captain, telling, funny, person, honey, goes, eyes, reason, inside, stand, break, means, number, tried, high, white, water, suppose, body, sick, game, excuse, party, women, country, answer, christ, waiting, office, send, pick, alive, sort, blood, black, daddy, line, husband, goddamn, book, fifty, thirty, fact, million, died, hands, power, stupid, started, shouldn, months, boys, city, sense, dinner, running, hour, shoot, fight, drive, speak, george, ship, living, figure, dear, street, ahead, lady, seven, scared, free, feeling, frank, able, children, safe, moment, outside, news, president, brought, write, happens, sent, bullshit, lose, light, glad, child, girls, sounds, sister, promise, lives, till, sound, weren, save, poor, cool, shall, asking, plan, king, bitch, daughter, weeks, beat, york, cold, worth, taken, harry, needs, piece, movie, fast, possible, small, goin, straight, human, hair, company, food, tired, lucky, pull, wonderful, touch, looked, thinks, state, picture, leaving, words, control, clear, known, special, buddy, luck, order, follow, expect, mary, catch, mouth, worked, mister, learn, playing, perfect, dream, calling, questions, hospital, takes, ride, coffee, miles, parents, works, secret, hotel, explain, kidding, worse, past, outta, general, felt, drop, unless, throw, interested, hang, certainly, absolutely, earth, loved, dark, wonder, accident, seeing, turned, clock, simple, doin, date, sweet, meeting, clean, sign, feet, handle, music, report, giving, army, fucked, cops, charlie, smart, yesterday, information, fall, fault, bank, class, month, blow, swear, caught, major, paul, road, talked, choice, plane, boss, david, paid, wear, american, worried, lord, paper, goodbye, clothes, ones, terrible, strange, given, mistake, finish, kept, blue, murder, hurry, apartment, sell, middle, nothin, careful, hasn, meant, walter, moving, changed, imagine, fair, difference, quiet, happening, near, quit, personal, marry, figured, future, rose, building, mama, michael, early, agent, kinda, watching, private, trip, record, certain, busy, jimmy, broke, sake, longer, store, boat, stick, finally, born, evening, sitting, bucks, ought, chief, lying, history, kiss, honor, darling, lunch, favor, fool, uncle, respect, rich, land, liked, killing, peter, tough, brain, interesting, completely, problems, welcome, nick, wake, honest, radio, dick, cash, dance, dude, james, bout, floor, weird, court, calls, jail, drunk, window, involved, johnny, officer, needed, asshole, situation, spend, books, relax, pain, service, grand, dangerous, letter, security, stopped, offer, realize, table, bastard, message, instead, killer, jake, deep, nervous, somethin, pass, evil, english, bought, short, step, ring, picked, likes, machine, eddie, voice, upset, forgot, carry, lived, afternoon, fear, quick, finished, count, forgive, wrote, named, decided, totally, space, team, pleasure, doubt, lawyer, station, gotten, suit, bother, prove, return, slow, pictures, bunch, strong, list, wearing, driving, join, tape, christmas, attack, appreciate, force, church, college, hungry, standing, present, dying, prison, missing, charge, board, truck, public, calm, gold, staying, ball, hardly, hadn, missed, lead, island, government, horse, cover, french, reach, joke, fish, star, mike, surprise, america, moved, soul, dress, seconds, club, self, putting, movies, lots, cost, listening, price, saved, smell, mark, peace, dreams, entire, crime, gives, usually, single, department, holy, beer, west, protect, stuck, wall, nose, ways, teach, forever, grow, train, type, awful, rock, detective, billy, walking, dumb, papers, beginning, planet, folks, park, attention, birthday, hide, card, master, share, reading, test, starting, lieutenant, field, partner, enjoy, twice, film, dollar, bomb, mess, blame, south, loves, girlfriend, round, records, using, plenty, especially, gentlemen, evidence, silly, experience, admit, fired, normal, talkin, mission, louis, memory, fighting, lock, notice, crap, wedding, promised, marriage, ground, guns, glass, idiot, orders, impossible, heaven, knock, hole, neck, animal, spent, green, wondering, nuts, press, drugs, broken, position, names, asleep, jerry, visit, boyfriend, acting, feels, plans, paris, smoke, tells, wind, cross, holding, sheriff, gimme, walked, mention, writing, double, brothers, code, judge, pardon, keeps, fellow, fell, closed, lovely, angry, cute, charles, surprised, percent, correct, bathroom, agree, address, andy, ridiculous, summer, tommy, rules, group, account, note, pulled, sleeping, sing, learned, proud, laugh, colonel, upstairs, river, difficult, built, jump, area, dirty, betty, bridge, breakfast, bobby, locked, amazing, north, feelings, alex, plus, definitely, worst, accept, kick, seriously, grace, steal, wild, stories, file, gettin, relationship, advice, nature, contact, spot, places, waste, knowing, beach, stole, apart, favorite, faith, level, loose, risk, song, eating, foot, played, patient, washington, turns, witness, action, build, obviously, begin, split, crew, command, games, tight, decide, nurse, keeping, runs, form, bird, copy, insane, complete, arrest, consider, taste, scene, jeffrey, teeth, shoes, career, henry, sooner, devil, monster, showed, weekend, gift, innocent, study, heavy, hall, comin, danger, greatest, track, keys, raise, destroy, concerned, program, carl, blind, apologize, suddenly, hanging, bruce, california, chicken, seventy, forward, drinking, sweetheart, medical, suspect, admiral, guard, shop, professor, legs, willing, camp, data, ticket, tree, goodnight, television, losing, senator, murdered, burn, dunno, paying, possibly, trick, dropped, credit, extra, starts, warm, hiding, meaning, sold, stone, taught, marty, lately, cheap, lookin, science, simply, jeff, corner, harold, following, majesty, queen, duty, cars, training, heads, seat, discuss, bear, enemy, helped, noticed, common, screw, dave)
>     topics: Array[Array[(String, Double)]] = Array(Array((just,0.030515134931284552), (like,0.02463563559747823), (want,0.022529385381465025), (damn,0.02094828832824297), (going,0.0203407289886203)), Array((yeah,0.10787301090151602), (look,0.0756831002291994), (know,0.04815746564274915), (wait,0.03897182014529944), (night,0.0341458394828345)), Array((gonna,0.08118584492034046), (money,0.051736711600637544), (shit,0.04620430294274594), (fuck,0.0399843125556081), (kill,0.03672740843080258)), Array((people,0.020091372023286612), (know,0.018613400462887356), (work,0.016775643603287843), (does,0.015522555458447744), (think,0.012161168331925723)), Array((know,0.031956573561538214), (just,0.030674598809934856), (want,0.027663491240851962), (tell,0.025727217382788027), (right,0.02300853167338119)), Array((love,0.05932570200934131), (father,0.030080735900045442), (life,0.01769248067468245), (true,0.016281752071881345), (young,0.014927950883812253)), Array((remember,0.03998401809663685), (went,0.01737965538107633), (lost,0.016916065536574213), (called,0.016443441316683228), (story,0.014849882671062261)), Array((house,0.028911209424810257), (miss,0.025669944694943093), (right,0.02091105252727788), (family,0.017862939987512365), (important,0.013959164390834044)), Array((saying,0.022939827090645636), (know,0.021335083902970984), (idea,0.017628999871937747), (business,0.017302568063786224), (police,0.012284217866942303)), Array((know,0.051876601466269136), (like,0.03828159069993671), (maybe,0.03754385940676905), (just,0.031938551661426284), (want,0.02876693222824349)), Array((years,0.032537676027398765), (going,0.030596831997667568), (case,0.02049555392502822), (doctor,0.018671171294737107), (working,0.017672067172167016)), Array((stuff,0.02236582778896705), (school,0.020057798194969816), (john,0.017134198006217606), (week,0.017075852415410653), (thousand,0.017013413435021035)), Array((little,0.08663446368316245), (girl,0.035120377589734936), (like,0.02992080326340266), (woman,0.0240813719635157), (baby,0.022471517953608963)), Array((know,0.0283115823590395), (leave,0.02744935904744228), (time,0.02050833156294194), (want,0.020124145131863225), (just,0.019466336438890477)), Array((didn,0.08220031921979461), (like,0.05062323326717784), (real,0.03087838046777391), (guess,0.02452989702353384), (says,0.022815035397008333)), Array((minutes,0.018541518543996716), (time,0.014737962244588431), (captain,0.012594614743931537), (thirty,0.01193707771669708), (ship,0.011260576815409516)), Array((okay,0.08153575328080886), (just,0.050004142902999975), (right,0.03438984898476042), (know,0.02821327795933634), (home,0.023397063860326372)), Array((country,0.011270500385627474), (power,0.010428408353623762), (president,0.009392162067926028), (fight,0.00799742811584178), (possible,0.007597974486019279)), Array((know,0.09541058020800194), (think,0.0698707939786508), (really,0.06881812755565207), (mean,0.02909700228968688), (just,0.028699687473471538)), Array((dead,0.03833642117149438), (like,0.017873711992106994), (hand,0.015280854355409379), (white,0.013718491413582671), (blood,0.012699265888344448)))

``` scala
top10ConversationsPerTopic(2)
```

>     res54: (Array[Long], Array[Double]) = (Array(22243, 39967, 18136, 18149, 59043, 61513, 34087, 75874, 66270, 68876),Array(0.9986758340945384, 0.99866200816902, 0.9982983538060165, 0.9982983538060165, 0.9982983538060165, 0.9982983538060165, 0.9982983538060165, 0.9982983538060165, 0.9982983538060165, 0.9982983538060165))

``` scala
top10ConversationsPerTopic(2)._1
```

>     res55: Array[Long] = Array(22243, 39967, 18136, 18149, 59043, 61513, 34087, 75874, 66270, 68876)

``` scala
val scenesForTopic2 = sc.parallelize(top10ConversationsPerTopic(2)._1).toDF("id")
```

>     scenesForTopic2: org.apache.spark.sql.DataFrame = [id: bigint]

``` scala
display(scenesForTopic2.join(corpusDF,"id"))
```

| id      | corpus                                                                             | movieTitle             | movieYear |
|---------|------------------------------------------------------------------------------------|------------------------|-----------|
| 22243.0 | Fuck him. :-()-: Don't. :-()-: Fuck her too.                                       | panic room             | 2002      |
| 59043.0 | Are you ok? :-()-: Fuck no.                                                        | magnolia               | 1999      |
| 66270.0 | Hey now... what the fuck... ? :-()-: Again.                                        | red white black & blue | 2006      |
| 75874.0 | What about Moliere? :-()-: Fuck off.                                               | the beach              | 2000/I    |
| 68876.0 | What the fuck is that? :-()-: A switchblade.                                       | seven                  | 1979      |
| 34087.0 | Fuck me!  Yes! :-()-: Uh...                                                        | american pie           | 1999      |
| 61513.0 | What the fuck is that?! :-()-: Screamer.                                           | arcade                 | 1993      |
| 18136.0 | What the fuck was that about? :-()-: She was jonesing for me.                      | made                   | 2001      |
| 18149.0 | C'mon... :-()-: Fuck...                                                            | made                   | 2001      |
| 39967.0 | Shit, shit, shit... :-()-: You're almost there, you can do it -- can do -- can do. | broadcast news         | 1987      |

``` scala
sc.parallelize(top10ConversationsPerTopic(2)._1).toDF("id").join(corpusDF,"id").show(10,false)
```

>     +-----+----------------------------------------------------------------------------------+----------------------+---------+
>     |id   |corpus                                                                            |movieTitle            |movieYear|
>     +-----+----------------------------------------------------------------------------------+----------------------+---------+
>     |22243|Fuck him. :-()-: Don't. :-()-: Fuck her too.                                      |panic room            |2002     |
>     |59043|Are you ok? :-()-: Fuck no.                                                       |magnolia              |1999     |
>     |66270|Hey now... what the fuck... ? :-()-: Again.                                       |red white black & blue|2006     |
>     |75874|What about Moliere? :-()-: Fuck off.                                              |the beach             |2000/I   |
>     |68876|What the fuck is that? :-()-: A switchblade.                                      |seven                 |1979     |
>     |34087|Fuck me!  Yes! :-()-: Uh...                                                       |american pie          |1999     |
>     |61513|What the fuck is that?! :-()-: Screamer.                                          |arcade                |1993     |
>     |18136|What the fuck was that about? :-()-: She was jonesing for me.                     |made                  |2001     |
>     |18149|C'mon... :-()-: Fuck...                                                           |made                  |2001     |
>     |39967|Shit, shit, shit... :-()-: You're almost there, you can do it -- can do -- can do.|broadcast news        |1987     |
>     +-----+----------------------------------------------------------------------------------+----------------------+---------+

``` scala
sc.parallelize(top10ConversationsPerTopic(5)._1).toDF("id").join(corpusDF,"id").show(10,false)
```

>     +-----+---------------------------------------------------------+-----------------+---------+
>     |id   |corpus                                                   |movieTitle       |movieYear|
>     +-----+---------------------------------------------------------+-----------------+---------+
>     |68250|I love you man :-()-: I love you too.                    |say anything...  |1989     |
>     |31256|I love you. :-()-: I love you.                           |total recall     |1990     |
>     |868  |I love you. :-()-: I love you.                           |8mm              |1999     |
>     |17285|Do me. :-()-: I love you. :-()-: I love you.             |little nicky     |2000     |
>     |56529|Why do you love me? :-()-: Why do you love me?           |jerry maguire    |1996     |
>     |67529|I love you, too. :-()-: I love you.  I love you.         |runaway bride    |1999     |
>     |82132|Why did you say that? :-()-: Say what? :-()-: I love you.|willow           |1988     |
>     |50163|I love you, Bud. :-()-: I love you more.                 |frequency        |2000     |
>     |39173|I love you. :-()-: I love you too, Dad.                  |body of evidence |1993     |
>     |57385|Yes? :-()-: I love you...                                |kramer vs. kramer|1979     |
>     +-----+---------------------------------------------------------+-----------------+---------+

``` scala
corpusDF.show(5)
```

>     +-----+--------------------+------------+---------+
>     |   id|              corpus|  movieTitle|movieYear|
>     +-----+--------------------+------------+---------+
>     |17668|This would be fun...|lost horizon|     1937|
>     |17598|Cave, eh? Where? ...|lost horizon|     1937|
>     |17663|Something grand a...|lost horizon|     1937|
>     |17593|You see? You get ...|lost horizon|     1937|
>     |17658|Let me up! Let me...|lost horizon|     1937|
>     +-----+--------------------+------------+---------+
>     only showing top 5 rows

We've managed to get some good results here. For example, we can easily infer that Topic 2 is about space, Topic 3 is about israel, etc.

We still get some ambiguous results like Topic 0.

To improve our results further, we could employ some of the below methods:

-   Refilter data for additional data-specific stopwords
-   Use Stemming or Lemmatization to preprocess data
-   Experiment with a smaller number of topics, since some of these topics in the 20 Newsgroups are pretty similar
-   Increase model's MaxIterations

Visualize Results
-----------------

We will try visualizing the results obtained from the EM LDA model with a d3 bubble chart.

``` scala
// Zip topic terms with topic IDs
val termArray = topics.zipWithIndex
```

>     termArray: Array[(Array[(String, Double)], Int)] = Array((Array((just,0.030515134931284552), (like,0.02463563559747823), (want,0.022529385381465025), (damn,0.02094828832824297), (going,0.0203407289886203)),0), (Array((yeah,0.10787301090151602), (look,0.0756831002291994), (know,0.04815746564274915), (wait,0.03897182014529944), (night,0.0341458394828345)),1), (Array((gonna,0.08118584492034046), (money,0.051736711600637544), (shit,0.04620430294274594), (fuck,0.0399843125556081), (kill,0.03672740843080258)),2), (Array((people,0.020091372023286612), (know,0.018613400462887356), (work,0.016775643603287843), (does,0.015522555458447744), (think,0.012161168331925723)),3), (Array((know,0.031956573561538214), (just,0.030674598809934856), (want,0.027663491240851962), (tell,0.025727217382788027), (right,0.02300853167338119)),4), (Array((love,0.05932570200934131), (father,0.030080735900045442), (life,0.01769248067468245), (true,0.016281752071881345), (young,0.014927950883812253)),5), (Array((remember,0.03998401809663685), (went,0.01737965538107633), (lost,0.016916065536574213), (called,0.016443441316683228), (story,0.014849882671062261)),6), (Array((house,0.028911209424810257), (miss,0.025669944694943093), (right,0.02091105252727788), (family,0.017862939987512365), (important,0.013959164390834044)),7), (Array((saying,0.022939827090645636), (know,0.021335083902970984), (idea,0.017628999871937747), (business,0.017302568063786224), (police,0.012284217866942303)),8), (Array((know,0.051876601466269136), (like,0.03828159069993671), (maybe,0.03754385940676905), (just,0.031938551661426284), (want,0.02876693222824349)),9), (Array((years,0.032537676027398765), (going,0.030596831997667568), (case,0.02049555392502822), (doctor,0.018671171294737107), (working,0.017672067172167016)),10), (Array((stuff,0.02236582778896705), (school,0.020057798194969816), (john,0.017134198006217606), (week,0.017075852415410653), (thousand,0.017013413435021035)),11), (Array((little,0.08663446368316245), (girl,0.035120377589734936), (like,0.02992080326340266), (woman,0.0240813719635157), (baby,0.022471517953608963)),12), (Array((know,0.0283115823590395), (leave,0.02744935904744228), (time,0.02050833156294194), (want,0.020124145131863225), (just,0.019466336438890477)),13), (Array((didn,0.08220031921979461), (like,0.05062323326717784), (real,0.03087838046777391), (guess,0.02452989702353384), (says,0.022815035397008333)),14), (Array((minutes,0.018541518543996716), (time,0.014737962244588431), (captain,0.012594614743931537), (thirty,0.01193707771669708), (ship,0.011260576815409516)),15), (Array((okay,0.08153575328080886), (just,0.050004142902999975), (right,0.03438984898476042), (know,0.02821327795933634), (home,0.023397063860326372)),16), (Array((country,0.011270500385627474), (power,0.010428408353623762), (president,0.009392162067926028), (fight,0.00799742811584178), (possible,0.007597974486019279)),17), (Array((know,0.09541058020800194), (think,0.0698707939786508), (really,0.06881812755565207), (mean,0.02909700228968688), (just,0.028699687473471538)),18), (Array((dead,0.03833642117149438), (like,0.017873711992106994), (hand,0.015280854355409379), (white,0.013718491413582671), (blood,0.012699265888344448)),19))

``` scala
// Transform data into the form (term, probability, topicId)
val termRDD = sc.parallelize(termArray)
val termRDD2 =termRDD.flatMap( (x: (Array[(String, Double)], Int)) => {
  val arrayOfTuple = x._1
  val topicId = x._2
  arrayOfTuple.map(el => (el._1, el._2, topicId))
})
```

>     termRDD: org.apache.spark.rdd.RDD[(Array[(String, Double)], Int)] = ParallelCollectionRDD[3066] at parallelize at <console>:109
>     termRDD2: org.apache.spark.rdd.RDD[(String, Double, Int)] = MapPartitionsRDD[3067] at flatMap at <console>:110

``` scala
// Create DF with proper column names
val termDF = termRDD2.toDF.withColumnRenamed("_1", "term").withColumnRenamed("_2", "probability").withColumnRenamed("_3", "topicId")
```

>     termDF: org.apache.spark.sql.DataFrame = [term: string, probability: double, topicId: int]

``` scala
display(termDF)
```

| term   | probability           | topicId |
|--------|-----------------------|---------|
| just   | 3.0515134931284552e-2 | 0.0     |
| like   | 2.463563559747823e-2  | 0.0     |
| want   | 2.2529385381465025e-2 | 0.0     |
| damn   | 2.094828832824297e-2  | 0.0     |
| going  | 2.03407289886203e-2   | 0.0     |
| yeah   | 0.10787301090151602   | 1.0     |
| look   | 7.56831002291994e-2   | 1.0     |
| know   | 4.815746564274915e-2  | 1.0     |
| wait   | 3.897182014529944e-2  | 1.0     |
| night  | 3.41458394828345e-2   | 1.0     |
| gonna  | 8.118584492034046e-2  | 2.0     |
| money  | 5.1736711600637544e-2 | 2.0     |
| shit   | 4.620430294274594e-2  | 2.0     |
| fuck   | 3.99843125556081e-2   | 2.0     |
| kill   | 3.672740843080258e-2  | 2.0     |
| people | 2.0091372023286612e-2 | 3.0     |
| know   | 1.8613400462887356e-2 | 3.0     |
| work   | 1.6775643603287843e-2 | 3.0     |
| does   | 1.5522555458447744e-2 | 3.0     |
| think  | 1.2161168331925723e-2 | 3.0     |
| know   | 3.1956573561538214e-2 | 4.0     |
| just   | 3.0674598809934856e-2 | 4.0     |
| want   | 2.7663491240851962e-2 | 4.0     |
| tell   | 2.5727217382788027e-2 | 4.0     |
| right  | 2.300853167338119e-2  | 4.0     |
| love   | 5.932570200934131e-2  | 5.0     |
| father | 3.0080735900045442e-2 | 5.0     |
| life   | 1.769248067468245e-2  | 5.0     |
| true   | 1.6281752071881345e-2 | 5.0     |
| young  | 1.4927950883812253e-2 | 5.0     |

Truncated to 30 rows

We will convert the DataFrame into a JSON format, which will be passed into d3.

``` scala
// Create JSON data
val rawJson = termDF.toJSON.collect().mkString(",\n")
```

>     rawJson: String = 
>     {"term":"just","probability":0.030515134931284552,"topicId":0},
>     {"term":"like","probability":0.02463563559747823,"topicId":0},
>     {"term":"want","probability":0.022529385381465025,"topicId":0},
>     {"term":"damn","probability":0.02094828832824297,"topicId":0},
>     {"term":"going","probability":0.0203407289886203,"topicId":0},
>     {"term":"yeah","probability":0.10787301090151602,"topicId":1},
>     {"term":"look","probability":0.0756831002291994,"topicId":1},
>     {"term":"know","probability":0.04815746564274915,"topicId":1},
>     {"term":"wait","probability":0.03897182014529944,"topicId":1},
>     {"term":"night","probability":0.0341458394828345,"topicId":1},
>     {"term":"gonna","probability":0.08118584492034046,"topicId":2},
>     {"term":"money","probability":0.051736711600637544,"topicId":2},
>     {"term":"shit","probability":0.04620430294274594,"topicId":2},
>     {"term":"fuck","probability":0.0399843125556081,"topicId":2},
>     {"term":"kill","probability":0.03672740843080258,"topicId":2},
>     {"term":"people","probability":0.020091372023286612,"topicId":3},
>     {"term":"know","probability":0.018613400462887356,"topicId":3},
>     {"term":"work","probability":0.016775643603287843,"topicId":3},
>     {"term":"does","probability":0.015522555458447744,"topicId":3},
>     {"term":"think","probability":0.012161168331925723,"topicId":3},
>     {"term":"know","probability":0.031956573561538214,"topicId":4},
>     {"term":"just","probability":0.030674598809934856,"topicId":4},
>     {"term":"want","probability":0.027663491240851962,"topicId":4},
>     {"term":"tell","probability":0.025727217382788027,"topicId":4},
>     {"term":"right","probability":0.02300853167338119,"topicId":4},
>     {"term":"love","probability":0.05932570200934131,"topicId":5},
>     {"term":"father","probability":0.030080735900045442,"topicId":5},
>     {"term":"life","probability":0.01769248067468245,"topicId":5},
>     {"term":"true","probability":0.016281752071881345,"topicId":5},
>     {"term":"young","probability":0.014927950883812253,"topicId":5},
>     {"term":"remember","probability":0.03998401809663685,"topicId":6},
>     {"term":"went","probability":0.01737965538107633,"topicId":6},
>     {"term":"lost","probability":0.016916065536574213,"topicId":6},
>     {"term":"called","probability":0.016443441316683228,"topicId":6},
>     {"term":"story","probability":0.014849882671062261,"topicId":6},
>     {"term":"house","probability":0.028911209424810257,"topicId":7},
>     {"term":"miss","probability":0.025669944694943093,"topicId":7},
>     {"term":"right","probability":0.02091105252727788,"topicId":7},
>     {"term":"family","probability":0.017862939987512365,"topicId":7},
>     {"term":"important","probability":0.013959164390834044,"topicId":7},
>     {"term":"saying","probability":0.022939827090645636,"topicId":8},
>     {"term":"know","probability":0.021335083902970984,"topicId":8},
>     {"term":"idea","probability":0.017628999871937747,"topicId":8},
>     {"term":"business","probability":0.017302568063786224,"topicId":8},
>     {"term":"police","probability":0.012284217866942303,"topicId":8},
>     {"term":"know","probability":0.051876601466269136,"topicId":9},
>     {"term":"like","probability":0.03828159069993671,"topicId":9},
>     {"term":"maybe","probability":0.03754385940676905,"topicId":9},
>     {"term":"just","probability":0.031938551661426284,"topicId":9},
>     {"term":"want","probability":0.02876693222824349,"topicId":9},
>     {"term":"years","probability":0.032537676027398765,"topicId":10},
>     {"term":"going","probability":0.030596831997667568,"topicId":10},
>     {"term":"case","probability":0.02049555392502822,"topicId":10},
>     {"term":"doctor","probability":0.018671171294737107,"topicId":10},
>     {"term":"working","probability":0.017672067172167016,"topicId":10},
>     {"term":"stuff","probability":0.02236582778896705,"topicId":11},
>     {"term":"school","probability":0.020057798194969816,"topicId":11},
>     {"term":"john","probability":0.017134198006217606,"topicId":11},
>     {"term":"week","probability":0.017075852415410653,"topicId":11},
>     {"term":"thousand","probability":0.017013413435021035,"topicId":11},
>     {"term":"little","probability":0.08663446368316245,"topicId":12},
>     {"term":"girl","probability":0.035120377589734936,"topicId":12},
>     {"term":"like","probability":0.02992080326340266,"topicId":12},
>     {"term":"woman","probability":0.0240813719635157,"topicId":12},
>     {"term":"baby","probability":0.022471517953608963,"topicId":12},
>     {"term":"know","probability":0.0283115823590395,"topicId":13},
>     {"term":"leave","probability":0.02744935904744228,"topicId":13},
>     {"term":"time","probability":0.02050833156294194,"topicId":13},
>     {"term":"want","probability":0.020124145131863225,"topicId":13},
>     {"term":"just","probability":0.019466336438890477,"topicId":13},
>     {"term":"didn","probability":0.08220031921979461,"topicId":14},
>     {"term":"like","probability":0.05062323326717784,"topicId":14},
>     {"term":"real","probability":0.03087838046777391,"topicId":14},
>     {"term":"guess","probability":0.02452989702353384,"topicId":14},
>     {"term":"says","probability":0.022815035397008333,"topicId":14},
>     {"term":"minutes","probability":0.018541518543996716,"topicId":15},
>     {"term":"time","probability":0.014737962244588431,"topicId":15},
>     {"term":"captain","probability":0.012594614743931537,"topicId":15},
>     {"term":"thirty","probability":0.01193707771669708,"topicId":15},
>     {"term":"ship","probability":0.011260576815409516,"topicId":15},
>     {"term":"okay","probability":0.08153575328080886,"topicId":16},
>     {"term":"just","probability":0.050004142902999975,"topicId":16},
>     {"term":"right","probability":0.03438984898476042,"topicId":16},
>     {"term":"know","probability":0.02821327795933634,"topicId":16},
>     {"term":"home","probability":0.023397063860326372,"topicId":16},
>     {"term":"country","probability":0.011270500385627474,"topicId":17},
>     {"term":"power","probability":0.010428408353623762,"topicId":17},
>     {"term":"president","probability":0.009392162067926028,"topicId":17},
>     {"term":"fight","probability":0.00799742811584178,"topicId":17},
>     {"term":"possible","probability":0.007597974486019279,"topicId":17},
>     {"term":"know","probability":0.09541058020800194,"topicId":18},
>     {"term":"think","probability":0.0698707939786508,"topicId":18},
>     {"term":"really","probability":0.06881812755565207,"topicId":18},
>     {"term":"mean","probability":0.02909700228968688,"topicId":18},
>     {"term":"just","probability":0.028699687473471538,"topicId":18},
>     {"term":"dead","probability":0.03833642117149438,"topicId":19},
>     {"term":"like","probability":0.017873711992106994,"topicId":19},
>     {"term":"hand","probability":0.015280854355409379,"topicId":19},
>     {"term":"white","probability":0.013718491413582671,"topicId":19},
>     {"term":"blood","probability":0.012699265888344448,"topicId":19}

We are now ready to use D3 on the rawJson data.

<p class="htmlSandbox">
<!DOCTYPE html>
<meta charset="utf-8">
<style>

circle {
  fill: rgb(31, 119, 180);
  fill-opacity: 0.5;
  stroke: rgb(31, 119, 180);
  stroke-width: 1px;
}

.leaf circle {
  fill: #ff7f0e;
  fill-opacity: 1;
}

text {
  font: 14px sans-serif;
}

</style>
<body>
<script src="https://cdnjs.cloudflare.com/ajax/libs/d3/3.5.5/d3.min.js"></script>
<script>

var json = {
 "name": "data",
 "children": [
  {
     "name": "topics",
     "children": [
      {"term":"just","probability":0.030515134931284552,"topicId":0},
{"term":"like","probability":0.02463563559747823,"topicId":0},
{"term":"want","probability":0.022529385381465025,"topicId":0},
{"term":"damn","probability":0.02094828832824297,"topicId":0},
{"term":"going","probability":0.0203407289886203,"topicId":0},
{"term":"yeah","probability":0.10787301090151602,"topicId":1},
{"term":"look","probability":0.0756831002291994,"topicId":1},
{"term":"know","probability":0.04815746564274915,"topicId":1},
{"term":"wait","probability":0.03897182014529944,"topicId":1},
{"term":"night","probability":0.0341458394828345,"topicId":1},
{"term":"gonna","probability":0.08118584492034046,"topicId":2},
{"term":"money","probability":0.051736711600637544,"topicId":2},
{"term":"shit","probability":0.04620430294274594,"topicId":2},
{"term":"fuck","probability":0.0399843125556081,"topicId":2},
{"term":"kill","probability":0.03672740843080258,"topicId":2},
{"term":"people","probability":0.020091372023286612,"topicId":3},
{"term":"know","probability":0.018613400462887356,"topicId":3},
{"term":"work","probability":0.016775643603287843,"topicId":3},
{"term":"does","probability":0.015522555458447744,"topicId":3},
{"term":"think","probability":0.012161168331925723,"topicId":3},
{"term":"know","probability":0.031956573561538214,"topicId":4},
{"term":"just","probability":0.030674598809934856,"topicId":4},
{"term":"want","probability":0.027663491240851962,"topicId":4},
{"term":"tell","probability":0.025727217382788027,"topicId":4},
{"term":"right","probability":0.02300853167338119,"topicId":4},
{"term":"love","probability":0.05932570200934131,"topicId":5},
{"term":"father","probability":0.030080735900045442,"topicId":5},
{"term":"life","probability":0.01769248067468245,"topicId":5},
{"term":"true","probability":0.016281752071881345,"topicId":5},
{"term":"young","probability":0.014927950883812253,"topicId":5},
{"term":"remember","probability":0.03998401809663685,"topicId":6},
{"term":"went","probability":0.01737965538107633,"topicId":6},
{"term":"lost","probability":0.016916065536574213,"topicId":6},
{"term":"called","probability":0.016443441316683228,"topicId":6},
{"term":"story","probability":0.014849882671062261,"topicId":6},
{"term":"house","probability":0.028911209424810257,"topicId":7},
{"term":"miss","probability":0.025669944694943093,"topicId":7},
{"term":"right","probability":0.02091105252727788,"topicId":7},
{"term":"family","probability":0.017862939987512365,"topicId":7},
{"term":"important","probability":0.013959164390834044,"topicId":7},
{"term":"saying","probability":0.022939827090645636,"topicId":8},
{"term":"know","probability":0.021335083902970984,"topicId":8},
{"term":"idea","probability":0.017628999871937747,"topicId":8},
{"term":"business","probability":0.017302568063786224,"topicId":8},
{"term":"police","probability":0.012284217866942303,"topicId":8},
{"term":"know","probability":0.051876601466269136,"topicId":9},
{"term":"like","probability":0.03828159069993671,"topicId":9},
{"term":"maybe","probability":0.03754385940676905,"topicId":9},
{"term":"just","probability":0.031938551661426284,"topicId":9},
{"term":"want","probability":0.02876693222824349,"topicId":9},
{"term":"years","probability":0.032537676027398765,"topicId":10},
{"term":"going","probability":0.030596831997667568,"topicId":10},
{"term":"case","probability":0.02049555392502822,"topicId":10},
{"term":"doctor","probability":0.018671171294737107,"topicId":10},
{"term":"working","probability":0.017672067172167016,"topicId":10},
{"term":"stuff","probability":0.02236582778896705,"topicId":11},
{"term":"school","probability":0.020057798194969816,"topicId":11},
{"term":"john","probability":0.017134198006217606,"topicId":11},
{"term":"week","probability":0.017075852415410653,"topicId":11},
{"term":"thousand","probability":0.017013413435021035,"topicId":11},
{"term":"little","probability":0.08663446368316245,"topicId":12},
{"term":"girl","probability":0.035120377589734936,"topicId":12},
{"term":"like","probability":0.02992080326340266,"topicId":12},
{"term":"woman","probability":0.0240813719635157,"topicId":12},
{"term":"baby","probability":0.022471517953608963,"topicId":12},
{"term":"know","probability":0.0283115823590395,"topicId":13},
{"term":"leave","probability":0.02744935904744228,"topicId":13},
{"term":"time","probability":0.02050833156294194,"topicId":13},
{"term":"want","probability":0.020124145131863225,"topicId":13},
{"term":"just","probability":0.019466336438890477,"topicId":13},
{"term":"didn","probability":0.08220031921979461,"topicId":14},
{"term":"like","probability":0.05062323326717784,"topicId":14},
{"term":"real","probability":0.03087838046777391,"topicId":14},
{"term":"guess","probability":0.02452989702353384,"topicId":14},
{"term":"says","probability":0.022815035397008333,"topicId":14},
{"term":"minutes","probability":0.018541518543996716,"topicId":15},
{"term":"time","probability":0.014737962244588431,"topicId":15},
{"term":"captain","probability":0.012594614743931537,"topicId":15},
{"term":"thirty","probability":0.01193707771669708,"topicId":15},
{"term":"ship","probability":0.011260576815409516,"topicId":15},
{"term":"okay","probability":0.08153575328080886,"topicId":16},
{"term":"just","probability":0.050004142902999975,"topicId":16},
{"term":"right","probability":0.03438984898476042,"topicId":16},
{"term":"know","probability":0.02821327795933634,"topicId":16},
{"term":"home","probability":0.023397063860326372,"topicId":16},
{"term":"country","probability":0.011270500385627474,"topicId":17},
{"term":"power","probability":0.010428408353623762,"topicId":17},
{"term":"president","probability":0.009392162067926028,"topicId":17},
{"term":"fight","probability":0.00799742811584178,"topicId":17},
{"term":"possible","probability":0.007597974486019279,"topicId":17},
{"term":"know","probability":0.09541058020800194,"topicId":18},
{"term":"think","probability":0.0698707939786508,"topicId":18},
{"term":"really","probability":0.06881812755565207,"topicId":18},
{"term":"mean","probability":0.02909700228968688,"topicId":18},
{"term":"just","probability":0.028699687473471538,"topicId":18},
{"term":"dead","probability":0.03833642117149438,"topicId":19},
{"term":"like","probability":0.017873711992106994,"topicId":19},
{"term":"hand","probability":0.015280854355409379,"topicId":19},
{"term":"white","probability":0.013718491413582671,"topicId":19},
{"term":"blood","probability":0.012699265888344448,"topicId":19}
     ]
    }
   ]
};

var r = 1500,
    format = d3.format(",d"),
    fill = d3.scale.category20c();

var bubble = d3.layout.pack()
    .sort(null)
    .size([r, r])
    .padding(1.5);

var vis = d3.select("body").append("svg")
    .attr("width", r)
    .attr("height", r)
    .attr("class", "bubble");

  
var node = vis.selectAll("g.node")
    .data(bubble.nodes(classes(json))
    .filter(function(d) { return !d.children; }))
    .enter().append("g")
    .attr("class", "node")
    .attr("transform", function(d) { return "translate(" + d.x + "," + d.y + ")"; })
    color = d3.scale.category20();
  
  node.append("title")
      .text(function(d) { return d.className + ": " + format(d.value); });

  node.append("circle")
      .attr("r", function(d) { return d.r; })
      .style("fill", function(d) {return color(d.topicName);});

var text = node.append("text")
    .attr("text-anchor", "middle")
    .attr("dy", ".3em")
    .text(function(d) { return d.className.substring(0, d.r / 3)});
  
  text.append("tspan")
      .attr("dy", "1.2em")
      .attr("x", 0)
      .text(function(d) {return Math.ceil(d.value * 10000) /10000; });

// Returns a flattened hierarchy containing all leaf nodes under the root.
function classes(root) {
  var classes = [];

  function recurse(term, node) {
    if (node.children) node.children.forEach(function(child) { recurse(node.term, child); });
    else classes.push({topicName: node.topicId, className: node.term, value: node.probability});
  }

  recurse(null, root);
  return {children: classes};
}
</script>
</p>

Step 1. Downloading and Loading Data into DBFS
----------------------------------------------

Here are the steps taken for downloading and saving data to the distributed file system. Uncomment them for repeating this process on your databricks cluster or for downloading a new source of data.

Unfortunately, the original data at:

-   [http://www.mpi-sws.org/~cristian/data/cornell*movie*dialogs\_corpus.zip](http://www.mpi-sws.org/~cristian/data/cornell_movie_dialogs_corpus.zip)

is not suited for manipulation and loading into dbfs easily. So the data has been downloaded, directory renamed without white spaces, superfluous OS-specific files removed, `dos2unix`'d, `tar -zcvf`'d and uploaded to the following URL for an easily dbfs-loadable download:

-   [http://lamastex.org/datasets/public/nlp/cornell*movie*dialogs\_corpus.tgz](http://lamastex.org/datasets/public/nlp/cornell_movie_dialogs_corpus.tgz)

``` scala
//%sh wget http://lamastex.org/datasets/public/nlp/cornell_movie_dialogs_corpus.tgz
```

>     --2017-01-11 01:23:41--  http://lamastex.org/datasets/public/nlp/cornell_movie_dialogs_corpus.tgz
>     Resolving lamastex.org (lamastex.org)... 166.62.28.100
>     Connecting to lamastex.org (lamastex.org)|166.62.28.100|:80... connected.
>     HTTP request sent, awaiting response... 200 OK
>     Length: 9914415 (9.5M) [application/x-tar]
>     Saving to: ‘cornell_movie_dialogs_corpus.tgz’
>
>          0K .......... .......... .......... .......... ..........  0%  125K 77s
>         50K .......... .......... .......... .......... ..........  1%  254K 57s
>        100K .......... .......... .......... .......... ..........  1% 7.71M 38s
>        150K .......... .......... .......... .......... ..........  2%  259K 38s
>        200K .......... .......... .......... .......... ..........  2% 9.99M 30s
>        250K .......... .......... .......... .......... ..........  3%  260K 31s
>        300K .......... .......... .......... .......... ..........  3% 23.1M 27s
>        350K .......... .......... .......... .......... ..........  4% 11.2M 23s
>        400K .......... .......... .......... .......... ..........  4%  261K 24s
>        450K .......... .......... .......... .......... ..........  5% 27.0M 22s
>        500K .......... .......... .......... .......... ..........  5% 42.4M 20s
>        550K .......... .......... .......... .......... ..........  6% 12.1M 18s
>        600K .......... .......... .......... .......... ..........  6%  262K 19s
>        650K .......... .......... .......... .......... ..........  7% 40.7M 18s
>        700K .......... .......... .......... .......... ..........  7% 32.6M 17s
>        750K .......... .......... .......... .......... ..........  8% 41.2M 15s
>        800K .......... .......... .......... .......... ..........  8% 15.9M 14s
>        850K .......... .......... .......... .......... ..........  9% 44.6M 14s
>        900K .......... .......... .......... .......... ..........  9%  263K 15s
>        950K .......... .......... .......... .......... .......... 10% 39.0M 14s
>       1000K .......... .......... .......... .......... .......... 10% 46.8M 13s
>       1050K .......... .......... .......... .......... .......... 11% 51.5M 12s
>       1100K .......... .......... .......... .......... .......... 11% 48.7M 12s
>       1150K .......... .......... .......... .......... .......... 12% 47.8M 11s
>       1200K .......... .......... .......... .......... .......... 12% 19.9M 11s
>       1250K .......... .......... .......... .......... .......... 13% 40.8M 10s
>       1300K .......... .......... .......... .......... .......... 13% 46.9M 10s
>       1350K .......... .......... .......... .......... .......... 14%  264K 11s
>       1400K .......... .......... .......... .......... .......... 14% 39.5M 10s
>       1450K .......... .......... .......... .......... .......... 15% 45.5M 10s
>       1500K .......... .......... .......... .......... .......... 16% 45.2M 9s
>       1550K .......... .......... .......... .......... .......... 16% 39.4M 9s
>       1600K .......... .......... .......... .......... .......... 17% 45.0M 9s
>       1650K .......... .......... .......... .......... .......... 17% 42.7M 8s
>       1700K .......... .......... .......... .......... .......... 18% 36.1M 8s
>       1750K .......... .......... .......... .......... .......... 18% 38.3M 8s
>       1800K .......... .......... .......... .......... .......... 19% 38.7M 8s
>       1850K .......... .......... .......... .......... .......... 19% 49.6M 7s
>       1900K .......... .......... .......... .......... .......... 20%  269K 8s
>       1950K .......... .......... .......... .......... .......... 20% 37.2M 8s
>       2000K .......... .......... .......... .......... .......... 21% 43.7M 7s
>       2050K .......... .......... .......... .......... .......... 21% 36.8M 7s
>       2100K .......... .......... .......... .......... .......... 22% 55.0M 7s
>       2150K .......... .......... .......... .......... .......... 22% 42.6M 7s
>       2200K .......... .......... .......... .......... .......... 23% 36.8M 7s
>       2250K .......... .......... .......... .......... .......... 23% 47.2M 6s
>       2300K .......... .......... .......... .......... .......... 24% 36.3M 6s
>       2350K .......... .......... .......... .......... .......... 24% 38.1M 6s
>       2400K .......... .......... .......... .......... .......... 25% 40.4M 6s
>       2450K .......... .......... .......... .......... .......... 25% 53.9M 6s
>       2500K .......... .......... .......... .......... .......... 26% 43.2M 6s
>       2550K .......... .......... .......... .......... .......... 26% 46.0M 5s
>       2600K .......... .......... .......... .......... .......... 27% 46.6M 5s
>       2650K .......... .......... .......... .......... .......... 27% 1.68M 5s
>       2700K .......... .......... .......... .......... .......... 28%  326K 5s
>       2750K .......... .......... .......... .......... .......... 28% 39.0M 5s
>       2800K .......... .......... .......... .......... .......... 29% 47.4M 5s
>       2850K .......... .......... .......... .......... .......... 29% 46.0M 5s
>       2900K .......... .......... .......... .......... .......... 30% 47.6M 5s
>       2950K .......... .......... .......... .......... .......... 30% 54.7M 5s
>       3000K .......... .......... .......... .......... .......... 31% 39.2M 5s
>       3050K .......... .......... .......... .......... .......... 32% 49.7M 5s
>       3100K .......... .......... .......... .......... .......... 32% 44.2M 5s
>       3150K .......... .......... .......... .......... .......... 33% 38.8M 4s
>       3200K .......... .......... .......... .......... .......... 33% 46.0M 4s
>       3250K .......... .......... .......... .......... .......... 34% 46.0M 4s
>       3300K .......... .......... .......... .......... .......... 34% 46.0M 4s
>       3350K .......... .......... .......... .......... .......... 35% 35.7M 4s
>       3400K .......... .......... .......... .......... .......... 35% 36.4M 4s
>       3450K .......... .......... .......... .......... .......... 36% 48.0M 4s
>       3500K .......... .......... .......... .......... .......... 36% 38.7M 4s
>       3550K .......... .......... .......... .......... .......... 37% 48.6M 4s
>       3600K .......... .......... .......... .......... .......... 37% 50.0M 4s
>       3650K .......... .......... .......... .......... .......... 38% 46.4M 4s
>       3700K .......... .......... .......... .......... .......... 38% 46.0M 3s
>       3750K .......... .......... .......... .......... .......... 39%  285K 4s
>       3800K .......... .......... .......... .......... .......... 39% 38.0M 4s
>       3850K .......... .......... .......... .......... .......... 40% 34.6M 4s
>       3900K .......... .......... .......... .......... .......... 40% 53.1M 3s
>       3950K .......... .......... .......... .......... .......... 41% 40.6M 3s
>       4000K .......... .......... .......... .......... .......... 41% 32.9M 3s
>       4050K .......... .......... .......... .......... .......... 42% 55.1M 3s
>       4100K .......... .......... .......... .......... .......... 42% 46.7M 3s
>       4150K .......... .......... .......... .......... .......... 43% 50.5M 3s
>       4200K .......... .......... .......... .......... .......... 43% 42.1M 3s
>       4250K .......... .......... .......... .......... .......... 44% 40.8M 3s
>       4300K .......... .......... .......... .......... .......... 44% 40.6M 3s
>       4350K .......... .......... .......... .......... .......... 45% 38.3M 3s
>       4400K .......... .......... .......... .......... .......... 45% 36.4M 3s
>       4450K .......... .......... .......... .......... .......... 46% 39.3M 3s
>       4500K .......... .......... .......... .......... .......... 46% 36.2M 3s
>       4550K .......... .......... .......... .......... .......... 47% 32.0M 3s
>       4600K .......... .......... .......... .......... .......... 48% 41.3M 3s
>       4650K .......... .......... .......... .......... .......... 48% 39.1M 3s
>       4700K .......... .......... .......... .......... .......... 49% 39.5M 2s
>       4750K .......... .......... .......... .......... .......... 49% 61.0M 2s
>       4800K .......... .......... .......... .......... .......... 50% 31.9M 2s
>       4850K .......... .......... .......... .......... .......... 50% 37.2M 2s
>       4900K .......... .......... .......... .......... .......... 51% 35.4M 2s
>       4950K .......... .......... .......... .......... .......... 51% 46.8M 2s
>       5000K .......... .......... .......... .......... .......... 52% 45.8M 2s
>       5050K .......... .......... .......... .......... .......... 52% 54.2M 2s
>       5100K .......... .......... .......... .......... .......... 53% 38.8M 2s
>       5150K .......... .......... .......... .......... .......... 53% 3.73M 2s
>       5200K .......... .......... .......... .......... .......... 54%  327K 2s
>       5250K .......... .......... .......... .......... .......... 54% 38.5M 2s
>       5300K .......... .......... .......... .......... .......... 55% 40.0M 2s
>       5350K .......... .......... .......... .......... .......... 55% 30.5M 2s
>       5400K .......... .......... .......... .......... .......... 56% 48.0M 2s
>       5450K .......... .......... .......... .......... .......... 56% 35.3M 2s
>       5500K .......... .......... .......... .......... .......... 57% 48.5M 2s
>       5550K .......... .......... .......... .......... .......... 57% 38.4M 2s
>       5600K .......... .......... .......... .......... .......... 58% 41.2M 2s
>       5650K .......... .......... .......... .......... .......... 58% 47.1M 2s
>       5700K .......... .......... .......... .......... .......... 59% 33.8M 2s
>       5750K .......... .......... .......... .......... .......... 59% 46.6M 2s
>       5800K .......... .......... .......... .......... .......... 60% 43.0M 2s
>       5850K .......... .......... .......... .......... .......... 60% 36.8M 2s
>       5900K .......... .......... .......... .......... .......... 61% 43.9M 2s
>       5950K .......... .......... .......... .......... .......... 61% 43.8M 2s
>       6000K .......... .......... .......... .......... .......... 62% 36.7M 2s
>       6050K .......... .......... .......... .......... .......... 63% 33.6M 2s
>       6100K .......... .......... .......... .......... .......... 63% 34.3M 1s
>       6150K .......... .......... .......... .......... .......... 64% 40.0M 1s
>       6200K .......... .......... .......... .......... .......... 64% 35.5M 1s
>       6250K .......... .......... .......... .......... .......... 65% 33.1M 1s
>       6300K .......... .......... .......... .......... .......... 65% 37.4M 1s
>       6350K .......... .......... .......... .......... .......... 66% 46.3M 1s
>       6400K .......... .......... .......... .......... .......... 66% 42.5M 1s
>       6450K .......... .......... .......... .......... .......... 67% 30.6M 1s
>       6500K .......... .......... .......... .......... .......... 67% 46.4M 1s
>       6550K .......... .......... .......... .......... .......... 68% 42.2M 1s
>       6600K .......... .......... .......... .......... .......... 68% 37.8M 1s
>       6650K .......... .......... .......... .......... .......... 69% 35.4M 1s
>       6700K .......... .......... .......... .......... .......... 69% 51.0M 1s
>       6750K .......... .......... .......... .......... .......... 70% 52.1M 1s
>       6800K .......... .......... .......... .......... .......... 70% 59.9M 1s
>       6850K .......... .......... .......... .......... .......... 71% 40.9M 1s
>       6900K .......... .......... .......... .......... .......... 71% 41.2M 1s
>       6950K .......... .......... .......... .......... .......... 72% 53.5M 1s
>       7000K .......... .......... .......... .......... .......... 72% 46.5M 1s
>       7050K .......... .......... .......... .......... .......... 73% 54.5M 1s
>       7100K .......... .......... .......... .......... .......... 73% 40.3M 1s
>       7150K .......... .......... .......... .......... .......... 74%  328K 1s
>       7200K .......... .......... .......... .......... .......... 74% 38.7M 1s
>       7250K .......... .......... .......... .......... .......... 75% 41.0M 1s
>       7300K .......... .......... .......... .......... .......... 75% 39.5M 1s
>       7350K .......... .......... .......... .......... .......... 76% 47.4M 1s
>       7400K .......... .......... .......... .......... .......... 76% 37.5M 1s
>       7450K .......... .......... .......... .......... .......... 77% 50.8M 1s
>       7500K .......... .......... .......... .......... .......... 77% 46.6M 1s
>       7550K .......... .......... .......... .......... .......... 78% 55.1M 1s
>       7600K .......... .......... .......... .......... .......... 79% 38.9M 1s
>       7650K .......... .......... .......... .......... .......... 79% 38.1M 1s
>       7700K .......... .......... .......... .......... .......... 80% 47.5M 1s
>       7750K .......... .......... .......... .......... .......... 80% 49.3M 1s
>       7800K .......... .......... .......... .......... .......... 81% 46.7M 1s
>       7850K .......... .......... .......... .......... .......... 81% 37.7M 1s
>       7900K .......... .......... .......... .......... .......... 82% 42.7M 1s
>       7950K .......... .......... .......... .......... .......... 82% 44.2M 1s
>       8000K .......... .......... .......... .......... .......... 83% 39.0M 1s
>       8050K .......... .......... .......... .......... .......... 83% 31.8M 1s
>       8100K .......... .......... .......... .......... .......... 84% 48.9M 1s
>       8150K .......... .......... .......... .......... .......... 84% 47.7M 1s
>       8200K .......... .......... .......... .......... .......... 85% 48.0M 0s
>       8250K .......... .......... .......... .......... .......... 85% 46.7M 0s
>       8300K .......... .......... .......... .......... .......... 86% 42.0M 0s
>       8350K .......... .......... .......... .......... .......... 86% 34.0M 0s
>       8400K .......... .......... .......... .......... .......... 87% 31.3M 0s
>       8450K .......... .......... .......... .......... .......... 87% 40.6M 0s
>       8500K .......... .......... .......... .......... .......... 88% 40.8M 0s
>       8550K .......... .......... .......... .......... .......... 88% 32.8M 0s
>       8600K .......... .......... .......... .......... .......... 89% 41.0M 0s
>       8650K .......... .......... .......... .......... .......... 89% 44.5M 0s
>       8700K .......... .......... .......... .......... .......... 90% 40.3M 0s
>       8750K .......... .......... .......... .......... .......... 90% 43.3M 0s
>       8800K .......... .......... .......... .......... .......... 91% 31.8M 0s
>       8850K .......... .......... .......... .......... .......... 91% 40.4M 0s
>       8900K .......... .......... .......... .......... .......... 92% 46.8M 0s
>       8950K .......... .......... .......... .......... .......... 92% 41.4M 0s
>       9000K .......... .......... .......... .......... .......... 93% 43.8M 0s
>       9050K .......... .......... .......... .......... .......... 93% 37.4M 0s
>       9100K .......... .......... .......... .......... .......... 94% 36.7M 0s
>       9150K .......... .......... .......... .......... .......... 95% 40.8M 0s
>       9200K .......... .......... .......... .......... .......... 95% 45.9M 0s
>       9250K .......... .......... .......... .......... .......... 96% 35.0M 0s
>       9300K .......... .......... .......... .......... .......... 96% 42.3M 0s
>       9350K .......... .......... .......... .......... .......... 97% 37.7M 0s
>       9400K .......... .......... .......... .......... .......... 97% 42.5M 0s
>       9450K .......... .......... .......... .......... .......... 98% 55.8M 0s
>       9500K .......... .......... .......... .......... .......... 98% 54.2M 0s
>       9550K .......... .......... .......... .......... .......... 99% 53.7M 0s
>       9600K .......... .......... .......... .......... .......... 99% 54.2M 0s
>       9650K .......... .......... .......... ..                   100% 47.7M=2.8s
>
>     2017-01-11 01:23:44 (3.34 MB/s) - ‘cornell_movie_dialogs_corpus.tgz’ saved [9914415/9914415]

Untar the file.

``` scala
//%sh tar zxvf cornell_movie_dialogs_corpus.tgz
```

>     cornell_movie_dialogs_corpus/
>     cornell_movie_dialogs_corpus/movie_lines.txt
>     cornell_movie_dialogs_corpus/movie_characters_metadata.txt
>     cornell_movie_dialogs_corpus/README.txt
>     cornell_movie_dialogs_corpus/raw_script_urls.txt
>     cornell_movie_dialogs_corpus/movie_titles_metadata.txt
>     cornell_movie_dialogs_corpus/movie_conversations.txt
>     cornell_movie_dialogs_corpus/chameleons.pdf

Let us list and load all the files into dbfs after `dbfs.fs.mkdirs(...)` to create the directory `dbfs:/datasets/sds/nlp/cornell_movie_dialogs_corpus/`.

``` scala
//%sh pwd && ls -al cornell_movie_dialogs_corpus
```

>     /databricks/driver
>     total 41552
>     drwxr-xr-x 2 ubuntu ubuntu     4096 Jan 11 01:03 .
>     drwxr-xr-x 5 root   root       4096 Jan 11 01:23 ..
>     -rw-r--r-- 1 ubuntu ubuntu   290691 May  9  2011 chameleons.pdf
>     -rw-r--r-- 1 ubuntu ubuntu   705695 Jan 11 01:02 movie_characters_metadata.txt
>     -rw-r--r-- 1 ubuntu ubuntu  6760930 Jan 11 01:02 movie_conversations.txt
>     -rw-r--r-- 1 ubuntu ubuntu 34641919 Jan 11 01:03 movie_lines.txt
>     -rw-r--r-- 1 ubuntu ubuntu    67289 Jan 11 01:02 movie_titles_metadata.txt
>     -rw-r--r-- 1 ubuntu ubuntu    56177 Jan 11 01:03 raw_script_urls.txt
>     -rw-r--r-- 1 ubuntu ubuntu     4181 Jan 11 01:03 README.txt

``` scala
/*
dbutils.fs.cp("file:///databricks/driver/cornell_movie_dialogs_corpus/movie_characters_metadata.txt","dbfs:/datasets/sds/nlp/cornell_movie_dialogs_corpus/")
dbutils.fs.cp("file:///databricks/driver/cornell_movie_dialogs_corpus/movie_conversations.txt","dbfs:/datasets/sds/nlp/cornell_movie_dialogs_corpus/")
dbutils.fs.cp("file:///databricks/driver/cornell_movie_dialogs_corpus/movie_lines.txt","dbfs:/datasets/sds/nlp/cornell_movie_dialogs_corpus/")
dbutils.fs.cp("file:///databricks/driver/cornell_movie_dialogs_corpus/movie_titles_metadata.txt","dbfs:/datasets/sds/nlp/cornell_movie_dialogs_corpus/")
dbutils.fs.cp("file:///databricks/driver/cornell_movie_dialogs_corpus/raw_script_urls.txt","dbfs:/datasets/sds/nlp/cornell_movie_dialogs_corpus/")
dbutils.fs.cp("file:///databricks/driver/cornell_movie_dialogs_corpus/README.txt","dbfs:/datasets/sds/nlp/cornell_movie_dialogs_corpus/")
*/
```

>     res9: Boolean = true

``` scala
display(dbutils.fs.ls("dbfs:/datasets/sds/nlp/cornell_movie_dialogs_corpus/"))
```

| path                                                                                   | name                            | size        |
|----------------------------------------------------------------------------------------|---------------------------------|-------------|
| dbfs:/datasets/sds/nlp/cornell\_movie\_dialogs\_corpus/README.txt                      | README.txt                      | 4181.0      |
| dbfs:/datasets/sds/nlp/cornell\_movie\_dialogs\_corpus/movie\_characters\_metadata.txt | movie\_characters\_metadata.txt | 705695.0    |
| dbfs:/datasets/sds/nlp/cornell\_movie\_dialogs\_corpus/movie\_conversations.txt        | movie\_conversations.txt        | 6760930.0   |
| dbfs:/datasets/sds/nlp/cornell\_movie\_dialogs\_corpus/movie\_lines.txt                | movie\_lines.txt                | 3.4641919e7 |
| dbfs:/datasets/sds/nlp/cornell\_movie\_dialogs\_corpus/movie\_titles\_metadata.txt     | movie\_titles\_metadata.txt     | 67289.0     |
| dbfs:/datasets/sds/nlp/cornell\_movie\_dialogs\_corpus/raw\_script\_urls.txt           | raw\_script\_urls.txt           | 56177.0     |