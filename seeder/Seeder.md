# Seeder

## Create Seeder

`php artisan make:seeder UserSeeder`

## Register the Seeder (Optional)

To run your seeder along with `php artisan db:seed`, you need to register it in the `DatabaseSeeder.php` file:

```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    /**
     * Seed the application's database.
     *
     * @return void
     */
    public function run()
    {
        $this->call([
            UserSeeder::class,
            // Add more seeders here
        ]);
    }
}
```

## Run Seeder

### Run a specific seeder:

`php artisan db:seed --class=UserSeeder`

### Run all seeders registered in DatabaseSeeder:

`php artisan db:seed`
