# search-engine-architecture
A practical guide to modern search engine architecture, covering web crawling, inverted indexes, TF-IDF, BM25, PageRank, query processing, embeddings, vector search, hybrid retrieval, and ranking.
How Search Engines Work: From Web Crawling and Indexing to Ranking and Semantic Search
Introduction
The internet contains billions of pages.
Yet when you search for something like:
how does blockchain work

a search engine can return relevant results almost instantly.
It may look like the search engine is searching the entire web at that moment.
It isn't.
Modern search engines perform most of the expensive work before you search.
They continuously discover pages, download content, analyze documents, build indexes, understand relationships between pages, and prepare data structures optimized for extremely fast retrieval.
A simplified search engine looks like this:
Internet
   ↓
Web Crawlers
   ↓
Page Processing
   ↓
Indexing
   ↓
Search Index
   ↓
User Query
   ↓
Candidate Retrieval
   ↓
Ranking
   ↓
Search Results

Modern systems go even further by combining traditional keyword search with machine learning, embeddings, vector search, and semantic understanding.
This article explains how the entire process works.
1. The Three Main Jobs of a Search Engine
At a high level, a search engine performs three major tasks:
1. Crawling
2. Indexing
3. Ranking

Crawling discovers information.
Indexing organizes it.
Ranking decides which results should appear first.
These stages solve very different problems.
2. Web Crawling
Before a search engine can return a webpage, it needs to know that the page exists.
This is the job of a web crawler.
A crawler is an automated program that visits webpages and discovers links.
Imagine it starts with:
https://example.com

The crawler downloads the page and discovers:
/about
/blog
/products
/contact

It adds those URLs to a queue.
Crawler
   ↓
Download Page
   ↓
Extract Links
   ↓
Add URLs to Queue
   ↓
Visit Next Page

This process continues across the web.
3. The URL Frontier
A large crawler cannot immediately visit every discovered URL.
Instead, URLs are stored in a structure often called a URL frontier.
Conceptually:
Discovered URLs
      ↓
┌─────────────────────┐
│     URL Frontier    │
├─────────────────────┤
│ example.com/page1   │
│ site.com/article    │
│ blog.com/post       │
│ docs.com/guide      │
└─────────────────────┘
      ↓
Crawler Workers

The frontier helps determine:
What should be crawled?

When should it be crawled?

How often should it be revisited?

Which pages have priority?

At large scale, crawl scheduling becomes a serious distributed-systems problem.
4. Discovering URLs
Search engines can discover URLs from several sources.
Common examples include:
Links from other pages
Sitemaps
Previously known URLs
Redirects
Submitted URLs

The web itself acts like a giant graph.
Page A ─────→ Page B
  │             │
  ↓             ↓
Page C ─────→ Page D

By following links, crawlers can discover enormous portions of the public web.
5. Robots.txt
Websites can provide crawling instructions through:
robots.txt

For example:
User-agent: *
Disallow: /private/

This communicates that crawlers matching the rule should not crawl the specified path.
A production crawler should respect applicable crawling policies rather than blindly downloading every discovered URL.
6. XML Sitemaps
Websites can also provide sitemaps.
A sitemap helps crawlers discover important URLs.
Conceptually:
<urlset>
    <url>
        <loc>https://example.com/</loc>
    </url>

    <url>
        <loc>https://example.com/blog</loc>
    </url>
</urlset>

A sitemap does not automatically guarantee that a page will be indexed or ranked.
It mainly helps with discovery and crawl organization.
7. Crawling at Scale
A small crawler could run on one machine.
A major search engine cannot.
Imagine needing to process:
Billions of URLs

The work must be distributed.
              URL Frontier
                   ↓
       ┌───────────┼───────────┐
       ↓           ↓           ↓
   Crawler 1   Crawler 2   Crawler 3
       ↓           ↓           ↓
    Pages        Pages        Pages

But this introduces additional challenges:
Duplicate URLs
Network Failures
Rate Limiting
DNS Lookups
Redirects
Crawler Traps
Scheduling
Storage

Crawling the web efficiently is itself a large engineering problem.
8. Parsing a Webpage
After downloading a page, the search engine needs to understand its structure.
An HTML page might contain:
<title>Python Tutorial</title>

<h1>Learn Python</h1>

<p>
Python is a popular programming language.
</p>

The parser can extract:
Title
Headings
Paragraphs
Links
Metadata
Structured Data

The raw HTML is transformed into information that later stages can analyze.
9. Content Normalization
Web content contains noise.
For example:
Navigation menus
Headers
Footers
Advertisements
Repeated widgets
Tracking parameters
Formatting

Search systems may normalize content before indexing.
Text may go through processes such as:
Lowercasing
Tokenization
Normalization
Language Detection
Duplicate Detection

The goal is to create a useful representation of the document.
10. Tokenization
Suppose a document contains:
Python is great for data science.

A tokenizer might produce:
python
is
great
for
data
science

These units are called tokens.
More sophisticated tokenization is required for different languages and writing systems.
Tokenization is an important bridge between raw text and searchable data.
11. Why Search Engines Need an Index
Imagine having one million documents.
A user searches:
machine learning

The naive solution would be:
Open Document 1 → Search
Open Document 2 → Search
Open Document 3 → Search
...
Open Document 1,000,000 → Search

Doing this for every query would be extremely inefficient.
Instead, search engines build an index beforehand.
12. The Inverted Index
One of the most important data structures in traditional search is the inverted index.
Suppose we have:
Document 1:
Python programming tutorial

Document 2:
Python data science

Document 3:
Java programming tutorial

Instead of storing only:
Document → Words

we create:
Word → Documents

The index becomes:
python      → Doc 1, Doc 2

programming → Doc 1, Doc 3

tutorial    → Doc 1, Doc 3

data        → Doc 2

science     → Doc 2

java        → Doc 3

Now searching for:
python

does not require scanning every document.
The engine can directly retrieve:
Doc 1
Doc 2

This is why it is called an inverted index.
13. Posting Lists
The list of documents associated with a term is often called a posting list.
For example:
blockchain
    ↓
[2, 8, 19, 41, 90]

These numbers could represent document IDs.
Real posting lists may store additional information:
Document ID
Term Frequency
Word Positions
Field Information

For example:
python →

Doc 4:
frequency = 7
positions = [3, 18, 45, ...]

Doc 21:
frequency = 2
positions = [8, 32]

This information helps ranking and phrase matching.
14. Query Processing
Now suppose the user searches:
python machine learning

The query may go through processing similar to document indexing.
Raw Query
    ↓
Normalization
    ↓
Tokenization
    ↓
Query Understanding
    ↓
Candidate Retrieval

The search engine can look up relevant posting lists.
python   → Doc 1, Doc 5, Doc 8

machine  → Doc 2, Doc 5, Doc 9

learning → Doc 2, Doc 5, Doc 10

Document 5 appears across all three.
It may therefore be a strong candidate.
But matching words is only the beginning.
The results still need to be ranked.
15. TF-IDF
A classic information-retrieval concept is TF-IDF.
It combines two ideas:
Term Frequency
+
Inverse Document Frequency

Term Frequency
If a word appears frequently in a document, it may be important to that document.
For example:
Document A:

cloud cloud cloud computing architecture

The word cloud has high term frequency.
Inverse Document Frequency
Words that appear in almost every document are usually less useful for distinguishing documents.
A rare term can carry more information.
Conceptually:
Important Term
=
Frequent in This Document
+
Relatively Rare Across Documents

TF-IDF provides a mathematical way to represent this idea.
16. BM25
Modern keyword search systems commonly use ranking functions from the BM25 family.
BM25 considers factors such as:
Term Frequency
Document Length
Term Rarity

It improves on simplistic approaches where repeating a keyword indefinitely would always increase relevance.
Conceptually:
Query
  ↓
Candidate Documents
  ↓
BM25 Score
  ↓
Ranked Results

BM25 remains an important baseline for text retrieval.
17. Ranking Is More Than Keyword Matching
Imagine two pages both contain:
how to learn python

One is a detailed tutorial from a strong source.
The other is an unrelated page repeating those keywords hundreds of times.
Pure keyword frequency would not be enough.
Search engines therefore consider many signals.
Depending on the search system, ranking signals might include:
Text Relevance
Page Structure
Link Relationships
Freshness
Language
Location Context
Quality Signals
User Intent

Modern ranking systems can combine many features rather than relying on a single formula.
18. PageRank
One famous idea in web search is PageRank.
The basic insight is that links can provide information about the importance of webpages.
Imagine:
Site A ──→ Page X

Site B ──→ Page X

Site C ──→ Page X

Several pages link to Page X.
This may indicate that Page X is important.
But not every link should necessarily have equal weight.
A link from an important page can itself carry more weight.
Conceptually:
Important Pages
      ↓ links
Another Page
      ↓
Potential Importance

This creates a recursive ranking problem.
19. The Web as a Graph
PageRank becomes easier to understand if we model the web as a graph.
A ─────→ B
↑        │
│        ↓
D ←───── C

Pages are nodes.
Links are edges.
Graph algorithms can analyze this structure.
This was an important shift because search quality could consider not only:
What's written on the page?

but also:
How is this page connected to the rest of the web?

20. Query Intent
The same words can represent very different intentions.
Consider:
python

The user might mean:
Python programming language
Python snake
Python software download
Python tutorial

Search engines attempt to understand query intent.
Common intent categories can include:
Informational
Navigational
Transactional
Local

For example:
"how does bitcoin mining work"

is mainly informational.
While:
"github login"

is likely navigational.
Understanding intent improves ranking.
21. Spelling and Query Expansion
Users do not always type perfect queries.
A search system may need to handle:
Typos
Alternative Spellings
Synonyms
Abbreviations
Related Terms

For example:
machin lerning

could potentially be interpreted as:
machine learning

A query can also be expanded with related concepts.
For example:
car

may have relationships with:
automobile
vehicle

But expansion must be used carefully because adding incorrect meanings can reduce relevance.
22. Traditional Keyword Search
Traditional search primarily relies on matching textual terms.
For example:
Query:

"distributed database"

The engine looks for documents containing terms related to:
distributed
database

This works extremely well for many searches.
But keyword matching has limitations.
Consider:
Query:
"software that stores information across multiple computers"

A highly relevant document might use the phrase:
distributed database

without containing the user's exact wording.
This motivates semantic search.
23. Semantic Search
Semantic search attempts to represent meaning, not just exact words.
For example:
Query:

"how computers understand human language"

could be semantically related to:
Natural Language Processing

even if the exact words differ.
A major technology behind modern semantic search is:
Embeddings.
24. Embeddings
An embedding converts text into a vector of numbers.
For example:
"machine learning"

↓

[0.21, -0.43, 0.87, 0.12, ...]

Another text:
"artificial intelligence models"

↓

[0.19, -0.39, 0.82, 0.17, ...]

Semantically related texts may have vectors that are relatively close in vector space.
Now search becomes partly a mathematical similarity problem.
25. Vector Search
Suppose every document has an embedding.
Document A → Vector A
Document B → Vector B
Document C → Vector C
...

The user's query is also converted into a vector.
Query
  ↓
Embedding Model
  ↓
Query Vector

The system searches for nearby vectors.
Query Vector
     ↓
Vector Search
     ↓
Nearest Documents

This enables retrieval based on semantic similarity.
26. Cosine Similarity
One common method for comparing vectors is Cosine Similarity.
Conceptually:
Vector A
   ↘
    \ angle
     \
      → Vector B

If two vectors point in similar directions, their cosine similarity is high.
The formula is:
cosine_similarity(A, B)

        A · B
= -----------------
   ||A|| × ||B||

This makes it possible to mathematically compare the semantic representations of documents and queries.
27. Why Vector Search Is Difficult at Scale
Imagine having:
1,000,000,000 document vectors

For every query, comparing against every vector would be extremely expensive.
Query
 ↓
Compare with Vector 1
Compare with Vector 2
...
Compare with Vector 1,000,000,000

Modern systems therefore use Approximate Nearest Neighbor (ANN) algorithms and specialized vector indexes.
The goal is:
Find very good nearby vectors
without checking every vector

This trades a small amount of exactness for dramatically faster retrieval.
28. Keyword Search vs Semantic Search
The two approaches have different strengths.
Keyword Search	Semantic Search
Matches terms	Matches meaning
Excellent for exact words	Better for conceptual similarity
Works well with rare identifiers	Handles different wording
Often uses inverted indexes	Often uses vector indexes
BM25 is common	Embedding similarity is common


Neither approach is universally better.
That leads to an important modern architecture:
Hybrid Search.
29. Hybrid Search
Hybrid search combines keyword and semantic retrieval.
                   Query
                     ↓
            ┌────────┴────────┐
            ↓                 ↓
      Keyword Search     Vector Search
            ↓                 ↓
          BM25             Embeddings
            ↓                 ↓
            └────────┬────────┘
                     ↓
               Merge Results
                     ↓
                  Ranking

Why combine them?
Consider searching:
RTX 5090 error code 43

Exact terms such as:
RTX 5090
43

can be extremely important.
Keyword search handles these well.
But semantic search can help find documents describing the same problem with different language.
Hybrid retrieval can benefit from both.
30. Re-Ranking
A search engine does not necessarily use its most expensive model across every document.
Instead:
Millions of Documents
        ↓
Fast Retrieval
        ↓
1,000 Candidates
        ↓
More Expensive Ranking
        ↓
100 Candidates
        ↓
Re-Ranking
        ↓
Top 10

This multi-stage architecture is extremely important.
Early stages optimize for:
Speed + Recall

Later stages can optimize more heavily for:
Relevance

because they operate on far fewer documents.
31. Search Engine Architecture
Putting everything together:
                    INTERNET
                       ↓
                  Web Crawlers
                       ↓
                 URL Frontier
                       ↓
                 Page Download
                       ↓
                     Parser
                       ↓
              Content Processing
                       ↓
          ┌────────────┴────────────┐
          ↓                         ↓
    Inverted Index             Embeddings
          ↓                         ↓
   Keyword Index              Vector Index
          │                         │
          └────────────┬────────────┘
                       │
USER                   │
 ↓                     │
Query ─────────→ Query Processing
                       ↓
              Candidate Retrieval
                       ↓
                    Ranking
                       ↓
                  Re-Ranking
                       ↓
               Search Results

The important point is that search is not one algorithm.
It is an entire pipeline.
32. Building a Tiny Search Engine with Python
Let's create a very simple keyword search engine.
import refrom collections import defaultdict, Counterdocuments = {    1: "Python is widely used for machine learning",    2: "Java is popular for enterprise applications",    3: "Machine learning uses data to train models",    4: "Python is useful for data science",}def tokenize(text):    return re.findall(r"\w+", text.lower())index = defaultdict(list)for doc_id, text in documents.items():    words = set(tokenize(text))    for word in words:        index[word].append(doc_id)def search(query):    query_words = tokenize(query)    scores = Counter()    for word in query_words:        for doc_id in index.get(word, []):            scores[doc_id] += 1    return scores.most_common()


The program first builds an inverted index.
For example:
python
  ↓
Doc 1
Doc 4

and:
learning
  ↓
Doc 1
Doc 3

Then the query:
python machine learning

retrieves matching documents and gives higher scores to documents matching more query terms.
This is obviously far simpler than a production search engine, but the fundamental idea is similar.
33. Scaling the Search Index
A real search engine cannot keep its entire index on one machine.
The index may be split into shards.
                    Query
                      ↓
                Search Router
          ┌───────────┼───────────┐
          ↓           ↓           ↓
       Shard 1     Shard 2     Shard 3
          ↓           ↓           ↓
       Results      Results      Results
          └───────────┼───────────┘
                      ↓
                Merge + Rank

Each shard searches part of the index.
Results are then combined.
Replication can also be used for availability and higher query throughput.
34. Freshness
The web constantly changes.
Pages are:
Created
Updated
Deleted
Moved

Therefore, crawling cannot happen only once.
Search engines must continuously revisit pages.
Page Crawled
    ↓
Indexed
    ↓
Time Passes
    ↓
Re-Crawl
    ↓
Changed?
  ↙     ↘
Yes     No
 ↓       ↓
Update   Keep
Index    Existing Data

Determining how frequently to revisit each page is another optimization problem.
A frequently changing news site may require different scheduling from a rarely updated documentation page.
35. Duplicate Content
The same content may appear under multiple URLs.
For example:
example.com/product?id=100

example.com/products/100

example.com/product?id=100&utm_source=x

Search engines need techniques for recognizing duplicates and selecting useful canonical representations.
Otherwise, the index can waste storage and search results can become repetitive.
36. Search Quality vs Search Speed
Search systems constantly balance:
Quality
   ↕
Latency
   ↕
Cost

A huge machine-learning model might improve ranking slightly but take several seconds.
Users usually expect search results almost immediately.
Therefore, search architectures use multiple stages:
Fast Candidate Retrieval
          ↓
More Accurate Ranking
          ↓
Expensive Re-Ranking
only on a small result set

This architecture provides a practical balance between quality and speed.
37. Search and Modern AI Systems
Search technology has become increasingly important in modern AI systems.
Consider an AI assistant that needs external knowledge.
A common architecture is:
User Question
      ↓
Search / Retrieval
      ↓
Relevant Documents
      ↓
Language Model
      ↓
Generated Answer

This is closely related to Retrieval-Augmented Generation (RAG).
Search engines therefore are no longer used only for traditional search pages.
Search and retrieval now power:
AI Assistants
Enterprise Knowledge Search
Document Q&A
Code Search
Product Discovery
Recommendation Systems
Research Tools

38. Final Mental Model
The easiest way to understand a modern search engine is:
                    OFFLINE WORK

Internet
   ↓
Crawl
   ↓
Parse
   ↓
Understand
   ↓
Build Indexes
   ↓
Store Searchable Representations


                    ONLINE WORK

User Query
    ↓
Understand Query
    ↓
Retrieve Candidates
   ↙              ↘
Keywords        Semantics
   ↓              ↓
Inverted        Vector
Index           Index
   ↘              ↙
     Merge Candidates
            ↓
          Rank
            ↓
        Re-Rank
            ↓
      Search Results

Most of the internet is not searched from scratch when you type a query.
Instead, search engines prepare enormous indexes ahead of time so that relevant information can be retrieved extremely quickly.
Conclusion
Search engines solve one of the largest information problems in computing:
How can we find a small amount of relevant information inside an enormous collection of documents?

The answer involves much more than matching keywords.
A modern search system combines:
Web Crawling
Indexing
Inverted Indexes
Information Retrieval
Ranking Algorithms
Link Analysis
Query Understanding
Machine Learning
Embeddings
Vector Search
Hybrid Retrieval

The full journey looks like:
Web Page
   ↓
Crawler
   ↓
Parser
   ↓
Index
   ↓
User Query
   ↓
Candidate Retrieval
   ↓
Ranking
   ↓
Best Results

Traditional search asks:
Which documents contain these words?

Semantic search adds another question:
Which documents express this meaning?

And modern hybrid search combines both:
Exact Words
    +
Semantic Meaning
    +
Ranking Signals
    ↓
Better Retrieval

Understanding this architecture provides a foundation for much more than web search. The same concepts appear in databases, AI systems, e-commerce platforms, documentation search, code search, and large-scale knowledge systems.
