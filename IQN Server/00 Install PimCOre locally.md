 
```sh
composer create-project pimcore/skeleton my-pimcore
cd my-pimcore
mysql -u root -p -e "CREATE DATABASE pimcore charset=utf8mb4;
./vendor/bin/pimcore-install
```
