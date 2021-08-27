# Usage

Use the factory to instanciate a Config collection class:

```php
use Qubus\Config\Collection;

$config = Collection::factory([
    'path' => __DIR__ . '/config'
]);
```

Optionally, you can also setup the environment. Setting up the environment will merge normal configurations 
with configurations in the environment directory. For example, if you setup the environment to be *prod*, 
the configurations from the directory ``config/prod/*`` will be loaded on top of the configurations from the 
directory ``config/*``. Consider the following example:

```php
use Qubus\Config\Collection;

$config = Collection::factory([
    'path' => __DIR__ . '/config',
    'environment' => 'prod'
]);
```

Optionally, you can also use dotenv to hide sensible information into a `.env` file. To do so, specify a directory
where the `.env` file. Like in this example:

```php
use Qubus\Config\Collection;

$config = Collection::factory([
    'path' => __DIR__ . '/config',
    'dotenv' => __DIR__,
    'environment' => 'prod'
]);
```

You can than use the configurations like this:

```php
$config->getConfigKey('app.timezone');
```
__Note:__ In the above example, `app` is the name of the file minus the `.php` extension, and `timezone` is a key found in `app.php`.

## Getter

The configuration getter uses a simple syntax: ``file_name.array_key``.

For example:

```php
$config->getConfigKey('app.timezone');
```

You can optionally set a default value like this:

```php
$config->getConfigKey('app.timezone', "America/New_York");
```

You can use the getter to access multidimensional arrays in your configurations:

```php
$config->getConfigKey('database.connections.default.host');
```

## Setter

Alternatively, you can set configurations from your application code:

```php
$config->setConfigKey('app.timezone', "Europe/Berlin");
```

You can set entire arrays of configurations:

```php
$config->setConfigKey('database', [
    'host' => "localhost",
    'dbname' => "my_database",
    'user' => "my_user",
    'password' => "my_password"
]);
```

## Invokable Classes
Sometimes in your own projects you may want to use config classes for storing application settings, without needing file I/O. You can do this by creating invokable classes:

```php
use Qubus\Config\Parser;

class SimpleConfig
{
    public function __invoke()
    {
        return [
            'database' => [
                'host' => 'localhost',
                'port'    => 443,
            ],
            'application' => [
                'name'   => 'configuration',
                'secret' => 's3cr3t',
            ],
        ];
    }
}

$config = new SimpleConfig();

[$file, $key, $sub] = Parser::getKey('file.application.secret');

echo Parser::getValue($config(), $key, $sub); // will print out `s3cr3t`
```

## Decorator
It is also possible to replace configuration variables by using the `VariableDecorator`. This allows you to define variables at runtime without changing configuration:

```php
use Qubus\Config\Collection;
use Qubus\Config\VariableDecorator;

// app.php contains the following:
# return [
#     'timezone' => 'America/Denver',
#     'test_dir' => '%vendorDir%/testdev1',
# ];

$config = Collection::factory([
    'path' =>  __DIR__ . "/files",
    'environment' => 'production',
    'dotenv' => __DIR__ . "/files"
]);

$decorator = new VariableDecorator($config);
$decorator->setVariables(['%vendorDir%' => __DIR__ . "/files"]);

echo $decorator->getConfigKey('app.test_dir'); // will print out `/some/directory/to/files/testdev1`
```

## PHP Configuration File

Example of a PHP configuration file:

```php
return [
    'timezone' => "America/New_York"
];
```

## Yaml Configuration File

Example of a YAML configuration file:

```yaml
timezone: America/New_York
```

## Dotenv

Example of using Dotenv in a PHP configuration file:

```php
use function Qubus\Config\Helpers\env;

return [
    'timezone' => env('TIMEZONE', "Denver")
];
```

And in the `.env` file:

```
TIMEZONE="America/New_York"
```