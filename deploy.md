Deployment
==========

## Backend

0. Configure and install PHP 5.6+ if your system doesn't have one.

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

1. Install composer & components.
   
   ```sh
   composer install
   ```

2. Set default timezone in `php.ini`.

   ```ini
   [Date]
   date.timezone = Asia/Shanghai
   ```

3. serialize handler in `php.ini` must be php.

4. Grant write permission

   ```sh
   chmod -R 777 app/cache
   chmod -R 777 app/logs
   ```

Before each production deployment:

1. clear route cache in `app/cache/route.cache`
