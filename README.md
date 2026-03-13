# Chadicus\Slim\OAuth2\Routes

> **Fork Notice:** This is a maintained fork of the abandoned [`chadicus/slim-oauth2-routes`](https://github.com/chadicus/slim-oauth2-routes) package. Updated for PHP 8.1+ with fixes for PHP 8.4+ deprecations.

[OAuth2 Server](http://bshaffer.github.io/oauth2-server-php-docs/) route callbacks for use within a [Slim Framework](http://www.slimframework.com/) API.

## Requirements

PHP 8.1 or later.

## Installation

```sh
composer require maksimovic/slim-oauth2-routes
```

## A Note on Using Views

The `authorize` and `receive-code` routes require `view` objects. The given view object must implement a `render()` method such as the one found in [slim/php-view](https://github.com/slimphp/PHP-View).

## Example Usage

```php
use Chadicus\Slim\OAuth2\Routes;
use OAuth2;
use OAuth2\GrantType;
use OAuth2\Storage;
use Slim;
use Slim\Views;

// Set up the OAuth2 Server
$storage = new Storage\Pdo(['dsn' => $dsn, 'username' => $username, 'password' => $password]);
$server = new OAuth2\Server($storage);
$server->addGrantType(new GrantType\AuthorizationCode($storage));
$server->addGrantType(new GrantType\ClientCredentials($storage));

// Set up the Slim Application
$app = new Slim\App([
    'view' => new Views\PhpRenderer('/path/to/maksimovic/slim-oauth2-routes/templates'),
]);

$container = $app->getContainer();

$app->map(['GET', 'POST'], Routes\Authorize::ROUTE, new Routes\Authorize($server, $container['view']))->setName('authorize');
$app->post(Routes\Token::ROUTE, new Routes\Token($server))->setName('token');
$app->map(['GET', 'POST'], Routes\ReceiveCode::ROUTE, new Routes\ReceiveCode($container['view']))->setName('receive-code');
$app->post(Routes\Revoke::ROUTE, new Routes\Revoke($server))->setName('revoke');

$app->run();
```

## Authorize and The UserIdProvider

Within the Authorization route, you can define a `UserIdProviderInterface` to extract the `user_id` from the incoming request. By default, the route will look in the `GET` query params.

```php
use Chadicus\Slim\OAuth2\Routes\UserIdProviderInterface;
use Psr\Http\Message\ServerRequestInterface;

class ArgumentUserIdProvider implements UserIdProviderInterface
{
    public function getUserId(ServerRequestInterface $request, array $arguments = [])
    {
        return isset($arguments['user_id']) ? $arguments['user_id'] : null;
    }
}

$authorizeRoute = new Routes\Authorize($server, $view, 'authorize.phtml', new ArgumentUserIdProvider());
$app->map(['GET', 'POST'], Routes\Authorize::ROUTE, $authorizeRoute)->setName('authorize');
```

## Development

```sh
composer install
composer test
composer test:coverage
```

## License

MIT
