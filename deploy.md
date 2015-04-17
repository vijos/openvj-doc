Deployment
==========

## MongoDB

OpenVJ requires MongoDB 3.0 or above and you should enable the WiredTiger storage engine.

See [Upgrade MongoDB to 3.0](http://docs.mongodb.org/manual/release-notes/3.0-upgrade/) for upgrading steps.

### OS X

Recommmended configuration file `/usr/local/etc/mongod.conf` for Homebrew:

```yaml
storage:
  dbPath: /usr/local/var/mongodb   # for other OS, modify this line
  directoryPerDB: true
  engine: wiredTiger
  journal:
    enabled: true

systemLog:
  destination: file
  path: /usr/local/var/log/mongodb/mongo.log  # for other OS, modify this line
  logAppend: true

net:
  bindIp: 127.0.0.1
```

## Elastic Search

OpenVJ uses Elastic Search to provide searching feature.

### OS X

1. Install elastic search (1.4.0+)

   ```sh
   brew install elasticsearch
   ```

   How to install older version: http://effectif.com/mac-os-x/installing-specific-version-of-homebrew-formula

   Elastic Search book: http://es.xiaoleilu.com/010_Intro/00_README.html

2. Install maven:

   ```sh
   brew install maven
   ```

3. Build ik plugin with maven

   ```sh
   cd /tmp
   wget https://github.com/medcl/elasticsearch-analysis-ik/archive/master.zip
   unzip master.zip
   cd elasticsearch-analysis-ik-master
   mvn package
   ```

4. Copy files & enable plugin index

   ```sh
   cd /tmp/elasticsearch-analysis-ik-master
   mkdir /usr/local/var/lib/elasticsearch/plugins/analysis-ik
   cp target/releases/elasticsearch-analysis-ik-1.2.9.zip /usr/local/var/lib/elasticsearch/plugins/analysis-ik/
   cp -R config/ik /usr/local/opt/elasticsearch/config/ # 复制字典
   cd /usr/local/var/lib/elasticsearch/plugins/analysis-ik
   unzip elasticsearch-analysis-ik-1.2.9.zip
   rm elasticsearch-analysis-ik-1.2.9.zip
   ```

   Append `/usr/local/opt/elasticsearch/config/elasticsearch.yml`:

   ```yaml
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

   ```sh
   launchctl unload ~/Library/LaunchAgents/homebrew.mxcl.elasticsearch.plist
   launchctl load ~/Library/LaunchAgents/homebrew.mxcl.elasticsearch.plist
   ```

6. test

   ```sh
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
### Ubuntu

1. Install elastic search (1.4.0+)

   ```sh
   wget -qO - https://packages.elasticsearch.org/GPG-KEY-elasticsearch | sudo apt-key add -
   echo "deb http://packages.elasticsearch.org/elasticsearch/1.5/debian stable main" | sudo tee -a /etc/apt/sources.list
   sudo apt-get update && sudo apt-get install elasticsearch
   sudo update-rc.d elasticsearch defaults 95 10
   ```

   How to install older version: http://effectif.com/mac-os-x/installing-specific-version-of-homebrew-formula

   Elastic Search book: http://es.xiaoleilu.com/010_Intro/00_README.html

2. Install maven:

   ```sh
   sudo apt-get install maven
   ```

3. Build ik plugin with maven

   ```sh
   cd /tmp
   wget https://github.com/medcl/elasticsearch-analysis-ik/archive/master.zip
   unzip master.zip
   cd elasticsearch-analysis-ik-master
   mvn package
   ```

4. Copy files & enable plugin index

   ```sh
   cd /tmp/elasticsearch-analysis-ik-master
   sudo mkdir /usr/share/elasticsearch/plugins/analysis-ik
   sudo cp target/releases/elasticsearch-analysis-ik-1.2.9.zip /usr/share/elasticsearch/plugins/analysis-ik/
   sudo cp -R config/ik /etc/elasticsearch/ # 复制字典
   cd /usr/share/elasticsearch/plugins/analysis-ik
   unzip elasticsearch-analysis-ik-1.2.9.zip
   rm elasticsearch-analysis-ik-1.2.9.zip
   ```

   Append `/etc/elasticsearch/elasticsearch.yml`:

   ```yaml
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

   ```sh
   sudo service elasticsearch restart
   ```

6. test

   ```sh
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

   OS X (Homebrew):
   
   ```sh
   brew install curl --with-openssl
   brew install php56 --with-homebrew-curl
   brew install php56-mongo php56-redis
   ```

   Linux (install from source):
   
   ```sh
   tar xf php.5.6.x.tar.xz
   cd php-5.6.x
   ./configure --enable-fpm --enable-mbstring --with-openssl --with-curl
   make -j8
   sudo make install -j8
   sudo pecl install redis mongo
   ```

  Ubuntu

  ```sh
  sudo apt-get update && sudo apt-get install python-software-properties
  sudo add-apt-repository ppa:ondrej/php5-5.6
  sudo apt-get install php5
  sudo pecl install redis mongo
  ```
   
   Be sure that you load the extensions in `php.ini`
   
   ```ini
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

## Server Configuration

### Apache

1. Enable `mod_rewrite`:

   ```apache
   LoadModule rewrite_module libexec/apache2/mod_rewrite.so
   ```

   Ubuntu
   ```sh
   sudo a2enmod rewrite
   sudo service apache2 restart
   ```

2. Sample configuration for HTTP server running on `80` port:

   ```apache
   <VirtualHost *:80>
       DocumentRoot /home/to/project/openvj/web
       ServerName openvj.org
       ServerAlias www.openvj.org static.openvj.org

       <Directory "/home/to/project/openvj/web">
           <IfModule mod_rewrite.c>
               RewriteEngine On
               RewriteCond %{REQUEST_FILENAME} !-f
               RewriteRule ^(.*)$ app.php [L]
           </IfModule>
       </Directory>
   </VirtualHost>
   ```

### Nginx

Sample configuration for HTTP server running on `80` port:

```nginx
server {
    listen 80 default;
    server_name openvj.org www.openvj.org static.openvj.org;

    root /home/to/project/openvj/web;
    
    location / {
        if (!-e $request_filename) {
            rewrite . /app.php last;
        }
    }

    error_page 404 /;
	
    location ~ \.php$ {
        fastcgi_pass  localhost:9000;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include       fastcgi_params;
    }

    location ~ ^(.*)(/)(\..+)(.*)$ {
        deny all;
    }
}
```

### Performance Note

1. use unix socket for php-fpm:
   
   In nginx:
   
   ```nginx
   # With php5-cgi alone:
   fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock;
   # With php5-fpm: fastcgi_pass
   # unix:/var/run/php5-fpm.sock;
   ```
   
   In php-fpm:
   
   ```ini
   listen = /var/run/php-fpm/php-fpm.sock
   ```

2. enable gzip for all responses:
   
   In nginx:
   
   ```nginx
   gzip on;
   gzip_buffers 16 64k;
   gzip_http_version 1.1;
   gzip_comp_level 9;
   gzip_types text/plain application/x-javascript text/css application/json application/xml text/javascript;
   ```

## Memorandum

Before each production deployment:

1. clear route cache in `app/cache/route.cache`
