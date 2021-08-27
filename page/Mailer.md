# Usage

## Configuration

Below are the contents of `config/mailer.php`:

```php
return [
    /*
    |--------------------------------------------------------------------------
    | SMTP Swift Mailer Transport
    |--------------------------------------------------------------------------
    */
    'smtp' => [
        'host' => env('SMTP_HOST'),
        'port' => 465,
        'username' => env('SMTP_USERNAME'),
        'password' => env('SMTP_PASSWORD'),
        'authmode' => 'login',
        'encryption' => 'ssl',
    ],

    /*
    |--------------------------------------------------------------------------
    | Sendmail Swift Mailer Transport
    |--------------------------------------------------------------------------
    */
    'sendmail' => [
        'command' => '/usr/sbin/sendmail -bs',
    ]
];
```

## Instantiate Class

```php
require('vendor/autoload.php');

use Qubus\Config\Collection;
use Qubus\Mail\Mailer;

$config = Collection::factory([
    'path' => __DIR__ . '/config'
]);

$mail = (new Mailer())->factory('smtp', $config);
```

## Sending an Email

```php
$mail->send(function ($message) {
    $message->to('myfriend@gmail.com');
    $message->from('roger@gmail.com', 'Roger Smith');
    $message->subject('Test Message');
    $message->body('This is a regular plain text message.');
    $message->charset('utf-8');
    $message->html(false);
});
```

## Sending an HTML Email

```php
$mail->send(function ($message) {
    $message->to('myfriend@gmail.com');
    $message->from('roger@gmail.com', 'Roger Smith');
    $message->subject('Test Message');
    $message->body('This is an <strong>html</strong> message.');
    $message->charset('utf-8');
    $message->html(true);
});
```

## Using Email Templates
You can send an email using an email template and pass in variables that can be replaced.

```php
$mail->send(function ($message) {
    $message->to('myfriend@gmail.com');
    $message->from('roger@gmail.com', 'Roger Smith');
    $message->subject('Test Message');
    $message->templatePath(__DIR__);
    $message->body(['MESSAGE' => 'This is an <strong>html</strong> message.'], [
        'template_name' => 'email.html',
    ]);
    $message->charset('utf-8');
    $message->html(true);
});
```

## Sending Attachment

```php
$mail->send(function ($message) {
    $message->to('myfriend@gmail.com');
    $message->from('roger@gmail.com', 'Roger Smith');
    $message->subject('Test Message');
    $message->body('This is a regular plain text message.');
    $message->charset('utf-8');
    $message->html(false);
    $message->attach('/path/to/file.pdf');
});
```