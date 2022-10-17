Nutch Manual
======

## 1. Nutch 프로젝트 clone 및 build

### 1-1. clone
GitHub 에서 clone 해준다. (예시는 1.19 버전을 클론함)  
``` shell
git clone -b release-1.19 https://github.com/apache/nutch
```
### 1-2. build
Nutch project root directory 로 진입한다.  
``` shell
cd nutch
```

ant 를 사용하여 빌드해주자. dependency 들은 ant 가 `build.xml` 을 읽어 알아서 해준다.  
아래 명령어를 실행하면 `build.xml` 에 default target 으로 기재된 `ant runtime` 이 실행된다:  
``` shell
ant
```

혹은:  
``` shell
ant runtime
```

#### IntelliJ 로 빌드할 시 (Linux)
1. File > Project Structure >> Project Settings > Project 에서 SDK 설정해주자.
2. Ant, ivyIDEA 플러그인 설치해주자.
3. target 중 runtime 을 실행시켜주자.


### 1-3. Nutch 실행하고 실행가능한 메뉴 보기
아래와 같이 Nutch 를 단순실행하면, 실행가능한 메뉴들의 목록 및 간단한 설명을 볼 수 있다.  
실행:  
``` shell
./runtime/local/bin/nutch
```

결과:  
``` text
nutch 1.19
Usage: nutch COMMAND [-Dproperty=value]... [command-specific args]...
where COMMAND is one of:
  readdb            read / dump crawl db
  mergedb           merge crawldb-s, with optional filtering
  readlinkdb        read / dump link db
  inject            inject new urls into the database
  generate          generate new segments to fetch from crawl db
  freegen           generate new segments to fetch from text files
  fetch             fetch a segment's pages
  parse             parse a segment's pages
  readseg           read / dump segment data
  mergesegs         merge several segments, with optional filtering and slicing
  updatedb          update crawl db from segments after fetching
  invertlinks       create a linkdb from parsed segments
  mergelinkdb       merge linkdb-s, with optional filtering
  index             run the plugin-based indexer on parsed segments and linkdb
  dedup             deduplicate entries in the crawldb and give them a special status
  dump              exports crawled data from segments into files
  commoncrawldump   exports crawled data from segments into common crawl data format encoded as CBOR
  solrindex         run the solr indexer on parsed segments and linkdb - DEPRECATED use the index command instead
  solrdedup         remove duplicates from solr - DEPRECATED use the dedup command instead
  solrclean         remove HTTP 301 and 404 documents from solr - DEPRECATED use the clean command instead
  clean             remove HTTP 301 and 404 documents and duplicates from indexing backends configured via plugins
  parsechecker      check the parser for a given url
  indexchecker      check the indexing filters for a given url
  filterchecker     check url filters for a given url
  normalizerchecker check url normalizers for a given url
  domainstats       calculate domain statistics from crawldb
  protocolstats     calculate protocol status code stats from crawldb
  crawlcomplete     calculate crawl completion stats from crawldb
  webgraph          generate a web graph from existing segments
  linkrank          run a link analysis program on the generated web graph
  scoreupdater      updates the crawldb with linkrank scores
  nodedumper        dumps the web graph's node scores
  plugin            load a plugin and run one of its classes main()
  junit             runs the given JUnit test
  startserver       runs the Nutch Server on localhost:8081
  warc              exports crawled data from segments at the WARC format
  updatehostdb      update the host db with records from the crawl db
  readhostdb        read / dump host db
  sitemap           perform Sitemap processing
  showproperties    print Nutch/Hadoop configuration properties to stdout
 or
  CLASSNAME         run the class named CLASSNAME
Most commands print help when invoked w/o parameters.
```

## 2. Nutch Injector 실행
### 2-1. Nutch Injector 사용법 확인하기
Nutch Injector 도 단순히 nutch 명령어 뒤에 `inject` 명령어만 기입하면 사용법을 볼 수 있다.

인자없이 `inject` 실행:  
``` shell
./runtime/local/bin/nutch inject
```

결과:  
``` text
Usage: Injector [-D...] <crawldb> <url_dir> [-overwrite|-update] [-noFilter] [-noNormalize] [-filterNormalizeAll]

  <crawldb>	Path to a crawldb directory. If not present, a new one would be created.
  <url_dir>	Path to URL file or directory with URL file(s) containing URLs to be injected.
           	A URL file should have one URL per line, optionally followed by custom metadata.
           	Blank lines or lines starting with a '#' would be ignored. Custom metadata must
           	be of form 'key=value' and separated by tabs.
           	Below are reserved metadata keys:

           		nutch.score: A custom score for a url
           		nutch.fetchInterval: A custom fetch interval for a url
           		nutch.fetchInterval.fixed: A custom fetch interval for a url that is not changed by AdaptiveFetchSchedule

           	Example:
           	 http://www.apache.org/
           	 http://www.nutch.org/ \t nutch.score=10 \t nutch.fetchInterval=2592000 \t userType=open_source

 -overwrite	Overwite existing crawldb records by the injected records. Has precedence over 'update'
 -update   	Update existing crawldb records with the injected records. Old metadata is preserved

 -noNormalize	Do not normalize URLs before injecting
 -noFilter 	Do not apply URL filters to injected URLs
 -filterNormalizeAll
           	Normalize and filter all URLs including the URLs of existing CrawlDb records

 -D...     	set or overwrite configuration property (property=value)
 -Ddb.update.purge.404=true
           	remove URLs with status gone (404) from CrawlDb
```


### 2-2. Nutch Injector 실행하고, 결과 확인해보기

`seed_urls` 라는 이름으로 seed URL 들을 저장하는 디렉토리를 만들어보자.
``` shell
mkdir seed_urls/
```

`seed_urls/seeds1.txt` 을 아래와 같이 작성해보자:  
``` text
# This is comment for Nutch's seed URL file
https://www.tmax.co.kr/tmaxai
https://www.tmax.co.kr/tmaxrg
```

예시 파일 `seed_urls/seeds1.txt` 의 첫째줄은 주석이며, 두번째 줄에서 Tmax AI 웹사이트 URL, 세번째 줄에서 Tmax RG 웹사이트 URL 을 seed로 선정하고 있다.  
아래 명령어로 injector 를 실행해보자:  
``` shell
./runtime/local/bin/nutch inject crawldb/ seed_urls/
```

`crawldb/` 디렉토리가 존재하지 않은 경우에는 Nutch 에서 디렉토리를 자동으로 생성해준다.  

아래와 같은 결과를 볼 수 있다:  
``` text
2022-10-13 15:44:33,418 INFO o.a.n.p.PluginManifestParser [main] Plugins: looking in: /home/jongho/nutch/runtime/local/plugins
2022-10-13 15:44:33,509 INFO o.a.n.p.PluginRepository [main] Plugin Auto-activation mode: [true]
2022-10-13 15:44:33,510 INFO o.a.n.p.PluginRepository [main] Registered Plugins:
2022-10-13 15:44:33,510 INFO o.a.n.p.PluginRepository [main] 	Regex URL Filter (urlfilter-regex)
2022-10-13 15:44:33,510 INFO o.a.n.p.PluginRepository [main] 	Html Parse Plug-in (parse-html)
2022-10-13 15:44:33,510 INFO o.a.n.p.PluginRepository [main] 	HTTP Framework (lib-http)
2022-10-13 15:44:33,510 INFO o.a.n.p.PluginRepository [main] 	the nutch core extension points (nutch-extensionpoints)
2022-10-13 15:44:33,510 INFO o.a.n.p.PluginRepository [main] 	Basic Indexing Filter (index-basic)
2022-10-13 15:44:33,510 INFO o.a.n.p.PluginRepository [main] 	Anchor Indexing Filter (index-anchor)
2022-10-13 15:44:33,510 INFO o.a.n.p.PluginRepository [main] 	Tika Parser Plug-in (parse-tika)
2022-10-13 15:44:33,510 INFO o.a.n.p.PluginRepository [main] 	Basic URL Normalizer (urlnormalizer-basic)
2022-10-13 15:44:33,511 INFO o.a.n.p.PluginRepository [main] 	Regex URL Filter Framework (lib-regex-filter)
2022-10-13 15:44:33,511 INFO o.a.n.p.PluginRepository [main] 	Regex URL Normalizer (urlnormalizer-regex)
2022-10-13 15:44:33,511 INFO o.a.n.p.PluginRepository [main] 	URL Validator (urlfilter-validator)
2022-10-13 15:44:33,511 INFO o.a.n.p.PluginRepository [main] 	CyberNeko HTML Parser (lib-nekohtml)
2022-10-13 15:44:33,511 INFO o.a.n.p.PluginRepository [main] 	OPIC Scoring Plug-in (scoring-opic)
2022-10-13 15:44:33,511 INFO o.a.n.p.PluginRepository [main] 	Pass-through URL Normalizer (urlnormalizer-pass)
2022-10-13 15:44:33,511 INFO o.a.n.p.PluginRepository [main] 	Http Protocol Plug-in (protocol-http)
2022-10-13 15:44:33,511 INFO o.a.n.p.PluginRepository [main] 	SolrIndexWriter (indexer-solr)
2022-10-13 15:44:33,511 INFO o.a.n.p.PluginRepository [main] Registered Extension-Points:
2022-10-13 15:44:33,511 INFO o.a.n.p.PluginRepository [main] 	 (Nutch Content Parser)
2022-10-13 15:44:33,511 INFO o.a.n.p.PluginRepository [main] 	 (Nutch URL Filter)
2022-10-13 15:44:33,511 INFO o.a.n.p.PluginRepository [main] 	 (HTML Parse Filter)
2022-10-13 15:44:33,511 INFO o.a.n.p.PluginRepository [main] 	 (Nutch Scoring)
2022-10-13 15:44:33,511 INFO o.a.n.p.PluginRepository [main] 	 (Nutch URL Normalizer)
2022-10-13 15:44:33,511 INFO o.a.n.p.PluginRepository [main] 	 (Nutch Publisher)
2022-10-13 15:44:33,511 INFO o.a.n.p.PluginRepository [main] 	 (Nutch Exchange)
2022-10-13 15:44:33,512 INFO o.a.n.p.PluginRepository [main] 	 (Nutch Protocol)
2022-10-13 15:44:33,512 INFO o.a.n.p.PluginRepository [main] 	 (Nutch URL Ignore Exemption Filter)
2022-10-13 15:44:33,512 INFO o.a.n.p.PluginRepository [main] 	 (Nutch Index Writer)
2022-10-13 15:44:33,512 INFO o.a.n.p.PluginRepository [main] 	 (Nutch Segment Merge Filter)
2022-10-13 15:44:33,512 INFO o.a.n.p.PluginRepository [main] 	 (Nutch Indexing Filter)
2022-10-13 15:44:33,513 INFO o.a.n.c.Injector [main] Injector: starting at 2022-10-13 15:44:33
2022-10-13 15:44:33,513 INFO o.a.n.c.Injector [main] Injector: crawlDb: crawldb
2022-10-13 15:44:33,513 INFO o.a.n.c.Injector [main] Injector: urlDir: seeds
2022-10-13 15:44:33,513 INFO o.a.n.c.Injector [main] Injector: Converting injected urls to crawl db entries.
2022-10-13 15:44:33,660 INFO o.a.n.c.Injector [main] Injecting seed URL file file:/home/jongho/nutch/seeds/seeds1.txt
2022-10-13 15:44:34,067 INFO o.a.n.n.u.r.RegexURLNormalizer [LocalJobRunner Map Task Executor #0] can't find rules for scope 'inject', using default
2022-10-13 15:44:34,132 INFO o.a.n.c.Injector [pool-5-thread-1] Injector: overwrite: false
2022-10-13 15:44:34,132 INFO o.a.n.c.Injector [pool-5-thread-1] Injector: update: false
2022-10-13 15:44:34,952 INFO o.a.n.c.Injector [main] Injector: Total urls rejected by filters: 0
2022-10-13 15:44:34,952 INFO o.a.n.c.Injector [main] Injector: Total urls injected after normalization and filtering: 2
2022-10-13 15:44:34,952 INFO o.a.n.c.Injector [main] Injector: Total urls injected but already in CrawlDb: 2
2022-10-13 15:44:34,952 INFO o.a.n.c.Injector [main] Injector: Total new urls injected: 0
2022-10-13 15:44:34,954 INFO o.a.n.c.Injector [main] Injector: finished at 2022-10-13 15:44:34, elapsed: 00:00:01
```

inject 실행 이후 `crawldb/` 디렉토리 내용은 아래와 같다:  
``` text
crawldb/
+-- current/
|   +-- part-r-00000/
|       +-- data
|       +-- index
|
+-- old/
    +-- part-r-00000/
        +-- data
        +-- index

```

data, index 등의 파일을 열어보면 글씨가 깨지는 현상이 나오는데,  
이는 `crawldb/` 에 저장되는 파일들이 HDFS (Hadoop File System) 형식으로 저장되기 때문이다.  


## 3. Nutch Generator 실행
TODO


## 4. Nutch Fetcher 실행
TODO
