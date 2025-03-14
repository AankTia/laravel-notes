# Command Cheat Sheet

## Create Model with Seed & Migration

`php artisan make:model ModelName --seed --migration`

## Create Controller 
`php artisan make:controller NameController`

Replace "Name" with the name you want to give your controller. 

There are several options you can use with this command:

1. Create a resource controller (with CRUD methods):
```bash
php artisan make:controller UserController --resource
```

2. Create an API resource controller:
```bash
php artisan make:controller UserController --api
```

3. Create an invokable controller (single action):
```bash
php artisan make:controller UserController --invokable
```

4. Create a controller with model binding:
```bash
php artisan make:controller UserController --model=User
```

5. Create a resourceful controller with model binding:
```bash
php artisan make:controller UserController --resource --model=User
```

The controller will be created in the `app/Http/Controllers` directory.

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
