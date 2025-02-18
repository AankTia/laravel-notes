# Migration

## Createing Model with Migration & Seeder

`php artisan make:model Todo --seed --migration`

## Migration Example

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateProductsTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('products', function (Blueprint $table) {
            // Primary key
            $table->id(); // Creates an auto-incrementing UNSIGNED BIGINT

            // Basic columns
            $table->string('name', 100);
            $table->string('slug')->unique();
            $table->text('description')->nullable();
            $table->decimal('price', 10, 2); // 10 digits, 2 decimal places
            $table->integer('stock')->default(0);
            $table->boolean('is_active')->default(true);

            // Various column types
            $table->unsignedBigInteger('category_id'); // For foreign key
            $table->enum('status', ['draft', 'published', 'archived'])->default('draft');
            $table->date('launch_date')->nullable();
            $table->dateTime('last_ordered_at')->nullable();
            $table->json('properties')->nullable();
            $table->timestamp('approved_at')->nullable();

            // Soft deletes
            $table->softDeletes();

            // Timestamps (created_at, updated_at)
            $table->timestamps();

            // Foreign key constraints
            $table->foreign('category_id')
                  ->references('id')
                  ->on('categories')
                  ->onDelete('cascade');

            // Indexes
            $table->index(['name', 'price']);
            $table->fullText('description');
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('products');
    }
}
```

### Creating a Pivot Table for Many-to-Many Relationship

```php
Schema::create('product_tag', function (Blueprint $table) {
    $table->id();
    $table->unsignedBigInteger('product_id');
    $table->unsignedBigInteger('tag_id');
    $table->timestamps();

    $table->unique(['product_id', 'tag_id']);

    $table->foreign('product_id')->references('id')->on('products')->onDelete('cascade');
    $table->foreign('tag_id')->references('id')->on('tags')->onDelete('cascade');
});
```

### Modifying an Existing Table

```php
Schema::table('users', function (Blueprint $table) {
    // Add new columns
    $table->string('middle_name')->nullable()->after('name');
    $table->boolean('is_admin')->default(false)->after('email');

    // Modify existing columns
    $table->string('name', 150)->change();

    // Rename columns
    $table->renameColumn('email', 'email_address');

    // Drop columns
    $table->dropColumn(['deleted_at', 'is_active']);

    // Drop foreign keys and indexes
    $table->dropForeign(['user_id']);
    $table->dropIndex(['email']);
});
```

### Creating a Table with UUID Primary Key

```php
Schema::create('orders', function (Blueprint $table) {
    $table->uuid('id')->primary();
    $table->string('order_number')->unique();
    $table->uuid('customer_id');
    $table->decimal('total', 12, 2);
    $table->enum('status', ['pending', 'processing', 'completed', 'cancelled']);
    $table->timestamps();

    $table->foreign('customer_id')
          ->references('id')
          ->on('customers')
          ->onDelete('restrict');
});
```

## Run Migration

`php artisan migrate`

If got errors:

```
UnexpectedValueException
The stream or file "/Applications/XAMPP/xamppfiles/htdocs/sinoppal-laravel/storage/logs/laravel.log" could not be opened in append mode: Failed to open stream: Permission denied
```

need grand permissions: `sudo chmod -R 777 /storage`

### Fresh Migration (Drop All Tables and Recreate)

`php artisan migrate:fresh`

This command drops all tables from your database and then runs all your migrations from scratch. **Be careful as this will delete all your data!**

### Refresh Migration (Rollback All and Migrate Again)

`php artisan migrate:refresh`

This command will roll back all of your migrations and then run the migrate command again. **This effectively rebuilds the database.**

### Reset and Re-Migrate

```
php artisan migrate:reset
php artisan migrate
```

Reset rolls back all migrations, and then the migrate command runs them again.

### Rollback All and Migrate with Seed Data

`php artisan migrate:refresh --seed`

This refreshes the migrations and also seeds the database with your seed data.

### Fresh Migration with Seed Data

`php artisan migrate:fresh --seed`

This drops all tables, runs all migrations, and seeds the database.
