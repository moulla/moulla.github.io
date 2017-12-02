---
layout: default
title: Open-source Vespa Enterprise Search
title_desc: Yahoo open-sourced Vespa.ai in September. We've been working with it since
header_image : /images/vespa-out-of-the-box_960x250.jpg
pub_date: 01 Dec 2017
abstract: Introducing Yahoo's Vespa.ai
tweet_id: 936694266553913344
tags:
- Data Lake
- Unstructured Data
- Vespa.ai
- Search
---

Vespa is Yahoo's 'big data serving engine' and it was open sourced it in September 2017. We've been working on getting it into production deployment for a great start-up called [Metsitaba](https://www.metsitaba.com/){:target="_blank"}.

Not being part of the Yahoo team, we don't know where the name came from but we'd hazard a guess that v**ES**pa originated in Enterprise Search.  If you want to read more about it's origins then look at the [reddit ama](https://www.reddit.com/r/programming/comments/72r7uq/yahoo_open_sources_its_search_engine_vespa/) from Jon Bratseth.  You can even ask the Vespa team yourself <a class="twitter-mention-button" href="https://twitter.com/intent/tweet?screen_name=vespaengine">Twitter</a>

It exists as an alternative to ElasticSearch and Lucene, platforms we don't have a lot of ES experience, so we'll point you [elsewhere](http://opensourceconnections.com/blog/2017/10/06/vespa-vs-lucene-initial-impressions/){:target="_blank"} for some comparisons which seem positive and agree with our initial conclusions.  The other benefit right out of the box is **instant scale**.

Vespa is a container for JDisc components that runs in a docker(or Vagrant (or god forbid Windows)) container which you can run locally or on a cloud container - Inception much? Once you get comfortable with accessing the various layers management is straight - forward using the included command tools.
<div class='w3-card'>
<img src="/images/vespa-onion.png" alt="The container layers" style="width:100%">
</div>
<span class="w3-opacity" markdown="1">Default docker deployment with container access over [port 8080](http://docs.vespa.ai/documentation/reference/files-processes-and-ports.html)</span>

Getting your first Vespa container up and running is as easy as following the [great tutorial](http://docs.vespa.ai/documentation/vespa-quick-start.html){:target="_blank"}.  Within that container you can run various services, store various documents and access them via a single REST search endpoint.  The container can similarly be monitored and administer over a parallel REST api by publishing those ports from docker with the appropriate security.

Within the Vespa container there is a sophisticated staging and deployment platform for your search application.  Each application has the structure shown below
<div class='w3-card'>
<img src="/images/vespa-sample-file-structure.png" alt="Vespa application structure" style="width:100%">
</div>
<span class="w3-opacity" markdown="1">Vespa application structure</span>

From there we worked on getting our data-set into the platform.  Vespa provides some great interfaces but this is where we encountered our first pieces of work.

Vespa doesn't solve traditional data architecture problems.  You will still need to do this yourself.  Sure, you can feed it a lake of unstructured text, just like ES, and Vespa will automatically build the nlp components to do effective text search on the lake, but that falls short of the goals of any enterprise search platform.

To do the platforms out of the box search ability  justice you will still need a database architecture and schema to express in the search definition files and document structures.

When staging your documents to Vespa, there are two things that must be kept in sync : the Search Definition(.sd) file and the JSON for mated input data.  It was for this reason that we implemented our pre processing step in python which reformatted our existing database into the required feeding format and dynamic .sd file generation with the desired field settings and other configuration. We found this pipeline worked well as we tried various approaches to the search solution and underlying data architecture.

If you already have a data architecture then [Vespa provides Hadoop, Pig and Oozie integration](http://docs.vespa.ai/documentation/feed-using-hadoop-pig-oozie.html){:target="_blank"} as well and you will only need to structure the .sd file correctly, however we are working end-to-end and are intend on building spiders to directly feed the Vespa instance via the [document-processing component](http://docs.vespa.ai/documentation/document-processing-overview.html){:target="_blank"}

The Vespa documentation is generally very good and comprehensive, but in some cases can be a bit reference like and finding the answer takes a bit of searching.  The team are very active and are improving the documentation all the time.  The two pieces I found lacking were the application structure which you now have above(figuring out where or in which file in the tree to add config was a bit trial and error) and the structure of the complete query along with that queries interaction with the various components.  Defaults exist for most query special-tokens but these are not visible in the skeleton application, instead being embedded in the core components.  You can dig this out of the [Search api reference](http://docs.vespa.ai/documentation/reference/search-api-reference.html){:target="_blank"} 

The host:port endpoint is the [Search Container](http://docs.vespa.ai/documentation/querying-vespa.html){:target="_blank"} with the general form of a search request being

    http://host:port/search/?param1=value1&param2=value2&...

more specifically including some of the more frequently used options from [here](http://docs.vespa.ai/documentation/query-language.html) and [here](http://docs.vespa.ai/documentation/reference/search-api-reference.html)


  Search impact                           | URL element
  --------------------------------------- | -------------
  Vespa Container(hosts.xml)              |  http://localhost:8080 
  Search api                              | /search/?
  User Query                              |  query=xyz
  YQL element                             | &yql=select * from sources * where default contains 'bad'
  include User Query in YQL               | AND userQuery();
  choose searchChain                      | &searchChain=default
  choose default index(sd)                | &model.defaultIndex=default
  choose document sources(services.xml)   | &model.sources=demo,demo2,demo3
  geolocation element                     | &pos.ll=None
  queryprofile can include api settings   | &queryProfile=default
  hits returned (if < maxhits)            | &hits=10
  choose a ranking profile                | &ranking.profile=default
  trace internal vespa logging(0=off)     | &tracelevel=0
  offset from first result for paging     | &offest=0
  turn off any search preprocess rules    | &rules.off
  
  
								
From here your query is fired off at a particular vespa instance as below.

<div class='w3-card'>
<img src="/images/vespa-query-path.png" alt="Collect" style="width:100%">
</div>


and there it encounters layer upon layer of customise search interaction and returns your results in JSON for your UI developer to make pretty.

The layers we have played with so far allow everything from search substitution (dealing with mis-spelling, n-grams, keyword to field transforms), differing ranking on a document basis allowing re-ranking of results.

This highly effective enterprise search platform works well right out of the box and with investment can be customised to provide the search experience you require.

Oh and the scale, it's not a moon ...



