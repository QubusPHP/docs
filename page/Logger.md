# Basic Usage

```php
require('vendor/autoload.php');

use League\Flysystem\Filesystem;
use League\Flysystem\Local\LocalFilesystemAdapter;
use Psr\Log\LogLevel;
use Qubus\Config\Collection;
use Qubus\Log\Logger;
use Qubus\Log\Loggers\FileLogger;
use Qubus\Log\Loggers\SwiftMailerLogger;
use Qubus\Mail\Mailer;

try {
    $storage = new SplObjectStorage();

    $adapter = new LocalFilesystemAdapter('storage/logs/');
    $filesystem = new Filesystem($adapter);

    $storage->attach(new FileLogger($filesystem, Psr\Log\LogLevel::INFO));

    $config = Collection::factory([
        'path' => __DIR__ . '/config'
    ]);
    $mail = (new Mailer())->factory('smtp', $config);

    $storage->attach(new SwiftMailerLogger($mail, LogLevel::INFO, [
        'from' => 'email@bob.com',
        'to' => 'logs@gmail.com',
        'subject' => '[System Error] Logger',
    ]));

    $logger = new Logger($storage);

    $logger->info('Info message.');
    $logger->alert('Alert message.');
    $logger->error('Error message.');
    $logger->debug('Debug message.');
    $logger->notice('Notice message.');
    $logger->warning('Warning message.');
    $logger->critical('Critical message.');
    $logger->emergency('Emergency message.');
} catch (Throwable $e) {
    echo $e->getMessage();
}
```