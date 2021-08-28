# Connection

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

# QueryBuilder

## Select Query


```php
// SELECT * FROM users;
$users = $db->select()->from('users')->execute();

foreach($users as $user) {
    var_dump($user);
}
```

### Compile
If you want to see the SQL statement that is generated, you can use the `compile()` method:

```php
$users = $db->select()->from('users')->where('id', 1);

// SELECT * FROM users WHERE id = '1';
echo $users->compile();
```

### Selecting A Table
```php
$select = $db->select();

// SELECT * FROM users;
$select->from('users');

// SELECT * FROM users AS u;
$select->from(['users', 'u']);
```

### Selecting Columns
```php
use Qubus\Dbal\DB;

// SELECT id, first_name, last_name, email FROM users;
$db->select('id', 'first_name', 'last_name', 'email')->from('users');

// SELECT first_name AS f_name FROM users;
$db->select(['first_name', 'f_name'])->from('users');

// SELECT first_name AS f_name, email FROM users;
$db->select(['first_name', 'f_name'], 'email')->from('users');

// SELECT COUNT(*) FROM users;
$db->select(DB::fnc('count', '*'))->from('users');

// SELECT COUNT(*) AS num FROM users;
$db->select(DB::fnc('count', '*'))->aliasTo('num')->from('users');

// SELECT COUNT(*) AS num_users FROM users;
$db->select([DB::fnc('count', '*'), 'num_users'])->from('users');
```

### Where Condition
```php
$select = $db->select()->from('users');

// SELECT * FROM users WHERE id = '1';
$select->where('id', 1);

// SELECT * FROM users WHERE first_name IS NULL;
$select->where('first_name', null);

// SELECT * FROM users WHERE first_name IS NOT NULL;
$select->where('first_name', '!=', null);

// SELECT * FROM users WHERE first_name = 'John' OR email IS NOT NULL;
$select->where('first_name', 'John')->orWhere('email', '!=', null);

// SELECT * FROM users WHERE first_name = 'John' AND email IS NOT NULL;
$select->where('first_name', 'John')->andWhere('email', '!=', null);

// SELECT * FROM users WHERE id IN [1, 3];
$select->where('id', 'in', [1, 3]);

// SELECT * FROM users WHERE id NOT IN [4, 6];
$select->where('id', 'not in', [4, 6]);

// SELECT * FROM users WHERE first_name LIKE 'Jo%';
$select->where('first_name', 'like', 'Jo%');
```

### Joining Tables
```php
$select = $db->select()->from('users');

// SELECT * FROM users JOIN address ON ('users.id' = 'address.user_id');
$select->join('address')->on('users.id', '=', 'address.user_id');

// SELECT * FROM users JOIN address ON ('users.id' = 'address.user_id' AND 'users.email' = 'address.email');
$select->join('address')->on('users.id', '=', 'address.user_id')->andOn('users.email', 'address.email');

// SELECT * FROM users JOIN address ON ('users.id' = 'address.user_id' OR 'users.email' = 'address.email');
$select->join('address')->on('users.id', '=', 'address.user_id')->orOn('users.email', 'address.email');
```

### Ordering and Grouping
```php
$select = $db->select()->from('users');

// SELECT * FROM users ORDER BY 'id';
$select->orderBy('id');

// SELECT * FROM users ORDER BY 'id' DESC;
$select->orderBy('id', 'desc');

// SELECT * FROM users GROUP BY 'id';
$select->groupBy('id');
```

### Limit and Offset
```php
$select = $db->select()->from('users');

// SELECT * FROM users LIMIT 10;
$select->limit(10);

// SELECT * FROM users LIMIT 10 OFFSET 50;
$select->limit(10)->offset(50);
```

### Having Condition
```php
$select = $db->select('users');

// SELECT * FROM users HAVING id = '3';
$select->having('id', 3);

// SELECT * FROM users HAVING id = '3' AND NOT last_name = 'Johnson';
$select->having('id', 3)->andNotHaving('last_name', 'Johnson');

// SELECT * FROM users HAVING email IS NOT NULL;
$select->having('email', '!=', null);
```

## Insert Query
```php
// INSERT INTO users (first_name, last_name, email) VALUES ('John', 'Smith', 'john_smith@gmail.com');
$insert = $db->insert('users')
        ->values([
            'first_name' => 'John',
            'last_name'  => 'Smith',
            'email'      => 'john_smith@gmail.com',
        ])
        ->execute();
// print the last inserted id
echo $insert->insertIdField();

// INSERT INTO users (first_name, last_name, email, created_at) VALUES ('John', 'Smith', 'john_smith@gmail.com', NOW());
$insert = $db->insert('users')
        ->values([
            'first_name' => 'John',
            'last_name'  => 'Smith',
            'email'      => 'john_smith@gmail.com',
            'created_at' => $db->fnc('now');
        ])
        ->execute();
```

## Update Query
```php
// UPDATE users SET email = 'j_smith@gmail.com' WHERE id = '1';
$update = $db->update('users')
        ->set([
            'email' => 'j_smith@gmail.com',
        ])
        ->where('id', 1)
        ->execute();
```

## Delete Query
```php
// DELETE FROM users WHERE id = '15'
$delete = $db->delete('users')
        ->where('id', 15)
        ->execute()
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