Deployment
==========

## MongoDB

You should enable WiredTiger storage engine. [Upgrade MongoDB to 3.0](http://docs.mongodb.org/manual/release-notes/3.0-upgrade/)

### Mac OS X

Recommmended configuration file `/usr/local/etc/mongod.conf` for Homebrew:

```yaml
storage:
  dbPath: /usr/local/var/mongodb
  directoryPerDB: true
  engine: wiredTiger
  journal:
    enabled: true

systemLog:
  destination: file
  path: /usr/local/var/log/mongodb/mongo.log
  logAppend: true

net:
  bindIp: 127.0.0.1
```

## Elastic search

### Mac OS X

1. Install elastic search (1.4.0+)

   ```bash
   brew install elasticsearch
   ```

   How to install older version: http://effectif.com/mac-os-x/installing-specific-version-of-homebrew-formula

   Elastic search book: http://es.xiaoleilu.com/010_Intro/00_README.html

2. Install maven:

   ```bash
   brew install maven
   ```

3. Build ik plugin with maven

   ```bash
   cd /tmp
   wget https://github.com/medcl/elasticsearch-analysis-ik/archive/master.zip
   unzip master.zip
   cd elasticsearch-analysis-ik-master
   mvn package
   ```

4. Copy files & enable plugin index

   ```bash
   cd /tmp/elasticsearch-analysis-ik-master
   mkdir /usr/local/var/lib/elasticsearch/plugins/analysis-ik
   cp target/releases/elasticsearch-analysis-ik-1.2.9.zip /usr/local/var/lib/elasticsearch/plugins/analysis-ik/
   cp -R config/ik /usr/local/opt/elasticsearch/config/ # 复制字典
   cd /usr/local/var/lib/elasticsearch/plugins/analysis-ik
   unzip elasticsearch-analysis-ik-1.2.9.zip
   rm elasticsearch-analysis-ik-1.2.9.zip
   ```

   Append `/usr/local/opt/elasticsearch/config/elasticsearch.yml`:

   ```yml
   index:
     analysis:
       analyzer:
         ik:
             alias: [ik_analyzer]
             type: org.elasticsearch.index.analysis.IkAnalyzerProvider
         ik_max_word:
             type: ik
             use_smart: false
         ik_smart:
             type: ik
             use_smart: true
   index.analysis.analyzer.default.type: ik
   ```

5. reload elastic search

   ```bash
   launchctl unload ~/Library/LaunchAgents/homebrew.mxcl.elasticsearch.plist
   launchctl load ~/Library/LaunchAgents/homebrew.mxcl.elasticsearch.plist
   ```

6. test

   ```bash
   # 创建索引
   curl -XPUT http://localhost:9200/index

   # 创建 mapping
   curl -XPOST http://localhost:9200/index/fulltext/_mapping -d'
   {
       "fulltext": {
           "_all": {
               "analyzer": "ik"
           },
           "properties": {
               "content": {
                   "type" : "string",
                   "boost" : 8.0,
                   "term_vector" : "with_positions_offsets",
                   "analyzer" : "ik",
                   "include_in_all" : true
               }
           }
       }
   }'

   # 测试分词
   curl 'http://localhost:9200/index/_analyze?analyzer=ik&pretty=true' -d '
   {
   "text":"世界你好"
   }'
   ```

## PHP Backend

1. Configure and install PHP 5.6+ if your system doesn't have one.

   Mac:

   ```
   brew install curl --with-openssl
   brew install php56 --with-homebrew-curl
   brew install php56-mongo php56-redis
   ```

   Linux:
   ```
   tar xf php.5.6.x.tar.xz
   cd php-5.6.x
   ./configure --enable-fpm --enable-mbstring --with-openssl --with-curl
   make -j8
   sudo make install -j8
   sudo pecl install redis mongo
   ```
   
   Load the extensions in php.ini
   
   ```
   extension=redis.so
   extension=mongo.so
   ```

2. Install composer & components.
   
   ```sh
   composer install
   ```

3. Install front-end components.

   ```sh
   cd web
   npm install
   bower install
   gulp shared:ext
   gulp shared:lib
   gulp shared:ui
   gulp shared:core
   gulp page:scripts
   ```

4. Set default timezone in `php.ini`.

   ```ini
   [Date]
   date.timezone = Asia/Shanghai
   ```

5. Grant write permission

   ```sh
   chmod -R 777 app/cache
   chmod -R 777 app/logs
   ```

Before each production deployment:

1. clear route cache in `app/cache/route.cache`
