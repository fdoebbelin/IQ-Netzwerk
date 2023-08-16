## Follow these steps
Initialize the skeleton or demo project using the pimcore/pimcore image


```
pimcore/skeleton
pimcore/demo
```

```sh
docker volume create pimcore_demo_php
docker run --rm -v pimcore_demo_php:/var/www/html pimcore/pimcore:php8.1-latest composer create-project pimcore/demo .
```

Create volume container and copy content

```sh
docker create --name dvc --volume pimcore_demo_php:/volume busybox
docker cp dvc:/volume/.docker .
```

Go to your new project

```sh
cd pimcore_demo/
```

Start the needed services with

```sh
docker compose up -d
```

Install pimcore and initialize the DB

```sh
docker compose exec php vendor/bin/pimcore-install
docker compose exec php chown -R www-data:www-data /var/www/html/var
docker compose exec php chown -R www-data:www-data public
```

When asked for admin user and password:

The frontend:

```
http://localhost
```


The admin interface, using the credentials you have chosen above:

```
http://localhost/admin
```
