# Table of Contents
* [Basic Routing](#basic-routing)
* [Closure Routing](#closure-routing)
* [JSON Routing](#json-routing)
* [Route Request](#route-request)
* [Route Response](#route-response)
* [HTTP Methods](#http-methods)
* [Route Parameters](#route-parameters)
* [Named Routes](#named-routes)
* [Controllers](#controllers)
* [Route Groups](#route-groups)
* [Middleware](#middleware)
* [Dispatching a Router](#dispatching-a-router)
* [Dependency Injection](#dependency-injection)
* [Events](#events)
* [BootManager](#boot-manager)
* [Misc](#misc)

# Basic Routing

Below is a basic example of setting up a route. The route's first parameter is the uri, and the second parameter is a closure or callback.

```php
/**
 * Step 1: Require autoloader and import a few needed classes.
 */
require('vendor/autoload.php');

use Psr\Http\Message\RequestInterface;
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ResponseFactoryInterface;
use Qubus\Http\Request;
use Qubus\Http\Response;
use Qubus\Injector\Config\Factory;
use Qubus\Injector\Psr11\Container;
use Qubus\Routing\Route\RouteCollector;
use Qubus\Routing\Router;

$container = new Container(Factory::create([
    Container::STANDARD_ALIASES => [
        RequestInterface::class => Request::class,
        ResponseInterface::class => Response::class,
        ResponseFactoryInterface::class => Laminas\Diactoros\ResponseFactory::class
    ]
]));

/**
 * Step 2: Instantiate the Router.
 */
$router = new Router(new RouteCollector(), $container);
//$router->setBasePath('/'); If the router is installed in a directory, then you need to set the base path.

/**
 * Step 3: Include the routes needed
 */
// Prints `Hello world!`.
$router->get('/hello-world/', function () {
    return 'Hello world!';
});
```

# Closure Routing

In passing a closure as a route handler, you need to pass in two arguments: `Psr\Http\Message\ServerRequestInterface` and `Psr\Http\Message\ResponseInterface`. Qubus Router requires [Laminas/Diactoros](https://github.com/laminas/laminas-diactoros). This library includes two classes that implement the two needed interfaces.

```php
/**
 * Step 1: Require autoloader and import a few needed classes.
 */
require('vendor/autoload.php');

use Psr\Http\Message\RequestInterface;
use Psr\Http\Message\ResponseInterface;
use Psr\Http\Message\ResponseFactoryInterface;
use Qubus\Http\Request;
use Qubus\Http\Response;
use Qubus\Http\ServerRequest;
use Qubus\Injector\Config\Factory;
use Qubus\Injector\Psr11\Container;
use Qubus\Routing\Route\RouteCollector;
use Qubus\Routing\Router;

$container = new Container(Factory::create([
    Container::STANDARD_ALIASES => [
        RequestInterface::class => Request::class,
        ResponseInterface::class => Response::class,
        ResponseFactoryInterface::class => Laminas\Diactoros\ResponseFactory::class
    ]
]));

/**
 * Step 2: Instantiate the Router.
 */
$router = new Router(new RouteCollector(), $container);
//$router->setBasePath('/'); If the router is installed in a directory, then you need to set the base path.

/**
 * Step 3: Include the routes needed
 */
// Get hello-world route.
$router->get('/hello-world/', function (ServerRequest $serverRequest, Response $response) {
    $response->getBody()->write('Hello World!');
    return $response;
});
```

# JSON Routing
You may also choose to load routes from a json file by using the `loadRoutesFromJson` method:

```php
$route->loadRoutesFromJson('routes.json');
```

## Simple JSON Route
```
{
    "routes": [
        {
            "path": "/users",
            "method": ["GET"],
            "callback": "\\Mvc\\App\\MyControllers\\UsersController@index"
        }
    ]
}
```
## Group JSON Route
```
{
   "routes":[
      {
         "group":{
            "routes":[
               {
                  "path":"/post/{postId}/comment/{commentId}/",
                  "method":["GET"],
                  "callback":"PostCommentController@show",
                  "name":"post.comment",
                  "namespace": "\\Mvc\\App\\MyControllers"
               },
               {
                  "path":"/post/{postId}/",
                  "method":["GET"],
                  "callback":"PostController@show",
                  "name":"post",
                  "namespace": "\\Mvc\\App\\MyControllers"
               }
            ]
         }
      }
   ]
}
```
## Combined Simple and Group Route
```
{
   "routes":[
      {
         "group":{
            "routes":[
               {
                  "path":"/post/{postId}/comment/{commentId}/",
                  "method":["GET"],
                  "callback":"PostCommentController@show",
                  "name":"post.comment",
                  "namespace": "\\Mvc\\App\\MyControllers"
               },
               {
                  "path":"/post/{postId}/",
                  "method":["GET"],
                  "callback":"PostController@show",
                  "name":"post.view",
                  "namespace": "\\Mvc\\App\\MyControllers"
               }
            ]
         }
      },
      {
         "path":"/hello-world/",
         "method":["GET"],
         "callback":"TestController@returnHelloWorld",
         "name":"hello.world",
         "namespace": "\\Mvc\\App\\MyControllers"
      }
   ]
}
```

# Route Request

The example below shows you how you can catch the request object.
```php
use Qubus\Http\ServerRequest;
use Qubus\Http\Factories\JsonResponseFactory;

$router->get('/', function (ServerRequest $serverRequest) {
    return JsonResponseFactory::create([
        'method'            => $serverRequest->getMethod(),
        'uri'               => $serverRequest->getUri(),
        'body'              => $serverRequest->getBody(),
        'parsedBody'        => $serverRequest->getParsedBody(),
        'headers'           => $serverRequest->getHeaders(),
        'queryParameters'   => $serverRequest->getQueryParams(),
        'attributes'        => $serverRequest->getAttributes()
    ]);
});
```

# Route Response

Qubus Router supports the following responses.

```php
use Qubus\Http\Factories\EmptyResponseFactory;
use Qubus\Http\Factories\HtmlResponseFactory;
use Qubus\Http\Factories\JsonResponseFactory;
use Qubus\Http\Factories\TextResponseFactory;
use Qubus\Http\Factories\XmlResponseFactory;

$router->get('/empty', function () {
    return EmptyResponseFactory::create();
});

$router->get('/html/1', function () {
    return '<html>This is an HTML response.</html>';
});

$router->get('/html/2', function () {
    return HtmlResponseFactory::create(
        '<html>This is another HTML response.</html>',
        200,
        ['Content-Type' => ['application/xhtml+xml']]
    );
});

$router->get('/json', function () {
    return JsonResponseFactory::create(
        'This is a JSON response.',
        200,
        ['Content-Type' => ['application/hal+json']]
    );
});

$router->get('/text', function () {
    return TextResponseFactory::create(
        'This is a text response.',
        200,
        ['Content-Type' => ['text/csv']]
    );
});

$router->get('/xml', function () {
    return XmlResponseFactory::create(
        'This is an xml response.',
        200,
        ['Content-Type' => ['application/hal+xml']]
    );
});
```

# HTTP Methods

Sometimes you might need to create a route that accepts multiple HTTP verbs. For this you can use the `map` method. However, If you need to match all HTTP verbs, you can use the `any` method.

```php
$router->map(['GET', 'POST'], '/', function() {
  // ...
});

$router->any('test', function() {
  // ...
});
```
## HTTP Verb Shortcuts

In most typical cases, you will only need to use one HTTP verb. The following can be used in those cases:

```php
$router->get('test/route', function () {});
$router->head('test/route', function () {});
$router->post('test/route', function () {});
$router->put('test/route', function () {});
$router->patch('test/route', function () {});
$router->delete('test/route', function () {});
$router->options('test/route', function () {});
$router->trace('test/route', function () {});
$router->connect('test/route', function () {});
$router->any('test/route', function () {});
```

# Route Parameters

Parameters can be defined on routes using the `{keyName}` syntax. When a route matches a contained parameter, an instance of the `RouteParams` object is passed to the action.

```php
use Qubus\Routing\Route\RouteParams;

$router->map(['GET'], 'posts/{id}', function($id) {
    return $id;
});
```

If you need to add constraints to a parameter, you can pass a regular expression pattern to the `where()` method of the defined `Route`:

```php
$router->map(['GET'], 'post/{id}/comment/{commentKey}', function ($id, $commentKey) {
    return $id;
})->where(['id', '[0-9]+'])->where(['commentKey', '[a-zA-Z]+']);
```

## Optional route Parameters

Sometimes you may want to use optional route parameters. To achieve this, you can add a `?` after the parameter name:

```php
$router->map(['GET'], 'posts/{id?}', function($id) {
    if (isset($id)) {
        // Parameter is set.
    } else {
        // Parameter is not set.
    }
});
```

# Named Routes

Routes can be named so that their URL can be generated programatically:

```php
$router->map(['GET'], 'post/all', function () {})->name('posts.index');

$url = $router->url('posts.index');
```

If the route requires parameters, you can pass an associative array as a second parameter:

```php
$router->map(['GET'], 'post/{id}', function () {})->name('posts.show');

$url = $router->url('posts.show', ['id' => 123]);
```

If a parameter fails the regex constraint applied, a `RouteParamFailedConstraintException` will be thrown.

# Controllers

## Basic Controller
If you'd rather use a class to group related route actions together you can pass a Controller String to `map()` instead of a closure. The string takes the format `{name of class}@{name of method}`. It is important that you use the complete namespace with the class name.

Example:

```php
// HelloWorldController.php
namespace Mvc\App\MyControllers;

class HelloWorldController
{
    public function sayHello()
    {
        return 'Hello World';
    }
}

// routes.php
$router->map(['GET'], 'hello-world', 'Mvc\App\MyControllers\HelloWorldController@sayHello');
// or set default namespace
$router->setDefaultNamespace('Mvc\App\MyControllers');
$router->map(['GET'], 'hello-world', 'HelloWorldController@sayHello');
```
## Resource Controller
You can create controllers that automatically handle all of the route/CRUD requests. When creating a resource controller, you can implement the resource controller interface: `Qubus\Routing\Interfaces\ResourceController`, but you don't have to. You can extend the interface to override some of the methods and their parameters or create your own interface based on the specifications of your project/application.
```php
namespace Qubus\Routing\Interfaces;

interface ResourceController
{
    /**
     * Display a listing of the resource.
     */
    public function index();

    /**
     * Show the form/view for creating a new resource.
     */
    public function create();

    /**
     * Display the specified resource.
     *
     * @param  int  $id
     */
    public function show($id);

    /**
     * Store a newly created resource in storage.
     */
    public function store();

    /**
     * Show the form/view for editing the specified resource.
     *
     * @param  int  $id
     */
    public function edit($id);

    /**
     * Update the specified resource in storage.
     *
     * @param  int  $id
     */
    public function update($id);

    /**
     * Remove the specified resource from storage.
     *
     * @param  int  $id
     */
    public function destroy($id);
}
```

If you want to use middleware in your resource controller, then your controller should extend the abstract controller class: `Qubus\Routing\Controller\BaseController`.

```php
use Qubus\Routing\Controller\BaseController;
use Qubus\Routing\Interfaces\ResourceController;
use Middleware\AddHeaderMiddleware;

class PostController extends BaseController implements ResourceController
{
    public function __construct()
    {
        $this->middleware(new AddHeaderMiddleware('X-Key1', 'abc');
    }
    
    public function index()
    {
        return 'Index of post controller.';
    }

    ```
}
```
Register a resourceful route to the controller:
```php
$router->resource('posts', 'PostController');
```
To register multiple resource controllers by passing in an array in the `resource` method:
```php
$router->resources([
  'posts' => 'PostController',
  'users' => 'UserController'
]);
```
**Actions Handled By Resource Controller**
| Verb  | Uri  | Action | Route Name|
|---|---|---|---|
| GET  | /posts  | index | posts.index|
| GET  | /posts/create  | create | posts.create|
| POST  | /posts/store  | store | posts.store|
| GET  | /posts/{posts}  | show | posts.show|
| GET  | /posts/{posts}/edit  | edit | posts.edit|
| PUT/PATCH  | /posts/{posts}  | update | posts.update|
| DELETE  | /posts/{posts}  | destroy | posts.destroy|

## RESTful Controller
You can conveniently create controllers that will be consumed by an api by using the `apiResource` method:
```php
$router->apiResource('users', 'UserRestController');
```
You can register several api resources by passing in an array to the `apiResources` method:
```php
$router->apiResources([
  'posts' => 'PostRestController',
  'users' => 'UserRestController'
]);
```

# Route Groups

It is common to group similar routes behind a common prefix. This can be achieved using Route Groups:

```php
$router->group(['prefix' => 'page'], function ($group) {
    $group->map(['GET'], '/route1/', function () {}); // `/page/route1/`
    $group->map(['GET'], '/route2/', function () {}); // `/page/route2/`
});
```
## Namespaces
Another common use-case for route groups is assigning the same PHP namespace to a group of controllers using the `namespace` parameter in the group array:
```php
$router->group(['namespace' => '\Mvc\App\MyControllers'], function ($group) {
    // Controllers Within The "\Mvc\App\MyControllers" Namespace
});
```
## Domain/Subdomain Routing
Route groups may also be used to handle domain/subdomain routing. The domain may be specified using the `domain` key on the group attribute array while the subdomain may be specified using the `subdomain` key:
```php
$router->group(['domain' => 'example.com'], function ($group) {
    $group->get('/user/{id}/', function ($id) {
        //
    });
});

$router->group(['subdomain' => 'api.example.com'], function ($group) {
    $group->get('/v1/get/', function () {
        //
    });
});
```
## Route Prefixes
The prefix group attribute may be used to prefix each route in the group with a given url. For example, you may want to prefix all route urls within the group with `admin`:
```php
$router->group(['prefix' => 'admin'], function ($group) {
    $group->get('/users/', function () {
        // Matches The "/admin/users/" URL
    });
});
```

# Middleware

PSR-7/15 Middleware can be added to both routes and groups.

#### Adding Middleware to a route

At it's simplest, adding Middleware to a route can be done by passing an object to the `middleware()` method:

```php
$middleware = new AddHeaderMiddleware('X-Key1', 'abc');

$router->get('hello-world', '\Mvc\App\MyControllers\HelloWorldController@sayHello')->middleware($middleware);
```

Multiple middleware can be added by passing more parameters to the `middleware()` method:

```php
$header = new AddHeaderMiddleware('X-Key1', 'abc');
$auth = new AuthMiddleware();

$router->get('auth', '\Mvc\App\MyControllers\TestController@testMethod')->middleware($header, $auth);
```

Or alternatively, you can also pass an array of middleware:

```php
$header = new AddHeaderMiddleware('X-Key1', 'abc');
$auth = new AuthMiddleware();

$router->get('auth', '\Mvc\App\MyControllers\TestController@testMethod')->middleware([$header, $auth]);
```

#### Adding Middleware to all routes

If you would like to add a middleware that is going to affect all routes, then use the `setBaseMiddleware` method.

```php
$router->setBaseMiddleware([
    new AddHeaderMiddleware('X-Key', 'abc'),
]);
```

#### Adding Middleware to a group

Middleware can also be added to a group. To do so you need to pass an array as the first parameter of the `group()` function instead of a string.

```php
$header = new AddHeaderMiddleware('X-Key1', 'abc');

$router->group(['prefix' => 'my-prefix', 'middleware' => $header]), function ($group) {
    $group->map(['GET'], 'route1', function () {}); // `/my-prefix/route1`
    $group->map(['GET'], 'route2', function () {}); // `/my-prefix/route2`
});
```

You can also pass an array of middleware if you need more than one:

```php
$header = new AddHeaderMiddleware('X-Key1', 'abc');
$auth = new AuthMiddleware();

$router->group(['prefix' => 'my-prefix', 'middleware' => [$header, $auth]]), function ($group) {
    $group->map(['GET'], 'route1', function () {}); // `/my-prefix/route1`
    $group->map(['GET'], 'route2', function () {}); // `/my-prefix/route2`
});
```

#### Defining Middleware on Controllers

You can also apply Middleware on a Controller class too. In order to do this your Controller must extend the `Qubus\Routing\Controller\BaseController` abstract class.

Middleware is added by calling the `middleware()` function in your Controller's `__constructor()`.

```php
use Mvc\App\MyControllers;

class MiddlewareController extends BaseController
{
    public function __construct()
    {
        // Add one at a time
        $this->middleware(new AddHeaderMiddleware('X-Key1', 'abc'));
        $this->middleware(new AuthMiddleware());

        // Add multiple with one method call
        $this->middleware([
            new AddHeaderMiddleware('X-Key1', 'abc'),
            new AuthMiddleware(),
        ]);
    }
}
```

By default all Middlewares added via a Controller will affect all methods on that class. To limit what methods a Middleware should be applied to, you can use `only()` and `except()`:

```php
use Mvc\App\MyControllers;

class MiddlewareController extends BaseController
{
    public function __construct()
    {
        // Only apply to `send()` method
        $this->middleware(new AddHeaderMiddleware('X-Key1', 'abc'))->only('send');

        // Apply to all methods except `show()` method
        $this->middleware(new AuthMiddleware())->except('show');

        // Multiple methods can be provided in an array to both methods
        $this->middleware(new AuthMiddleware())->except(['send', 'show']);
    }
}
```

# Dispatching a Router

You can dispatch a router by using [Laminas SapiEmitter](https://github.com/laminas/laminas-httphandlerrunner/blob/master/src/Emitter/SapiEmitter.php) in combination with `ServerRequest` or use `ServerRequest` in combination with `HttpPublisher`.

```php
/**
 * Step 4: Dispath the router.
 */
 use Qubus\Http\ServerRequest;
 use Laminas\HttpHandlerRunner\Emitter\SapiEmitter as EmitResponse;

 return (new EmitResponse)->emit(
     $router->match(
         ServerRequest::fromGlobals(
             $_SERVER,
             $_GET,
             $_POST,
             $_COOKIE,
             $_FILES
         )
     )
 );

 /**
  * Or you can use the HttpPublisher:
  */
  use Qubus\Http\ServerRequest;
  use Qubus\Http\HttpPublisher;

  return (new HttpPublisher)->publish(
      $router->match(
          ServerRequest::fromGlobals(
              $_SERVER,
              $_GET,
              $_POST,
              $_COOKIE,
              $_FILES
          )
      ),
      null
  );
```

# Dependency Injection

The router can also be used with a PSR-11 compatible Container of your choosing ([PHP-DI](https://github.com/PHP-DI/PHP-DI) is highly recommended). This allows you to type hint dependencies in your route closures or Controller methods.

To make use of a container, simply pass it as a parameter to the Router's constructor:

```php
use DI\Container;
use Qubus\Routing\Route\RouteCollector;
use Qubus\Routing\Router;

$container = new Container();
$router = new Router(new RouteCollector, $container);
```

After which, your route closures and Controller methods will be automatically type hinted:

```php
$container = new Container();

$testServiceInstance = new TestService();
$container->set(TestService::class, $testServiceInstance);

$router = new Router(new RouteCollector, $container);

$router->get('/my/route', function (TestService $service) {
    // $service is now the same object as $testServiceInstance
});
```

# Events

This section will help you understand how to register your own callbacks to events in the router. It will also cover the basics of event-handlers; how to use the handlers provided with the router and how to create your own custom event-handlers.

## Available Events
This section contains all available events that can be registered using the `RoutingEventHandler`.

All event callbacks will retrieve a `RoutingEventArgument` object as parameter. This object contains easy access to event-name, router- and request instance and any special event-arguments related to the given event. You can see what special event arguments each event returns in the list below.

| Name                        | Special arguments | Description               | 
| -------------               |-----------      | ----                      | 
| `EVENT_ALL`                 | - | Fires when a event is triggered. |
| `EVENT_INIT`                | - | Fires when router is initializing and before routes are loaded. |
| `EVENT_LOAD`                | `loadedRoutes` | Fires when all routes have been loaded and rendered, just before the output is returned. |
| `EVENT_ADD_ROUTE`           | `route` | Fires when route is added to the router. |
| `EVENT_BOOT`                | `bootmanagers` | Fires when the router is booting. This happens just before boot-managers are rendered and before any routes has been loaded. |
| `EVENT_RENDER_BOOTMANAGER`  | `bootmanagers`<br>`bootmanager` | Fires before a boot-manager is rendered. |
| `EVENT_LOAD_ROUTES`         | `routes` | Fires when the router is about to load all routes. |
| `EVENT_FIND_ROUTE`          | `name` | Fires whenever the `has` method is used.|
| `EVENT_GET_URL`             | `name`<br>`params` | Fires whenever the `Router::url` method is called and the router tries to find the route. |
| `EVENT_MATCH_ROUTE`         | `route` | Fires when a route is matched and valid (correct request-type etc). and before the route is rendered. |
| `EVENT_RENDER_MIDDLEWARES`  | `route`<br>`middlewares` | Fires before middlewares for a route is rendered. |

## Registering New Event
To register a new event you need to create a new instance of the `Qubus\Routing\Events\RoutingEventHandler` object. On this object you can add as many callbacks as you like by calling the `registerEvent` method.

When you've registered events, make sure to add it to your routes file by calling the `addEventHandler` method from the router. We recommend that you add your event-handlers within your `routes.php`.
```php
use Qubus\Routing\Events\RoutingEventHandler;
use Qubus\Routing\Events\RoutingEventArgument;

// --- your routes goes here ---

$eventHandler = new RoutingEventHandler();

// Add event that fires when a route is rendered
$eventHandler->register(RoutingEventHandler::EVENT_ADD_ROUTE, function(RoutingEventArgument $argument) {
   
   // Get the route by using the special argument for this event.
   $route = $argument->route;
   
   // DO STUFF...
    
});

$router->addEventHandler($eventHandler);
```
## Custom EventHandlers
`Qubus\Routing\Events\RoutingEventHandler` is the class that manages events and must inherit from the `Qubus\Routing\Events\EventHandler` contract. The handler knows how to handle events for the given handler type.

Most of the time the basic `Qubus\Routing\Events\RoutingEventHandler` class will be more than enough for most people as you simply register an event which fires when triggered.

Let's go over how to create your very own event handler class.

Below is a basic example of a custom event-handler called `DatabaseDebugHandler`. The idea of the example below is to log all events to the database when triggered. Hopefully this is enough to give you an idea of how event handlers work.
```php
namespace Mvc\App\Events;

use Qubus\Routing\Router;
use Qubus\Routing\Events\RoutingEventArgument;
use Qubus\Routing\Events\EventHandler;

class DatabaseDebugHandler implements EventHandler
{

    /**
     * Debug callback
     * @var \Closure
     */
    protected $callback;

    public function __construct()
    {
        $this->callback = function (RoutingEventArgument $argument) {
            // todo: store log in database
        };
    }

    /**
     * Get events.
     *
     * @param string|null $name Filter events by name.
     * @return array
     */
    public function getEvents(?string $name): array
    {
        return [
            $name => [
                $this->callback,
            ],
        ];
    }

    /**
     * Fires any events registered with given event name
     *
     * @param Router $router Router instance
     * @param string $name Event name
     * @param array ...$eventArgs Event arguments
     */
    public function fireEvents(Router $router, string $name, ...$eventArgs): void
    {
        $callback = $this->callback;
        $callback(new RoutingEventArgument($router, $eventArgs));
    }

    /**
     * Set debug callback
     *
     * @param \Closure $event
     */
    public function setCallback(\Closure $event): void
    {
        $this->callback = $event;
    }

}
```

# Bootmanager

Sometimes you might find it necessary to store urls in a database, file or similar. In this example, we want the url `/router/article/view/1/` to load the route `/router/hello-world/` which the router knows, because it's defined in the routing file (i.e. routes.php). Please note the that `/router` part of the url is an example of when the route is installed in a subdirectory. For this example, the route is installed in a subdirectory named `router`.

To interfere with the router, we create a class that implements the `Qubus\Routing\Interfaces\BootManager` interface. This class will be loaded before any other rules in routes.php and allow us to "change" the current route, if any of our criteria are fulfilled (like coming from the url `/router/article/view/1/`).
```php
<?php

namespace Mvc\App\Rules;

use Psr\Http\Message\RequestInterface;
use Qubus\Routing\Interfaces\BootManager;
use Qubus\Routing\Router;

class CustomRouterRules implements BootManager
{

    /**
     * Called when router is booting and before the routes are loaded.
     *
     * @param \Qubus\Routing\Router $router
     * @param \Psr\Http\Message\RequestInterface $request
     */
    public function boot(Router $router, RequestInterface $request): void
    {
        $rewriteRules = [
            '/router/article/view/1/' => '/router/hello-world/'
        ];

        foreach ($rewriteRules as $url => $rule) {
            /**
             * If the current url matches the rewrite url, we use our custom route.
             */
            if ($request->getUrl()->getPath() === $url) {
                $request->setRewriteUrl($rule);
            }
        }
    }
}
```
The last thing we need to do, is to add our custom boot-manager to the `routes.php` file. You can create as many bootmanagers as you like and easily add them in your `routes.php` file.
```php
$router->addBootManager(new \Mvc\App\Rules\CustomRouterRules());
```

# Misc

If you return an instance of `Response` from your closure it will be sent back un-touched. If however you return something else, it will be wrapped in an instance of `Response` with your return value as the content.

## Responsable objects

If you return an object from your closure that implements the `Responsable` interface, it's `toResponse()` object will be automatically called for you.

```php
use Laminas\Diactoros\Response\TextResponse;
use Psr\Http\Message\RequestInterface;
use Psr\Http\Message\ResponseInterface;
use Qubus\Routing\Interfaces\Responsable;

class HelloWorldObject implements Responsable
{
    public function toResponse(RequestInterface $request): ResponseInterface
    {
        return new TextResponse('Hello World!');
    }
}

$router->get('hello-world', function () {
    return new HelloWorldObject();
});
```

## 404

If no route matches the request, a `Response` object will be returned with it's status code set to `404`;

## Accessing current route

The currently matched `Route` can be retrieved by calling:

```php
$route = $router->currentRoute();
```

If no route matches or `match()` has not been called, `null` will be returned.

You can also access the name of the currently matched `Route` by calling:

```php
$name = $router->currentRouteName();
```

If no route matches or `match()` has not been called or the matched route has no name, `null` will be returned.