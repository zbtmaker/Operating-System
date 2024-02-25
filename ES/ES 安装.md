# ES 安装


```bash
zbt@192 ~ % curl -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.11.3-darwin-x86_64.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  405M  100  405M    0     0  18.1M      0  0:00:22  0:00:22 --:--:-- 22.1M
zbt@192 ~ % curl https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.11.3-darwin-x86_64.tar.gz.sha512 | shasum -a 512 -c - 
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   172  100   172    0     0    240      0 --:--:-- --:--:-- --:--:--   241
^[[Celasticsearch-8.11.3-darwin-x86_64.tar.gz: OK
zoubaitao@192 ~ % curl https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.11.3-darwin-x86_64.tar.gz.sha512 | shasum -a 512 -c - 
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   172  100   172    0     0    225      0 --:--:-- --:--:-- --:--:--   226
elasticsearch-8.11.3-darwin-x86_64.tar.gz: OK
zbt@192 ~ % tar -xzf elasticsearch-8.11.3-darwin-x86_64.tar.gz
zbt@192 ~ % ls
Applications					Downloads					Pictures					elasticsearch-8.11.3
Blog						Library						Programming					elasticsearch-8.11.3-darwin-x86_64.tar.gz
Desktop						Movies						Public						go
Documents					Music						Software					logs
zbt@192 ~ % cd elasticsearch-8.11.3
zbt@192 elasticsearch-8.11.3 % ls
LICENSE.txt	NOTICE.txt	README.asciidoc	bin		config		jdk.app		lib		logs		modules		plugins
zbt@192 elasticsearch-8.11.3 % cd bin
zbt@192 bin % l
zsh: command not found: l
zbt@192 bin % ls 
elasticsearch				elasticsearch-create-enrollment-token	elasticsearch-geoip			elasticsearch-reconfigure-node		elasticsearch-setup-passwords		elasticsearch-syskeygen
elasticsearch-certgen			elasticsearch-croneval			elasticsearch-keystore			elasticsearch-reset-password		elasticsearch-shard			elasticsearch-users
elasticsearch-certutil			elasticsearch-env			elasticsearch-node			elasticsearch-saml-metadata		elasticsearch-sql-cli
elasticsearch-cli			elasticsearch-env-from-file		elasticsearch-plugin			elasticsearch-service-tokens		elasticsearch-sql-cli-8.11.3.jar
zbt@192 bin % elasticsearch
zsh: command not found: elasticsearch
zbt@192 bin % ./elasticsearch
warning: ignoring JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_301.jdk/Contents/Home; using bundled JDK
十二月 16, 2023 11:11:45 上午 sun.util.locale.provider.LocaleProviderAdapter <clinit>
WARNING: COMPAT locale provider will be removed in a future release
[2023-12-16T11:11:48,769][INFO ][o.a.l.i.v.PanamaVectorizationProvider] [192.168.0.103] Java vector incubator API enabled; uses preferredBitSize=256
[2023-12-16T11:11:49,218][INFO ][o.e.n.Node               ] [192.168.0.103] version[8.11.3], pid[13381], build[tar/64cf052f3b56b1fd4449f5454cb88aca7e739d9a/2023-12-08T11:33:53.634979452Z], OS[Mac OS X/12.6/x86_64], JVM[Oracle Corporation/OpenJDK 64-Bit Server VM/21.0.1/21.0.1+12-29]
[2023-12-16T11:11:49,219][INFO ][o.e.n.Node               ] [192.168.0.103] JVM home [/Users/zoubaitao/elasticsearch-8.11.3/jdk.app/Contents/Home], using bundled JDK [true]
[2023-12-16T11:11:49,219][INFO ][o.e.n.Node               ] [192.168.0.103] JVM arguments [-Des.networkaddress.cache.ttl=60, -Des.networkaddress.cache.negative.ttl=10, -Djava.security.manager=allow, -XX:+AlwaysPreTouch, -Xss1m, -Djava.awt.headless=true, -Dfile.encoding=UTF-8, -Djna.nosys=true, -XX:-OmitStackTraceInFastThrow, -Dio.netty.noUnsafe=true, -Dio.netty.noKeySetOptimization=true, -Dio.netty.recycler.maxCapacityPerThread=0, -Dlog4j.shutdownHookEnabled=false, -Dlog4j2.disable.jmx=true, -Dlog4j2.formatMsgNoLookups=true, -Djava.locale.providers=SPI,COMPAT, --add-opens=java.base/java.io=org.elasticsearch.preallocate, -XX:+UseG1GC, -Djava.io.tmpdir=/var/folders/gz/xx_hsf5s0wx4qnqqqt6fc9t00000gn/T/elasticsearch-740136646129116962, --add-modules=jdk.incubator.vector, -XX:+HeapDumpOnOutOfMemoryError, -XX:+ExitOnOutOfMemoryError, -XX:HeapDumpPath=data, -XX:ErrorFile=logs/hs_err_pid%p.log, -Xlog:gc*,gc+age=trace,safepoint:file=logs/gc.log:utctime,level,pid,tags:filecount=32,filesize=64m, -Xms8192m, -Xmx8192m, -XX:MaxDirectMemorySize=4294967296, -XX:InitiatingHeapOccupancyPercent=30, -XX:G1ReservePercent=25, -Des.distribution.type=tar, --module-path=/Users/zoubaitao/elasticsearch-8.11.3/lib, --add-modules=jdk.net, --add-modules=ALL-MODULE-PATH, -Djdk.module.main=org.elasticsearch.server]
[2023-12-16T11:11:54,835][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [repository-url]
[2023-12-16T11:11:54,835][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [rest-root]
[2023-12-16T11:11:54,836][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [x-pack-core]
[2023-12-16T11:11:54,836][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [x-pack-redact]
[2023-12-16T11:11:54,836][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [ingest-user-agent]
[2023-12-16T11:11:54,836][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [x-pack-async-search]
[2023-12-16T11:11:54,836][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [x-pack-monitoring]
[2023-12-16T11:11:54,836][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [repository-s3]
[2023-12-16T11:11:54,837][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [x-pack-analytics]
[2023-12-16T11:11:54,837][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [x-pack-ent-search]
[2023-12-16T11:11:54,837][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [x-pack-autoscaling]
[2023-12-16T11:11:54,837][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [lang-painless]
[2023-12-16T11:11:54,837][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [x-pack-ml]
[2023-12-16T11:11:54,837][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [legacy-geo]
[2023-12-16T11:11:54,838][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [lang-mustache]
[2023-12-16T11:11:54,838][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [x-pack-ql]
[2023-12-16T11:11:54,838][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [rank-rrf]
[2023-12-16T11:11:54,838][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [analysis-common]
[2023-12-16T11:11:54,838][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [transport-netty4]
[2023-12-16T11:11:54,838][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [aggregations]
[2023-12-16T11:11:54,838][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [ingest-common]
[2023-12-16T11:11:54,839][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [x-pack-identity-provider]
[2023-12-16T11:11:54,839][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [frozen-indices]
[2023-12-16T11:11:54,839][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [x-pack-text-structure]
[2023-12-16T11:11:54,839][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [x-pack-shutdown]
[2023-12-16T11:11:54,839][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [snapshot-repo-test-kit]
[2023-12-16T11:11:54,839][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [ml-package-loader]
[2023-12-16T11:11:54,839][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [kibana]
[2023-12-16T11:11:54,840][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [constant-keyword]
[2023-12-16T11:11:54,840][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [x-pack-logstash]
[2023-12-16T11:11:54,840][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [x-pack-graph]
[2023-12-16T11:11:54,840][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [x-pack-ccr]
[2023-12-16T11:11:54,840][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [x-pack-esql]
[2023-12-16T11:11:54,840][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [parent-join]
[2023-12-16T11:11:54,841][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [x-pack-enrich]
[2023-12-16T11:11:54,841][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [repositories-metering-api]
[2023-12-16T11:11:54,841][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [transform]
[2023-12-16T11:11:54,841][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [repository-azure]
[2023-12-16T11:11:54,841][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [repository-gcs]
[2023-12-16T11:11:54,841][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [spatial]
[2023-12-16T11:11:54,841][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [apm]
[2023-12-16T11:11:54,841][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [mapper-extras]
[2023-12-16T11:11:54,842][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [mapper-version]
[2023-12-16T11:11:54,842][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [x-pack-rollup]
[2023-12-16T11:11:54,842][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [percolator]
[2023-12-16T11:11:54,842][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [data-streams]
[2023-12-16T11:11:54,842][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [x-pack-stack]
[2023-12-16T11:11:54,842][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [rank-eval]
[2023-12-16T11:11:54,842][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [reindex]
[2023-12-16T11:11:54,843][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [x-pack-security]
[2023-12-16T11:11:54,843][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [blob-cache]
[2023-12-16T11:11:54,843][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [searchable-snapshots]
[2023-12-16T11:11:54,843][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [x-pack-slm]
[2023-12-16T11:11:54,843][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [snapshot-based-recoveries]
[2023-12-16T11:11:54,843][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [x-pack-watcher]
[2023-12-16T11:11:54,843][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [old-lucene-versions]
[2023-12-16T11:11:54,843][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [x-pack-ilm]
[2023-12-16T11:11:54,844][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [x-pack-voting-only-node]
[2023-12-16T11:11:54,844][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [x-pack-deprecation]
[2023-12-16T11:11:54,844][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [x-pack-fleet]
[2023-12-16T11:11:54,844][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [x-pack-profiling]
[2023-12-16T11:11:54,844][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [x-pack-aggregate-metric]
[2023-12-16T11:11:54,844][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [x-pack-downsample]
[2023-12-16T11:11:54,845][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [ingest-geoip]
[2023-12-16T11:11:54,845][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [inference]
[2023-12-16T11:11:54,845][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [x-pack-write-load-forecaster]
[2023-12-16T11:11:54,845][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [search-business-rules]
[2023-12-16T11:11:54,845][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [wildcard]
[2023-12-16T11:11:54,845][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [ingest-attachment]
[2023-12-16T11:11:54,845][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [unsigned-long]
[2023-12-16T11:11:54,845][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [x-pack-sql]
[2023-12-16T11:11:54,846][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [runtime-fields-common]
[2023-12-16T11:11:54,846][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [x-pack-async]
[2023-12-16T11:11:54,846][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [vector-tile]
[2023-12-16T11:11:54,846][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [lang-expression]
[2023-12-16T11:11:54,846][INFO ][o.e.p.PluginsService     ] [192.168.0.103] loaded module [x-pack-eql]
[2023-12-16T11:11:57,137][INFO ][o.e.e.NodeEnvironment    ] [192.168.0.103] using [1] data paths, mounts [[/System/Volumes/Data (/dev/disk1s2)]], net usable_space [232.5gb], net total_space [465.6gb], types [apfs]
[2023-12-16T11:11:57,137][INFO ][o.e.e.NodeEnvironment    ] [192.168.0.103] heap size [8gb], compressed ordinary object pointers [true]
[2023-12-16T11:11:57,163][INFO ][o.e.n.Node               ] [192.168.0.103] node name [192.168.0.103], node ID [cEOQD9KzSzWkKMhV8rhU4g], cluster name [elasticsearch], roles [data_content, data_warm, master, remote_cluster_client, data, data_cold, ingest, data_frozen, ml, data_hot, transform]
[2023-12-16T11:11:59,710][INFO ][o.e.x.m.p.l.CppLogMessageHandler] [192.168.0.103] [controller/13383] [Main.cc@123] controller (64 bit): Version 8.11.3 (Build c16ff912638f0a) Copyright (c) 2023 Elasticsearch BV
[2023-12-16T11:12:00,020][INFO ][o.e.x.s.Security         ] [192.168.0.103] Security is enabled
[2023-12-16T11:12:00,538][INFO ][o.e.x.s.a.s.FileRolesStore] [192.168.0.103] parsed [0] roles from file [/Users/zoubaitao/elasticsearch-8.11.3/config/roles.yml]

[2023-12-16T11:12:01,198][INFO ][o.e.x.p.ProfilingPlugin  ] [192.168.0.103] Profiling is enabled
[2023-12-16T11:12:01,216][INFO ][o.e.x.p.ProfilingPlugin  ] [192.168.0.103] profiling index templates will not be installed or reinstalled
[2023-12-16T11:12:01,947][INFO ][o.e.t.n.NettyAllocator   ] [192.168.0.103] creating NettyAllocator with the following configs: [name=elasticsearch_configured, chunk_size=1mb, suggested_max_allocation_size=1mb, factors={es.unsafe.use_netty_default_chunk_and_page_size=false, g1gc_enabled=true, g1gc_region_size=4mb}]
[2023-12-16T11:12:01,971][INFO ][o.e.i.r.RecoverySettings ] [192.168.0.103] using rate limit [40mb] with [default=40mb, read=0b, write=0b, max=0b]
[2023-12-16T11:12:02,011][INFO ][o.e.d.DiscoveryModule    ] [192.168.0.103] using discovery type [multi-node] and seed hosts providers [settings]
[2023-12-16T11:12:03,099][INFO ][o.e.n.Node               ] [192.168.0.103] initialized
[2023-12-16T11:12:03,100][INFO ][o.e.n.Node               ] [192.168.0.103] starting ...
[2023-12-16T11:12:03,171][INFO ][o.e.x.s.c.f.PersistentCache] [192.168.0.103] persistent cache index loaded
[2023-12-16T11:12:03,172][INFO ][o.e.x.d.l.DeprecationIndexingComponent] [192.168.0.103] deprecation component started
[2023-12-16T11:12:03,284][INFO ][o.e.t.TransportService   ] [192.168.0.103] publish_address {127.0.0.1:9300}, bound_addresses {[::1]:9300}, {127.0.0.1:9300}
[2023-12-16T11:12:03,549][INFO ][o.e.c.c.ClusterBootstrapService] [192.168.0.103] this node has not joined a bootstrapped cluster yet; [cluster.initial_master_nodes] is set to [192.168.0.103]
[2023-12-16T11:12:03,554][INFO ][o.e.c.c.Coordinator      ] [192.168.0.103] setting initial configuration to VotingConfiguration{cEOQD9KzSzWkKMhV8rhU4g}
[2023-12-16T11:12:03,851][INFO ][o.e.c.s.MasterService    ] [192.168.0.103] elected-as-master ([1] nodes joined in term 1)[_FINISH_ELECTION_, {192.168.0.103}{cEOQD9KzSzWkKMhV8rhU4g}{SAlFFRxSQlWPg8BqAVPzHw}{192.168.0.103}{127.0.0.1}{127.0.0.1:9300}{cdfhilmrstw}{8.11.3}{7000099-8500003} completing election], term: 1, version: 1, delta: master node changed {previous [], current [{192.168.0.103}{cEOQD9KzSzWkKMhV8rhU4g}{SAlFFRxSQlWPg8BqAVPzHw}{192.168.0.103}{127.0.0.1}{127.0.0.1:9300}{cdfhilmrstw}{8.11.3}{7000099-8500003}]}
[2023-12-16T11:12:03,930][INFO ][o.e.c.c.CoordinationState] [192.168.0.103] cluster UUID set to [SfXwYH5hR8GXMVW2rMBSxA]
[2023-12-16T11:12:04,003][INFO ][o.e.c.s.ClusterApplierService] [192.168.0.103] master node changed {previous [], current [{192.168.0.103}{cEOQD9KzSzWkKMhV8rhU4g}{SAlFFRxSQlWPg8BqAVPzHw}{192.168.0.103}{127.0.0.1}{127.0.0.1:9300}{cdfhilmrstw}{8.11.3}{7000099-8500003}]}, term: 1, version: 1, reason: Publication{term=1, version=1}
[2023-12-16T11:12:04,037][INFO ][o.e.c.f.AbstractFileWatchingService] [192.168.0.103] starting file watcher ...
[2023-12-16T11:12:04,040][INFO ][o.e.c.f.AbstractFileWatchingService] [192.168.0.103] file settings service up and running [tid=80]
[2023-12-16T11:12:04,046][INFO ][o.e.c.c.NodeJoinExecutor ] [192.168.0.103] node-join: [{192.168.0.103}{cEOQD9KzSzWkKMhV8rhU4g}{SAlFFRxSQlWPg8BqAVPzHw}{192.168.0.103}{127.0.0.1}{127.0.0.1:9300}{cdfhilmrstw}{8.11.3}{7000099-8500003}] with reason [completing election]
[2023-12-16T11:12:04,049][INFO ][o.e.h.AbstractHttpServerTransport] [192.168.0.103] publish_address {192.168.0.103:9200}, bound_addresses {[::]:9200}
[2023-12-16T11:12:04,049][INFO ][o.e.n.Node               ] [192.168.0.103] started {192.168.0.103}{cEOQD9KzSzWkKMhV8rhU4g}{SAlFFRxSQlWPg8BqAVPzHw}{192.168.0.103}{127.0.0.1}{127.0.0.1:9300}{cdfhilmrstw}{8.11.3}{7000099-8500003}{xpack.installed=true, transform.config_version=10.0.0, ml.machine_memory=17179869184, ml.allocated_processors=12, ml.allocated_processors_double=12.0, ml.max_jvm_size=8589934592, ml.config_version=11.0.0}
[2023-12-16T11:12:04,174][INFO ][o.e.g.GatewayService     ] [192.168.0.103] recovered [0] indices into cluster_state
[2023-12-16T11:12:04,314][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding component template [behavioral_analytics-events-mappings]
[2023-12-16T11:12:04,327][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding index template [search-acl-filter] for index patterns [.search-acl-filter-*]
[2023-12-16T11:12:04,343][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding index template [.monitoring-ent-search-mb] for index patterns [.monitoring-ent-search-8-*]
[2023-12-16T11:12:04,347][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding index template [.ml-state] for index patterns [.ml-state*]
[2023-12-16T11:12:04,366][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding component template [elastic-connectors-mappings]
[2023-12-16T11:12:04,374][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding index template [.ml-notifications-000002] for index patterns [.ml-notifications-000002]
[2023-12-16T11:12:04,377][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding component template [elastic-connectors-settings]
[2023-12-16T11:12:04,392][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding index template [.monitoring-kibana-mb] for index patterns [.monitoring-kibana-8-*]
[2023-12-16T11:12:04,395][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding component template [elastic-connectors-sync-jobs-settings]
[2023-12-16T11:12:04,400][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding component template [elastic-connectors-sync-jobs-mappings]
[2023-12-16T11:12:04,407][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding index template [.ml-stats] for index patterns [.ml-stats-*]
[2023-12-16T11:12:04,423][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding index template [.monitoring-logstash-mb] for index patterns [.monitoring-logstash-8-*]
[2023-12-16T11:12:04,437][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding index template [.ml-anomalies-] for index patterns [.ml-anomalies-*]
[2023-12-16T11:12:04,444][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding template [.monitoring-logstash] for index patterns [.monitoring-logstash-7-*]
[2023-12-16T11:12:04,448][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding template [.monitoring-alerts-7] for index patterns [.monitoring-alerts-7]
[2023-12-16T11:12:04,457][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding template [.monitoring-beats] for index patterns [.monitoring-beats-7-*]
[2023-12-16T11:12:04,462][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding template [.monitoring-kibana] for index patterns [.monitoring-kibana-7-*]
[2023-12-16T11:12:04,477][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding template [.monitoring-es] for index patterns [.monitoring-es-7-*]
[2023-12-16T11:12:04,494][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding index template [.monitoring-beats-mb] for index patterns [.monitoring-beats-8-*]
[2023-12-16T11:12:04,516][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding index template [.monitoring-es-mb] for index patterns [.monitoring-es-8-*]
[2023-12-16T11:12:04,518][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding component template [metrics-tsdb-settings]
[2023-12-16T11:12:04,523][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding component template [synthetics-mappings]
[2023-12-16T11:12:04,526][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding component template [metrics-settings]
[2023-12-16T11:12:04,529][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding component template [metrics-mappings]
[2023-12-16T11:12:04,532][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding component template [synthetics-settings]
[2023-12-16T11:12:04,543][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding component template [data-streams-mappings]
[2023-12-16T11:12:04,548][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding component template [logs-mappings]
[2023-12-16T11:12:04,554][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding index template [.kibana-reporting] for index patterns [.kibana-reporting*]
[2023-12-16T11:12:04,558][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding component template [ecs@dynamic_templates]
[2023-12-16T11:12:04,562][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding index template [.slm-history] for index patterns [.slm-history-5*]
[2023-12-16T11:12:04,567][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding index template [ilm-history] for index patterns [ilm-history-5*]
[2023-12-16T11:12:04,576][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding index template [.watch-history-16] for index patterns [.watcher-history-16*]
[2023-12-16T11:12:04,580][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding component template [.deprecation-indexing-mappings]
[2023-12-16T11:12:04,582][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding component template [.deprecation-indexing-settings]
[2023-12-16T11:12:04,586][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding index template [.fleet-fileds-tohost-meta] for index patterns [.fleet-fileds-tohost-meta-*]
[2023-12-16T11:12:04,590][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding index template [.fleet-fileds-fromhost-meta] for index patterns [.fleet-fileds-fromhost-meta-*]
[2023-12-16T11:12:04,595][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding index template [.fleet-fileds-fromhost-data] for index patterns [.fleet-fileds-fromhost-data-*]
[2023-12-16T11:12:04,599][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding index template [.fleet-fileds-tohost-data] for index patterns [.fleet-fileds-tohost-data-*]
[2023-12-16T11:12:04,736][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding index template [elastic-connectors-sync-jobs] for index patterns [.elastic-connectors-sync-jobs-v1]
[2023-12-16T11:12:04,740][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding index template [elastic-connectors] for index patterns [.elastic-connectors-v1]
[2023-12-16T11:12:04,743][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding index template [synthetics] for index patterns [synthetics-*-*]
[2023-12-16T11:12:04,747][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding index template [metrics] for index patterns [metrics-*-*]
[2023-12-16T11:12:04,750][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding index template [.deprecation-indexing-template] for index patterns [.logs-deprecation.*]
[2023-12-16T11:12:04,824][INFO ][o.e.x.i.a.TransportPutLifecycleAction] [192.168.0.103] adding index lifecycle policy [behavioral_analytics-events-default_policy]
[2023-12-16T11:12:04,923][INFO ][o.e.x.i.a.TransportPutLifecycleAction] [192.168.0.103] adding index lifecycle policy [ml-size-based-ilm-policy]
[2023-12-16T11:12:04,999][INFO ][o.e.x.i.a.TransportPutLifecycleAction] [192.168.0.103] adding index lifecycle policy [.monitoring-8-ilm-policy]
[2023-12-16T11:12:05,072][INFO ][o.e.x.i.a.TransportPutLifecycleAction] [192.168.0.103] adding index lifecycle policy [logs]
[2023-12-16T11:12:05,147][INFO ][o.e.x.i.a.TransportPutLifecycleAction] [192.168.0.103] adding index lifecycle policy [metrics]
[2023-12-16T11:12:05,221][INFO ][o.e.x.i.a.TransportPutLifecycleAction] [192.168.0.103] adding index lifecycle policy [synthetics]
[2023-12-16T11:12:05,295][INFO ][o.e.x.i.a.TransportPutLifecycleAction] [192.168.0.103] adding index lifecycle policy [7-days-default]
[2023-12-16T11:12:05,371][INFO ][o.e.x.i.a.TransportPutLifecycleAction] [192.168.0.103] adding index lifecycle policy [30-days-default]
[2023-12-16T11:12:05,445][INFO ][o.e.x.i.a.TransportPutLifecycleAction] [192.168.0.103] adding index lifecycle policy [90-days-default]
[2023-12-16T11:12:05,518][INFO ][o.e.x.i.a.TransportPutLifecycleAction] [192.168.0.103] adding index lifecycle policy [180-days-default]
[2023-12-16T11:12:05,593][INFO ][o.e.x.i.a.TransportPutLifecycleAction] [192.168.0.103] adding index lifecycle policy [365-days-default]
[2023-12-16T11:12:05,668][INFO ][o.e.x.i.a.TransportPutLifecycleAction] [192.168.0.103] adding index lifecycle policy [slm-history-ilm-policy]
[2023-12-16T11:12:05,743][INFO ][o.e.x.i.a.TransportPutLifecycleAction] [192.168.0.103] adding index lifecycle policy [watch-history-ilm-policy-16]
[2023-12-16T11:12:05,817][INFO ][o.e.x.i.a.TransportPutLifecycleAction] [192.168.0.103] adding index lifecycle policy [ilm-history-ilm-policy]
[2023-12-16T11:12:05,890][INFO ][o.e.x.i.a.TransportPutLifecycleAction] [192.168.0.103] adding index lifecycle policy [.deprecation-indexing-ilm-policy]
[2023-12-16T11:12:05,964][INFO ][o.e.x.i.a.TransportPutLifecycleAction] [192.168.0.103] adding index lifecycle policy [.fleet-actions-results-ilm-policy]
[2023-12-16T11:12:06,037][INFO ][o.e.x.i.a.TransportPutLifecycleAction] [192.168.0.103] adding index lifecycle policy [.fleet-file-tohost-data-ilm-policy]
[2023-12-16T11:12:06,113][INFO ][o.e.x.i.a.TransportPutLifecycleAction] [192.168.0.103] adding index lifecycle policy [.fleet-file-fromhost-data-ilm-policy]
[2023-12-16T11:12:06,189][INFO ][o.e.x.i.a.TransportPutLifecycleAction] [192.168.0.103] adding index lifecycle policy [.fleet-file-tohost-meta-ilm-policy]
[2023-12-16T11:12:06,262][INFO ][o.e.x.i.a.TransportPutLifecycleAction] [192.168.0.103] adding index lifecycle policy [.fleet-file-fromhost-meta-ilm-policy]
[2023-12-16T11:12:06,499][INFO ][o.e.h.n.s.HealthNodeTaskExecutor] [192.168.0.103] Node [{192.168.0.103}{cEOQD9KzSzWkKMhV8rhU4g}] is selected as the current health node.
[2023-12-16T11:12:06,643][INFO ][o.e.x.c.t.IndexTemplateRegistry] [192.168.0.103] adding ingest pipeline behavioral_analytics-events-final_pipeline
[2023-12-16T11:12:06,643][INFO ][o.e.x.c.t.IndexTemplateRegistry] [192.168.0.103] adding ingest pipeline ent-search-generic-ingestion
[2023-12-16T11:12:06,644][INFO ][o.e.x.c.t.IndexTemplateRegistry] [192.168.0.103] adding ingest pipeline logs@json-message
[2023-12-16T11:12:06,646][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding component template [behavioral_analytics-events-settings]
[2023-12-16T11:12:06,726][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding index template [behavioral_analytics-events-default] for index patterns [behavioral_analytics-events-*]
[2023-12-16T11:12:06,914][INFO ][o.e.l.ClusterStateLicenseService] [192.168.0.103] license [3f582a6d-94d0-4054-80c8-3e270ef7ab1c] mode [basic] - valid
[2023-12-16T11:12:06,915][INFO ][o.e.x.s.a.Realms         ] [192.168.0.103] license mode is [basic], currently licensed security realms are [reserved/reserved,file/default_file,native/default_native]
[2023-12-16T11:12:07,051][INFO ][o.e.x.c.t.IndexTemplateRegistry] [192.168.0.103] adding ingest pipeline logs-default-pipeline
[2023-12-16T11:12:07,053][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding component template [logs-settings]
[2023-12-16T11:12:07,132][INFO ][o.e.c.m.MetadataIndexTemplateService] [192.168.0.103] adding index template [logs] for index patterns [logs-*-*]
[2023-12-16T11:12:13,207][INFO ][o.e.x.s.InitialNodeSecurityAutoConfiguration] [192.168.0.103] HTTPS has been configured with automatically generated certificates, and the CA's hex-encoded SHA-256 fingerprint is [6c67f1239fd7d14e0a71d8100b4c7eec2e5d03635ec003484a87b5874bc179b0]
[2023-12-16T11:12:13,210][INFO ][o.e.x.s.s.SecurityIndexManager] [192.168.0.103] security index does not exist, creating [.security-7] with alias [.security]
[2023-12-16T11:12:13,223][INFO ][o.e.x.s.e.InternalEnrollmentTokenGenerator] [192.168.0.103] Will not generate node enrollment token because node is only bound on localhost for transport and cannot connect to nodes from other hosts
[2023-12-16T11:12:13,254][INFO ][o.e.c.m.MetadataCreateIndexService] [192.168.0.103] [.security-7] creating index, cause [api], templates [], shards [1]/[0]
[2023-12-16T11:12:13,285][INFO ][o.e.x.s.s.SecurityIndexManager] [192.168.0.103] security index does not exist, creating [.security-7] with alias [.security]
[2023-12-16T11:12:13,739][INFO ][o.e.i.m.MapperService    ] [192.168.0.103] [.security-7] reloading search analyzers
[2023-12-16T11:12:14,110][INFO ][o.e.c.r.a.AllocationService] [192.168.0.103] current.health="GREEN" message="Cluster health status changed from [YELLOW] to [GREEN] (reason: [shards started [[.security-7][0]]])." previous.health="YELLOW" reason="shards started [[.security-7][0]]"

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Elasticsearch security features have been automatically configured!
✅ Authentication is enabled and cluster connections are encrypted.

ℹ️  Password for the elastic user (reset with `bin/elasticsearch-reset-password -u elastic`):
  fhw8nsf=1-_e9lXzocJD

ℹ️  HTTP CA certificate SHA-256 fingerprint:
  6c67f1239fd7d14e0a71d8100b4c7eec2e5d03635ec003484a87b5874bc179b0

ℹ️  Configure Kibana to use this cluster:
• Run Kibana and click the configuration link in the terminal when Kibana starts.
• Copy the following enrollment token and paste it into Kibana in your browser (valid for the next 30 minutes):
  eyJ2ZXIiOiI4LjExLjMiLCJhZHIiOlsiMTkyLjE2OC4wLjEwMzo5MjAwIl0sImZnciI6IjZjNjdmMTIzOWZkN2QxNGUwYTcxZDgxMDBiNGM3ZWVjMmU1ZDAzNjM1ZWMwMDM0ODRhODdiNTg3NGJjMTc5YjAiLCJrZXkiOiJNUzJjY0l3QmlMRGt5UVV5TDBITTpNMVppanlQTVJWdXVQYnFEMGtUamJBIn0=

ℹ️  Configure other nodes to join this cluster:
• On this node:
  ⁃ Create an enrollment token with `bin/elasticsearch-create-enrollment-token -s node`.
  ⁃ Uncomment the transport.host setting at the end of config/elasticsearch.yml.
  ⁃ Restart Elasticsearch.
• On other nodes:
  ⁃ Start Elasticsearch with `bin/elasticsearch --enrollment-token <token>`, using the enrollment token that you generated.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
上面是启动日志

![avatar](/Users/zoubaitao/Blog/ES/ES_UP.png)，注意要使用英文符号


Kibana安装和使用
```bash
zbt@192 kibana-8.11.3 % bin/kibana
Kibana is currently running with legacy OpenSSL providers enabled! For details and instructions on how to disable see https://www.elastic.co/guide/en/kibana/8.11/production.html#openssl-legacy-provider
{"log.level":"info","@timestamp":"2023-12-16T03:23:24.350Z","log":{"logger":"elastic-apm-node"},"agentVersion":"4.1.0","env":{"pid":14329,"proctitle":"bin/../node/bin/node","os":"darwin 21.6.0","arch":"x64","host":"192.168.0.103","timezone":"UTC+0800","runtime":"Node.js v18.18.2"},"config":{"serviceName":{"source":"start","value":"kibana","commonName":"service_name"},"serviceVersion":{"source":"start","value":"8.11.3","commonName":"service_version"},"serverUrl":{"source":"start","value":"https://kibana-cloud-apm.apm.us-east-1.aws.found.io/","commonName":"server_url"},"logLevel":{"source":"default","value":"info","commonName":"log_level"},"active":{"source":"start","value":true},"contextPropagationOnly":{"source":"start","value":true},"environment":{"source":"start","value":"production"},"globalLabels":{"source":"start","value":[["git_rev","cc11667953f4734af414e8d8977b8d9dda5698ef"]],"sourceValue":{"git_rev":"cc11667953f4734af414e8d8977b8d9dda5698ef"}},"secretToken":{"source":"start","value":"[REDACTED]","commonName":"secret_token"},"breakdownMetrics":{"source":"start","value":false},"captureSpanStackTraces":{"source":"start","sourceValue":false},"centralConfig":{"source":"start","value":false},"metricsInterval":{"source":"start","value":120,"sourceValue":"120s"},"propagateTracestate":{"source":"start","value":true},"transactionSampleRate":{"source":"start","value":0.1,"commonName":"transaction_sample_rate"},"captureBody":{"source":"start","value":"off","commonName":"capture_body"},"captureHeaders":{"source":"start","value":false}},"activationMethod":"require","ecs":{"version":"1.6.0"},"message":"Elastic APM Node.js Agent v4.1.0"}
[2023-12-16T11:23:26.348+08:00][INFO ][root] Kibana is starting
[2023-12-16T11:23:26.387+08:00][INFO ][node] Kibana process configured with roles: [background_tasks, ui]
[2023-12-16T11:23:44.453+08:00][INFO ][plugins-service] Plugin "cloudChat" is disabled.
[2023-12-16T11:23:44.460+08:00][INFO ][plugins-service] Plugin "cloudExperiments" is disabled.
[2023-12-16T11:23:44.460+08:00][INFO ][plugins-service] Plugin "cloudFullStory" is disabled.
[2023-12-16T11:23:44.460+08:00][INFO ][plugins-service] Plugin "cloudGainsight" is disabled.
[2023-12-16T11:23:44.643+08:00][INFO ][plugins-service] Plugin "profilingDataAccess" is disabled.
[2023-12-16T11:23:44.644+08:00][INFO ][plugins-service] Plugin "profiling" is disabled.
[2023-12-16T11:23:44.708+08:00][INFO ][plugins-service] Plugin "securitySolutionServerless" is disabled.
[2023-12-16T11:23:44.709+08:00][INFO ][plugins-service] Plugin "serverless" is disabled.
[2023-12-16T11:23:44.709+08:00][INFO ][plugins-service] Plugin "serverlessObservability" is disabled.
[2023-12-16T11:23:44.709+08:00][INFO ][plugins-service] Plugin "serverlessSearch" is disabled.
[2023-12-16T11:23:44.937+08:00][INFO ][http.server.Preboot] http server running at http://localhost:5601
[2023-12-16T11:23:45.157+08:00][INFO ][plugins-system.preboot] Setting up [1] plugins: [interactiveSetup]
[2023-12-16T11:23:45.160+08:00][INFO ][preboot] "interactiveSetup" plugin is holding setup: Validating Elasticsearch connection configuration…
[2023-12-16T11:23:45.231+08:00][INFO ][root] Holding setup until preboot stage is completed.


i Kibana has not been configured.

Go to http://localhost:5601/?code=638797 to get started.
```