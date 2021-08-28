## Config-Driven Qubus Injector

* Injector configuration can be done through a Config file.
* Aliases are case-sensitive.
* Closures can receive an `InjectionChain` object that let you iterate over the instantiation hierarchy.

## Table Of Contents

* [Basic Usage](#basic-usage)
    * [Standard Aliases](#standard-aliases)
    * [Shared Aliases](#shared-aliases)
    * [Argument Definitions](#argument-definitions)
    * [Argument Providers](#argument-providers)
    * [Delegations](#delegations)
    * [Preparations](#preparations)
* [Registering Additional Mappings](#registering-additional-mappings)

## Basic Usage

> This documentation only deals with passing in mappings through a `Config` file. Documentation for the basic methods still needs to be synced. For this, refer to the [original docs](https://github.com/QubusPHP/docs/blob/master/page/Injector.md) for these.

The Qubus Injector expects to get an object through its constructor that implements the `Qubus\Config\Config`. You need to pass in the correct "sub-configuration", so that the keys that the `Injector` is looking for are found at the root level.

The `Injector` looks for six configuration keys: `sharedAliases`, `standardAliases`, `argumentDefinitions`, `argumentProviders`, `delegations`, and `preparations`.

The injector works by letting you map aliases to implementations. An alias is a specific name that you want to be able to instantiate. Aliases can be classes, abstract classes, interfaces or arbitrary strings. You can map each alias to a concrete implementation that the injector should instantiate.

This allows you to have your classes only depend on interfaces, through constructor injection, and choose the specific implementation to use through the injector config file.

As an example, imagine the following class:

```PHP
class BookReader
{
    /** @var BookInterface */
    protected $book;

    public function __construct(BookInterface $book)
    {
        $this->book = $book;
    }

    public function read()
    {
        echo $this->book->getContents();
    }
}
```

If we now define an alias `'BookInterface' => 'LatestBestseller'`, we can have code like the following:

```PHP
<?php
$bookReader = $injector->make('BookReader');
// This will now echo the result of LatestBestseller::getContents().
$bookReader->read();
```

### Standard Aliases

A standard alias is an alias that behaves like a normal class. So, for each new instantiation (using `Injector::make()`), you'll get a fresh new instance.

Standard aliases are defined through the `Injector::STANDARD_ALIASES` configuration key:

```PHP
// Format:
//    '<class/interface>' => '<concrete class to instantiate>',
Injector::STANDARD_ALIASES => [
    'Qubus\Config\Config' => 'Qubus\Config\InjectorConfig',
]
```

### Shared Aliases

A shared alias is an alias that behaves similarly to a static variable, in that they get reused across all instantiations. So, for each new instantiation (using `Injector::make()`), you'll get exactly the same instance each time. The object is only truly instantiated the first time it is needed, and this instance is then shared.

Shared aliases are defined through the `Injector::SHARED_ALIASES` configuration key:

```PHP
// Format:
//    '<class/interface>' => '<concrete class to instantiate>',
Injector::SHARED_ALIASES => [
    'ShortcodeManager' => 'Qubus\Shortcode\ShortcodeManager',
]
```

### Argument Definitions

The argument definitions allow you to let the `Injector` know what to pass in to arguments when you need to inject scalar values.

```PHP
// Format:
//
// '<alias to provide argument for>' => [
//    '<argument>' => '<callable or scalar that returns the value>',
// ],
Injector::ARGUMENT_DEFINITIONS => [
	'PDO' => [
		'dsn'      => $dsn,
		'username' => $username,
		'passwd'   => $password,
	]
]
```

By default, the values you pass in as definitions are assumed to be raw values to be used as they are. If you want to pass in an alias through the `Injector::ARGUMENT_DEFINITIONS` key, wrap it in a `Qubus\Injector\Injection` class, like so:

```PHP
Injector::ARGUMENT_DEFINITIONS => [
	'config' => [
		'config' => new Injection( 'My\Custom\ConfigClass' ),
	]
]
```

### Argument Providers

The argument providers allow you to let the `Injector` know what to pass in to arguments when you need to instantiate objects, like `$config` or `$logger`. As these are probably different for each object, we need a way to map them to specific aliases (instead of having one global value to pass in). This is done by mapping each alias to a callable that returns an object of the correct type.

The Injector will create a light-weight proxy object for each of these. These proxies are instantiated and replaced by the real objects when they are first referenced.

If you want to map aliases to specific subtrees of Config files, you can do this by providing a callable for each alias. When the `Injector` tries to instantiate that specific alias, it will invoke the corresponding callable and hopefully get a matching Config back.

```PHP
// Format:
// '<argument>' => [
//    'interface' => '<interface/class that the argument accepts>',
//    'mappings'  => [
//        '<alias to provide argument for>' => <callable that returns a matching object>,
//    ],
// ],
Injector::ARGUMENT_PROVIDERS => [
    'routeCollector' => [
        'interface' => Collector::class,
        'mappings'  => [
            Router::class => function () {
                return new RouterCollector();
            },
        ],
    ],
]
```

### Delegations

The delegations allow you to let the `Injector` delegate the instantiation for a given alias to a provided factory. The factory can be any callable that will return an object that is of a matching type to satisfy the alias.

If you need to act on the injection chain, like finding out what the object is for which you currently need to instantiate a dependency, add a `Qubus\Injector\InjectionChain $injectionChain` argument to your factory callable. You will then be able to query the passed-in injection chain. To query the injection chain, pass the index you want to fetch into `InjectionChain::getByIndex($index)`. If you provide a negative index, you will get the nth element starting from the end of the queue counting backwards.

As an example, consider an `ExampleClass` with a constructor `__construct( ExampleDependency $dependency )`. The injection chain would be the following (in `namespace Example\Namespace`) :

```
[0] => 'Example\Namespace\ExampleClass'
[1] => 'Example\Namespace\ExampleDependency'
```

So, in the example below, we use `getByIndex(-2)` to fetch the second-to-last element from the list of injections.

```PHP
// Format:
//    '<alias>' => <callable to use as factory>
Injector::DELEGATIONS => [
	'Example\Namespace\ExampleDependency' => function ( InjectionChain $injectionChain ) {
		$parent = $injectionChain->getByIndex(-2);
		$factory = new \Example\Namespace\ExampleFactory();
		return $factory->createFor( $parent );
	},
]
```

### Preparations

The preparations allow you to let the `Injector` define additional preparation steps that need to be done after instantiation, but before the object is actually used.

The callable will receive two arguments, the object to prepare, as well as a reference to the injector.

```PHP
// Format:
//    '<alias>' => <callable to execute after instantiation>
Injector::PREPARATIONS => [
	'PDO' => function ( $instance, $injector ) {
		/** @var $instance PDO */
		$instance->setAttribute(
			PDO::ATTR_DEFAULT_FETCH_MODE,
			PDO::FETCH_OBJ
		);
	},
]
```

## Registering Additional Mappings

You can register additional mappings at any time by simply passing additional Configs to the `Injector::registerMappings()` method. It takes the exact same format as the constructor.

```PHP
$config = Factory::create([
    Injector::STANDARD_ALIASES => [
        'ExampleInterface' => 'ConcreteExample'
    ]
]);
$injector->registerMappings($config);
// Here, `$object` will be an instance of `ConcreteExample`.
$object = $injector->make('ExampleInterface');
```

__Note__: For such a simple example, creating a configuration file is of course overkill. You can just as well use the basic `Injector` [alias functionality](https://github.com/QubusPHP/docs/blob/master/page/Injector.md#type-hint-aliasing) and just `Injector::alias()` an additional alias.