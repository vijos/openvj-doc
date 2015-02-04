Deployment
==========

## Backend

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