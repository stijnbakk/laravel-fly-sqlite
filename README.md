# Laravel Fly deploy with sqlite database

## 1. Setup fly

```
fly launch
# follow the instructions
```

## 2. Create a volume for the sqlite database

Create volume on fly.io

```
flyctl volumes create sqlite_data --region ams --size 1
```

Update `fly.toml`

```toml
# fly.toml
[[mounts]]
  source = "sqlite_data"
  destination = "/var/www/storage/"
# ...
[env]
  DB_CONNECTION = 'sqlite'
  DB_DATABASE = '/var/www/html/storage/database/database.sqlite'
```

Add initialisation script to create database on first deploy.

```sh
# .fly/scripts/1_storage_init.sh

# Add this below the storage folder initialization snippet
FOLDER=/var/www/html/storage/database
if [ ! -d "$FOLDER" ]; then
    echo "$FOLDER is not a directory, initializing database"
    mkdir /var/www/html/storage/database
    touch /var/www/html/storage/database/database.sqlite
fi
```

## 3. Deploy

Use fly cli to deploy

```bash
fly deploy
```

SSH into the machine to perform migrations and seeding

```bash
flyctl ssh console

ls
# Dockerfile  README.md  app  artisan  bootstrap  composer.json  composer.lock  config  database  index.nginx-debian.html  package-lock.json  package.json  phpunit.xml  postcss.config.js  public  resources  routes  storage  tailwind.config.js  tests  vendor  vite.config.js
```

```bash
php artisan migrate
```

```bash
php artisan db:seed
```
