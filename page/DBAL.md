# Usage

```php
use Qubus\Dbal\DB;

$config = [
    'driver' => 'mysql',
    'username' => 'root',
    'password' => 'root',
    'dbname' => 'test',
];

$db = DB::connection($config);
```

## QueryBuilder

```php
// SELECT * FROM users WHERE id = 1
$db->select()->from('users')->where('id', 1)->execute();

// Equivalent raw query
$db->query('SELECT * FROM users')->first();
```

## Schema Builder
```php
use Qubus\Dbal\Schema\CreateTable;

$db->schema()->create('users', function (CreateTable $table) {
    $table->integer('id')->size('big')->primary();
    $table->string('email')->unique();
    $table->string('username')->unique();
    $table->string('first_name');
    $table->string('last_name');
});
```