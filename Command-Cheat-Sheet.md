# Command Cheat Sheet

## Create Model with Seed & Migration

`php artisan make:model ModelName --seed --migration`

## Create Controller 
`php artisan make:controller NameController`

## Create Provider
`php artisan make:provider NameProvider`

## Run Migrate

`php artisan migrate`

## Drop all tables, runs all migrations, and seeds the database.

`php artisan migrate:fresh --seed`

## Clear the cache
```bash
php artisan cache:clear
php artisan config:clear
```
